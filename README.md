# 8-bit Multiplier Report

## Introduction
In EEP1 CPU configuration, there are no implemented multipliers. If we try to perform multiplication
in EEP1 by fully software implementation, it needs 34 clock cycles to complete for a 16-bit unsigned
multiplication based on the result from Task 4 or 40 clock cycles for 32-bit unsigned multiplication.

Therefore, it's necessary to implement a multiplier which is able to calculate the signed and unsigned
product of two inputs with less time taken. In this design, I tried to implement a fully combinational
multiplier, which means it takes the same number of clock cycles to complete the instructions as other
EEP1 machine code instruction do.

I only made an 8-bit multiplier instead of a 16-bit multiplier (which is equal to the width of registers)
is because that will give a 32-bit product, which is far bigger than the width of registers. 

## Mathematical Background and Implementation
### 1-bit unsigned multiplier
Let's first look at the mathematical and Boolean algorithm for 1-bit unsigned multiplication:
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
- \( 0 	* 0 = 0 \)
- \( 0 	* 1 = 0 \)
- \( 1 	* 0 = 0 \)
- \( 1 	* 1 = 1 \)

Thus, the output \( P \) is simply the logical AND operation between A and B.

### 2-bit unsigned multiplier
A 2-bit unsigned multiplier takes two 2-bit binary numbers (A and B) and produces a 4-bit product (P). 
The multiplication follows standard binary multiplication rules, where each bit of B is multiplied by 
each bit of A.
Consider two 2-bit numbers:

$$
A = A_1A_0, \quad B = B_1B_0 
$$

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


$$
A = A_1A_0, \quad B = B_1B_0
$$




#### Boolean Algorithm
Each bit of the product can be derived using AND operations:

$$
P_0 = A_0 B_0
$$

$$
P_1 = (A_1 B_0) \oplus (A_0 B_1)
$$

$$
P_2 = (A_1 B_1) \oplus \left( (A_0 B_1) \cdot (A_1 B_0) \right)
$$

$$
P_3 = A_1 B_1 \cdot A_0 B_1 \cdot A_1 B_0
$$

And this is the implementation:

However, the problem of this implementation is it takes 4 multiplication steps (or 16 steps if we count
the multiplications inside the 2-bit unsigned adder). And as we know, the multiplication is much more
complex in computer than addition. Therefore, we can try to reduce the steps of multiplication.

By observing the long multiplication diagram for 2-bit unsigned multiplier, we can find out that the final
product is
$$
P = 4 \times (A_1 B_1) + 2 \times (A_1 B_0 + A_0 B_1) + (A_0 B_0)
$$
, and
$$
A_1 B_0 + A_0 B_1 = (A_1 + A_2)(B_1 + B_2) - A_1 B_1 - A_2 B_2
$$
therefore, we can reduce four steps multiplication to three steps multiplication. And this is the
implementation based on this algorithm.

A problem of this kind implementation is the cost of hardware is so much higher than normal 4-bit multipliers
, and by the way, in order to implement this algorithm, we need to use a 2-bit multiplier before we figured
out a 2-bit multiplication implementation.Then, what's the aim of using a 2-bit multiplier
to make a 2-bit multiplier? Therefore, I gave up this algorithm and backed to the normal
one.

#### 4-Bit Unsigned Multiplier

Let: A = A₃A₂A₁A₀ (4-bit binary number) and B = B₃B₂B₁B₀ (4-bit binary number). We are looking to compute P = A × B, which will be an 8-bit number.

##### Partial Products (Table Representation)

We can break the multiplication into smaller parts as follows:

A = A₃A₂A₁A₀ and B = B₃B₂B₁B₀. The multiplication will be performed bit by bit. Each bit of B will multiply the entire number A, and the results will be shifted accordingly.

| Multiplicand (A) | Multiplier bit B₀ | Multiplier bit B₁ | Multiplier bit B₂ | Multiplier bit B₃ |
|------------------|-------------------|-------------------|-------------------|-------------------|
| A × B₀           | A₃A₂A₁A₀ × B₀   | Shifted left by 1: A₃A₂A₁A₀ × B₁ | Shifted left by 2: A₃A₂A₁A₀ × B₂ | Shifted left by 3: A₃A₂A₁A₀ × B₃ |

##### Breaking it Down into the Four 2-bit Multiplications

To simplify, we divide the multiplication into smaller parts, using the 2-bit multipliers. The four 2-bit multiplications are:

1. **P₀ = A[1:0] × B[1:0]**: This multiplies the lower 2 bits of A and B. The result is a 4-bit product.
2. **P₁ = A[1:0] × B[3:2]**: This multiplies the lower 2 bits of A with the upper 2 bits of B. The result is a 4-bit product.
3. **P₂ = A[3:2] × B[1:0]**: This multiplies the upper 2 bits of A with the lower 2 bits of B. The result is a 4-bit product.
4. **P₃ = A[3:2] × B[3:2]**: This multiplies the upper 2 bits of A with the upper 2 bits of B. The result is a 4-bit product.

##### Mathematical Algorithm for Each Bit of the Product

Now, let's define the **mathematical algorithm** to compute each bit of the product P = A × B.

For A = A₃A₂A₁A₀ and B = B₃B₂B₁B₀, the product P = P₇P₆P₅P₄P₃P₂P₁P₀ can be broken down as follows:

- **P₀**: The least significant bit is the result of multiplying the least significant bits of A and B, i.e., A₀ × B₀:  
  P₀ = A₀ * B₀

- **P₁**: The second bit is computed by adding the products of the lower bits of A with the higher bits of B, using XOR for addition:  
  P₁ = (A₁ * B₀) + (A₀ * B₁)

- **P₂**: The third bit is obtained similarly by considering the multiplication of the next set of bits:  
  P₂ = (A₁ * B₁) + (A₀ * B₂) + (A₁ * B₀)

- **P₃**: This bit involves a further combination of products and shifting:  
  P₃ = (A₁ * B₂) + (A₀ * B₃) + (A₁ * B₁)

- **P₄**: The next bits in the higher order are derived by considering the 2-bit parts of A and B, using similar multiplication and shifting logic as above:  
  P₄ = (A₃ * B₁) + (A₂ * B₀)

- **P₅**: Similarly, this bit is obtained from the remaining multiplication of higher bits:  
  P₅ = (A₃ * B₂) + (A₂ * B₁)

- **P₆**: The second highest bit is calculated by:  
  P₆ = (A₃ * B₃) + (A₂ * B₂)

- **P₇**: The highest bit is computed from the multiplication of the most significant bits of both A and B:  
  P₇ = A₃ * B₃

#### 8‑Bit Unsigned Multiplier Based on a 4‑Bit Unsigned Multiplier

The way of implementing an 8-bit unsigned multiplier is similar to implementing a 4-bit
multiplier, and the biggest difference is we need to double the width of adders used.

Let:
- **A** = A₇ A₆ A₅ A₄ A₃ A₂ A₁ A₀ (8‑bit binary number)
- **B** = B₇ B₆ B₅ B₄ B₃ B₂ B₁ B₀ (8‑bit binary number)

We want to compute the product **P = A × B**, which is a 16‑bit number:

![Product Equation](https://latex.codecogs.com/png.latex?P%20=%20A%20%5Ctimes%20B)

In expanded form:

![Expanded Product Bits](https://latex.codecogs.com/png.latex?P%20=%20P_{15}P_{14}%20...%20P_0)

---

##### Decomposition into 4‑Bit Halves

We split each 8‑bit number into high and low 4‑bit parts.

For **A**:

![A Decomposition](https://latex.codecogs.com/png.latex?A%20=%20A_H%20%5Ctimes%202^4%20&plus;%20A_L%2C%20%5Cquad%20A_H%20=%20A_7A_6A_5A_4%2C%20A_L%20=%20A_3A_2A_1A_0)

For **B**:

![B Decomposition](https://latex.codecogs.com/png.latex?B%20=%20B_H%20%5Ctimes%202^4%20&plus;%20B_L%2C%20%5Cquad%20B_H%20=%20B_7B_6B_5B_4%2C%20B_L%20=%20B_3B_2B_1B_0)

Multiplying the decomposed forms gives:

![Expanded Multiplication](https://latex.codecogs.com/png.latex?P%20=%20%28A_H%20%5Ctimes%202^4%20&plus;%20A_L%29%28B_H%20%5Ctimes%202^4%20&plus;%20B_L%29%20=%20A_HB_H%20%5Ctimes%202^8%20&plus;%20%28A_HB_L&plus;A_LB_H%29%20%5Ctimes%202^4%20&plus;%20A_LB_L)

---

##### Using 4‑Bit Multipliers

Assume we have a 4‑bit unsigned multiplier module that multiplies two 4‑bit numbers to yield a 8‑bit result. We compute four partial products:

1. **\( P_0 = A_L \times B_L \)** (no shift)  
   ![P0](https://latex.codecogs.com/png.latex?P_0%20=%20A_L%20%5Ctimes%20B_L)

2. **\( P_1 = A_L \times B_H \)** (shift left by 4 bits)  
   ![P1](https://latex.codecogs.com/png.latex?P_1%20=%20A_L%20%5Ctimes%20B_H)

3. **\( P_2 = A_H \times B_L \)** (shift left by 4 bits)  
   ![P2](https://latex.codecogs.com/png.latex?P_2%20=%20A_H%20%5Ctimes%20B_L)

4. **\( P_3 = A_H \times B_H \)** (shift left by 8 bits)  
   ![P3](https://latex.codecogs.com/png.latex?P_3%20=%20A_H%20%5Ctimes%20B_H)

The final 16‑bit product is obtained by aligning and adding these partial products:

![Final Product Equation](https://latex.codecogs.com/png.latex?P%20=%20P_3%20%5Ctimes%202^8%20&plus;%20%28P_1%20&plus;%20P_2%29%20%5Ctimes%202^4%20&plus;%20P_0)

---

### Signed multiplication 

Different for addition and subtraction, we can not directly use the hardware for unsigned 
multiplication in signed one, therefore, we need to design a signed multiplier.

However, it doesn't mean we need to start from 0. The first step of signed multiplication is to 
convert signed value to unsigned absolute value. Then, the unsigned absolute values of two 
operands could just use the unsigned multiplier, and the sign of the final product is determined
by the sign of MSBs of two operands by an XOR gate. 

Therefore, we first need an absolute value generator:
Based on 2's compliment signed number, the absolute value is equal to:

| Condition              | Two's Complement Representation    | Absolute Value Formula                   | Example (8-bit)                   |
|------------------------|--------------------------------------|------------------------------------------|-----------------------------------|
| Nonnegative (MSB = 0)  | x, with MSB = 0                      | \(|x| = x\)                              | 00101010 (42 in decimal): \(|42| = 42\) |
| Negative (MSB = 1)     | x, with MSB = 1                      | \(|x| = \sim x + 1\) or \(2^n - x\)       | 11111111 (−1 in decimal): \(|-1| = 256 - 255 = 1\) |

And this is the absolute value:


### Complete 8-bit Signed and Unsigned Multiplier
Since the product of signed and unsigned multiplication is different, we must need a controller
that controls whether signed ot unsigned output is desired.

For signed multiplication itself, we also need to select whether the product is positive or
negative. If the product is positive, we can just output it, and if it's negative, we need to
again use 2's compliment to transfer it back to negative number. Here's the hardware implementation:

