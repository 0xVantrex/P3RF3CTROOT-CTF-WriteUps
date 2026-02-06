### **Category:** Reverse Engineering

### **Difficulty:** Beginner/Intermediate

### **Summary:**

The binary asks for a username and a math answer, but even with the correct sum, it refuses to give the flag. The real check is hidden behind an environment variable.

---

## **1. Initial Execution**

Running the binary normally:

`./perfetcuser`

It prompts:

- **Name**
    
- **13 + 37**
    

Even with the correct answer `50`, or any guess, you only get:

`Correct, but not what I needed to see.`

So the math check isn’t the real path.

---

## **2. Recon: Searching for Clues**

### **Strings**

A quick `strings` scan shows interesting values:

`p3rf3ctr00t TRUE Correct, but not what I needed to see. Correct, see this: %s`

That immediately hints at:

- a hardcoded username: `p3rf3ctr00t`
    
- a hardcoded expected value: `TRUE`
    
- a hidden success message
    

---

## **3. Digging Into the Binary (.rodata)**

Dumping `.rodata`:

`objdump -s -j .rodata perfetcuser`

We find the important part:

`... p3rf3ctr00t ... TRUE ... Correct, see this: %s`

And right after the username is the literal string `"TRUE"`.  
This means the binary likely calls:

`getenv("p3rf3ctr00t")`

and checks if the value returned is `"TRUE"`.

---

## **4. Disassembly Confirms It**

Looking at the logic around `strcmp`:

`objdump -d -M intel perfetcuser | grep -A 20 getenv`

We see:

1. A string is loaded.
    
2. `getenv()` is called with that string.
    
3. The result is compared with `"TRUE"` via `strcmp`.
    

If the comparison fails → print _"Correct, but not what I needed to see."_

If it succeeds → print `"Correct, see this: %s"` (flag).

---

## **5. Exploitation – Set the Environment Variable**

The trick:

The **environment variable name = username**  
The **required value = TRUE**

So you export it:

`export p3rf3ctr00t=TRUE`

Then run the binary normally:

`./perfetcuser`

Enter:

`p3rf3ctr00t 50`

or any answer — the math doesn’t matter anymore.

Finally you get:

`Correct, see this: r00t{1234_welc0me_t0_th3_CTF!!}`

---

## **6. Final Notes**

The challenge tested:

- reading `.rodata`
    
- recognizing embedded strings as clues
    
- understanding how a binary uses `getenv()`
    
- tracing logic around `strcmp`
    
- bypassing false logic paths
    

The twist: the program hides the real condition behind an environment variable named after the required username.