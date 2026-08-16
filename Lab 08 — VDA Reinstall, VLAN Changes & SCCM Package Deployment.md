# Lab 08 — VDA Reinstall, VLAN Changes & SCCM Package Deployment

This lab covers:

* Reinstalling the **Citrix VDA** on existing VMs
* Changing VLAN assignments for running VMs
* Creating and deploying software packages to client VMs via **SCCM/MECM**

---

## 🧪 Task 1: Reinstall the Citrix VDA on an Existing VM

> **ℹ️ Note:** VDA = **Virtual Delivery Agent**. Reinstall the VDA when it is corrupted, outdated, or after operating system upgrades.

### Step 1 — Uninstall the Existing VDA

Log in to the target VM.

1. Open **Programs and Features**.
2. Locate **Citrix Virtual Delivery Agent**.
3. Select **Uninstall**.
4. Restart the VM.

### Step 2 — Obtain the VDA Installer

Download the latest VDA installer matching your **CVAD site version** from:

* Citrix
* Approved network share

### Step 3 — Run the VDA Installer

Run the installer as **Administrator**.

Select the appropriate VDA type:

* **Virtual Delivery Agent for Windows Desktop OS**
* **Virtual Delivery Agent for Server OS**

### Step 4 — Select the Environment

Choose the appropriate environment:

* **Create a master image (MCS/PVS)**
* **Enable brokered connections (persistent VMs)**

### Step 5 — Specify Delivery Controllers

Enter the **Delivery Controller FQDN(s)** for the Citrix Broker Services servers.

Example:

```text
ctxbroker01.corp.local
```

### Step 6 — Select VDA Components

Select the required components, including:

* **Citrix Profile Management**
* **Machine Identity Service**

### Step 7 — Configure Firewall Ports

On the **Firewall** page, open the required ports:

| Protocol | Port |
| -------- | ---: |
| TCP      |   80 |
| TCP      |  443 |
| TCP      | 1494 |
| TCP      | 2598 |

The installer can configure these ports automatically.

### Step 8 — Complete Installation and Verify Registration

1. Complete the installation wizard.
2. Restart the VM.
3. Open **Citrix Studio**.
4. Verify that the machine shows:

```text
Registration State = Registered
```

---

## 💻 Silent VDA Installation

Use the following command for a silent VDA installation:

```cmd
VDAServerSetup.exe /quiet /components vda ^
  /controllers "ctxbroker01.corp.local" ^
  /enable_hdx_ports ^
  /enable_real_time_transport ^
  /noreboot
```

### 🔎 Check VDA Registration Using Citrix SDK

```powershell
Get-BrokerMachine -MachineName 'CORP\VM-VDA-01' |
    Select MachineName, RegistrationState
```

---

# 🧪 Task 2: Change the VLAN for an Existing VM

### Step 1 — Identify Current and Target Port Groups

Identify:

* Current port group
* Target port group

Confirm that the target VLAN exists on **all cluster hosts**.

### Step 2 — Confirm Physical Network Configuration

Notify the networking team and confirm that:

* The new VLAN is configured on the physical switches.
* ESXi uplinks are configured to carry the new VLAN.

### Step 3 — Change the VM Network Adapter

Right-click the VM and navigate to:

**Edit Settings > Network Adapter**

Select the new port group.

Example:

```text
VLAN200-NEW
```

Click **OK**.

> ⚠️ **Expected Behavior:** A brief network interruption is expected when changing the VLAN assignment.

### Step 4 — Update the Guest OS Network Configuration

If the VM uses a static IP address, update the following settings to match the new VLAN subnet:

* IP address
* Default Gateway
* DNS

### Step 5 — Test Connectivity

From inside the VM, ping the default gateway of the new VLAN.

Example:

```cmd
ping <new-vlan-default-gateway>
```

Confirm that connectivity is working.

### Step 6 — Update Dependent Configurations

Update the following where applicable:

* Firewall rules
* Application configurations referencing the old IP
* CMDB records
* DNS A records

---

## 💻 PowerCLI — Bulk VLAN Change for Multiple VMs

The following PowerCLI example changes the port group for multiple VMs:

```powershell
$vms = Get-VM -Name 'VDA-*'

foreach ($vm in $vms) {
    $nic = Get-NetworkAdapter -VM $vm
    $pg = Get-VirtualPortGroup -Name 'VLAN200-NEW'

    Set-NetworkAdapter `
        -NetworkAdapter $nic `
        -Portgroup $pg `
        -Confirm:$false
}
```

---

# 🧪 Task 3: Create a Package and Deploy to Client VMs via SCCM

### Step 1 — Copy the Installer to a UNC Share

Copy the installer to a UNC share accessible by SCCM.

Example:

```text
\\SCCM-Server\Sources\Software\AppName\
```

### Step 2 — Create the SCCM Package

In the SCCM Console, navigate to:

**Software Library > Application Management > Packages**

Right-click and select:

**Create Package**

### Step 3 — Configure Package Information

Enter:

* **Name**
* **Manufacturer**
* **Version**

Select:

**This package contains source files**

Browse to the appropriate UNC source path.

### Step 4 — Configure the Standard Program

Choose:

**Standard program**

Set the command line:

```cmd
setup.exe /silent /norestart
```

Configure:

* **Run:** Hidden
* **Program can run:** Whether or not a user is logged on

### Step 5 — Deploy the Package

Right-click the package and select:

**Deploy**

Select the target collection.

Example:

```text
All-VDA-Machines
```

Set the deployment purpose to:

**Required**

### Step 6 — Distribute Content and Configure the Schedule

Distribute the package content to **Distribution Points (DPs)** first.

Configure the deployment schedule:

* Available time
* Mandatory deadline

### Step 7 — Complete and Monitor the Deployment

Click **Finish**.

Monitor deployment status through:

**Monitoring > Deployments**

Verify success or failure for each machine.

---

# 💻 SCCM Client Troubleshooting Commands

## Force SCCM Policy Retrieval

Run the following command on the client VM:

```powershell
Invoke-WMIMethod `
    -Namespace root\ccm `
    -Class SMS_Client `
    -Name TriggerSchedule `
    -ArgumentList '{00000000-0000-0000-0000-000000000021}'
```

## Policy Evaluation

Run:

```powershell
Invoke-WMIMethod `
    -Namespace root\ccm `
    -Class SMS_Client `
    -Name TriggerSchedule `
    -ArgumentList '{00000000-0000-0000-0000-000000000022}'
```

## View the SCCM Client Log

Review the `execmgr.log` file:

```text
C:\Windows\CCM\Logs\execmgr.log
```

The provided Linux-style command for viewing the log is:

```bash
tail -f C:\Windows\CCM\Logs\execmgr.log
```

---

# ⭐ Best Practice Tips

* Always test **VDA reinstall** on a non-production VM before rolling it out to the entire VDA farm.
* Use the **Citrix VDA Cleanup Utility** if normal uninstall fails — it removes all leftover registry keys.
* When changing VLANs, change the **network adapter setting first**, then update the IP in the OS — this minimizes downtime.
* In SCCM, always distribute content to **DPs before scheduling a deployment** to avoid failures.
* Use **Detection Methods** (registry key, file version) for reliable re-run prevention.
* Use a dedicated **SCCM collection for VDA machines** for Citrix-specific deployments.
* Log all VLAN change requests in a **change management system (ServiceNow, Jira)** before executing.
