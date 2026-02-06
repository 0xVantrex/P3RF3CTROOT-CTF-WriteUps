# 

**Category:** DFIR  
**Points:** 200  
**Challenge:** _Dropping the Shield_

---

## **Objective**

Identify the **exact command** the attacker used to disable antivirus protection on the compromised Windows host.

---

## **1. Expanding the Evidence**

The challenge shipped two ZIP files (prof1.zip, prof2.zip).  
After extracting them, we inspected the folder structure and immediately focused on:

`Users/<user>/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/`

This directory stores **ConsoleHost_history.txt**, which logs executed PowerShell commands — prime evidence for attacker activity.

---

## **2. Hunting for Antivirus Disable Commands**

Because Windows Defender modifications usually involve:

- `Set-MpPreference`
    
- `WinDefend`
    
- `DisableRealtimeMonitoring`
    
- `sc stop` / `net stop`
    
- `MpCmdRun`
    

…we used targeted grep searches:

`grep -RniE "mp|defender|WinDefend|Disable" Users/*/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/`

This returned:

`1: powershell -command 'Set-MpPreference -DisableRealtimeMonitoring $true -DisableScriptScanning $true -DisableBehaviorMonitoring $true -DisableIOAVProtection $true -DisableIntrusionPreventionSystem $true' 2: Set-MpPreference -DisableRealtimeMonitoring $true 3: Set-MpPreference -DisableScriptScanning $true 4: Set-MpPreference -DisableBehaviorMonitoring $true 5: Set-MpPreference -DisableIOAVProtection $true 6: Set-MpPreference -DisableIntrusionPreventionSystem $true`

We also scanned the entire disk image for fallback signatures:

`grep -RniE "Set-MpPreference|WinDefend|DisableRealtime|DisableAntiSpyware|sc stop|net stop|MpCmdRun" .`

This confirmed **no other Defender shutdown commands** existed outside this history.

---

## **3. Identifying the “Single Command”**

The challenge states:

> “The attacker disabled antivirus protection with a **single command**.”

We found multiple PowerShell lines, but only **one** was a full one‑liner that disables EVERYTHING at once:

`powershell -command 'Set-MpPreference -DisableRealtimeMonitoring $true -DisableScriptScanning $true -DisableBehaviorMonitoring $true -DisableIOAVProtection $true -DisableIntrusionPreventionSystem $true'`

Lines 2–6 were run afterward but each disables only one module.  
They are _not_ the “single command”.

The **first line** is the actual kill-switch:

- It calls PowerShell
    
- Executes Set‑MpPreference with **five disable flags at once**
    
- Fits the challenge wording perfectly
    
- Appears exactly once in the logs
    
- Disables Windows Defender’s major protection layers in a single operation
    

---

## **4. Final Answer**

{r00t{`powershell -command 'Set-MpPreference -DisableRealtimeMonitoring $true -DisableScriptScanning $true -DisableBehaviorMonitoring $true -DisableIOAVProtection $true -DisableIntrusionPreventionSystem $true'`}

This is the exact command the attacker executed.