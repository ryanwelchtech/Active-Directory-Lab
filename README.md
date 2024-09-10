# Home Lab: Active Directory Setup with Oracle VirtualBox

This project demonstrates the setup of a basic home lab to simulate a corporate network environment using Oracle VirtualBox, Microsoft Windows 10, and Microsoft Server 2019. The setup includes configuring Active Directory, DNS, DHCP, and Remote Access Server (RAS) for NAT (Network Address Translation). Additionally, a custom PowerShell script is used to automate the provisioning, maintenance, and de-provisioning of 1000 user accounts.

## Table of Contents

- [Overview](#overview)
- [Technologies Used](#technologies-used)
- [Setup Guide](#setup-guide)
- [PowerShell Script](#powershell-script)
- [Network Configuration](#network-configuration)
- [Active Directory Configuration](#active-directory-configuration)
- [DNS and DHCP Setup](#dns-and-dhcp-setup)
- [Remote Access Server (RAS) Setup](#remote-access-server-ras-setup)
- [Contributing](#contributing)
- [License](#license)

## Overview

The goal of this project was to simulate a corporate network environment in a controlled lab setting using virtualization technologies. By using Oracle VirtualBox to host both a Windows 10 client and a Windows Server 2019 machine, this environment allows for practicing network administration, automation, and user management tasks.

The key features of the project include:

- Active Directory setup with 1000 users created via a custom PowerShell script.
- Remote Access Server (RAS) configured to provide NAT, allowing the virtual machines to access the internet.
- Automated DNS and DHCP services to manage network configurations.
- Automating user account lifecycle (provisioning, maintenance, and de-provisioning).

## Technologies Used

- **Oracle VirtualBox** – Virtualization software for creating virtual machines.
- **Microsoft Server 2019** – Configured as the Active Directory Domain Controller.
- **Microsoft Windows 10** – Client machine joined to the domain.
- **PowerShell** – Used for scripting and automation.
- **Remote Access Server (RAS)** – For providing NAT and remote access support.
- **DNS/DHCP** – For dynamic IP assignment and name resolution.

## Setup Guide

### Prerequisites

- Oracle VirtualBox installed on your host machine.
- Windows Server 2019 and Windows 10 ISO files for virtual machine creation.
- A stable internet connection for downloading updates and connecting to external resources.

### Steps

1. **Create Virtual Machines**:
   - Use Oracle VirtualBox to create two virtual machines:
     - One with Microsoft Server 2019.
     - One with Microsoft Windows 10.

2. **Install Active Directory**:
   - Install and configure Active Directory on the Windows Server 2019 machine.
   - Promote the server to a Domain Controller and configure the domain.

3. **PowerShell Script for User Management**:
   - Run the custom PowerShell script to create 1000 users in Active Directory.  
   - The script will also handle user account maintenance and de-provisioning.

4. **Configure Remote Access Server (RAS)**:
   - Set up RAS on the server to provide NAT and remote access support to the virtual machines.

5. **DNS and DHCP Setup**:
   - Configure DNS and DHCP services on the Windows Server to manage IP assignments for the Windows 10 machine and future clients.

## PowerShell Script

The custom PowerShell script automates the following tasks:

- **User Creation**: Automatically provisions 1000 users in Active Directory.
- **User Maintenance**: Handles bulk updates and changes to user accounts.
- **Deprovisioning**: Automatically disables and removes user accounts based on defined criteria.

### Account generation Powershell script

```powershell
# ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$USER_FIRST_LAST_LIST = Get-Content .\names.txt
# ------------------------------------------------------ #

$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()
    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
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

### Account maintenance/update Powershell script

```powershell
# ----- Edit these Variables for your own Use Case ----- #
$USER_FIRST_LAST_LIST = Get-Content .\names.txt
$NEW_DEPARTMENT = "IT Support"
$NEW_PHONE_NUMBER = "123-456-7890"
# ------------------------------------------------------ #

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()

    $user = Get-ADUser -Filter { SamAccountName -eq $username }

    if ($user) {
        Write-Host "Updating user: $($username)" -BackgroundColor Black -ForegroundColor Yellow

        # Update user attributes (Department and Phone Number in this example)
        Set-ADUser -Identity $user `
                   -Department $NEW_DEPARTMENT `
                   -OfficePhone $NEW_PHONE_NUMBER
    } else {
        Write-Host "User not found: $($username)" -BackgroundColor Black -ForegroundColor Red
    }
}

```

### Account de-provisioning Powershell script

```powershell
# ----- Edit these Variables for your own Use Case ----- #
$USER_FIRST_LAST_LIST = Get-Content .\names.txt
$DISABLED_OU = "OU=Disabled_Users,$(([ADSI]`"").distinguishedName)"
$INACTIVITY_THRESHOLD_DAYS = 90
# ------------------------------------------------------ #

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()

    $user = Get-ADUser -Filter { SamAccountName -eq $username } -Properties LastLogonDate

    if ($user) {
        $lastLogon = $user.LastLogonDate
        $daysInactive = (Get-Date) - $lastLogon

        if ($daysInactive.TotalDays -gt $INACTIVITY_THRESHOLD_DAYS) {
            Write-Host "Disabling user: $($username)" -BackgroundColor Black -ForegroundColor Red

            # Disable the user account
            Disable-ADAccount -Identity $user

            # Move the user to the Disabled_Users OU
            Move-ADObject -Identity $user.DistinguishedName -TargetPath $DISABLED_OU
        }
    } else {
        Write-Host "User not found: $($username)" -BackgroundColor Black -ForegroundColor Red
    }
}


```

## Network Configuration
The network is configured with NAT through RAS to allow the virtual machines to connect to the internet. This setup provides the flexibility to simulate real-world networking scenarios, including external and internal traffic.

## Active Directory Configuration
Active Directory was set up to manage users, computers, and resources within the domain. The domain controller handles authentication and group policy management for the lab environment.

## Domain Structure
Domain Name: homelab.local
User Base: 1000 users created via PowerShell.

## DNS and DHCP Setup
DNS: Configured to resolve internal domain names and provide name resolution services to clients.
DHCP: Configured to automatically assign IP addresses to virtual machines within the network.

## Remote Access Server (RAS) Setup
RAS was implemented to support NAT and provide secure remote access. This allows virtual machines to access the internet while staying within the isolated lab environment.

## Contributing
If you'd like to contribute to this project, feel free to fork the repository and submit a pull request. Issues and suggestions for improvements are welcome!

## License
This project is licensed under the MIT License.
