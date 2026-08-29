# Windows DFIR Lab 65 - InstallUtil Abuse Investigation

## Overview

This lab investigates potential abuse of `InstallUtil.exe`, a legitimate Microsoft .NET Framework utility that can process .NET assemblies.

A controlled and benign .NET assembly is used to create predictable laboratory activity. The investigation focuses on validating the trusted binary, preparing the test assembly, establishing a filesystem baseline, and examining endpoint telemetry through Sysmon and Wazuh.

The lab also documents a compilation failure encountered while preparing the assembly and demonstrates why troubleshooting and artifact validation should occur before execution.

---

## Lab Objectives

- Understand the forensic significance of `InstallUtil.exe`.
- Identify available `InstallUtil.exe` binaries on the endpoint.
- Validate the selected binary using metadata, SHA256 hashing, and Authenticode signature verification.
- Prepare a controlled .NET assembly for the laboratory.
- Troubleshoot the `CS5001` compilation error.
- Establish a pre-execution filesystem baseline.
- Analyze Sysmon Event ID 1 for process creation.
- Analyze Sysmon Event ID 11 for file creation.
- Investigate parent-child process relationships.
- Correlate timestamps, processes, and file artifacts.
- Use Wazuh as an additional telemetry source.
- Distinguish InstallUtil-related evidence from unrelated PowerShell and WMI activity.
- Practice evidence-based attribution during DFIR investigations.

---

## Investigation Scenario

A Windows endpoint is being examined for possible `InstallUtil.exe` abuse involving a legitimate Microsoft .NET Framework utility. The analyst needs to determine whether the utility was actually executed and whether any observed endpoint activity can be reliably associated with it.

The investigation is performed in a controlled environment using a benign .NET assembly:

- `InstallUtil.exe` is identified and validated.
- A dedicated laboratory workspace is created.
- A benign assembly is prepared for controlled testing.
- Sysmon and Wazuh provide endpoint visibility.
- A filesystem baseline is established before execution.
- The assembly encounters a `CS5001` compilation error.
- PowerShell and WMI-related Sysmon activity is also observed.
- Process lineage, timestamps, and file artifacts are reviewed before making an attribution.

The investigation therefore focuses on determining what can be **confirmed from telemetry** rather than assuming that every event occurring around the same time belongs to the InstallUtil activity.

---

## Environment

| Component | Observed Value |
|---|---|
| Operating System | Windows 11 |
| Hostname | `DESKTOP-9MMM37V` |
| Sysmon Service | `Sysmon64` |
| Sysmon Status | Running |
| Wazuh Service | `WazuhSvc` |
| Wazuh Status | Running |
| .NET Framework | `v4.0.30319` |
| Lab Root | `C:\InstallUtilAbuseLab` |

---
## Laboratory Directory Structure

The laboratory workspace was created as:

```text
C:\InstallUtilAbuseLab\
├── Evidence\
├── Output\
└── Payload\
```

### Payload

```text
C:\InstallUtilAbuseLab\Payload\
```

Contains the benign assembly source and the intended compiled assembly.

### Output

```text
C:\InstallUtilAbuseLab\Output\
```

Used for the predictable output artifact produced by the benign test assembly.

### Evidence

```text
C:\InstallUtilAbuseLab\Evidence\
```

Used to store investigation evidence such as the filesystem baseline.

---

## InstallUtil Discovery

The endpoint was searched for `InstallUtil.exe`.

Two versions were identified:

```text
C:\Windows\Microsoft.NET\Framework\v4.0.30319\InstallUtil.exe
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe
```

The 64-bit version selected for the investigation was:

```text
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe
```

Observed metadata:

```text
Length:
34768 bytes

LastWriteTime:
07-05-2022 10:50:28
```

---

## InstallUtil Binary Validation

The selected binary was examined using PowerShell.

### SHA256

Observed SHA256:

```text
4F02DE543316367945FDFB89DAFEB3A50E6C1E54DF015AD1732C15962206B647
```

### Authenticode Signature

The signature validation returned:

```text
Status:
Valid

StatusMessage:
Signature verified.
```

The result indicates that the selected binary had a valid Authenticode signature during the investigation.

The recorded hash represents the specific binary present on this laboratory endpoint.

---

## Monitoring Validation

The endpoint monitoring services were checked before continuing with the investigation.

### Sysmon

```text
Status   Name       DisplayName
------   ----       -----------
Running  Sysmon64   Sysmon64
```

### Wazuh

```text
Status   Name       DisplayName
------   ----       -----------
Running  WazuhSvc   Wazuh
```

Both monitoring components were therefore active during the laboratory preparation.

---

## Benign Assembly

The laboratory uses a controlled .NET assembly rather than malicious software.

The intended output artifact is:

```text
C:\InstallUtilAbuseLab\Output\installutil-execution-result.txt
```

The assembly is designed to perform a predictable local file-write operation.

It does not intentionally perform:

- Persistence
- Network communication
- File downloading
- Account creation
- Security-control modification
- Command-and-control activity
- Destructive actions

The purpose is to generate a known artifact that can be correlated with endpoint telemetry.

---

## Assembly Source

The source file was created at:

```text
C:\InstallUtilAbuseLab\Payload\InstallUtilBenignLab.cs
```

The .NET Framework compiler was confirmed to exist using:

```powershell
Test-Path "C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe"
```

Result:

```text
True
```

---

## Compilation Attempt

The assembly was initially compiled as an executable using the .NET Framework C# compiler.

The compiler reported:

```text
Microsoft (R) Visual C# Compiler version 4.8.9032.0
for C# 5
```

The compilation failed with:

```text
error CS5001: Program 'c:\InstallUtilAbuseLab\Payload\InstallUtilBenignLab.exe' does not contain a static 'Main' method suitable for an entry point
```

Because compilation failed, the expected executable was not produced.

The payload directory contained:

```text
InstallUtilBenignLab.cs
```

but did not contain:

```text
InstallUtilBenignLab.exe
```

---

## Compilation Finding

The `CS5001` error occurred because the source was compiled as an executable while not providing a conventional `Main()` entry point.

This is important because an InstallUtil-compatible installer assembly does not necessarily follow the same structure as a normal standalone console application.

The failure therefore occurred during **artifact preparation**, before successful InstallUtil execution could be established.

---

## Filesystem Baseline

A baseline was created using:

```powershell
Get-ChildItem "C:\InstallUtilAbuseLab" -Recurse -Force |
Select-Object FullName, Length, CreationTime, LastWriteTime |
Out-File "C:\InstallUtilAbuseLab\Evidence\baseline.txt"
```

The baseline recorded:

- Full path
- File size
- Creation time
- Last write time

The baseline included:

```text
C:\InstallUtilAbuseLab\Evidence
C:\InstallUtilAbuseLab\Output
C:\InstallUtilAbuseLab\Payload
C:\InstallUtilAbuseLab\Evidence\baseline.txt
```

The presence of `baseline.txt` in the results is expected because the evidence file was created inside the directory being enumerated.

---

## Pre-Execution State

The current time was recorded as:

```text
29 August 2026 07:41:43
```

The output directory was checked:

```powershell
Get-ChildItem "C:\InstallUtilAbuseLab\Output" -Force
```

No files were displayed.

This established that the expected output artifact was not present during the documented pre-execution check.

---

## Sysmon Evidence

The Sysmon Operational log contained a very large volume of events.

The supplied Event Viewer screenshots showed approximately:

```text
46,000+ Sysmon events
```

and more than:

```text
23,000 Event ID 1 events
```

after filtering.

The investigation therefore focuses on specific event fields rather than manually reviewing the entire event volume.

---

## Sysmon Event ID 1

Sysmon Event ID 1 represents process creation.

Important fields for this investigation include:

```text
UtcTime
Image
CommandLine
ProcessId
ProcessGuid
ParentImage
ParentCommandLine
ParentProcessId
ParentProcessGuid
User
IntegrityLevel
```

The most important question is whether a process creation event shows:

```text
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe
```

and references the known laboratory assembly.

---

## Sysmon Event ID 11

Sysmon Event ID 11 represents file creation.

Important fields include:

```text
UtcTime
Image
TargetFilename
ProcessId
ProcessGuid
User
```

The expected laboratory artifact is:

```text
C:\InstallUtilAbuseLab\Output\installutil-execution-result.txt
```

A file creation event for this artifact would provide useful evidence when correlated with a preceding InstallUtil process event.

---

## Observed PowerShell Activity

A supplied Sysmon Event ID 11 recorded:

```text
UtcTime:
2026-08-29 02:15:29.417

ProcessId:
17176

Image:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

TargetFilename:
C:\WINDOWS\Temp\_PSScriptPolicyTest_hphtfxff.tww.ps1

User:
NT AUTHORITY\SYSTEM
```

This confirms that PowerShell created a temporary file.

However, the event does not establish that `InstallUtil.exe` caused the activity.

---

## Observed WMI Activity

A supplied Sysmon Event ID 1 contained:

```text
ParentProcessId:
10236

ParentImage:
C:\WINDOWS\System32\wbem\WmiPrvSE.exe

ParentCommandLine:
C:\WINDOWS\system32\wbem\wmiprvse.exe -Embedding

ParentUser:
NT AUTHORITY\SYSTEM

IntegrityLevel:
System
```

This demonstrates WMI-related process activity.

The available fields do not establish that the process was launched by or related to `InstallUtil.exe`.

---

## Evidence Correlation

The intended evidence chain for a successful controlled execution would look like:

```text
InstallUtil.exe Process Create
        |
        v
Known benign assembly referenced
        |
        v
Installer logic executes
        |
        v
Expected output file created
        |
        v
Sysmon Event ID 11
        |
        v
Wazuh correlation
```

Each relationship must be independently verified.

A matching timestamp alone is not sufficient to establish causality.

---

## Investigation Findings

### Finding 1 - InstallUtil binaries identified

Both 32-bit and 64-bit versions of `InstallUtil.exe` were present.

### Finding 2 - Selected InstallUtil binary validated

The selected 64-bit binary had a valid Authenticode signature.

### Finding 3 - Monitoring was active

Both Sysmon and Wazuh services were running.

### Finding 4 - Laboratory workspace established

The required `Payload`, `Output`, and `Evidence` directories were created.

### Finding 5 - Baseline established

A filesystem baseline was collected before the intended execution phase.

### Finding 6 - Assembly compilation failed

The initial compilation returned `CS5001`.

### Finding 7 - Expected assembly was not created

`InstallUtilBenignLab.exe` was absent following the failed compilation.

### Finding 8 - Output directory was clean

No output artifact was present during the documented pre-execution check.

### Finding 9 - PowerShell and WMI activity was observed

Sysmon recorded PowerShell and WMI-related activity during the investigation window.

### Finding 10 - InstallUtil execution is not confirmed

The supplied evidence does not currently prove that `InstallUtil.exe` successfully processed the benign assembly.

---

## Investigation Status

| Activity | Status |
|---|---|
| InstallUtil discovery | Complete |
| InstallUtil metadata collection | Complete |
| InstallUtil SHA256 | Complete |
| InstallUtil signature validation | Complete |
| Lab directory creation | Complete |
| Sysmon verification | Complete |
| Wazuh verification | Complete |
| Filesystem baseline | Complete |
| Assembly source creation | Complete |
| Assembly compilation | Failed - `CS5001` |
| Compiled assembly validation | Pending |
| InstallUtil execution | Not confirmed |
| InstallUtil Sysmon correlation | Pending |
| Wazuh correlation | Pending |

---

## Lessons Learned

### Trusted Binary Abuse Requires Context

The presence of a legitimate Microsoft binary does not establish malicious activity.

### Payload Validation Comes First

A failed compilation means the expected test artifact does not exist and execution cannot be reliably investigated.

### Process Lineage Matters

Parent and child process relationships provide stronger evidence than timestamps alone.

### File Artifacts Strengthen Correlation

A predictable output file provides an additional artifact that can be correlated with process telemetry.

### Avoid Premature Attribution

PowerShell or WMI activity occurring near the investigation window should not automatically be attributed to InstallUtil.

---

## Safety Scope

This repository documents a controlled defensive security laboratory.

The test assembly is designed only to produce a predictable local artifact. The lab does not intentionally perform persistence, credential theft, network communication, payload downloading, security-control modification, command-and-control behavior, or destructive activity.
