# Lab 06 — Health Checks for VMs and ESXi Hosts

This lab covers a structured health check routine for **virtual machines (VMs)** and **ESXi hosts**, ensuring proactive identification of:

* Performance issues
* Hardware faults
* Configuration drift

---

## 🧪 Task 1: VM Health Check Checklist

### Step 1 — Verify VM Status and VMware Tools

Select the VM in vCenter and open the **Summary** tab.

Verify that **VMware Tools** is:

* Running
* Up-to-date

### Step 2 — Check CPU and Memory Usage

Review:

* **CPU Usage (%)**
* **Memory Usage (%)**

> ⚠️ **Attention:** Sustained usage above **85%** on either CPU or memory is a concern.

### Step 3 — Review Advanced Performance Metrics

Navigate to:

**Monitor > Performance > Advanced**

Review the following metrics:

| Metric                  | Recommended Threshold |
| ----------------------- | --------------------: |
| **CPU Ready**           |                   <5% |
| **Disk Latency**        |                <20 ms |
| **Memory Balloon/Swap** |                     0 |

### Step 4 — Check Disk I/O

Review disk I/O performance.

High **read/write latency** or **queue depth** can indicate a storage bottleneck.

### Step 5 — Review VM Alarms

Open the VM **Alarms** tab.

Investigate and resolve any active alarms.

### Step 6 — Check Guest OS Disk Space

Check available disk space within the guest operating system.

**Linux:**

```bash id="t8z4w1"
df -h
```

**Windows:**

```powershell id="f6h5l3"
Get-PSDrive
```

> ⚠️ **Alert Threshold:** Generate an alert if any volume exceeds **85%** utilization.

### Step 7 — Verify Snapshot Status

Verify the VM snapshot configuration.

> ⚠️ **Production Requirement:** No production VM should have snapshots older than **72 hours**.

---

## 💻 PowerCLI — VM Health Report

Use the following PowerCLI command to generate a VM health report:

```powershell id="6v2f6r"
Get-VM | Select Name, PowerState,
@{N='CPU_MHz';E={$_.ExtensionData.Summary.QuickStats.OverallCpuUsage}},
@{N='MemUsedMB';E={$_.ExtensionData.Summary.QuickStats.GuestMemoryUsage}},
@{N='ToolsStatus';E={$_.ExtensionData.Guest.ToolsStatus}} |
Format-Table -AutoSize
```

## 💻 Linux — Disk Space Check

The following command reports volumes exceeding the 85% utilization threshold:

```bash id="0f7l4c"
df -h | awk '$5+0 > 85 {print "WARNING: "$0}'
```

---

# 🧪 Task 2: ESXi Host Health Check Checklist

### Step 1 — Verify ESXi Host Status

Select the ESXi host in vCenter and open the **Summary** tab.

Confirm:

* **Connection State = Connected**
* **Power State = Powered On**

### Step 2 — Check CPU and Memory Utilization

Review CPU and memory utilization across the environment.

> ⚠️ **Attention:** Sustained CPU utilization above **80% across all hosts** indicates potential resource exhaustion.

### Step 3 — Check Hardware Health

Navigate to:

**Monitor > Hardware Health**

Verify that all sensors are green, including:

* Temperature
* Fan speed
* Power supply

### Step 4 — Verify NTP Synchronization

Verify that ESXi hosts are correctly synchronized with the configured NTP source.

> ⚠️ **Important:** Time drift greater than **30 seconds** can break HA, vMotion, and authentication.

### Step 5 — Check Storage Adapters

Verify the status of the storage adapters:

* All HBAs should show **Online**.
* Verify the path count for multipathed datastores.

### Step 6 — Verify Physical Network Adapters

Check the physical network adapters (`vmnics`).

All active adapters should:

* Show **Up**
* Display the expected link speed

### Step 7 — Run Detailed Diagnostics

SSH to the ESXi host and run **NCC / esxcli health commands** for detailed diagnostics.

---

# 💻 SSH into ESXi — Health Commands

### Check ESXi Version

```bash id="2b2v0h"
esxcli system version get
```

### Check Hardware Memory

```bash id="d4zq5e"
esxcli hardware memory get
```

### Check Storage Adapters

```bash id="r2h3zy"
esxcli storage core adapter list
```

### Check Network NICs

```bash id="0d1rks"
esxcli network nic list
```

### Check VMkernel Core Dump Configuration

```bash id="s6t8r2"
vmkchkdump -f
```

### Check NTP Servers

```bash id="v0q8j9"
esxcli network ntp server list
```

### Check System Time

```bash id="r7j5b1"
esxcli system time get
```

---

# 💻 Nutanix NCC Health Check

Run the Nutanix Cluster Check:

```bash id="3d8w4m"
ncc health_checks run_all
```

---

# ⭐ Best Practice Tips

* Schedule automated daily health checks using **PowerCLI** or **vRealize Operations Manager**.
* **CPU Ready >5%** indicates VM CPU contention — reduce vCPU count or move the VM to a less loaded host.
* **Memory balloon/swap** in VMs means the host is memory-starved — add RAM or migrate VMs.
* Always configure **syslog on ESXi hosts** to forward logs to a centralized **SIEM or log server**.
* Use **vCenter Alarms** to automatically notify via email when thresholds are breached.
* Run **Nutanix NCC checks weekly** and after any infrastructure changes.

---

## 📋 Health Check Workflow

```text
                    Infrastructure Health Check
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
           VM Health                   ESXi Health
                │                           │
        ┌───────┼───────┐           ┌───────┼───────┐
        ▼       ▼       ▼           ▼       ▼       ▼
      CPU     Memory  Storage      CPU/    Hardware Network
      /RAM            /I/O        Memory    Health   /Storage
        │       │       │           │       │       │
        └───────┴───────┘           └───────┴───────┘
                │                           │
                └─────────────┬─────────────┘
                              ▼
                     Review Alerts & Logs
                              │
                              ▼
                       Remediate Issues
                              │
                              ▼
                       Document Findings
```
