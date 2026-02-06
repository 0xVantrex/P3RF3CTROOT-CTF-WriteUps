# 

### **Question:**

_“The adversary quietly deployed a zip utility to prepare for data exfiltration. When was this tool likely installed?”_  
**Answer Format:** `r00t{yyyy-mm-dd HH:MM}` (UTC)()

---

## **Overview**

We were provided with a Windows NTFS artifact dump (`$MFT`, `$LogFile`, system directories, etc.).  
The challenge states that the attacker deployed a “zip utility,” meaning some form of 7-Zip or similar compression tool used to prepare data for exfiltration.

Since no EVTX logs were included, the installation or execution time had to be recovered from **file system forensics**.

---

# **Step 1 — Locating the Zip Utility**

From the root of the extracted logs, I searched for any files corresponding to common zip tools:

`find . -type f \( -iname "*7z*" -o -iname "*zip.exe" -o -iname "*winzip*" -o -iname "*zip*" \) -printf '%p\n'`

This returned:

`./Windows/prefetch/7ZFM.EXE-7C92DCA0.pf ./Windows/prefetch/7Z2501-X64.EXE-36B52C5B.pf`

These are **Windows Prefetch files** for 7‑Zip (`7ZFM.EXE` and `7Z2501-X64.EXE`).  
This proves the attacker ran a **portable 7-Zip executable** — the common method for stealth exfil prep.

---

# **Step 2 — Prefetch = First Execution Timestamp**

Prefetch files store the **first execution time** of a program.  
This is exactly what the question means by "installed" or "deployed" — the moment the attacker executed the tool.

I retrieved filesystem metadata:

`stat "./Windows/prefetch/7ZFM.EXE-7C92DCA0.pf" stat "./Windows/prefetch/7Z2501-X64.EXE-36B52C5B.pf"`

Relevant field:

`Modify: 2025-11-24 08:25:54 -0500 Modify: 2025-11-24 08:25:04 -0500`

Windows NTFS PF **Modify** timestamps (rounded to the minute) reflect the embedded _last execution timestamp_, and for newly created PFs this matches the **first run time**.

Both confirm the first execution was around **08:25 local time**.

Local timezone in dump: **UTC-5**  
Convert to UTC:

`08:25 - 5 hours = 13:25 UTC`

---

# **Final Answer**

`r00t{2025-11-24 13:25}`

---

# **Conclusion**

By identifying the prefetch artifacts and extracting their execution timestamps, we determined that the zip utility (7‑Zip) was first run on **2025‑11‑24 at 13:25 UTC**, marking the attacker’s deployment of their compression tool in preparation for exfiltration.