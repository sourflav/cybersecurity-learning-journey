# NAT - Network Address Translation

## What It Is

NAT is a technique that allows multiple devices on a private network to access the internet using a single public IP address.

When a device on a private network sends traffic to the internet, NAT intercepts the packet and replaces the private source IP address with the public IP address of the router or gateway. When the response comes back, NAT reverses the process and forwards the packet to the correct device on the internal network.

---

## Why It Matters

IPv4 addresses are limited. There are not enough public IP addresses to assign one to every device in the world. NAT solves this problem by allowing an entire network of private devices to share a single public IP.

Beyond address conservation, NAT also provides a basic layer of security: devices on the internal network are not directly reachable from the internet, because their private addresses are not routable outside the local network.

---

## Private vs Public IP Addresses

Not all IP addresses are equal. Some ranges are reserved for private use and are never routed on the public internet.

| Range | Subnet Mask | Common Use |
|-------|-------------|------------|
| `10.0.0.0 – 10.255.255.255` | 255.0.0.0 | Large private networks, labs |
| `172.16.0.0 – 172.31.255.255` | 255.240.0.0 | Medium private networks |
| `192.168.0.0 – 192.168.255.255` | 255.255.0.0 | Home and small office networks |

Any device using one of these addresses needs NAT to communicate with the public internet.

---

## How NAT Works

### Basic flow

1. A client on the internal network (`10.0.0.25`) sends a request to `8.8.8.8` (Google DNS)
2. The packet reaches the NAT gateway (e.g. `10.0.0.1`)
3. The gateway replaces the source IP `10.0.0.25` with its own public IP and records the mapping in a translation table
4. The packet is sent to `8.8.8.8` with the public IP as source
5. `8.8.8.8` replies to the public IP
6. The gateway receives the reply, looks up the translation table, and forwards the packet back to `10.0.0.25`

The internal client is never directly visible to the outside world.

---

## Types of NAT

| Type | Description | Common Use |
|------|-------------|------------|
| **Static NAT** | One private IP is permanently mapped to one public IP | Hosting a server accessible from outside |
| **Dynamic NAT** | Private IPs are mapped to a pool of public IPs on demand | Less common today |
| **PAT / NAT Overload** | Multiple private IPs share a single public IP using different ports | Home routers, most real-world scenarios |

> **PAT (Port Address Translation)** is the most widely used form of NAT. It is what your home router does every time you browse the internet. It is also called **NAT Overload**.

---

## NAT and Port Numbers

PAT tracks connections using port numbers in addition to IP addresses. Each outgoing connection is assigned a unique source port, so the gateway can distinguish between multiple simultaneous sessions from different internal devices.

Example:

| Internal Device | Internal Port | Public IP | Assigned Port |
|----------------|--------------|-----------|---------------|
| 10.0.0.20 | 49152 | 203.0.113.5 | 60001 |
| 10.0.0.21 | 49153 | 203.0.113.5 | 60002 |
| 10.0.0.22 | 49154 | 203.0.113.5 | 60003 |

All three devices share the same public IP but are distinguishable by their assigned port.

---

## NAT in VirtualBox

VirtualBox includes a built-in NAT adapter that simulates exactly this behavior. When a virtual machine uses the NAT network adapter:

- The VM receives a private IP in the `10.0.2.x` range
- The VirtualBox engine acts as the NAT gateway (`10.0.2.2`)
- The VM can reach the internet through the host machine
- The VM is not directly reachable from outside

This is the standard configuration used in lab environments to give VMs internet access without exposing them to the network.

| VirtualBox NAT defaults | Value |
|------------------------|-------|
| VM IP range | 10.0.2.x |
| Gateway | 10.0.2.2 |
| DNS | 10.0.2.3 |

---

## NAT vs Routing

It is important not to confuse NAT with routing. They are related but different concepts.

| Concept | Description |
|---------|-------------|
| **Routing** | Decides where to forward a packet based on its destination IP |
| **NAT** | Modifies the source or destination IP address of a packet in transit |

In practice, NAT is almost always combined with routing. A gateway that performs NAT also routes packets between the internal network and the internet.

---

## Useful Commands

### Windows — verify NAT connectivity

```cmd
# Check current IP and gateway
ipconfig /all

# Test internet connectivity through NAT
ping 8.8.8.8

# Test DNS resolution through NAT
nslookup google.com
```

### Windows Server — enable NAT via Routing and Remote Access

NAT on Windows Server is configured through the **Routing and Remote Access** role:

1. Install role: Server Manager → Add Roles and Features → Remote Access → Routing
2. Open: Server Manager → Tools → Routing and Remote Access
3. Right-click the server → Configure and Enable Routing and Remote Access
4. Select **Network Address Translation (NAT)**
5. Select the external adapter (the one connected to the internet)
6. Complete the wizard

---

## Common Issues and Solutions

| Problem | Likely Cause | Solution |
|---------|-------------|---------|
| Internal clients cannot reach the internet | Wrong adapter selected during NAT setup | Re-run the wizard and select the correct external adapter |
| NAT configured but no internet on clients | DHCP gateway not pointing to NAT server | Verify that the default gateway distributed by DHCP matches the NAT server IP |
| Clients can ping IPs but not domain names | DNS not configured on clients | Check DNS option in DHCP scope or set DNS manually on the client |
| VMs can reach internet but not each other | VMs on different network types in VirtualBox | Ensure both VMs use the same Internal Network adapter name |

---

## Lab Notes

Tested in a VirtualBox lab with two Windows Server 2022 virtual machines:

- **DC1** - NAT gateway
  - Adapter 1: NAT (VirtualBox) - IP `10.0.2.x` - connected to internet
  - Adapter 2: Internal Network `lan` - IP `10.0.0.1` - connected to DC2
- **DC2** - internal client, routes all traffic through DC1

### Configuration on DC1

The NAT role was installed as part of the **Remote Access** server role, with the **Routing** sub-role enabled.

During the Routing and Remote Access wizard:
- Selected: **Network Address Translation (NAT)**
- External interface: the adapter with IP `10.0.2.x` (VirtualBox NAT adapter)
- Internal interface: automatically detected as `10.0.0.1`

### Verification

From DC2, after receiving IP via DHCP:

```cmd
ping 8.8.8.8
```

Reply received - internet access through NAT confirmed.

---

## Summary

| Concept | Key Point |
|---------|-----------|
| NAT | Translates private IPs to a public IP for internet access |
| PAT | Most common form of NAT — uses port numbers to track multiple sessions |
| Private IP | Address not routable on the public internet (10.x, 172.16.x, 192.168.x) |
| Translation table | Internal record the NAT gateway uses to match replies to internal devices |
| VirtualBox NAT | Built-in NAT adapter — assigns IPs in the 10.0.2.x range |
