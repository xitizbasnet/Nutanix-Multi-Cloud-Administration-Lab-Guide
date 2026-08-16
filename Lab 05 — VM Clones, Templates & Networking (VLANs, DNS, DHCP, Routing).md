# Lab 05 — VM Clones, Templates & Networking (VLANs, DNS, DHCP, Routing)

This lab covers rapid VM deployment using **clones and templates in vCenter**, and configuration of key networking services:

* VLANs
* Routing
* Network namespaces
* DNS
* DHCP

---

## 🧪 Task 1: Convert a VM to Template and Deploy Clones

### Step 1 — Prepare a Golden VM

Prepare the golden VM by:

* Installing the operating system.
* Applying all patches.
* Installing base software.
* Running **Sysprep** on Windows or **cloud-init** on Linux to generalize the VM.

### Step 2 — Convert the VM to a Template

Power off the VM.

Right-click the VM and select:

**Template > Convert to Template**

### Step 3 — Deploy a VM from the Template

To deploy a new VM:

1. Right-click the template.
2. Select **New VM from This Template**.
3. Enter the VM name.
4. Select the deployment location.
5. Select the storage.

### Step 4 — Configure Customization

During the **Customize** step, select a **Customization Specification** to automatically configure:

* Hostname
* IP address
* Domain join
* Administrator password

### Step 5 — Complete the Deployment

Finish the wizard.

The cloned VM boots with the customized identity.

Verify:

* Hostname
* IP address
* Domain membership

### Step 6 — Understand Clone Types

| Clone Type       | Description                 | Recommended Use       |
| ---------------- | --------------------------- | --------------------- |
| **Full Clone**   | Independent copy            | Production            |
| **Linked Clone** | Shares the parent base disk | Development/Test only |

### 💻 PowerCLI — Deploy 5 VMs from a Template

```powershell
$template = Get-Template -Name 'WS2019-Gold'
$spec = Get-OSCustomizationSpec -Name 'Win-Domain-Spec'

foreach ($i in 1..5) {
    New-VM `
        -Name "APP-VM-0$i" `
        -Template $template `
        -OSCustomizationSpec $spec `
        -Datastore 'SAN-DS01' `
        -ResourcePool 'PROD-RP'
}
```

---

## 🌐 Task 2: Configure VLANs on vSphere Standard Switch (VSS)

### Step 1 — Open Virtual Switch Configuration

In vCenter, navigate to:

**Host > Configure > Networking > Virtual Switches**

Select the required **vSwitch** and choose:

**Add Networking**

### Step 2 — Create a VM Port Group

1. Choose **Virtual Machine Port Group**.
2. Enter a descriptive name.

Example:

```text
VLAN100-PROD
```

3. Set the **VLAN ID** to:

```text
100
```

4. Click **Finish**.

### Step 3 — Assign VMs to the VLAN Port Group

Navigate to:

**Edit VM Settings > Network Adapter**

Select:

```text
VLAN100-PROD
```

### Step 4 — Configure the Physical Switch

On the physical switch, ensure the **ESXi uplink port** is configured as a **trunk port** carrying VLAN 100.

### 💻 ESXi CLI — Add a Port Group with VLAN

Create the port group:

```bash
esxcli network vswitch standard portgroup add \
  -v vSwitch0 \
  -p VLAN200-DEV
```

Set the VLAN ID:

```bash
esxcli network vswitch standard portgroup set \
  -p VLAN200-DEV \
  --vlan-id 200
```

List standard switch port groups:

```bash
esxcli network vswitch standard portgroup list
```

---

## 🌐 Task 3: Configure Static Routing and Network Namespaces (Linux)

### Step 1 — View the Routing Table

```bash
ip route show
```

### Step 2 — Add a Static Route

Use the following syntax:

```bash
ip route add <network>/<prefix> via <gateway> dev <interface>
```

Example:

```bash
ip route add 192.168.50.0/24 via 10.0.0.1 dev eth0
```

### Step 3 — Make the Route Persistent

For RHEL, edit:

```text
/etc/sysconfig/network-scripts/route-eth0
```

### Step 4 — Create a Network Namespace

Create the namespace:

```bash
ip netns add ns-app1
```

Move the interface into the namespace:

```bash
ip link set veth0 netns ns-app1
```

### Step 5 — Configure an IP Address in the Namespace

```bash
ip netns exec ns-app1 ip addr add 10.1.1.1/24 dev veth0
```

### 💻 Routing Commands

View the routing table:

```bash
ip route show
```

Add a static route:

```bash
ip route add 192.168.50.0/24 via 10.0.0.1 dev eth0
```

### 💻 Network Namespace Commands

Create the namespace:

```bash
ip netns add ns-app1
```

List network namespaces:

```bash
ip netns list
```

Bring the loopback interface up:

```bash
ip netns exec ns-app1 ip link set lo up
```

Configure the IP address:

```bash
ip netns exec ns-app1 ip addr add 10.1.1.1/24 dev veth0
```

---

## 🌐 Task 4: Configure DHCP and DNS Services

### Step 1 — Configure Windows DHCP

Open:

**DHCP Manager > IPv4 > New Scope**

Define:

* IP range
* Subnet mask
* Gateway
* DNS
* Lease duration

Activate the scope.

### Step 2 — Verify Client IP Assignment

On the client, run:

```cmd
ipconfig /release
ipconfig /renew
```

Verify that the client receives the expected IP configuration.

### Step 3 — Configure Linux DHCP

For **isc-dhcp-server**:

1. Edit:

```text
/etc/dhcp/dhcpd.conf
```

2. Define:

   * Subnet
   * IP range
   * Gateway
   * DNS

3. Restart the DHCP service.

### Step 4 — Configure Windows DNS

In **Windows DNS Manager**:

1. Create **Forward Lookup Zones**.
2. Add **A records** for servers.
3. Add **CNAME records** for aliases.

### Step 5 — Test DNS

Use:

```cmd
nslookup server01.corp.local <dns-ip>
```

Verify both:

* Forward lookups
* Reverse lookups

### 💻 Linux DHCP Configuration

Example `/etc/dhcp/dhcpd.conf` configuration:

```conf
subnet 10.0.10.0 netmask 255.255.255.0 {
    range 10.0.10.100 10.0.10.200;
    option routers 10.0.10.1;
    option domain-name-servers 10.0.10.5, 8.8.8.8;
    default-lease-time 86400;
}
```

### 💻 DNS Testing

Use `nslookup`:

```bash
nslookup prod-db-01.corp.local
```

Use `dig` against the specified DNS server:

```bash
dig @10.0.10.5 prod-db-01.corp.local
```

---

# ⭐ Best Practice Tips

* Always run **Sysprep** before converting a Windows VM to a template to reset SID and hostname.
* Use **VDS (vSphere Distributed Switch)** instead of VSS for consistent port group configuration across all hosts.
* Tag all VLANs with environment labels (**PROD, DEV, DMZ**) for clarity during troubleshooting.
* Use **DHCP reservations** for servers based on MAC address — this provides the benefit of both DHCP and static IP.
* Enable **DNS scavenging** to automatically remove stale DNS records.
* Document your IP addressing scheme in a centralized **IPAM** tool (**phpIPAM, NetBox**).
