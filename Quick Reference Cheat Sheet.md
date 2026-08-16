# Quick Reference Cheat Sheet

> 📘 **Purpose:** Use this cheat sheet as a quick operational reference for common Nutanix, VMware, Windows, Linux, PowerCLI, and Citrix administration commands and actions.

| Command / Action                     | Platform    | Purpose                                  |
| ------------------------------------ | ----------- | ---------------------------------------- |
| `New-VM from Template (PowerCLI)`    | PowerCLI    | Deploy VM from vSphere template          |
| `esxcli software profile update`     | ESXi SSH    | Upgrade ESXi from offline bundle         |
| `Get-VM \| Select Name, PowerState`  | PowerCLI    | List all VM names and power states       |
| `ip route add <net> via <gw>`        | Linux       | Add static route                         |
| `ip netns add <name>`                | Linux       | Create network namespace                 |
| `ncli alert ls`                      | Nutanix CVM | List Nutanix alerts                      |
| `ncc health_checks run_all`          | Nutanix CVM | Run all Nutanix NCC checks               |
| `esxtop`                             | ESXi SSH    | Real-time ESXi performance monitoring    |
| `acli vm.list`                       | Nutanix CVM | List all VMs on AHV cluster              |
| `gpupdate /force`                    | Windows     | Force Group Policy update                |
| `wuauclt /detectnow`                 | Windows     | Force Windows Update detection           |
| `Move-VM -VM $vm -Destination $host` | PowerCLI    | Perform vMotion (live migration)         |
| `VDAServerSetup.exe /quiet`          | Windows     | Silent Citrix VDA install                |
| `resize2fs /dev/sda1`                | Linux       | Extend ext4 filesystem after disk expand |
| `esxcli network nic list`            | ESXi SSH    | List physical NICs and link state        |
| `Get-DrsRecommendation`              | PowerCLI    | View DRS migration recommendations       |
| `Set-NetworkAdapter -Portgroup`      | PowerCLI    | Change VM VLAN/port group                |

---

## 🧭 Platform Reference

| Platform        | Key Administration Areas                              |
| --------------- | ----------------------------------------------------- |
| **PowerCLI**    | VM deployment, VM inventory, vMotion, DRS, networking |
| **ESXi SSH**    | Host upgrades, NICs, performance monitoring           |
| **Nutanix CVM** | VM management, alerts, cluster health                 |
| **Linux**       | Routing, network namespaces, filesystem management    |
| **Windows**     | Group Policy and Windows Update                       |
| **Citrix VDA**  | Silent VDA installation and deployment                |

---

## ⚡ Common Administrative Actions

### 🖥️ VMware / PowerCLI

* Deploy VMs from templates
* List VM power states
* Perform live VM migration using vMotion
* Review DRS migration recommendations
* Change VM VLAN/port group assignments

### 🔧 ESXi

* Upgrade ESXi using an offline bundle
* Monitor real-time host performance
* Review physical NICs and link states

### 🏢 Nutanix

* List AHV VMs
* Review Nutanix alerts
* Run Nutanix Cluster Check (NCC)

### 🌐 Linux

* Add static routes
* Create network namespaces
* Extend ext4 filesystems after disk expansion

### 🪟 Windows

* Force Group Policy updates
* Force Windows Update detection

### 🖥️ Citrix

* Perform silent Citrix VDA installation

> 💡 **Operational Tip:** Use this cheat sheet together with **Labs 01–08** for detailed procedures, prerequisites, verification steps, and best practices.
