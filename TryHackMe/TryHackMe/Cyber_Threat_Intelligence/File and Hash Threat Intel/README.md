# Lab Investigation: File and Hash Threat Intelligence

## 📌 Executive Summary
* **Scenario:** During a routine alert sweep, multiple suspicious binaries were flagged across workstations. The objective of this investigation is to triage a curated package of malware samples within 60 minutes to determine if they are benign or malicious.
* **Objective:** Verify, enrich, and determine the nature of unknown files using static heuristics, cryptographic hashing, and automated sandbox telemetry.
* **Tools Used:** Windows CMD/PowerShell, VirusTotal, MalwareBazaar, Hybrid Analysis, MITRE ATT&CK Framework.

---

## 🛠️ Phase 1: Static Heuristics (Filenames & Paths)
Initial triage involves examining human-readable strings and file properties for indicators of malicious tradecraft before execution.

**Q: One file displays one of the indicators mentioned. Can you identify the file and the indicator?**
* **Finding:** `payroll.pdf`, Double extensions
* **Analysis:** Attackers frequently abuse default Windows settings that hide known file extensions. By appending a legitimate document extension (`.pdf`) before the hidden executable extension, the file visually masquerades as a benign payroll document to trick the end-user into executing the payload.

> 📸 **[SCREENSHOT: File Explorer]** 
> *Capture the directory showing `payroll.pdf.exe` or its Properties window highlighting the true `.exe` file type.*

---

## 🔑 Phase 2: Cryptographic Fingerprinting & Hash Lookups
To establish immutable identities for the binaries regardless of their filenames, cryptographic hashes were generated and enriched using Threat Intelligence platforms.

### Target: `bl0gger.exe`
**Q: What is the SHA256 hash of the file bl0gger?**
* **Finding:** `2672b6688d7b32a90f9153d2ff607d6801e6cbde61f509ed36d0450745998d58`
* **Analysis:** The hash was extracted using the native Windows command-line utility `certutil -hashfile bl0gger.exe SHA256`. This creates a reliable fingerprint for cross-referencing against global threat databases.

> 📸 **[SCREENSHOT: Terminal]** 
> *Capture the CMD or PowerShell window showing the executed hash command and the resulting SHA256 output.*

**Q: On VirusTotal, what is the threat label used to identify the malicious file?**
* **Finding:** `trojan.graftor/flystudio` (or `trojan.graftor/blackmoon`)
* **Analysis:** Querying the extracted hash on VirusTotal revealed a high detection confidence, with multiple AV engines categorizing the binary as a Trojan associated with the Graftor/Flystudio family, known for backdoor capabilities.

**Q: When was the file first submitted for analysis?**
* **Finding:** `2025-05-15 12:03:49 UTC`
* **Analysis:** Analyzing the "First Submission" timestamp under the Details tab provides context on the age of the campaign. Older, widely submitted hashes indicate known commodity malware, whereas newly submitted or unindexed hashes suggest a targeted or zero-day attack.

### Target: `Morse-Code-Analyzer.exe`
**Q: According to MalwareBazaar, which vendor classified the Morse-Code-Analyzer file as non-malicious?**
* **Finding:** `CyberFortress`
* **Analysis:** Not all security vendors utilize the same detection signatures. Discrepancies between vendors highlight the importance of aggregating intelligence across multiple platforms rather than relying on a single engine's verdict.

**Q: On VirusTotal, what MITRE technique has been flagged for persistence and privilege escalation for the Morse-Code-Analyzer file?**
* **Finding:** DLL Side-Loading
* **Analysis:** The behavior tab indicates the binary attempts to execute malicious payloads by dropping a spoofed DLL into a directory where a legitimate application expects to load it. This technique allows the malware to evade detection by running within the context of a trusted process.

> 📸 **[SCREENSHOT: VirusTotal]** 
> *Capture the VirusTotal Behavior tab showing the MITRE ATT&CK mapping for DLL Side-Loading.*

---

## 🔬 Phase 3: Automated Sandbox Analysis (Dynamic Telemetry)
To understand the actual functional payload and bypass static analysis limitations, sandbox telemetry (Hybrid Analysis) was reviewed.

### Target: `bl0gger.exe`
**Q: What tags are used to identify the bl0gger.exe malicious file on Hybrid Analysis?**
* **Finding:** `BlackMoon, Discovery, windows-server-utility`

**Q: What was the stealth command line executed from the file?**
* **Finding:** `regsvr32 %WINDIR%\Media\ActiveX.ocx /s`
* **Analysis:** The malware utilizes `regsvr32.exe`, a native Windows signed binary, to silently (`/s`) register a malicious OLE control extension (`.ocx`). This is a classic "Living off the Land" (LotL) technique used to bypass application whitelisting and execute code stealthily.

**Q: Which other process was spawned according to the process tree?**
* **Finding:** `werfault.exe`
* **Analysis:** The malware likely spawned the Windows Error Reporting process (`werfault.exe`) either as a byproduct of a crash during execution or intentionally for process hollowing—injecting malicious code into a legitimate Windows error process to evade EDR detection.

> 📸 **[SCREENSHOT: Hybrid Analysis]** 
> *Capture the process tree from Hybrid Analysis showing `bl0gger.exe` spawning `regsvr32` and `werfault.exe`.*

### Target: `payroll.pdf`
**Q: The payroll.pdf application seems to be masquerading as which known Windows file?**
* **Finding:** `svchost.exe`
* **Analysis:** By impersonating the Service Host process, the malware attempts to blend into standard Windows background noise, making visual identification by an analyst difficult in Task Manager.

**Q: What associated URL is linked to the file?**
* **Finding:** `hxxp://121[.]182[.]174[.]27:3000/server.exe` *(Note: Defanged for safety)*
* **Analysis:** Sandbox telemetry captured an outbound network connection to a suspicious IP over port 3000, attempting to pull down a secondary payload (`server.exe`). This IP represents a critical network Indicator of Compromise (IOC) to block at the firewall.

**Q: How many extracted strings were identified from the sandbox analysis of the file?**
* **Finding:** `454`

---

## 🎯 Phase 4: Final Threat Intelligence Challenge 
Consolidating static and dynamic analysis to investigate a final, unknown sample (`Challenge.bin.sample`).

**Q: What is the SHA256 hash of the file?**
* **Finding:** `43b0ac119ff957bb209d86ec206ea1ec3c51dd87bebf7b4a649c7e6c7f3756e7`

**Q: What family labels are assigned to the file on VirusTotal?**
* **Finding:** `akira, filecryptor`
* **Analysis:** The hash lookup confirms the file belongs to the Akira ransomware family, a known threat actor group that utilizes file cryptors to encrypt victim data for extortion.

**Q: How many security vendors have flagged the file as malicious?**
* **Finding:** `61`

**Q: Name the text file dropped during the execution of the malicious file.**
* **Finding:** `akira_readme.txt`
* **Analysis:** This is the standard ransomware ransom note dropped onto the filesystem after the encryption routine completes, providing instructions to the victim.

**Q: What PowerShell script is observed to be executed?**
* **Finding:** `Get-WmiObject Win32_Shadowcopy | Remove-WmiObject`
* **Analysis:** This PowerShell command uses WMI to locate and delete all Volume Shadow Copies on the machine. This is a critical ransomware behavior designed to prevent the victim from recovering their encrypted files from local Windows backups.

**Q: What is the MITRE ATT&CK ID associated with this execution?**
* **Finding:** `T1490` (Inhibit System Recovery)
* **Analysis:** The deletion of shadow copies maps directly to MITRE ATT&CK T1490, confirming the primary objective of the executed payload is to maximize the impact of the extortion attempt by disabling recovery mechanisms.

> 📸 **[SCREENSHOT: Execution Flow]** 
> *Capture the VirusTotal Behavior tab or Sandbox activity showing the malicious PowerShell command execution.*
