<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
# 8-bit-multiplier
## Introduction
In EEP1 CPU configuration, there is no implemented multipliers. If we try to perform multiplication
in EEP1 by fully software implementation, it needs 34 clock cycles to complete for a 16-bit unsigned
multiplication based on the result from Task 4 or 40 clock cycles for 32-bit unsigned multiplication.

Therefore, it's necessary to implement a multiplier which is able to calculate the signed and unsigned
product of two inputs with less time taken. In this design, I tried to implement a fully combinational
multiplier, which means it takes the same number of clock cycles to complete the instructions as other
EEP1 machine code instruction do.

I only made a 8-bit multiplier instead of a 16-bit multiplier (which is equal to the width of registers)
is because that will give a 32 bit product, which is far bigger than the width of registers. 

## Mathematical Background
### 1-bit unsigned multiplier
Let's first look at the mathematical and Boolean algorithm for 1-bit unsigned multiplcation:
A 1-bit unsigned multiplier takes two 1-bit binary numbers (A and B) and produces a 1-bit 
product (P). Since binary multiplication follows the same principles as decimal multiplication but 
with only two possible values (0 and 1), the truth table for a 1-bit multiplier is simple:

| A | B | P (A × B) |
|---|---|---------|
| 0 | 0 | 0       |
| 0 | 1 | 0       |
| 1 | 0 | 0       |
| 1 | 1 | 1       |

The multiplication follows:
- \( 0 	imes 0 = 0 \)
- \( 0 	imes 1 = 0 \)
- \( 1 	imes 0 = 0 \)
- \( 1 	imes 1 = 1 \)

Thus, the output \( P \) is simply the logical AND operation between A and B.

### 2-bit unsigned multiplier
A 2-bit unsigned multiplier takes two 2-bit binary numbers (A and B) and produces a 4-bit product (P). 
The multiplication follows standard binary multiplication rules, where each bit of B is multiplied by 
each bit of A.
Consider two 2-bit numbers:

\[ A = A_1A_0, \quad B = B_1B_0 \]

The multiplication is performed as follows:

```
       A1  A0
   ×   B1  B0
  ------------
       A1B0 A0B0  (Partial Product 1)
+  A1B1 A0B1  0   (Partial Product 2, shifted left)
  ------------
    P3  P2  P1  P0
```

The final 4-bit product is:

\[ P = P_3P_2P_1P_0 \]
#### Boolean Algorithm
Each bit of the product can be derived using AND operations:

\[
P_0 = A_0 B_0
\]
\[
P_1 = (A_1 B_0) \oplus (A_0 B_1)
\]
\[
P_2 = (A_1 B_1) \oplus (A_0 B_1 \cdot A_1 B_0)
\]
\[
P_3 = A_1 B_1 \cdot A_0 B_1 \cdot A_1 B_0
\] <bar>

And this is the implementation:
A 2-bit multiplier requires multiple AND and XOR gates:



