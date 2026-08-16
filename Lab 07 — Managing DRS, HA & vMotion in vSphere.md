# Lab 07 — Managing DRS, HA & vMotion in vSphere

This lab covers configuration and management of **vSphere cluster features**, including:

* **DRS (Distributed Resource Scheduler)** for load balancing
* **HA (High Availability)** for automatic VM restart on host failure
* **vMotion** for live VM migration

---

## 🧪 Task 1: Configure vSphere DRS (Distributed Resource Scheduler)

### Step 1 — Enable vSphere DRS

Select:

**Cluster > Configure > Services > vSphere DRS**

Click **Edit**.

Toggle **vSphere DRS** to **ON**.

### Step 2 — Configure the Automation Level

Select the automation level:

**Fully Automated**

> 💡 **Recommended:** Fully Automated is recommended for production environments when appropriate for the organization's operational requirements.

### Step 3 — Set the Migration Threshold

Set the **Migration Threshold** to:

**Level 3 (Aggressive)**

This provides balanced workload distribution across the cluster.

### Step 4 — Enable Predictive DRS

Enable **Predictive DRS** if **vROps** is integrated.

Predictive DRS can pre-emptively balance workloads before anticipated peak loads.

### Step 5 — Configure DRS Rules

Configure:

**VM-VM Anti-Affinity**

Use VM-VM anti-affinity rules to spread critical VMs across different hosts.

### Step 6 — Configure VM-Host Affinity Rules

Configure **VM-Host Affinity Rules** to pin specific VMs to specific hosts.

Example use case:

* Licensing compliance

### 💻 PowerCLI — Apply DRS Recommendations

View DRS recommendations:

```powershell id="2xqg0h"
Get-Cluster 'PROD-Cluster' |
    Get-DrsRecommendation |
    Format-Table -AutoSize
```

Apply DRS recommendations:

```powershell id="0z8m2n"
Get-Cluster 'PROD-Cluster' |
    Get-DrsRecommendation |
    Apply-DrsRecommendation
```

---

# 🧪 Task 2: Configure vSphere HA (High Availability)

### Step 1 — Enable vSphere HA

Select:

**Cluster > Configure > Services > vSphere Availability**

Click **Edit**.

Toggle **HA** to **ON**.

### Step 2 — Configure Host Failure and Isolation Responses

Set:

**Host Failure Response:**

```text
Restart VMs
```

Set:

**Response for Host Isolation:**

```text
Shut Down and Restart VMs
```

### Step 3 — Configure Admission Control

Reserve:

* **25% CPU**
* **25% Memory**

This ensures sufficient resources are available for HA to restart VMs when one host fails.

### Step 4 — Configure Heartbeat Datastores

Select **2 shared datastores** for HA host isolation validation.

### Step 5 — Configure VM Overrides

Set the **Highest restart priority** for critical VMs, including:

* Domain controllers
* DNS
* AD

### Step 6 — Enable VM Monitoring

Enable **VM Monitoring**.

HA can restart VMs that lose the **VMware Tools heartbeat**.

---

# 🧪 Task 3: Perform vMotion (Live VM Migration)

### Step 1 — Verify the vMotion VMkernel Adapter

Navigate to:

**Host > Configure > VMkernel Adapters**

Verify that **vMotion** is enabled on:

```text
vmk1
```

### Step 2 — Start the Migration

Right-click the running VM and select:

**Migrate > Change compute resource only**

### Step 3 — Select the Destination Host

Select the destination host.

vCenter validates compatibility.

If an **EVC mismatch** occurs, address the mismatch as required, such as by enabling **EVC on the cluster**.

### Step 4 — Select the Destination Network

Select the destination network.

Set the migration priority to:

**High Priority**

This provides higher priority for the migration.

### Step 5 — Complete the Migration

Click **Finish**.

The VM remains fully operational during the migration.

### Step 6 — Verify the New Host

Select the VM and open:

**Summary > Host**

Verify that the **Host** field shows the new destination host.

---

# 💻 PowerCLI — vMotion

Define the VM and destination host:

```powershell id="6j2e8p"
$vm = Get-VM -Name 'PROD-APP-01'
$destHost = Get-VMHost -Name 'esxi03.corp.local'

Move-VM `
    -VM $vm `
    -Destination $destHost `
    -Confirm:$false
```

---

# 💾 PowerCLI — Storage vMotion

Define the destination datastore:

```powershell id="e6q6wr"
$ds = Get-Datastore -Name 'SSD-DS02'

Move-VM `
    -VM $vm `
    -Datastore $ds `
    -Confirm:$false
```

---

# ⭐ Best Practice Tips

* Enable **Enhanced vMotion Compatibility (EVC)** on clusters to allow vMotion between different CPU generations.
* Dedicate a **VMkernel adapter for vMotion** on a separate VLAN with **10GbE minimum bandwidth**.
* Set **DRS anti-affinity rules** for clustered applications, such as **Oracle RAC**, to spread them across hosts.
* **HA Admission Control** prevents powering on VMs that would leave insufficient failover capacity.
* Test HA by placing a **non-critical host in Maintenance Mode** and verifying that VMs restart on the remaining hosts.
* For **multi-site DR**, use **vSphere Replication** or **SRM** in conjunction with HA.
