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

#### Create ps1 file: 
```bash
function Get-AVInfo { gcim -Namespace root/SecurityCenter2 -ClassName AntivirusProduct }
```
#### Run it :
```bash
Import-Module C:\tools\windows_endpoint_introduction\get_avinfo.ps1
```
```bash
Get-AVInfo
```






