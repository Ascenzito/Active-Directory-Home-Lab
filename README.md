# Active Directory Home Lab

## Objective

The Active Directory Home Lab project aimed to simulate a small enterprise network environment by deploying a Windows Server 2019 Domain Controller inside Oracle VirtualBox. The goal was to gain practical experience configuring core Windows Server services including Active Directory Domain Services, DNS, DHCP, and RAS/NAT and to automate bulk user provisioning using PowerShell. This environment mirrors the kind of infrastructure found in real-world corporate networks, providing a foundation for understanding identity management, network services, and domain-joined client behaviour.

### Skills Learned

- Deployed and configured a Windows Server 2019 Domain Controller in a virtualised environment using Oracle VirtualBox.
- Installed and configured Active Directory Domain Services (AD DS) and promoted a server to a Domain Controller.
- Configured Remote Access Server (RAS) with NAT to allow internal network clients to reach the internet through the Domain Controller.
- Set up a DHCP server on the Domain Controller to automatically assign IP addresses to domain-joined clients.
- Wrote and executed a PowerShell script to bulk-create over 1,000 user accounts in Active Directory from a names list.
- Joined a Windows 10 client machine to the domain and verified end-to-end connectivity and user login.
- Developed understanding of how internal DNS resolution works within a domain environment.
- Gained familiarity with network adapter configuration (NAT vs Internal Network) in VirtualBox to simulate real corporate network segmentation.

### Tools Used

- **Oracle VirtualBox** — Hypervisor used to host both the Windows Server 2019 Domain Controller and the Windows 10 client VM.
- **Windows Server 2019** — Server operating system used to host Active Directory, DNS, DHCP, and RAS/NAT services.
- **Active Directory Domain Services (AD DS)** — Directory service for managing users, computers, and policies within the domain.
- **PowerShell** — Used to automate the creation of bulk user accounts within Active Directory.
- **Windows 10** — Client machine joined to the domain to verify functionality and simulate an end-user workstation.

---

## Steps

### Step 1 – Download and Install Oracle VirtualBox

Downloaded Oracle VirtualBox and the VirtualBox Extension Pack from the official site. Also downloaded the Windows Server 2019 evaluation ISO from Microsoft and a Windows 10 ISO for the client VM.

*Ref 1: VirtualBox installation with Extension Pack installed*

---

### Step 2 – Create the Domain Controller VM

Created a new VM in VirtualBox named `DC` (Domain Controller). Configured it with two network adapters:
- **Adapter 1:** NAT — connects to the internet via the host machine.
- **Adapter 2:** Internal Network — connects to the private lab network where the Windows 10 client will live.

Mounted the Windows Server 2019 ISO and completed the OS installation.

*Ref 2: VirtualBox VM settings showing dual NIC configuration*

---

### Step 3 – Configure IP Addressing

After booting into Windows Server 2019, identified the two network adapters and renamed them for clarity (`INTERNET` and `INTERNAL`). Assigned a static IP address to the internal NIC:

- **IP:** 172.16.0.1
- **Subnet Mask:** 255.255.255.0
- **DNS:** 127.0.0.1 (loopback — the DC will serve as its own DNS once AD DS is installed)

*Ref 3: Static IP configuration on the internal network adapter*

---

### Step 4 – Install Active Directory Domain Services

Opened Server Manager and added the **Active Directory Domain Services** role. After installation, promoted the server to a Domain Controller by creating a new forest with the domain name `mydomain.com`. The server rebooted and the domain was live.

*Ref 4: AD DS role installation and domain promotion wizard*

---

### Step 5 – Create a Dedicated Admin Account

Rather than using the built-in Administrator account day-to-day, created a new Organisational Unit (OU) called `_ADMINS` in Active Directory Users and Computers, then created a personal admin user account (e.g. `a-pascenzo`) and added it to the **Domain Admins** group.

*Ref 5: New admin OU and user account created in ADUC*

---

### Step 6 – Configure RAS / NAT

Installed the **Remote Access** role with the **Routing** sub-feature to allow the internal network clients to reach the internet through the Domain Controller. Configured NAT on the external (internet-facing) NIC so that traffic from the internal network is translated and routed outbound.

*Ref 6: RAS/NAT configured with the internet-facing adapter selected*

---

### Step 7 – Set Up DHCP

Installed the **DHCP Server** role on the Domain Controller. Created a new scope to automatically assign IP addresses to domain-joined clients:

- **Scope Name:** 172.16.0.100-200
- **Range:** 172.16.0.100 – 172.16.0.200
- **Subnet Mask:** 255.255.255.0
- **Default Gateway:** 172.16.0.1 (the DC itself)
- **DNS:** 172.16.0.1

Authorised the DHCP server in Active Directory and activated the scope.

*Ref 7: DHCP scope configured and activated in DHCP Manager*

---

### Step 8 – Bulk Create Users with PowerShell

Used a PowerShell script to automate the creation of over 1,000 user accounts in Active Directory. The script reads from a plain-text `names.txt` file, generates usernames in `firstname.lastname` format, sets a default password, and creates all accounts inside a new OU called `_USERS`.

```powershell
# Snippet: core user creation loop
$PASSWORD_FOR_USERS = "Password1"
$USER_FIRST_LAST_LIST = Get-Content .\names.txt

$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last  = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()

    New-AdUser -AccountPassword $password `
               -GivenName $first `
               -Surname $last `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_USERS,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
}
```

*Ref 8: PowerShell script running — users being created in real time*

---

### Step 9 – Create and Join the Windows 10 Client VM

Created a second VM in VirtualBox named `CLIENT1` with a single network adapter set to **Internal Network** (same internal network as the DC). Installed Windows 10, then:

1. Verified the machine received an IP address from the DHCP server (in the 172.16.0.100–200 range).
2. Confirmed internet connectivity through the DC's NAT.
3. Joined the machine to `mydomain.com` via System Properties.
4. Rebooted and logged in using one of the bulk-created domain user accounts.

*Ref 9: CLIENT1 joined to mydomain.com and domain user login successful*

---

### Step 10 – Verify End-to-End Functionality

Confirmed the full lab was functioning correctly:
- `CLIENT1` obtained a DHCP lease from the DC.
- Internal DNS resolved `mydomain.com` correctly.
- Domain users could log into `CLIENT1` with their AD credentials.
- Internet access worked through the NAT configured on the DC.

*Ref 10: ipconfig output on CLIENT1 showing DHCP-assigned address and default gateway*
