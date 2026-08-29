# Investigation Notes - Lab 65

## Investigation Overview

This investigation examines the potential use of `InstallUtil.exe` on a Windows endpoint.

The focus is on identifying the trusted binary, validating its legitimacy, preparing a controlled assembly, establishing a baseline, and determining what Sysmon and Wazuh evidence would be required to confirm execution.

---

## Endpoint

| Field | Value |
|---|---|
| Hostname | `DESKTOP-9MMM37V` |
| Operating System | Windows 11 |
| Sysmon | Running |
| Wazuh | Running |
| .NET Framework | `v4.0.30319` |
| Lab Root | `C:\InstallUtilAbuseLab` |

---

## 1. InstallUtil Discovery

The endpoint was searched for `InstallUtil.exe`.

Discovered paths:

```text
C:\Windows\Microsoft.NET\Framework\v4.0.30319\InstallUtil.exe
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe
```

The 64-bit binary was selected:

```text
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe
```

Observed size:

```text
34768 bytes
```

Observed last write time:

```text
07-05-2022 10:50:28
```

---

## 2. InstallUtil Hash

SHA256 was collected for the selected binary.

```text
4F02DE543316367945FDFB89DAFEB3A50E6C1E54DF015AD1732C15962206B647
```

This value represents the specific InstallUtil binary present on the laboratory endpoint.

---

## 3. Signature Validation

The selected InstallUtil binary was checked using Authenticode validation.

Observed result:

```text
Status:
Valid

StatusMessage:
Signature verified.
```

This supports the conclusion that the selected binary had a valid digital signature at the time of testing.

---

## 4. Monitoring Validation

Sysmon status:

```text
Running
Sysmon64
```

Wazuh status:

```text
Running
WazuhSvc
```

Both monitoring services were active before continuing with the investigation.

---

## 5. Laboratory Workspace

The following directories were created:

```text
C:\InstallUtilAbuseLab\
├── Evidence\
├── Output\
└── Payload\
```

Purpose:

| Directory | Purpose |
|---|---|
| `Payload` | Assembly source and compiled payload |
| `Output` | Expected execution artifact |
| `Evidence` | Baseline and investigation evidence |

---

## 6. Assembly Preparation

The benign source file was created at:

```text
C:\InstallUtilAbuseLab\Payload\InstallUtilBenignLab.cs
```

The assembly was intended to perform a controlled local action and create:

```text
C:\InstallUtilAbuseLab\Output\installutil-execution-result.txt
```

The expected behavior is limited to producing a predictable laboratory artifact.

---

## 7. Compiler Verification

The .NET Framework C# compiler was checked:

```powershell
Test-Path "C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe"
```

Result:

```text
True
```

The compiler reported:

```text
Microsoft (R) Visual C# Compiler version 4.8.9032.0
for C# 5
```

---

## 8. Compilation Failure

The initial compilation attempt attempted to create:

```text
C:\InstallUtilAbuseLab\Payload\InstallUtilBenignLab.exe
```

The compiler returned:

```text
error CS5001: Program 'c:\InstallUtilAbuseLab\Payload\InstallUtilBenignLab.exe' does not contain a static 'Main' method suitable for an entry point
```

This prevented the expected executable from being created.

---

## 9. Compilation Interpretation

The source was compiled as an executable.

The compiler therefore expected a conventional entry point.

Because the source did not contain a suitable:

```csharp
static void Main()
```

method, compilation failed.

This means the problem occurred during assembly preparation and not during confirmed InstallUtil execution.

---

## 10. Payload Directory Verification

After the compiler error, the payload directory was checked.

Observed:

```text
InstallUtilBenignLab.cs
```

Not present:

```text
InstallUtilBenignLab.exe
```

Therefore:

```text
Compiled assembly = Not created
```

---

## 11. `corflags.exe` Check

An attempt was made to access:

```text
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\corflags.exe
```

PowerShell reported that the executable was not recognized at that location.

Because the assembly compilation had already failed, there was no successfully compiled assembly available for inspection.

This is therefore recorded as a tooling availability issue rather than evidence of a payload failure.

---

## 12. Filesystem Baseline

The baseline was created using:

```powershell
Get-ChildItem "C:\InstallUtilAbuseLab" -Recurse -Force |
Select-Object FullName, Length, CreationTime, LastWriteTime |
Out-File "C:\InstallUtilAbuseLab\Evidence\baseline.txt"
```

The baseline recorded the laboratory directory structure and available files.

The baseline itself appeared in the enumeration because it was created inside the directory being measured.

---

## 13. Pre-Execution Output Check

The recorded time was:

```text
29 August 2026 07:41:43
```

The output directory was checked:

```powershell
Get-ChildItem "C:\InstallUtilAbuseLab\Output" -Force
```

No files were returned.

Therefore:

```text
Expected output artifact = Not present
```

during the documented pre-execution check.

---

## 14. Sysmon Event Volume

The supplied Event Viewer screenshots showed a very large number of Sysmon events.

Observed approximately:

```text
46,000+ total Sysmon events
```

The Event ID 1 filter showed approximately:

```text
23,000+ Process Create events
```

Because of the event volume, the investigation should prioritize relevant process and file attributes.

---

## 15. Sysmon Event ID 1

Event ID 1 represents Process Create.

Relevant fields include:

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

The primary question is whether an event contains:

```text
Image:
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe
```

and whether its command line references the known laboratory assembly.

---

## 16. Sysmon Event ID 11

Event ID 11 represents File Create.

Relevant fields include:

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

A matching Event ID 11 would become stronger evidence if it can be correlated with an InstallUtil process event.

---

## 17. PowerShell Event

A supplied Event ID 11 showed:

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

### Interpretation

This confirms PowerShell file creation.

It does not prove that the activity was caused by InstallUtil.

---

## 18. WMI Process Activity

A supplied Event ID 1 included:

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

### Interpretation

The event demonstrates WMI-related process activity.

The available information does not establish a direct InstallUtil relationship.

---

## 19. Process Lineage Requirements

For a reliable InstallUtil attribution, investigate:

```text
ProcessGuid
ParentProcessGuid
ProcessId
ParentProcessId
ParentImage
CommandLine
User
UtcTime
```

The preferred evidence relationship is:

```text
Parent Process
      |
      v
InstallUtil.exe
      |
      v
Known laboratory assembly
```

A process should not be attributed to InstallUtil merely because it occurred during the same time window.

---

## 20. File Correlation Requirements

The expected artifact is:

```text
C:\InstallUtilAbuseLab\Output\installutil-execution-result.txt
```

The investigation should compare:

```text
InstallUtil Process Create timestamp
```

against:

```text
Expected output File Create timestamp
```

and then examine:

```text
ProcessGuid
ProcessId
Image
TargetFilename
```

for additional correlation.

---

## 21. Wazuh Correlation

Wazuh was active during the investigation.

Wazuh should be used to identify and correlate:

- Process creation
- File creation
- Sysmon events
- Process paths
- Command lines
- User context
- Event timestamps

The available evidence does not currently demonstrate a confirmed Wazuh record showing successful InstallUtil processing of the benign assembly.

---

## 22. Evidence Assessment

### Confirmed

```text
InstallUtil.exe exists
```

```text
Selected InstallUtil binary is signed
```

```text
Sysmon is running
```

```text
Wazuh is running
```

```text
Laboratory workspace exists
```

```text
Filesystem baseline exists
```

```text
Benign assembly source exists
```

```text
Compilation returned CS5001
```

```text
Expected compiled executable was not created
```

```text
Output directory was empty during the pre-execution check
```

### Observed but Not Attributed to InstallUtil

```text
PowerShell activity
```

```text
WMI-related process activity
```

### Not Confirmed

```text
Successful InstallUtil execution
```

```text
InstallUtil processing the benign assembly
```

```text
Expected output artifact generated by InstallUtil
```

---

## 23. Investigation Conclusion

The laboratory environment was successfully prepared and the selected InstallUtil binary was validated.

The main preparation issue was the `CS5001` compilation failure, which prevented the intended assembly from being created.

Sysmon also showed PowerShell and WMI-related activity, but the supplied evidence does not establish that these events were caused by InstallUtil.

The investigation should therefore record successful InstallUtil execution as **not confirmed** until direct process creation and artifact correlation are obtained.
