# Timeline - Lab 65

## Timeline Overview

This timeline documents the preparation of the InstallUtil investigation, assembly compilation failure, baseline creation, pre-execution checks, and relevant endpoint telemetry.

The timeline distinguishes confirmed laboratory actions from endpoint activity that has not been directly attributed to InstallUtil.

---

## 2026-08-29

### 07:28 - Laboratory Workspace Created

The main laboratory directory was created:

```text
C:\InstallUtilAbuseLab
```

The following directories were created:

```text
C:\InstallUtilAbuseLab\Payload
C:\InstallUtilAbuseLab\Output
C:\InstallUtilAbuseLab\Evidence
```

**Classification:** Confirmed laboratory activity.

---

### 07:28 - Directory Structure Verified

The laboratory root was enumerated.

Observed:

```text
Evidence
Output
Payload
```

**Classification:** Confirmed laboratory activity.

---

### 07:34 - Filesystem Baseline Created

A baseline was written to:

```text
C:\InstallUtilAbuseLab\Evidence\baseline.txt
```

The baseline recorded:

```text
FullName
Length
CreationTime
LastWriteTime
```

The baseline file itself appeared in the enumeration because it was created inside the directory being measured.

**Classification:** Confirmed laboratory activity.

---

### 07:39 - Benign Assembly Source Created

The source file was created:

```text
C:\InstallUtilAbuseLab\Payload\InstallUtilBenignLab.cs
```

**Classification:** Confirmed laboratory activity.

---

### 07:39 - C# Compiler Verified

The .NET Framework C# compiler was checked:

```text
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe
```

The check returned:

```text
True
```

**Classification:** Confirmed laboratory activity.

---

### 07:39 - Assembly Compilation Attempted

Compilation was attempted to produce:

```text
C:\InstallUtilAbuseLab\Payload\InstallUtilBenignLab.exe
```

The compiler reported:

```text
Microsoft (R) Visual C# Compiler version 4.8.9032.0
for C# 5
```

**Classification:** Confirmed laboratory activity.

---

### 07:39 - Compilation Failed

The compiler returned:

```text
error CS5001: Program 'c:\InstallUtilAbuseLab\Payload\InstallUtilBenignLab.exe' does not contain a static 'Main' method suitable for an entry point
```

The expected executable was therefore not created.

**Classification:** Confirmed compilation failure.

---

### 07:39 - Payload Directory Checked

The payload directory was checked after the compilation failure.

Observed:

```text
InstallUtilBenignLab.cs
```

The expected:

```text
InstallUtilBenignLab.exe
```

was absent.

**Classification:** Confirmed laboratory state.

---

### 07:40+ - `corflags.exe` Check Attempted

An attempt was made to access:

```text
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\corflags.exe
```

PowerShell reported that the executable was not recognized at that location.

**Classification:** Confirmed diagnostic result.

---

### 07:41:43 - Output Directory Checked

The recorded time was:

```text
29 August 2026 07:41:43
```

The output directory was checked:

```text
C:\InstallUtilAbuseLab\Output
```

No files were displayed.

**Classification:** Confirmed pre-execution state.

---

### 07:43:05 - Sysmon Process Activity Observed

A supplied Sysmon Event ID 1 showed WMI-related process activity.

Relevant fields included:

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

**Classification:** Observed endpoint telemetry.

**InstallUtil attribution:** Not established.

---

### 07:45:29 - Sysmon Process Activity Observed

Additional Sysmon Event ID 1 activity was observed around:

```text
29-08-2026 07:45:29
```

The supplied telemetry included PowerShell-related process activity.

**Classification:** Observed endpoint telemetry.

**InstallUtil attribution:** Not established.

---

### 07:45:29 - Sysmon File Creation Observed

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

**Classification:** Observed endpoint telemetry.

**Interpretation:** PowerShell created a temporary file.

**InstallUtil attribution:** Not established.

---

### 07:45:30 - Additional Sysmon Activity Observed

The Event Viewer displayed additional Event ID 1 and Event ID 11 activity.

The supplied screenshots showed a very large Sysmon event volume.

**Classification:** Observed endpoint telemetry.

**InstallUtil attribution:** Not established.

---

# Confirmed Laboratory Timeline

```text
07:28
Lab directories created
        |
        v
07:34
Filesystem baseline created
        |
        v
07:39
Benign assembly source created
        |
        v
07:39
Compilation attempted
        |
        v
07:39
CS5001 compilation failure
        |
        v
07:41
Output directory confirmed empty
```

---

# Observed Endpoint Telemetry

```text
07:43+
WMI-related process activity observed
        |
        v
07:45+
PowerShell process/file activity observed
        |
        v
07:45+
Additional Sysmon Event ID 1 and Event ID 11 activity
```

These events should not be merged into the confirmed InstallUtil execution chain without supporting process lineage.

---

# Evidence Attribution Table

| Time | Activity | Classification |
|---|---|---|
| 07:28 | Lab directories created | Confirmed laboratory activity |
| 07:34 | Baseline generated | Confirmed laboratory activity |
| 07:39 | Assembly source created | Confirmed laboratory activity |
| 07:39 | Compilation attempted | Confirmed laboratory activity |
| 07:39 | `CS5001` returned | Confirmed compilation failure |
| 07:41 | Output directory checked | Confirmed laboratory activity |
| 07:43+ | WMI-related process activity | Observed endpoint activity |
| 07:45+ | PowerShell/file activity | Observed endpoint activity |
| 07:45+ | Additional Sysmon events | Observed endpoint activity |
| Later | Successful InstallUtil execution | Not confirmed |

---

# Expected Successful Execution Timeline

A successful controlled execution should produce evidence resembling:

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
Expected output artifact created
        |
        v
Sysmon Event ID 11
        |
        v
Wazuh correlation
```

The actual timestamps, process IDs, GUIDs, and file metadata must be obtained from telemetry rather than assumed.

---

# Investigation Status

| Stage | Status |
|---|---|
| InstallUtil discovery | Complete |
| InstallUtil validation | Complete |
| Lab workspace creation | Complete |
| Sysmon verification | Complete |
| Wazuh verification | Complete |
| Filesystem baseline | Complete |
| Assembly source creation | Complete |
| Assembly compilation | Failed - `CS5001` |
| Compiled assembly validation | Pending |
| InstallUtil execution | Not confirmed |
| Sysmon InstallUtil correlation | Pending |
| Wazuh correlation | Pending |

---

# Timeline Conclusion

The documented timeline confirms successful preparation of the laboratory environment, validation of the selected InstallUtil binary, creation of the filesystem baseline, and preparation of the benign assembly source.

The assembly compilation then failed with `CS5001`, preventing the expected executable from being created.

PowerShell and WMI-related Sysmon events were subsequently observed, but the supplied evidence does not establish a direct relationship between those events and InstallUtil. Successful InstallUtil execution therefore remains **unconfirmed** until direct process and artifact correlation is obtained.
