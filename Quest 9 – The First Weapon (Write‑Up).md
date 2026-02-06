# **Quest 9 – The First Weapon (Write‑Up)**

**Objective:**  
Identify which executable was used as the attacker’s **initial launcher** on the compromised Windows system.  
Answer expected in **lowercase**.

---

## **1. Understanding the Task**

The challenge describes the attacker’s **first weapon** — meaning the very first program executed to begin the intrusion. Typically this is a **LOLBIN** (Living‑off‑the‑Land binary) used to launch a script, payload, or remote content.

So we needed to determine **which executable initiated the attack chain**.

---

## **2. Evidence Reviewed**

We inspected the forensic logs extracted from the `Windows` directory:

### **a. Prefetch (`Windows/Prefetch/`)**

Prefetch files show recently executed programs.  
We listed them:

`ls Windows/prefetch`

This revealed many executables including `cmd.exe`, `powershell.exe`, `wscript.exe`, etc.  
However, none of these directly indicated the initial infection vector.

### **b. Amcache / AppCompatCache**

Using:

`grep -RniE "\.exe" Windows/AppCompat/Programs/*`

We confirmed various executables, but binary matches alone don’t confirm _initial_ execution.

At this point, simply seeing that “an EXE ran” wasn’t enough. We needed to identify **what attackers normally use for initial staging**.

---

## **3. Key Insight**

Revisiting the _challenge description_ is what broke it open:  
“The first weapon” strongly hinted toward a **LOLBin used for initial execution**, not a payload itself.

One of the most common initial launchers in real‑world attacks is:

### **`mshta.exe`**

- Built‑in Windows binary
    
- Executes **HTA files** containing JavaScript/VBScript
    
- Supports **remote URLs**, making it ideal for phishing/lure‑based attacks
    
- Widely used by APTs, ransomware operators, and red‑teamers
    

This fits perfectly with the challenge context:  
Attack started with a **launcher**, not the malware itself.

---

## **4. Verification with Prefetch**

Searching prefetch confirmed execution:

`ls Windows/Prefetch | grep -i mshta`

Result example:

`MSHTA.EXE-XXXXX.pf`

This confirmed `mshta.exe` was executed on the system.

---

## **5. Conclusion**

The attacker’s **first launcher** — their “first weapon” — was:

`mshta.exe`

---

## **Final Answer:**

**mshta.exe**