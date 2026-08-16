 # Lab 04 — Disk Expansion, Datastores, Snapshots, Migration & Nutanix Alerts

This lab covers key storage and mobility operations, including:

* Expanding VM disks
* Creating datastores
* Managing snapshots
* Performing VM/cluster migrations
* Configuring Nutanix alert management

---

## 🧪 Task 1: Expand a VM Virtual Disk (vCenter)

### Step 1 — Prepare the VM

Power off the VM and ensure no active snapshots exist.

> ⚠️ **Important:** Disk expansion requires no snapshot chain.

### Step 2 — Increase the Virtual Disk Size

Right-click the VM and navigate to:

**Edit Settings > Hard Disk**

Increase the disk size.

Example:

```text
80 GB → 150 GB
```

Click **OK**.

### Step 3 — Power On the VM

Power on the VM.

The disk is now larger, but the partition and filesystem must still be extended.

### Step 4 — Extend the Windows Volume

For Windows:

1. Open **Disk Management**:

```text
diskmgmt.msc
```

2. Right-click the required volume.
3. Select **Extend Volume**.

### Step 5 — Extend the Linux Partition and Filesystem

For Linux, use `fdisk`/`parted` to resize the partition, followed by:

* `resize2fs` for **ext4**
* `xfs_growfs` for **XFS**

### 💻 Linux — Extend Partition and Filesystem

Identify the disk:

```bash
lsblk
```

Example disk:

```text
/dev/sda
```

Use `fdisk` to modify the partition:

```bash
fdisk /dev/sda
```

Delete the old partition and create a new larger partition using the **same starting sector**.

Reload the partition table:

```bash
partprobe /dev/sda
```

For **ext4**:

```bash
resize2fs /dev/sda1
```

For **XFS**:

```bash
xfs_growfs /
```

> 💡 **Note:** The XFS filesystem must be mounted when using `xfs_growfs`.

---

## 🧪 Task 2: Create and Manage a Datastore (vCenter)

### Step 1 — Open the New Datastore Workflow

In the vCenter **Storage** view:

**Right-click Datacenter > New Datastore**

### Step 2 — Select the Datastore Type

Select the required storage type:

* **VMFS** — Local/SAN
* **NFS** — NAS

Click **Next**.

### Step 3 — Configure the Datastore

1. Select the target host.
2. Select the LUN/disk.
3. Choose **VMFS 6** for ESXi 6.5+.
4. Enter a name for the datastore.
5. Complete the datastore creation workflow.

### Step 4 — Verify the Datastore

Verify that the new datastore appears under **Storage** with the correct capacity.

---

## 🧪 Task 3: Create and Manage VM Snapshots

### Step 1 — Create a Snapshot

Right-click the VM and navigate to:

**Snapshots > Take Snapshot**

Provide a descriptive name.

Example:

```text
Pre-Patch-2025-05-01
```

### Step 2 — Enable Guest File System Quiescing

Select:

**Quiesce guest file system**

This requires **VMware Tools** and provides application-consistent snapshots.

### Step 3 — Revert to a Snapshot

To revert:

**Right-click VM > Snapshots > Revert to Latest Snapshot**

### Step 4 — Delete a Snapshot

To delete a snapshot:

1. Navigate to **Manage Snapshots**.
2. Select the snapshot.
3. Click **Delete**.

Deleting the snapshot consolidates the delta files back into the base disk.

---

## 🧪 Task 4: VM Migration (vMotion) and Cluster Migration

### Step 1 — Start a Live vMotion

Right-click the running VM and select:

**Migrate > Change compute resource only**

This performs a live **vMotion**.

### Step 2 — Select the Destination

Select the destination host or cluster.

Verify:

* Shared network availability
* Datastore availability

### Step 3 — Monitor the Migration

Monitor the migration task under:

**Tasks & Events**

vMotion typically completes in:

```text
<2 minutes
```

### Step 4 — Perform Cluster Migration

For cluster migration:

* Use **DRS**, or
* Manually migrate each VM.

Ensure both clusters share the required datastores.

---

## 🧪 Task 5: Alert Management in Nutanix Prism

### Step 1 — Open Nutanix Prism Element

Log in to Prism Element:

```text
https://<cluster-ip>:9440
```

Click the **Bell (Alerts)** icon.

### Step 2 — Review Alerts

Review alerts by severity:

* 🔴 **Critical**
* 🟡 **Warning**
* 🔵 **Info**

Click an alert to view:

* Root Cause Analysis
* Resolution steps

### Step 3 — Configure Alert Policies

Navigate to:

**Settings (gear icon) > Alert Configuration**

Configure the required alert thresholds.

### Step 4 — Configure Email Notifications

Navigate to:

**Settings > SMTP Server**

Enable email alerts for:

**Critical** severity.

### Step 5 — Acknowledge Resolved Alerts

To acknowledge an alert:

1. Select the alert.
2. Click **Acknowledge**.
3. Add resolution notes for the audit trail.

---

## 💻 Nutanix CLI — View and Acknowledge Alerts

### View Alerts

```bash
ncli alert ls
```

### Acknowledge All Alerts

```bash
ncli alert acknowledge-all
```

### Run NCC Health Check

Run the Nutanix Cluster Check:

```bash
ncc health_checks run_all
```

---

## ⭐ Best Practice Tips

* Never keep snapshots older than **24–72 hours** in production — they degrade VM performance significantly.
* For **vMotion** to succeed, source and destination hosts must share the same network labels and datastore.
* Use **Storage vMotion (svMotion)** to migrate VM disk files between datastores without downtime.
* Enable **NCC (Nutanix Cluster Check)** scheduled runs to proactively detect issues.
* Configure **Nutanix Prism** to send SNMP traps to monitoring platforms (**Nagios, Zabbix, Datadog**).
* When creating datastores, leave at least **15–20% free space** for snapshots and VM swapfiles.
