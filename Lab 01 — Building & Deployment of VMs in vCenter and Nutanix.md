# Lab 01 — Building & Deployment of VMs in vCenter and Nutanix

This lab walks through creating and deploying virtual machines on both **VMware vCenter** and **Nutanix AHV**. You will learn the full VM provisioning workflow, including hardware configuration, OS installation, and network assignment.

---

## 🧪 Task 1: Deploy a VM in VMware vCenter

### Prerequisites

Before beginning, ensure you have:

* Access to **vSphere Client (HTML5)**
* Administrator credentials
* ISO image or template available in a datastore

### Step 1 — Open vSphere Client

Open a browser and navigate to:

```text
https://<vcenter-ip>/ui
```

Log in with administrator credentials.

### Step 2 — Select the Target Host

Navigate to:

**Datacenter > Cluster > Host**

Right-click the target host and select **New Virtual Machine**.

### Step 3 — Create a New Virtual Machine

Select:

**Create a new virtual machine**

Click **Next**.

### Step 4 — Specify the VM Name and Location

Enter a VM name, for example:

```text
PROD-APP-01
```

Select the **datacenter** as the deployment location.

### Step 5 — Select Compute Resource and Datastore

Select:

* The compute resource (**ESXi host or cluster**)
* The **datastore** to be used for VM storage

### Step 6 — Set VM Compatibility

Set the VM compatibility level.

Select:

**ESXi 7.0 and later**

This provides maximum feature support.

### Step 7 — Select the Guest OS

Select the appropriate:

* **Guest OS Family:** Windows/Linux
* **Guest OS Version:** Specific version, for example, Windows Server 2019

### Step 8 — Customize Virtual Hardware

Configure the VM hardware according to workload requirements:

* **vCPUs**
* **Memory (GB)**
* **Hard Disk size**
* **Disk provisioning:** Thin/Thick
* **Network Adapter:** Attach to the correct port group

### Step 9 — Attach the OS ISO

Under **CD/DVD Drive**:

1. Select **Datastore ISO File**.
2. Browse to the OS ISO.
3. Select **Connect at power on**.

### Step 10 — Review, Deploy, and Install the OS

1. Review the VM configuration summary.
2. Click **Finish**.
3. Right-click the VM and select **Power On**.
4. Open **Web Console**.
5. Complete the operating system installation.

### 🔎 Verify the VM Using PowerCLI

Use PowerCLI to verify the VM configuration and power state:

```powershell
Connect-VIServer -Server <vcenter-ip> -User admin -Password <password>

Get-VM -Name 'PROD-APP-01' |
    Select Name, PowerState, NumCpu, MemoryGB
```

---

## 🧪 Task 2: Deploy a VM in Nutanix AHV (Prism Central)

### Step 1 — Log In to Prism Central

Open a browser and navigate to:

```text
https://<prism-central-ip>:9440
```

Log in using the appropriate credentials.

### Step 2 — Open the VM Creation Workflow

Navigate to:

**Compute & Storage > VMs > Create VM**

### Step 3 — Specify VM Name and Cluster

Enter the VM name and select the target **Cluster**.

### Step 4 — Configure CPU and Memory

Set the following according to workload requirements:

* **vCPUs**
* **Cores per vCPU**
* **Memory (GiB)**

### Step 5 — Configure Disks

Under **Disks**, click **Attach Disk**.

Select one of the following options:

* **Clone from Image** — Template-based deployment
* **Allocate on Storage Container** — Blank disk

### Step 6 — Configure the Network

Under **Networks**:

1. Click **Attach to Subnet**.
2. Select the correct VLAN-backed subnet.

Example:

```text
VLAN100-PROD
```

### Step 7 — Attach the OS ISO

For a fresh OS installation:

1. Select **Add CD-ROM**.
2. Select **Clone from Image Library**.
3. Select the required OS ISO.

### Step 8 — Save, Power On, and Install the OS

1. Click **Save**.
2. Power on the VM.
3. Click **Launch Console**.
4. Complete the OS installation.

### Step 9 — Install Nutanix Guest Tools (NGT)

Install **Nutanix Guest Tools (NGT)** after the operating system installation:

1. Right-click the VM.
2. Select **Install NGT**.
3. Mount the ISO in the guest operating system.
4. Run the NGT installer.

---

## 💻 Nutanix CLI Verification

### Nutanix CLI on CVM

Use `acli` to list VMs:

```bash
acli vm.list
```

### Prism Central CLI

Use `nucalm` to retrieve VMs:

```bash
nucalm get vms --count 20
```

---

## ⭐ Best Practice Tips

* Use **Thin Provisioning** in dev/test; use **Thick Eager Zeroed** for databases and critical production VMs.
* Attach VMs to port groups/subnets matching the correct VLAN to prevent network segregation issues.
* Install **VMware Tools (vCenter)** or **NGT (Nutanix)** immediately after OS install.
* Use a consistent VM naming convention:

```text
<ENV>-<ROLE>-<NUMBER>
```

Examples:

```text
PROD-DB-01
DEV-WEB-02
```

* Document **CPU/RAM/Storage allocation** in a CMDB for capacity planning.

---

## 📋 Lab Completion Summary

Upon completion of this lab, the administrator should have experience with:

| Area                      | VMware vCenter | Nutanix AHV       |
| ------------------------- | -------------- | ----------------- |
| VM Creation               | ✅              | ✅                 |
| Compute Configuration     | ✅              | ✅                 |
| Memory Configuration      | ✅              | ✅                 |
| Disk Configuration        | ✅              | ✅                 |
| Network Assignment        | ✅              | ✅                 |
| ISO-Based OS Installation | ✅              | ✅                 |
| Guest Tools Installation  | VMware Tools   | NGT               |
| CLI Verification          | PowerCLI       | `acli` / `nucalm` |

