# Lab 03 — Managing VMware ESXi 5.5, 6.5, 7.0 & Host Upgrades

This lab covers day-to-day **ESXi host management** and upgrading ESXi hosts from older versions (**5.5, 6.5**) to **ESXi 7.0** using **VMware Lifecycle Manager (vLCM)**.

> ⚠️ **Version Note:** The upgrade workflow described in this lab is specifically focused on upgrading supported ESXi hosts to ESXi 7.0 using vLCM.

---

## 🧪 Task 1: ESXi Host Daily Management Tasks

### Step 1 — Access the ESXi Host Client

Access the ESXi Host Client using:

```text
https://<esxi-host-ip>/ui
```

Alternatively, manage the host through **vCenter**.

### Step 2 — Check Host Status

In vCenter, navigate to:

**Hosts > Select Host > Summary**

Verify:

* CPU usage
* Memory usage
* Uptime
* Connection state

### Step 3 — Review Active Alarms

Open the **Alarms** tab.

Investigate any **red** or **yellow** alarms before proceeding with administrative activities.

### Step 4 — Check Hardware Health

Navigate to:

**Monitor > Hardware Health**

Verify that all hardware sensors are healthy/green, including:

* CPU temperature
* Fans
* Power

### Step 5 — Verify NTP Synchronization

Connect to the ESXi host using SSH and run:

```bash
esxcli system time get
esxcli network ntp server list
```

Verify that the host time is synchronized correctly and that the expected NTP configuration is present.

### Step 6 — Check Datastores

Navigate to:

**Storage > Datastores**

Confirm that:

* All required datastores are accessible.
* Datastores have adequate free space.
* Free space is greater than **20%**.

### Step 7 — Verify Network Adapters

Navigate to:

**Networking > Physical Adapters**

Verify that all required `vmnic` ports show an **Up** state.

---

## 💻 ESXi SSH Key Commands

Use SSH to connect to the ESXi host and perform common management and troubleshooting tasks.

### Check ESXi Version

```bash
esxcli system version get
```

### List CPU Information

```bash
esxcli hardware cpu list
```

### List Storage Devices

```bash
esxcli storage core device list
```

### Test VMkernel Connectivity

```bash
vmkping -I vmk0
```

### List VMkernel Network Interfaces

```bash
esxcli network ip interface list
```

### Real-Time Performance Monitoring

```bash
esxtop
```

> 💡 **Tip:** `esxtop` provides a real-time performance monitor for troubleshooting CPU, memory, storage, and network performance.

---

# 🧪 Task 2: Upgrade ESXi Host Using VMware Lifecycle Manager (vLCM)

> **📌 Note:** Applicable for upgrading **ESXi 6.5/6.7 to ESXi 7.0**. vLCM is available in **vCenter 7.0+**.

### Step 1 — Enter Maintenance Mode

Right-click the host and select:

**Maintenance Mode > Enter Maintenance Mode**

VMs migrate via **vMotion** automatically when the environment is configured appropriately.

### Step 2 — Open Lifecycle Manager

In vCenter, navigate to:

**Menu > Lifecycle Manager**

Select the cluster containing the target host.

### Step 3 — Configure the Cluster to Use an Image

Switch the cluster to use an image.

Select:

**Manage with a Single Image**

### Step 4 — Select the Desired ESXi Image

Select the required image.

Example:

```text
ESXi 7.0 Update 3
```

Add vendor add-ons if required, such as:

* HPE customized image
* Dell customized image

### Step 5 — Check Compliance

Click:

**Check Compliance**

The host will show **Non-Compliant** because it is running an older ESXi version.

### Step 6 — Remediate the Host

Click:

**Remediate All**

Review the proposed changes and click:

**Start Remediation**

The host reboots automatically during the remediation process.

### Step 7 — Verify the Host After Reboot

After the host reboots:

1. Verify that the host rejoins **vCenter**.
2. Confirm that it is running **ESXi 7.0**.
3. Verify the host exits maintenance mode as expected.

### Step 8 — Recheck Compliance

Run:

**Check Compliance**

The status should now be:

```text
Compliant
```

### Step 9 — Power On and Validate VMs

Power on the migrated VMs.

Upgrade the **VM hardware version** if needed for compatibility with new features.

---

# 💻 Manual ESXi Upgrade via CLI — Offline Bundle

The following provides the manual ESXi upgrade method using an offline bundle:

```bash
esxcli software profile update \
  -d /vmfs/volumes/datastore1/ESXi700-update.zip \
  -p ESXi-7.0.0-15843807-standard
```

## 🔎 Verify the ESXi Version After Upgrade

Run:

```bash
vmware -vl
esxcli system version get
```

---

# ⭐ Best Practice Tips

* Always snapshot or back up critical VMs before entering maintenance mode.
* Verify hardware compatibility on the **VMware HCL** before upgrading.
* Use vendor-customized **ESXi images (Dell/HPE)** to ensure driver compatibility.
* Upgrade **vCenter Server BEFORE upgrading ESXi hosts** — vCenter must be at an equal or higher version.
* After the upgrade, update **VMware Tools** and **VM hardware version** on all VMs.
* Schedule upgrades during maintenance windows and communicate outages to stakeholders.

---

## 📋 ESXi Upgrade Workflow

```text
Verify Prerequisites
        │
        ▼
Check VMware HCL
        │
        ▼
Verify vCenter Version
        │
        ▼
Enter ESXi Host Maintenance Mode
        │
        ▼
Configure vLCM Single Image
        │
        ▼
Select ESXi 7.0 Image
        │
        ▼
Check Compliance
        │
        ▼
Remediate All
        │
        ▼
Host Reboot
        │
        ▼
Verify ESXi 7.0
        │
        ▼
Check Compliance
        │
        ▼
Power On / Validate VMs
```

---
