# VirtualBox and Windows Server 2022 Domain Controller Lab Setup

**Goal:** Install VirtualBox and create two Windows Server 2022 virtual machines (DC1 and DC2) for Active Directory domain controller lab practice

## Step 1: VirtualBox Installation

**Version:** VirtualBox 7.2.4 r170995 (Qt 6.8.0)  
**Download source:** https://www.virtualbox.org/

## Step 2: Windows Server 2022 ISO

**Download source:** https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022  
**Edition:** Windows Server 2022 Standard Evaluation  
**Installation type:** Desktop Experience (GUI)  
**License:** 180-day evaluation license

## Step 3: First VM Creation - DC1

**VM name:** DC1  
**Purpose:** Primary domain controller

### Configuration:
- **RAM:** 2 GB
- **CPU cores:** 1
- **Disk size:** 100 GB (dynamically allocated)
- **Disk controller:** SATA
- **Network adapters:**
  - Adapter 1: NAT (internet access)
  - Adapter 2: Internal Network (named "lan")

### Installation process:
1. Created new VM with Windows 2022 (64-bit) profile
2. **Installation method:** Manual (supervised) installation -  skipped VirtualBox unattended installation option
3. Mounted Windows Server 2022 ISO
4. Started VM and followed installation wizard step by step
5. Selected "Windows Server 2022 Standard Evaluation (Desktop Experience)"
6. Custom installation on entire disk
7. Set administrator password
8. Installation time: approximately 15 minutes

### Post-installation:
Installed VirtualBox Guest Additions:
- Mounted Guest Additions ISO from Devices menu
- Ran VBoxWindowsAdditions.exe as administrator
- Rebooted VM
- Verified screen resolution and clipboard sharing working correctly

## Step 4: Second VM Creation - DC2

**VM name:** DC2  
**Purpose:** Secondary domain controller (backup/replication)

### Configuration:
- **RAM:** 2 GB
- **CPU cores:** 1
- **Disk size:** 100 GB (dynamically allocated)
- **Disk controller:** SATA
- **Network adapters:**
  - Adapter 1: Internal Network (named "internal-lan") - no NAT adapter

### Installation process:
Same steps as DC1, with the difference in network configuration.

### Post-installation:
Installed VirtualBox Guest Additions following the same procedure as DC1.

## Network Configuration Strategy

**Design choice:**
- DC1 has both NAT (internet) and Internal LAN (domain communication)
- DC2 has only Internal LAN (isolated, domain-only communication)

This setup simulates a real scenario where:
- DC1 can download updates and access external resources
- DC2 is isolated from internet for security
- Both can communicate on the internal domain network

## Current Progress

### Completed:
- [x] VirtualBox installation
- [x] Two Windows Server 2022 VMs created
- [x] Guest Additions installed on both VMs
- [x] Network adapters configured

### Next Steps:
1. Configure static IP address on DC1's Internal LAN adapter
2. Create VM snapshots for both machines (clean state before domain configuration)
3. Install Active Directory Domain Services on DC1
4. Promote DC1 to domain controller
5. Join DC2 to the domain
6. Promote DC2 to secondary domain controller

## Technical Notes

**Why 2 GB RAM per VM?**  
Windows Server 2022 minimum requirement is 512 MB, but 2 GB is the recommended minimum for Active Directory operations. With 64 GB host RAM, this leaves plenty of resources for the host OS and other applications.

**Why 1 CPU core?**  
Sufficient for lab purposes and domain controller operations in a testing environment. Can be increased if performance issues occur.

**Why 100 GB disk?**  
Windows Server 2022 requires approximately 32 GB. 100 GB provides space for:
- OS and updates
- Active Directory database
- Logs and diagnostic data
- Future lab exercises

## Snapshot Strategy

Before proceeding with Active Directory configuration, snapshots will be created:
- **DC1-Clean:** Fresh install with Guest Additions and network configured
- **DC2-Clean:** Fresh install with Guest Additions and network configured

This allows rolling back to a clean state if configuration errors occur during domain setup.

## References

- VirtualBox documentation: https://www.virtualbox.org/manual/
- Windows Server 2022 documentation: https://learn.microsoft.com/en-us/windows-server/
- Active Directory installation guide: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-
