# DHCP - Dynamic Host Configuration Protocol

## What It Is

DHCP is a network protocol that automatically assigns IP configuration to devices on a network.

Without DHCP, every device would need to be configured manually with an IP address, subnet mask, gateway, and DNS server. DHCP eliminates this work by handling the assignment automatically, the moment a device connects to the network.

---

## Why It Matters

In a network with many devices, manual IP configuration is slow, error prone, and hard to maintain. DHCP centralizes this process on a single server, making the network easier to manage and scale.

It is one of the first services configured in any network infrastructure, whether in a home router, a corporate environment, or a lab.

---

## How It Works — The DORA Process

When a device connects to a network, DHCP follows a four step process known as **DORA**:

| Step | Name | Description |
|------|------|-------------|
| 1 | **Discover** | The client broadcasts a message looking for a DHCP server |
| 2 | **Offer** | The server responds with an available IP address and configuration |
| 3 | **Request** | The client formally requests the offered IP address |
| 4 | **Acknowledge** | The server confirms the assignment and the client starts using the IP |

> The entire exchange happens in seconds and is invisible to the user.

---

## What DHCP Assigns

A DHCP server can assign the following parameters to each client:

| Parameter | Description | Example |
|-----------|-------------|---------|
| IP Address | Unique address for the device on the network
| Subnet Mask | Defines the size of the network
| Default Gateway | The router used to reach other networks
| DNS Server | The server used to resolve domain names
| Lease Duration | How long the IP assignment is valid

---

## Key Concepts

### Scope
The scope is the pool of IP addresses the DHCP server is allowed to distribute.

Example: a scope from `10.0.0.20` to `10.0.0.100` means the server will only assign addresses within that range.

### Lease
The lease is the time period for which an IP address is assigned to a client. When the lease expires, the client must renew it. If the client is no longer on the network, the address returns to the pool and can be reassigned.

### Exclusions
Exclusions are IP addresses within the scope range that the server will never assign. They are used to protect addresses already assigned statically to servers or other fixed devices.

### Reservations
A reservation permanently binds a specific IP address to a specific device, identified by its MAC address. The device always receives the same IP, even though it still goes through the DORA process.

### Options
DHCP options are additional configuration parameters distributed alongside the IP address. The most common ones are:

- **Option 003** — Default Gateway
- **Option 006** — DNS Server
- **Option 015** — DNS Domain Name
- **Option 044** — WINS Server (legacy)

---

## DHCP in a Domain vs Workgroup

| Scenario | DNS Server distributed by DHCP |
|----------|-------------------------------|
| **Domain** (e.g. Active Directory) | IP of the local DNS server (e.g. 10.0.0.1) |
| **Workgroup** | Public DNS server (e.g. 8.8.8.8 or 1.1.1.1) |

In a domain environment, the DNS server is typically the Domain Controller itself. Clients need to resolve internal names (like `dc1.esempio.com`) so they must point to the local DNS, not a public one.

In a workgroup, there is no local DNS server, so a public DNS must be used to resolve internet names.

---

## Useful Commands

### Windows

```cmd
# View current IP configuration
ipconfig /all

# Release the current IP address
ipconfig /release

# Request a new IP address from the DHCP server
ipconfig /renew
```

### Linux

```bash
# View current IP configuration
ip addr show

# Request a new IP address (replace eth0 with your interface name)
sudo dhclient eth0

# Release the current IP address
sudo dhclient -r eth0
```

---

## Common Issues and Solutions

| Problem | Likely Cause | Solution |
|---------|-------------|---------|
| Client gets IP `169.254.x.x` | No DHCP server found | Check if DHCP service is running and the scope is active |
| Client does not get an IP | Scope is exhausted or inactive | Verify scope range, check active leases |
| Wrong DNS assigned | Incorrect Option 006 in scope | Correct the DNS address in DHCP scope options |
| IP conflict on network | Static IP inside scope range | Add an exclusion for that IP in the DHCP scope |

> An IP address starting with `169.254.x.x` is called an **APIPA address** (Automatic Private IP Addressing). It means the device could not reach a DHCP server and assigned itself a fallback address. It is a clear sign of a DHCP problem.

---

## Lab Notes

Tested in a VirtualBox lab with two Windows Server 2022 virtual machines:

- **DC1** — DHCP server, static IP `10.0.0.1` on the internal LAN adapter
- **DC2** — DHCP client, receives IP automatically from DC1

### Scope configured in the lab

| Parameter | Value |
|-----------|-------|
| Scope name | lan_scope |
| Start IP | 10.0.0.20 |
| End IP | 10.0.0.100 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 10.0.0.1 |
| DNS Server | 10.0.0.1 |
| Lease Duration | 8 days |

### Installation on Windows Server 2022

1. Open Server Manager → Manage → Add Roles and Features
2. Select **DHCP Server** in the Server Roles list
3. Complete the installation wizard
4. After installation, click the yellow notification flag → **Complete DHCP configuration** → Commit

### Verification

After configuration, DC2 received the following via DHCP (verified with `ipconfig /all`):
