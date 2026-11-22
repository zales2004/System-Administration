# Windows System Administration — Day 01

*Personal study notes · Windows basics, PowerShell and local account administration*

## 1. Topics covered

These are the main ideas and commands covered in this session:

1. Explain the role of an operating system, kernel, user mode, and kernel mode.
2. Identify important Windows directories and understand the purpose of NTFS and the Registry.
3. Inspect a Windows computer and identify its current user and hostname.
4. Create, inspect, and remove a local user account.
5. Explain local groups and apply the principle of least privilege.
6. Inspect processes and services using Task Manager and PowerShell.
7. Perform basic disk, system-file, and network troubleshooting.
8. Read Windows event logs and recognize commonly encountered event IDs.
9. Use PowerShell pipelines to filter and sort administrative information.
10. Follow a repeatable troubleshooting and documentation workflow.

## 2. Windows operating-system fundamentals

An operating system manages hardware and software resources and provides services that applications and users rely on.

### 2.1 Kernel

The kernel is the core of the operating system. Its responsibilities include:

- Scheduling processes and threads on the CPU.
- Managing memory.
- Coordinating access to hardware through drivers.
- Managing system resources and low-level operations.

### 2.2 User mode and kernel mode

- **User mode:** Most ordinary applications run with restricted privileges. A crash in one application usually does not crash the entire operating system.
- **Kernel mode:** The Windows kernel and certain trusted components execute with high privileges and can access protected system resources.

### 2.3 File system and NTFS

NTFS (New Technology File System) is the standard Windows file system on most modern Windows installations. It supports file and folder permissions, ownership, auditing, large volumes, and other advanced features.

### 2.4 Windows Registry

The Registry is a hierarchical database that stores configuration information used by Windows, drivers, users, and applications.

Open Registry Editor:

```powershell
regedit
```

**Safety:** Do not modify Registry values without understanding their purpose. Back up relevant keys and use a documented change procedure before making changes.

## 3. Important Windows directories

| Path | Purpose |
|---|---|
| `C:\\Windows` | Main Windows operating-system directory |
| `C:\\Windows\\System32` | Important Windows utilities, libraries, and system components |
| `C:\\Program Files` | Common location for 64-bit applications |
| `C:\\Program Files (x86)` | Common location for 32-bit applications on 64-bit Windows |
| `C:\\Users` | User profile directories |
| `C:\\Users\\<user>\\Desktop` | User desktop files |
| `C:\\Users\\<user>\\Downloads` | Downloaded files |
| `C:\\ProgramData` | Shared application data |
| `C:\\Windows\\Temp` | Windows temporary files |

Useful navigation commands:

```powershell
pwd
Get-Location
Get-ChildItem
dir
cd C:\Windows\System32
```

- `pwd` / `Get-Location`: show the current working directory.
- `Get-ChildItem` / `dir`: list directory contents.
- `cd`: change the current directory.

## 4. Inspecting the computer

The following commands were run in the first screenshot.

```powershell
pwd
hostname
whoami
Get-ComputerInfo
```

| Command | Purpose |
|---|---|
| `pwd` | Displays the current working directory |
| `hostname` | Displays the computer's hostname |
| `whoami` | Displays the current security identity |
| `Get-ComputerInfo` | Displays detailed operating-system and computer information |

### Lab observation

The screenshot shows:

- The PowerShell working directory was `C:\Users\alenz`.
- The current identity was displayed in computer-name\\user-name format.
- `Get-ComputerInfo` returned Windows product, build, installation, BIOS, and other system properties.

**Version interpretation note:** The screenshot's `WindowsProductName` field reports **Windows 10 Home Single Language**. It also displays build and display-version fields. When documenting a machine, report the exact values shown by the system rather than inferring the edition from a single build number. You can verify the edition and version with:

```powershell
winver
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsDisplayVersion, WindowsBuildLabEx
```

## 5. Local users and groups

A **local user** is an account stored on that Windows computer. A **local group** is a collection of accounts to which permissions can be assigned.

### 5.1 Account types

- **Standard user:** Intended for everyday work with limited ability to make system-wide changes.
- **Administrator:** Can perform privileged system-management tasks after elevation.
- **Built-in and service accounts:** Used for specific operating-system or service functions. They should not be treated as ordinary employee accounts.

### 5.2 Why groups matter

Assigning permissions to groups is easier to manage than assigning the same permissions separately to every user. For example, an IT Support group may be granted access to a support folder.

**Least privilege:** Give each account only the permissions needed to perform its role. Do not make a user an administrator simply because it is convenient.

### 5.3 Local account commands

Run PowerShell as Administrator for account-management operations.

List local users:

```powershell
Get-LocalUser
```

List local groups:

```powershell
Get-LocalGroup
```

Create a local user with a securely prompted password:

```powershell
New-LocalUser -Name "Supportuser" -Password (Read-Host -AsSecureString "Enter Password")
```

Inspect the account:

```powershell
Get-LocalUser -Name "Supportuser"
```

Add the test account to the local Administrators group (lab demonstration only):

```powershell
Add-LocalGroupMember -Group "Administrators" -Member "Supportuser"
```

Verify group membership:

```powershell
Get-LocalGroupMember -Group "Administrators"
```

Remove the test account from the Administrators group:

```powershell
Remove-LocalGroupMember -Group "Administrators" -Member "Supportuser"
```

Delete the test account:

```powershell
Remove-LocalUser -Name "Supportuser"
```

**Important:** The lab account was temporarily added to the Administrators group to demonstrate group membership. It was then removed from that group and deleted. Do not leave test accounts or unnecessary privileged memberships on a real workstation.

### 5.4 Understanding the Access Denied error

The first attempt to run `New-LocalUser` returned **Access denied**. The command was subsequently run in an elevated PowerShell window titled **Administrator: Windows PowerShell**, and the account was created successfully.

An account may belong to the Administrators group while a normal PowerShell window still runs with a filtered, non-elevated token because of User Account Control (UAC).

To open an elevated shell:

1. Search for PowerShell in the Start menu.
2. Right-click it and choose **Run as administrator**.
3. Approve the User Account Control prompt.
4. Re-run the administrative command.

A practical elevation check:

```powershell
net session
```

If the command returns **Access is denied**, the shell is not elevated or the account lacks the required rights. In an elevated shell, it should run and may report that there are no entries.

**Interview point:** Being a member of Administrators and running an elevated process are related but different. UAC requires explicit elevation for many privileged operations.

## 6. Processes and Task Manager

A **process** is a running instance of a program. Windows assigns a Process ID (PID) to identify it.

Open Task Manager:

```text
Ctrl + Shift + Esc
```

Useful Task Manager areas:

| Area | What to inspect |
|---|---|
| Processes | Applications and background processes; CPU, memory, disk, and network use |
| Performance | Overall CPU, memory, disk, and network activity |
| Users | Signed-in user sessions |
| Details | Process names, PIDs, and additional process details |
| Services | Service status and associated process information |
| Startup apps | Applications configured to launch during sign-in |

PowerShell process commands:

```powershell
Get-Process
Get-Process chrome
Get-Process -Id 1234
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
Get-Process | Sort-Object WorkingSet64 -Descending | Select-Object -First 5
```

Stop a known test application only when it is safe to do so:

```powershell
Stop-Process -Name notepad
```

- `CPU` in the process object is accumulated processor time, not a live CPU percentage.
- `WorkingSet64` represents the process working-set memory in bytes.
- Do not terminate unfamiliar system processes or force-close applications with unsaved work.

## 7. Windows services

A **service** is a background component that provides a system or application function. Examples include Print Spooler, DHCP Client, DNS Client, Windows Update, and Windows Defender Firewall.

Open the Services console:

```text
Win + R
services.msc
```

Common service states and startup types:

- **Running:** The service is currently active.
- **Stopped:** The service is not currently running.
- **Automatic:** Windows is configured to start it automatically.
- **Automatic (Delayed Start):** Windows starts it after the normal automatic-start sequence.
- **Manual:** It can be started when needed, often by a user or another component.
- **Disabled:** It cannot be started until its startup configuration is changed.

PowerShell service commands:

```powershell
Get-Service
Get-Service -Name Spooler
Get-Service -Name wuauserv
Get-Service -Name Dhcp
Get-Service -Name Dnscache
Start-Service -Name Spooler
Stop-Service -Name Spooler
Restart-Service -Name Spooler
Get-Service | Where-Object { $_.Status -eq "Stopped" }
```

Inspect services configured for automatic startup:

```powershell
Get-CimInstance Win32_Service |
    Where-Object { $_.StartMode -eq "Auto" } |
    Select-Object Name, State, StartMode
```

### Practical scenario: printing failure

If a user cannot print:

1. Confirm the correct printer is selected and reachable.
2. Check whether the printer has paper, power, and no visible error.
3. Open `services.msc` and inspect **Print Spooler**.
4. If the service is stopped, investigate the reason and restart it if appropriate.
5. Send a test print.
6. Record the result and any remaining issue.

Example commands:

```powershell
Get-Service -Name Spooler
Start-Service -Name Spooler
Get-Service -Name Spooler
```

Do not restart services blindly. Some services have dependencies or business impact.

## 8. Disk and system-file troubleshooting

Open Disk Management:

```text
Win + R
diskmgmt.msc
```

Disk Management can display disks and volumes, assign drive letters, create or format volumes, and extend or shrink partitions where supported.

**Warning:** Formatting, deleting, shrinking, or extending partitions can cause data loss. Confirm the target disk and backup requirements before making changes.

Inspect storage with PowerShell:

```powershell
Get-Volume
Get-Disk
Get-Partition
Get-PhysicalDisk
```

### Repair and integrity tools

Check a volume:

```powershell
chkdsk C:
```

Online scan of the C: file system:

```powershell
chkdsk C: /scan
```

Check and repair protected Windows system files:

```powershell
sfc /scannow
```

Repair the Windows component store using DISM:

```powershell
DISM /Online /Cleanup-Image /RestoreHealth
```

| Tool | Primary purpose |
|---|---|
| CHKDSK | Checks a volume's file system and reports or repairs file-system issues |
| SFC | Checks protected Windows system files and attempts repairs |
| DISM | Repairs the Windows component store used by servicing and system-file repair |

These tools solve different classes of problems. Run them with appropriate privileges and allow them to finish.

## 9. Windows network troubleshooting

An IP configuration commonly includes:

| Setting | Purpose |
|---|---|
| IP address | Identifies the interface on an IP network |
| Subnet mask / prefix | Defines the local network range |
| Default gateway | Next-hop router used to reach other networks |
| DNS server | Resolves hostnames to IP addresses |
| DHCP | Automatically leases network configuration to clients |

### 9.1 Essential commands

Show IP configuration:

```powershell
ipconfig
ipconfig /all
```

Test IP connectivity:

```powershell
ping 8.8.8.8
ping <default-gateway-address>
```

Query DNS:

```powershell
nslookup google.com
```

Trace the route to a destination:

```powershell
tracert google.com
```

Display connections and listening ports with process IDs:

```powershell
netstat -ano
```

Test TCP connectivity to HTTPS port 443:

```powershell
Test-NetConnection google.com -Port 443
```

Renew DHCP configuration (may temporarily interrupt connectivity):

```powershell
ipconfig /release
ipconfig /renew
```

Clear the local DNS resolver cache:

```powershell
ipconfig /flushdns
```

Additional PowerShell network inspection:

```powershell
Get-NetAdapter
Get-NetIPAddress
Get-NetTCPConnection
```

### 9.2 A systematic troubleshooting sequence

When a user reports that the internet is not working:

1. Check the cable or Wi-Fi connection and confirm the correct network is selected.
2. Run `ipconfig /all`; inspect the address, subnet, gateway, DNS, and DHCP details.
3. Test the default gateway with `ping <gateway>`.
4. Test a known external IP address, such as `ping 8.8.8.8`.
5. Test name resolution with `nslookup google.com`.
6. If required, test a specific TCP service with `Test-NetConnection`.
7. Identify whether the issue is local connectivity, routing, DNS, or an application/service.
8. Apply a targeted fix and verify the result.

A failed ping alone does not prove that a host is down; ICMP may be blocked. Use multiple tests and interpret them together.

## 10. PowerShell administration essentials

PowerShell cmdlets commonly use a **Verb-Noun** naming convention.

Examples:

- `Get-Service`
- `New-LocalUser`
- `Remove-LocalUser`
- `Restart-Service`
- `Get-Process`

Useful discovery commands:

```powershell
Get-Help Get-Service
Get-Command
Get-Command *Service*
Clear-Host
```

### 10.1 Pipeline and filtering

The pipe character `|` sends objects from one command to another.

Show running services:

```powershell
Get-Service | Where-Object { $_.Status -eq "Running" }
```

Show stopped services:

```powershell
Get-Service | Where-Object { $_.Status -eq "Stopped" }
```

Sort processes by accumulated CPU time:

```powershell
Get-Process |
    Sort-Object CPU -Descending |
    Select-Object -First 10
```

Sort processes by working-set memory:

```powershell
Get-Process |
    Sort-Object WorkingSet64 -Descending |
    Select-Object -First 5
```

Understand the syntax:

- `|`: pipeline operator.
- `Where-Object`: filters objects based on a condition.
- `$_`: current object in the pipeline.
- `.Status`: property of the current object.
- `-eq`: equality comparison.
- `Sort-Object`: sorts objects by a property.
- `-Descending`: highest values first.
- `Select-Object -First 5`: returns the first five objects.

## 11. Event Viewer and Windows logs

Open Event Viewer:

```text
Win + R
eventvwr.msc
```

Important Windows log categories:

| Log | Typical contents |
|---|---|
| Application | Events recorded by applications |
| Security | Logon, account, and audit events, depending on audit policy |
| System | Windows components, drivers, services, and system events |
| Setup | Operating-system setup and servicing events |
| Forwarded Events | Events forwarded from other computers |

Common event IDs:

| Event ID | Meaning / use |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4720 | User account created |
| 4726 | User account deleted |
| 7045 | A service was installed |
| 1074 | Shutdown or restart initiated by a process or user |
| 6005 | Event Log service started |
| 6006 | Event Log service stopped |
| 6008 | Previous shutdown was unexpected |

**Audit note:** Security events depend on audit policy and log availability. Event IDs should be interpreted with their event details, account, timestamp, logon type, and surrounding activity. An event ID alone is not proof of malicious activity.

Read recent System events:

```powershell
Get-WinEvent -LogName System -MaxEvents 20
```

Read recent failed logon events (may require an elevated shell and suitable audit logging):

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 10
```

## 12. Troubleshooting playbooks

### Scenario A — Computer is slow

1. Open Task Manager.
2. Check CPU, memory, disk, and startup activity.
3. Identify the process consuming resources.
4. Check available disk space using `Get-Volume`.
5. Review relevant Application and System events.
6. Investigate the cause before closing applications or changing startup settings.
7. Verify performance after the corrective action.

### Scenario B — User cannot print

1. Confirm printer power, connection, queue, and selected printer.
2. Inspect Print Spooler with `Get-Service Spooler`.
3. Review relevant system or printer events.
4. Restart the service only if appropriate.
5. Print a test page and document the result.

### Scenario C — Internet access failure

1. Check Wi-Fi/Ethernet status.
2. Inspect `ipconfig /all`.
3. Test the gateway.
4. Test an external IP.
5. Test DNS with `nslookup`.
6. Test the required TCP port if the problem is application-specific.
7. Record the failing layer and resolution.

### Scenario D — Windows system-file problem

1. Record the symptoms and recent changes.
2. Review relevant Event Viewer entries.
3. Run `DISM /Online /Cleanup-Image /RestoreHealth` if component-store repair is indicated.
4. Run `sfc /scannow`.
5. Reboot if required and permitted.
6. Verify that the issue is resolved.

## 13. Practical work

### 13.1 Checking the computer

Commands used:

```powershell
pwd
hostname
whoami
Get-ComputerInfo
```

**Screenshot — PowerShell system information**

![PowerShell system information commands and output](evidence/01-system-information.jpg)

*The registered-owner email is hidden in this copy.*

### 13.2 Creating and managing a local user

Commands used:

```powershell
New-LocalUser -Name "Supportuser" -Password (Read-Host -AsSecureString "Enter Password")
Get-LocalUser -Name "Supportuser"
Add-LocalGroupMember -Group "Administrators" -Member "Supportuser"
Get-LocalGroupMember -Group "Administrators"
Remove-LocalGroupMember -Group "Administrators" -Member "Supportuser"
Remove-LocalUser -Name "Supportuser"
```

**Screenshot — local user and Administrators group**

![PowerShell local user creation, group membership and cleanup commands](evidence/02-local-user-administration.jpg)

### Lab outcome

- The initial non-elevated attempt returned **Access denied**.
- The command succeeded after opening PowerShell as Administrator.
- The local test account was created and verified.
- The test account was added to the local Administrators group and its membership was displayed.
- The test account was removed from the group and the account deletion command was executed.

The screenshot documents command execution. It does not independently show a final post-deletion query; if you want to verify deletion in a repeat lab, run:

```powershell
Get-LocalUser -Name "Supportuser"
```

A not-found result confirms that the local account no longer exists.

## 14. Commands to keep handy

| Task | Command |
|---|---|
| Current directory | `pwd` |
| Computer name | `hostname` |
| Current identity | `whoami` |
| System information | `Get-ComputerInfo` |
| List files | `Get-ChildItem` / `dir` |
| List local users | `Get-LocalUser` |
| List local groups | `Get-LocalGroup` |
| Create local user | `New-LocalUser` |
| Add group member | `Add-LocalGroupMember` |
| List group members | `Get-LocalGroupMember` |
| Remove group member | `Remove-LocalGroupMember` |
| Delete local user | `Remove-LocalUser` |
| List processes | `Get-Process` |
| Stop process | `Stop-Process` |
| List services | `Get-Service` |
| Start / stop / restart service | `Start-Service` / `Stop-Service` / `Restart-Service` |
| List volumes | `Get-Volume` |
| List disks / partitions | `Get-Disk` / `Get-Partition` |
| Check file system | `chkdsk` |
| Check protected system files | `sfc /scannow` |
| Repair component store | `DISM /Online /Cleanup-Image /RestoreHealth` |
| IP configuration | `ipconfig /all` |
| Connectivity test | `ping` |
| DNS lookup | `nslookup` |
| Route tracing | `tracert` |
| Connections and PIDs | `netstat -ano` |
| TCP port test | `Test-NetConnection` |
| DHCP renew | `ipconfig /release`, `ipconfig /renew` |
| Clear DNS cache | `ipconfig /flushdns` |
| Network adapters / addresses | `Get-NetAdapter` / `Get-NetIPAddress` |
| Read System events | `Get-WinEvent -LogName System -MaxEvents 20` |
| Read Security failed logons | `Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625}` |

## 15. Questions to revise

1. **What is an operating system?**  
   System software that manages hardware and software resources and provides common services for applications and users.

2. **What is the kernel?**  
   The core operating-system component responsible for low-level resource management and hardware coordination.

3. **What is the difference between user mode and kernel mode?**  
   User mode restricts application access; kernel mode has privileged access to system resources.

4. **What is the difference between a process and a service?**  
   A process is a running program instance. A service is a background component that provides a defined function; it runs within one or more processes.

5. **What is a PID?**  
   A Process ID used to identify a running process.

6. **What is Active Directory?**  
   Microsoft's directory service for centrally managing identities, computers, groups, and access in a Windows domain. It is a Windows Server topic covered in a later module.

7. **What is DNS?**  
   A naming system that resolves hostnames to IP addresses and supports other DNS record types.

8. **What is DHCP?**  
   A protocol that automatically leases IP configuration to network clients.

9. **What is the difference between RAM and storage?**  
   RAM is volatile working memory; storage retains data when power is off.

10. **Why can an administrator account receive Access Denied?**  
    The current process may not be elevated. UAC commonly starts applications with a filtered token until elevation is approved.

11. **What is Event Viewer?**  
    A Windows utility for reviewing application, system, security, and other event logs.

12. **How would you investigate a slow computer?**  
    Inspect CPU, memory, disk, and startup activity; identify the responsible process; check storage and relevant logs; make a targeted change and verify the result.

## 16. Revision checklist

- [ ] Explain kernel, user mode, and kernel mode.
- [ ] Navigate common Windows directories.
- [ ] Identify hostname, current user, and system information.
- [ ] List local users and groups.
- [ ] Create and remove a local test account.
- [ ] Add and remove a group membership.
- [ ] Explain UAC and elevated PowerShell.
- [ ] Inspect processes and services.
- [ ] Inspect disks and volumes.
- [ ] Explain CHKDSK, SFC, and DISM.
- [ ] Diagnose a basic DNS or connectivity issue.
- [ ] Find relevant events in Event Viewer.
- [ ] Explain the practical lab evidence and its limitations.

---

## Microsoft documentation

Use Microsoft's official documentation when validating command syntax and behavior:

- [PowerShell documentation](https://learn.microsoft.com/powershell/)
- [LocalAccounts PowerShell module](https://learn.microsoft.com/powershell/module/microsoft.powershell.localaccounts/)
- [Windows commands](https://learn.microsoft.com/windows-server/administration/windows-commands/windows-commands)
- [Windows Event Viewer](https://learn.microsoft.com/windows/security/operating-system-security/system-security/auditing/)
- [DISM and SFC repair guidance](https://support.microsoft.com/windows/using-system-file-checker-in-windows-365e0031-36b1-6031-f804-8fd86e0ef4ca)


