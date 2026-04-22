# DNS - Domain Name System

## What It Is

DNS is the protocol that translates human readable domain names into IP addresses that computers can use to communicate.

When you type `google.com` in a browser, your computer does not know where that is. It sends a query to a DNS server, which responds with the corresponding IP address (e.g. `142.250.180.46`). Your computer then uses that IP to establish the connection.

DNS is often described as the **phone book of the internet**: you look up a name and get back a number.

---

## Why It Matters

IP addresses are hard to remember and can change over time. Domain names are stable, readable, and meaningful. DNS is the layer that connects the two.

Without DNS, every connection to a website or service would require knowing the exact IP address. DNS makes the entire internet usable for humans.

In a network infrastructure, DNS is also critical for internal name resolution - servers, domain controllers, and services are referenced by name, not by IP.

---

## How DNS Works - The Resolution Process

When a client needs to resolve a domain name, the following steps occur:

1. **Local cache check** - the client checks if it has recently resolved this name and cached the result
2. **Hosts file check** - the client checks the local `hosts` file for a static entry
3. **Recursive query to DNS resolver** - the client sends the query to its configured DNS server
4. **Root servers** - if the resolver does not know the answer, it queries one of the 13 root DNS servers
5. **TLD nameserver** - the root server directs the query to the nameserver responsible for the top-level domain (e.g. `.com`)
6. **Authoritative nameserver** - the TLD server directs the query to the authoritative DNS server for the specific domain
7. **Response** - the authoritative server returns the IP address, which is cached and returned to the client

> This entire process typically completes in milliseconds.

---

## DNS Record Types

DNS stores information in records. Each record type serves a different purpose.

| Record | Name | Description | Example |
|--------|------|-------------|---------|
| **A** | Address | Maps a domain name to an IPv4 address | `dc1.esempio.com → 10.0.0.1` |
| **AAAA** | IPv6 Address | Maps a domain name to an IPv6 address | `dc1.esempio.com → ::1` |
| **CNAME** | Canonical Name | Alias pointing to another domain name | `www → dc1.esempio.com` |
| **MX** | Mail Exchange | Specifies the mail server for a domain | `mail.esempio.com` |
| **PTR** | Pointer | Reverse lookup - maps IP to domain name | `10.0.0.1 → dc1.esempio.com` |
| **NS** | Name Server | Lists the authoritative DNS servers for a domain | `ns1.esempio.com` |
| **SOA** | Start of Authority | Contains administrative info about the zone | Serial, refresh, retry intervals |
| **TXT** | Text | Stores arbitrary text, used for verification and security | SPF, DKIM records |

---

## Key Concepts

### Zone
A DNS zone is a portion of the DNS namespace managed by a specific DNS server. For example, the zone `esempio.com` contains all DNS records for that domain.

### Forward Lookup Zone
Resolves names to IP addresses. This is the standard direction: you know the name, you want the IP.

### Reverse Lookup Zone
Resolves IP addresses to names. Useful for diagnostics and logging. Uses a special domain called `in-addr.arpa`.

### Forwarders
Forwarders are external DNS servers that a local DNS server uses to resolve names it does not know. For example, a Windows DNS server configured with `8.8.8.8` as a forwarder will send all unknown queries to Google DNS.

### TTL - Time to Live
TTL is the time (in seconds) that a DNS record can be cached by a resolver. A short TTL means changes propagate quickly but generate more DNS traffic. A long TTL reduces traffic but slows propagation.

### Recursive vs Iterative Query
- **Recursive**: the client asks the resolver to do all the work and return the final answer
- **Iterative**: the resolver returns the best answer it has (e.g. "ask this server next") and the client continues querying

---

## DNS in a Domain vs Workgroup

| Scenario | DNS Configuration |
|----------|------------------|
| **Active Directory Domain** | DNS is installed on the Domain Controller. Clients use the DC's IP as DNS server. Internal names (e.g. `dc1.esempio.com`) are resolved locally. |
| **Workgroup** | No local DNS server. Clients use a public DNS server (e.g. `8.8.8.8`). Only internet names are resolved. |

In an Active Directory environment, DNS is tightly integrated with the domain. The DC registers its own DNS records automatically, and clients must use the local DNS server to locate domain services.

---

## DNS and Active Directory

Active Directory depends entirely on DNS. When a client wants to log in to the domain or find a Domain Controller, it queries DNS for special records called **SRV records**.

For example, to find a Domain Controller for `esempio.com`, the client queries:

```
_ldap._tcp.esempio.com
```

If DNS is misconfigured or unreachable, clients cannot join or authenticate to the domain.

> This is why, when adding a machine to a domain, the DNS server must be set to the Domain Controller's IP **before** attempting to join.

---

## Useful Commands

### Windows

```cmd
# Display current DNS configuration
ipconfig /all

# Flush the local DNS cache
ipconfig /flushdns

# Query a specific DNS record
nslookup google.com

# Query using a specific DNS server
nslookup google.com 8.8.8.8

# Query a specific record type
nslookup -type=MX google.com
```

### Linux

```bash
# Query DNS resolution
dig google.com

# Query a specific record type
dig MX google.com

# Query using a specific DNS server
dig @8.8.8.8 google.com

# Simple name resolution
nslookup google.com

# Flush DNS cache (systemd-resolved)
sudo systemd-resolve --flush-caches
```

---

## Common Issues and Solutions

| Problem | Likely Cause | Solution |
|---------|-------------|---------|
| `nslookup` times out | DNS server unreachable | Check DNS server IP in `ipconfig /all`, verify connectivity |
| Domain names not resolving but IPs work | DNS misconfigured or unreachable | Set correct DNS server on the client |
| Internal names not resolving | Client using public DNS instead of local | Set DNS to Domain Controller IP |
| Cannot join domain | DNS not pointing to Domain Controller | Set preferred DNS to DC IP before joining |
| `nslookup` works but browser does not | Browser proxy or hosts file issue | Check browser settings and `C:\Windows\System32\drivers\etc\hosts` |
| Forwarders not working | Wrong forwarder IP or firewall blocking port 53 | Verify forwarder addresses, check firewall rules for UDP/TCP port 53 |

---

## DNS Port

DNS uses **port 53** on both UDP and TCP:

- **UDP 53** - standard queries (fast, connectionless)
- **TCP 53** - used for large responses and zone transfers

Firewalls must allow traffic on port 53 for DNS to function correctly.

---

## Lab Notes

Tested in a VirtualBox lab with two Windows Server 2022 virtual machines:

- **DC1** - DNS server (role installed alongside Active Directory)
- **DC2** - DNS client, configured to use `10.0.0.1` as DNS server via DHCP

### Installation on Windows Server 2022

The DNS role can be installed:
- **Automatically** - as a dependency when installing Active Directory Domain Services
- **Manually** - via Server Manager → Add Roles and Features → DNS Server

### Forwarder configuration on DC1

After installation, forwarders were configured to allow DC1 to resolve internet names:

1. Server Manager → Tools → DNS
2. Right-click DC1 → Properties → Forwarders → Edit
3. Added the following forwarders:

| Address | Provider |
|---------|----------|
| `8.8.8.8` | Google DNS |
| `8.8.4.4` | Google DNS secondary |
| `1.1.1.1` | Cloudflare DNS |

### Common issue encountered

During lab setup, DC2 was initially configured with DNS `10.0.0.10` (incorrect IP) instead of `10.0.0.1`. This caused the domain join to fail with the error:

> *"Unable to contact an Active Directory Domain Controller for the domain"*

The issue was resolved by:
1. Manually correcting the DNS to `10.0.0.1` on DC2
2. Correcting Option 006 in the DHCP scope on DC1

After the fix, `nslookup esempio.com` returned the correct IP and the domain join succeeded.

### Verification

From DC2:

```cmd
nslookup google.com
```

```
Server:  dc1.esempio.com
Address: 10.0.0.1

Non-authoritative answer:
Name:    google.com
Address: 142.250.180.46
```

Internal and external name resolution confirmed working.

---

## Summary

| Concept | Key Point |
|---------|-----------|
| DNS | Translates domain names into IP addresses |
| A Record | Maps a name to an IPv4 address |
| PTR Record | Reverse lookup - maps an IP to a name |
| Forwarder | External DNS server used for unknown queries |
| TTL | Time a DNS record is cached before being refreshed |
| Port 53 | DNS uses UDP 53 (queries) and TCP 53 (zone transfers) |
| SRV Record | Used by Active Directory to locate domain services |
| APIPA + DNS | If DNS fails, domain authentication fails entirely |
