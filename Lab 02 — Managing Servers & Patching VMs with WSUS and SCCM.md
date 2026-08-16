# Lab 02 — Managing Servers & Patching VMs with WSUS and SCCM

This lab covers patch management for Windows servers and VMs using **WSUS** and **SCCM/MECM**. You will configure update groups, approve patches, and deploy software updates at scale.

---

## 🧪 Task 1: Configure WSUS and Approve Patches

### Step 1 — Open Windows Server Update Services

Open:

**Windows Server Manager > Tools > Windows Server Update Services**

### Step 2 — Verify Client Computers

Navigate to:

**Computers > All Computers**

If machines are missing, run the following command on client VMs:

```cmd
gpupdate /force
```

### Step 3 — Synchronize Updates

Navigate to:

**Synchronizations > Synchronize Now**

This pulls the latest updates from **Microsoft Update**.

### Step 4 — Review Available Updates

Navigate to:

**Updates > All Updates**

Filter updates using:

* **Approval:** Unapproved
* **Status:** Needed

Review the available:

* Critical updates
* Security updates

### Step 5 — Approve Updates

1. Right-click the selected updates.
2. Select **Approve**.
3. Choose the appropriate **Computer Group**, for example:

   * `Test Servers`
4. Set the approval action to **Install**.

### Step 6 — Configure the Installation Deadline

Schedule a **Deadline** for automatic installation.

For production environments, use defined maintenance windows.

Example:

```text
Sunday 02:00 AM
```

### Step 7 — Verify Update Compliance

Navigate to:

**Reports > Update Status Summary**

Use the report to monitor installation progress and update compliance.

### 💻 Force a Windows Update Check from a Client

```cmd
wuauclt /detectnow
wuauclt /reportnow
```

### 💻 PowerShell — Check Pending Updates

```powershell
Get-WindowsUpdate -MicrosoftUpdate
```

---

## 🧪 Task 2: Deploy Software Updates via SCCM/MECM

### Step 1 — Open SCCM Console

Navigate to:

**SCCM Console > Software Library > Software Updates > All Software Updates**

### Step 2 — Synchronize Software Updates

Click:

**Synchronize Software Updates**

This synchronizes updates with **WSUS/Microsoft catalog**.

### Step 3 — Filter Required Updates

Apply the following filters:

| Filter             | Value               |
| ------------------ | ------------------- |
| **Classification** | Security Updates    |
| **Product**        | Windows Server 2019 |
| **Superseded**     | No                  |

### Step 4 — Create a Software Update Group

1. Select the required updates.
2. Right-click the selection.
3. Select **Create Software Update Group**.
4. Provide a descriptive name.

Example:

```text
May-2025-Security
```

### Step 5 — Deploy the Update Group

1. Right-click the **Update Group**.
2. Select **Deploy**.
3. Set the **Collection** containing the target machines.
4. Set **Deployment Type** to:

```text
Required
```

### Step 6 — Configure the Deployment Schedule

Configure:

* **Available Time**
* **Deadline Time**

Set appropriate **maintenance windows** for standard patching activities.

### Step 7 — Distribute Updates

Ensure updates are downloaded to **Distribution Points** before the deployment deadline.

Click **Finish** to complete the deployment.

### Step 8 — Monitor Deployment Compliance

Navigate to:

**Monitoring > Deployments**

Check compliance for individual machines under:

**Asset Details**

---

## 💻 Force SCCM Policy Evaluation on a Target Machine

Run the following command to force SCCM policy evaluation:

```powershell
Invoke-WMIMethod -Namespace root\ccm `
-Class SMS_Client `
-Name TriggerSchedule `
-ArgumentList '{00000000-0000-0000-0000-000000000108}'
```

### 🔎 Check the SCCM Agent

Verify the SCCM client service:

```powershell
Get-Service -Name CcmExec
```

---

## ⭐ Best Practice Tips

* Always patch a **test/staging group first**, wait **48 hours**, then patch production.
* Use **SCCM Maintenance Windows** to ensure updates only apply during off-peak hours.
* Decline superseded updates in **WSUS** regularly to keep the catalog clean.
* Enable automatic **WSUS synchronization daily at 03:00 AM** so patches are always current.
* Set **SCCM Deployment Alerts** to get notified when compliance drops below **95%**.
* Take **VM snapshots before patch deployment** to allow quick rollback if issues arise.

---

## 📋 Lab Workflow Summary

```text
WSUS
  │
  ├── Synchronize Updates
  │
  ├── Review Critical/Security Updates
  │
  ├── Approve Updates
  │
  ├── Set Computer Group
  │
  └── Monitor Compliance
          │
          ▼
      SCCM/MECM
          │
          ├── Synchronize Software Updates
          ├── Filter Required Updates
          ├── Create Software Update Group
          ├── Deploy to Collection
          ├── Configure Schedule
          ├── Distribute to Distribution Points
          └── Monitor Deployment Compliance
```

---

