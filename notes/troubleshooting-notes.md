# Troubleshooting Notes - Lab 65

## Purpose

This document records the issues encountered while preparing the controlled InstallUtil investigation.

The primary problem was the failure to compile the benign assembly because the source was compiled as an executable without a suitable `Main()` entry point.

---

## 1. Incorrect `dotnet` Information Command

### Command Used

```powershell
Get-Item dotnet --info
```

### Error

```text
Get-Item: A positional parameter cannot be found that accepts argument '--info'.
```

### Cause

`Get-Item` is a PowerShell cmdlet for retrieving filesystem objects.

It does not execute the `dotnet` command.

### Correct Command

If the .NET CLI is installed:

```powershell
dotnet --info
```

or:

```powershell
dotnet --list-sdks
```

### Lesson

Use an executable directly when the goal is to execute a command.

Use `Get-Item` when the goal is to inspect an item such as a file or directory.

---

## 2. C# Compiler Availability

The .NET Framework C# compiler was checked using:

```powershell
Test-Path "C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe"
```

Result:

```text
True
```

This confirmed that the compiler was available.

The compiler reported:

```text
Microsoft (R) Visual C# Compiler version 4.8.9032.0
for C# 5
```

Therefore, the compiler itself was not the primary problem.

---

## 3. Compilation Failure - CS5001

### Error

```text
error CS5001: Program 'c:\InstallUtilAbuseLab\Payload\InstallUtilBenignLab.exe' does not contain a static 'Main' method suitable for an entry point
```

### Cause

The source was compiled using:

```text
/target:exe
```

An executable requires a valid entry point.

A typical console application entry point is:

```csharp
static void Main()
{
}
```

The installer-oriented source did not provide a suitable `Main()` method.

As a result, the compiler stopped before producing the executable.

---

## 4. Result of the Compilation Failure

The expected output:

```text
C:\InstallUtilAbuseLab\Payload\InstallUtilBenignLab.exe
```

was not created.

The payload directory contained:

```text
InstallUtilBenignLab.cs
```

but not:

```text
InstallUtilBenignLab.exe
```

This confirms that the compilation failure directly prevented the expected artifact from being created.

---

## 5. Correct Assembly Concept

An InstallUtil-compatible assembly should be treated differently from a conventional standalone console application.

A normal executable generally expects:

```text
C# source
    |
    v
Main()
    |
    v
EXE
```

An installer assembly instead provides installer-related classes and logic that the installation utility can process.

Conceptually:

```text
C# source
    |
    v
Installer class
    |
    v
.NET assembly
    |
    v
InstallUtil.exe
```

Therefore, the assembly should be compiled and validated according to the intended InstallUtil-compatible structure rather than simply forcing it into a normal console application model.

---

## 6. Expected Assembly Output

A library-style assembly may be produced as:

```text
InstallUtilBenignLab.dll
```

instead of:

```text
InstallUtilBenignLab.exe
```

The important requirement is that the resulting assembly contains the expected installer implementation and can be independently validated before execution.

---

## 7. Missing Compiled Assembly

The following command was used:

```powershell
Get-Item "C:\InstallUtilAbuseLab\Payload\InstallUtilBenignLab.exe"
```

PowerShell reported that the path did not exist.

### Explanation

This was a consequence of the earlier compiler error.

The filesystem was not failing to locate an existing file.

The file simply had never been successfully produced.

---

## 8. `corflags.exe` Not Found

### Command

```powershell
& "C:\Windows\Microsoft.NET\Framework64\v4.0.30319\corflags.exe"
```

### Result

PowerShell reported that the executable was not recognized at that location.

### Interpretation

`corflags.exe` was not available at the exact path tested.

This does not by itself indicate that the .NET Framework installation is damaged.

Additionally, the assembly compilation had already failed, so there was no valid compiled assembly available for inspection.

---

## 9. Baseline Contains `baseline.txt`

The baseline was created using:

```powershell
Get-ChildItem "C:\InstallUtilAbuseLab" -Recurse -Force |
Select-Object FullName, Length, CreationTime, LastWriteTime |
Out-File "C:\InstallUtilAbuseLab\Evidence\baseline.txt"
```

The resulting output included:

```text
C:\InstallUtilAbuseLab\Evidence\baseline.txt
```

### Cause

The command enumerated:

```text
C:\InstallUtilAbuseLab
```

including its subdirectories.

The output file was created inside the same directory tree.

Therefore, the baseline file appeared in the results.

### Lesson

For a cleaner baseline, store the baseline outside the measured directory or explicitly exclude the baseline file from the enumeration.

---

## 10. Large Sysmon Event Volume

The supplied Event Viewer screenshots showed approximately:

```text
46,000+ Sysmon events
```

and more than:

```text
23,000 Event ID 1 events
```

### Problem

Manually reviewing thousands of events is inefficient and increases the risk of missing the relevant process.

### Recommended Focus

For Event ID 1:

```text
Image
CommandLine
ProcessId
ProcessGuid
ParentImage
ParentProcessId
ParentProcessGuid
User
UtcTime
```

For Event ID 11:

```text
Image
TargetFilename
ProcessId
ProcessGuid
User
UtcTime
```

---

## 11. PowerShell Event Attribution

Observed:

```text
Image:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

and:

```text
TargetFilename:
C:\WINDOWS\Temp\_PSScriptPolicyTest_hphtfxff.tww.ps1
```

### Interpretation

The event confirms PowerShell file creation.

It does not prove:

```text
PowerShell
    |
    v
InstallUtil
```

or:

```text
InstallUtil
    |
    v
PowerShell
```

Without process lineage, the relationship should remain unconfirmed.

---

## 12. WMI Event Attribution

Observed:

```text
ParentImage:
C:\WINDOWS\System32\wbem\WmiPrvSE.exe
```

and:

```text
ParentCommandLine:
C:\WINDOWS\system32\wbem\wmiprvse.exe -Embedding
```

### Interpretation

The event demonstrates WMI-related activity.

It should not automatically be classified as part of the InstallUtil investigation.

Process GUID and parent-child relationships should be followed before making an attribution.

---

## 13. InstallUtil Execution Not Confirmed

The following evidence is confirmed:

```text
InstallUtil.exe exists
```

```text
InstallUtil.exe is signed
```

```text
Sysmon is running
```

```text
Wazuh is running
```

However, the available evidence does not yet prove:

```text
InstallUtil.exe was executed
```

or:

```text
InstallUtil.exe processed the benign assembly
```

or:

```text
InstallUtil.exe created the expected output artifact
```

These should remain pending until direct telemetry is obtained.

---

## 14. Recommended Troubleshooting Sequence

Use the following sequence when continuing the lab:

```text
Validate source
        |
        v
Validate compiler
        |
        v
Compile correct assembly type
        |
        v
Confirm output file
        |
        v
Inspect assembly structure
        |
        v
Record file metadata
        |
        v
Calculate SHA256
        |
        v
Confirm Output directory is clean
        |
        v
Perform controlled execution
        |
        v
Search Sysmon Event ID 1
        |
        v
Search Sysmon Event ID 11
        |
        v
Correlate Wazuh telemetry
        |
        v
Document findings
```

---

## 15. Artifact Validation Checklist

Before executing the assembly, confirm:

```text
[ ] Assembly exists
[ ] Assembly size is non-zero
[ ] Assembly opens successfully
[ ] Expected installer class exists
[ ] Expected output path is known
[ ] SHA256 hash recorded
[ ] Creation time recorded
[ ] Last write time recorded
[ ] Output directory is clean
[ ] Sysmon is running
[ ] Wazuh is running
```

---

## 16. Current Troubleshooting Status

| Issue | Status |
|---|---|
| Incorrect `dotnet` command | Identified |
| C# compiler availability | Confirmed |
| `CS5001` compilation error | Identified |
| Missing EXE | Explained |
| `corflags.exe` unavailable at tested path | Identified |
| Baseline includes evidence file | Explained |
| High Sysmon event volume | Identified |
| PowerShell attribution | Requires correlation |
| WMI attribution | Requires correlation |
| Valid compiled assembly | Pending |
| InstallUtil execution | Not confirmed |

---

## Key Troubleshooting Lesson

The main lesson from this phase is that **artifact preparation must be completed before execution analysis begins**.

The `CS5001` error prevented the expected assembly from being created. Therefore, subsequent telemetry should not be interpreted as successful InstallUtil execution unless direct evidence confirms that the intended assembly was actually processed.
