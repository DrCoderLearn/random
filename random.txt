# **Case Study: Formal Verification of HTU**  

This case study presents our approach to formally verifying the Hardware Throttling Unit (HTU). We focused on two key aspects: **Counter Reduction and Abstraction** and **Register Data Integrity Check**.  

---

## **1. Counter Reduction and Abstraction**  

### **Challenge**  
The HTU includes a **hysteresis filter** that adds hysteresis when the input goes low. A **25-bit counter** controls when the output is allowed to go low, based on a programmed threshold stored in an **SFR register**.  

- The counter increments every clock cycle until it matches the SFR value.  
- This creates **16,777,216 possible states**, which is too large for formal tools to exhaustively verify, leading to inconclusive results.  
- While smaller values could be formally verified, this was insufficient for sign-off.  

### **Solution: Counter Abstraction**  
To **reduce state-space complexity**, we applied **counter abstraction**:  

1. **Stop Points:**  
   - We introduced **stopats** on:  
     - The SFR holding the counter threshold.  
     - The RTL counter, which increments every clock.  

2. **Key Assumptions:**  
   - **SFR Coverage:** We used `$onehot(sfr)` to ensure that each of the 25 bits was tested as high at some point.  
   - **Counter Behavior:** Instead of incrementing by 1 every cycle, we assumed:  
     ```
     counter == $past(counter) << 1
     ```
     This effectively **reduced sequential depth**, making verification feasible.  

### **Outcome**  
- The abstraction significantly improved formal convergence.  
- The verification property ensured that when the counter reached the SFR value, the **primary output correctly responded** to the primary input.  

---

## **2. Register Data Integrity Check**  

### **Challenge**  
We needed to verify that **SFR writes were correctly stored and read back later**, ensuring data integrity.  

- A value written to an SFR must **persist** and match when read at any future cycle.  
- The read could happen at **any arbitrary time after the write** (not necessarily the next cycle).  

### **Solution: Using Non-Consecutive Goto (`[=n]`)**  

1. **Initial Approach (Limitation)**  
   - We first considered `write [->1] ##1 read`, enforcing a **strict one-cycle delay** between write and read.  
   - **Issue:** This prevented checking for reads that occurred **much later** after the write.  

2. **Optimized Approach**  
   - We used `write [=1] ##1 read`, which allows the read to occur **at any arbitrary future cycle** after the write.  
   - To track correctness, we:  
     - Stored the written value in a **local property variable**.  
     - Ensured the **future read matched the stored value** for assertion success.  

### **Outcome**  
- Achieved a **flexible, robust data integrity check** for all SFRs.  
- Improved **formal completeness** without unnecessary restrictions on read timing.  

---

## **Conclusion**  
By strategically applying **counter abstraction** and **optimized property writing**, we significantly enhanced the efficiency and completeness of HTU formal verification. These techniques ensured that:  

✅ **State-space complexity was reduced**, allowing exhaustive verification.  
✅ **Data integrity was preserved** across multiple SFRs, regardless of when reads occurred.  
✅ **Verification convergence improved**, leading to **conclusive results** for sign-off.  

This structured approach provided a **scalable and efficient** formal verification methodology for HTU.
