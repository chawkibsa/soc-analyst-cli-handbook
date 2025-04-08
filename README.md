# soc-analyst-cli-handbook
> A comprehensive and actionable guide for Security Operations Center (SOC) analysts to perform endpoint investigations, triage, and data collection using native command-lines and scripting approaches.
---

## ✅ Usage Guidelines

This repository is intended for SOC analysts and incident responders. It includes practical command-line snippets, categorized per investigation scenario. Use it as a live reference during incident triage or while building automation playbooks.

---
## ☕ Warming up

An overview of Windows endpoint internals to understand and detect malicious behavior through scripting, and built-in security mechanisms.

<details>
<summary>Click to expand</summary>

### 📌 Use Case: *Command prompt*

- **Goal:** The Windows command prompt25is the most commonly-used command-line interface for the operating system.
- **Platform:** Windows
- **Skill Level:** Beginner

#### 🔧 Command(s)

```bash
@ECHO OFF 
TITLE Example Batch File 
ECHO This batchfile will show Windows 10 Operating System information 
systeminfo | findstr /C:"Host Name" systeminfo | findstr /C:"OS Name" 
systeminfo | findstr /C:"OS Version" systeminfo | findstr /C:"System Type" 
systeminfo | findstr /C:"Registered Owner" 
PAUSE
```
### 📌 Use Case: *Powershell*

- **Goal:** The PowerShell prompt is backwards-compatible with the syntax of cmd.exe (TIP: Get-ExecutionPolicy)
- **Platform:** Windows
- **Skill Level:** Beginner

#### 🔧 Command(s)

```bash
Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object -Property CSName, Caption, Version,BuildNumber, BuildType, OSType, RegisteredUser, OSArchitecture, ServicePackMajorVersion, ServicePackMinorVersion
```
```bash
Get-Service | Where-Object { $_.Status -eq "Running" }
```
```bash
Get-Help Get-CimInstance
```
```bash
Get-Alias gcim
```

#### Create ps1 file
```bash
function Get-AVInfo { gcim -Namespace root/SecurityCenter2 -ClassName AntivirusProduct }
```
#### Run it 
```bash
Import-Module C:\tools\windows_endpoint_introduction\get_avinfo.ps1
```
```bash
Get-AVInfo
```
#### Event XML view sample
```bash
- <Event xmlns="http://schemas.microsoft.com/win/2004/08/events/event">
 - <System>
    <Provider Name="Microsoft-Windows-Security-Auditing" Guid="{54849361-4382-4991-a1ba-3e3b0125c30d}" /> 
    <EventID>4624</EventID> 
    <Version>3</Version> 
    <Level>0</Level> 
    <Task>12544</Task> 
    <Opcode>0</Opcode> 
    <Keywords>0x8020000000000000</Keywords> 
    <TimeCreated SystemTime="2025-04-08T13:10:00.7179797Z" /> 
    <EventRecordID>24040</EventRecordID> 
    <Correlation ActivityID="{4e6aad10-a887-0001-03ae-6a4e87a8db01}" /> 
    <Execution ProcessID="780" ThreadID="840" /> 
    <Channel>Security</Channel> 
    <Computer>DESKTOP</Computer> 
    <Security /> 
    </System>
- <EventData>
    <Data Name="SubjectUserSid">S-1-5-18</Data> 
    <Data Name="SubjectUserName">DESKTOP$</Data> 
    <Data Name="SubjectDomainName">WORKGROUP</Data> 
    <Data Name="SubjectLogonId">0x3e7</Data> 
    <Data Name="TargetUserSid">S-1-5-18</Data> 
    <Data Name="TargetUserName">SYSTEM</Data> 
    <Data Name="TargetDomainName">NT AUTHORITY</Data> 
    <Data Name="TargetLogonId">0x3e7</Data> 
    <Data Name="LogonType">5</Data> 
    <Data Name="LogonProcessName">Advapi</Data> 
    <Data Name="AuthenticationPackageName">Negotiate</Data> 
    <Data Name="WorkstationName">-</Data> 
    <Data Name="LogonGuid">{00000000-0000-0000-0000-000000000000}</Data> 
    <Data Name="TransmittedServices">-</Data> 
    <Data Name="LmPackageName">-</Data> 
    <Data Name="KeyLength">0</Data> 
    <Data Name="ProcessId">0x2f8</Data> 
    <Data Name="ProcessName">C:\Windows\System32\services.exe</Data> 
    <Data Name="IpAddress">-</Data> 
    <Data Name="IpPort">-</Data> 
    <Data Name="ImpersonationLevel">%%1833</Data> 
    <Data Name="RestrictedAdminMode">-</Data> 
    <Data Name="RemoteCredentialGuard">-</Data> 
    <Data Name="TargetOutboundUserName">-</Data> 
    <Data Name="TargetOutboundDomainName">-</Data> 
    <Data Name="VirtualAccount">%%1843</Data> 
    <Data Name="TargetLinkedLogonId">0x0</Data> 
    <Data Name="ElevatedToken">%%1842</Data> 
 </EventData>
</Event>
```
#### Use Get-WinEventwith the -ListLogargument to get the details about the four Windows logs.
``` bash
Get-WinEvent -ListLog Application, Security, Setup, System
```
#### Use Get-WinEvent -LogName Security | Select-Object -First 10 in PowerShell to retrieve the latest 10 Security log entries, the Message field may appear truncated in the output.
``` bash
Get-WinEvent -LogName Security | Select-Object -first 10
```
#### Filter Security log events with ID 4624 using Where-Object, and display only TimeCreated and Message fields using Select-Object for concise output.
``` bash
Get-WinEvent -LogName 'Security' | Where-Object { $_.Id -eq "4624" } | Select-Object -Property TimeCreated,Message -first 10
```
#### Use Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624; StartTime='04/23/2021 14:00:00'; EndTime='04/23/2021 14:30:00'} to efficiently retrieve Logon events within a specific time range using multiple filter conditions.
##### Note: FilterHashtable requires a LogName, ProviderName,  or Path
``` bash
Get-WinEvent -FilterHashtable @{LogName='Security'; StartTime="4/23/2021 14:00:00"; EndTime="4/23/2021 14:30:00"; ID=4624} | Select-Object -Property TimeCreated,Message
```
#### indices - EvendData values map sample
```bash
Index 0 is "SubjectUserSid" 
Index 1 is "SubjectUserName" 
Index 2 is "SubjectDomainName" 
Index 3 is "SubjectLogonId" 
Index 4 is "TargetUserSid" 
Index 5 is "TargetUserName" 
Index 6 is "TargetDomainName" 
Index 7 is "TargetLogonId" 
Index 8 is "LogonType" ... 
```
#### To detect potential unauthorized remote access during off-hours, use Get-WinEvent with -FilterHashtable to retrieve Security logon events (ID 4624) between 04/23/2021 19:00:00 and 04/26/2021 07:00:00. Then, apply Where-Object to filter for LogonType 10 (Remote Desktop logons) by accessing the 8th EventData property. Format the output with Format-List to view full event details including source IP:
``` bash
Get-WinEvent -FilterHashTable @{LogName='Security'; StartTime="4/23/2021 00:00:00"; EndTime="4/26/2021 07:00:00"; ID=4624 } | Where-Object { $_.properties[8].value -eq 10 } | Format-List
```








