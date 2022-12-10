---
title: "Active Directory Domain Controller Deployment with Powershell"

date: 2021-10-22
url: /addc-powershell/
image: images/2021-thumbs/addc-powershell.webp
categories:
  - Windows
tags:
  - activedirectory
draft: false
-----

# Rename the Server
```powershell
PS C:\Users\Administrator> Rename-Computer XOGS-E-DC01
WARNING: The changes will take effect after you restart the computer XOGS-DC01.
PS C:\Users\Administrator> Restart-Computer
```
Wait for the server to reboot.

```powershell
PS C:\Users\Administrator> hostname XOGS-E-DC01
```

# Install ADDC Role
```powershell
PS C:\Users\Administrator> Install-windowsfeature -name AD-Domain-Services -IncludeManagementTools

Success Restart Needed Exit Code      Feature Result
------- -------------- ---------      --------------
True    No             Success        {Active Directory Domain Services, Group P...

PS C:\Users\Administrator> Get-WindowsFeature | Format-List Name,InstallState


Name         : AD-Certificate
InstallState : Available

Name         : ADCS-Cert-Authority
InstallState : Available

Name         : ADCS-Enroll-Web-Pol
InstallState : Available

Name         : ADCS-Enroll-Web-Svc
InstallState : Available

Name         : ADCS-Web-Enrollment
InstallState : Available

Name         : ADCS-Device-Enrollment
InstallState : Available

Name         : ADCS-Online-Cert
InstallState : Available

Name         : AD-Domain-Services
InstallState : Installed

Name         : ADFS-Federation
InstallState : Available

Name         : ADLDS
InstallState : Available
```

# Promote the Server to Domain Controller
Firstly, use the Test cmdlets to run prerequisite checks for the installation. If this is a real deployment, please use a strong password.

```powershell
PS C:\Users\Administrator> Test-ADDSForestInstallation -DomainName lab.xogs.io -SafeModeAdministratorPassword (ConvertTo-SecureString -String "P@55w0rd1" -AsPlainText -Force)
<-- OMITTED -->
Message                          Context                                  RebootRequired  Status
-------                          -------                                  --------------  ------
Operation completed successfully Test.VerifyDcPromoCore.DCPromo.General.3          False Success
````

If the operation was successful run the installation for real. If this is a real deployment, please use a strong password.

```poweshell
PS C:\Users\Administrator> Install-ADDSForest -DomainName lab.xogs.io -SafeModeAdministratorPassword (ConvertTo-SecureString -String "P@55w0rd1" -AsPlainText -Force)
The target server will be configured as a domain controller and restarted when this operation is complete.
Do you want to continue with this operation?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"):
```

After the server has rebooted, we can check the DCs configured in the lab.xogs.io domain where we should see our newly promoted DC.

```powershell
PS C:\Users\Administrator> Get-ADDomainController | Format-List Name,Domain,IPv4Address
Name        : XOGS-E-DC01
Domain      : lab.xogs.io
IPv4Address : 10.10.0.5
```
