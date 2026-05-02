# 🔒 Secure 2-Bit Reversible ALU (Tiny Tapeout)

## Project Overview
This project is a hardware-secured, 2-bit Reversible Arithmetic Logic Unit (ALU). 

Traditional classical computing destroys information during logic operations (e.g., an AND gate compresses 2 bits of input into 1 bit of output). According to Landauer's Principle, this erasure of information physically manifests as heat. **Reversible Computing** solves this by utilizing bijective (1-to-1) mapping, ensuring no information is ever erased, paving the way for ultra-low-power and quantum computing architectures.

This chip implements a fully reversible ALU using Toffoli, Feynman, Peres, and Fredkin gates, constrained to an 8-input / 8-output DIP footprint.

## The Reversible Architecture

To maintain strict thermodynamic reversibility and zero information loss, the architecture is divided into four main functional zones:

### 1. Key Validation & Obfuscation (Phases 1 & 2)
The circuit intercepts the user's operational mode instruction with a 2-bit Hardware Security Key. 
*   **Valid Key (10):** Instructions pass through normally.
*   **Invalid Key:** An obfuscation layer intentionally flips the internal control signals. The circuit sabotages the routing tree, forcing the ALU to output the result of a different operation than requested (e.g., performing XOR instead of ADD).

### 2. The Reversible ALU Engine (Phase 3)
Instead of calculating one math operation at a time, the engine calculates ADD, XOR, and SWAP simultaneously.
*   **Peres Gates:** The ripple-carry adder is constructed using Peres gates. The Peres gate is highly optimized for reversible ALUs because it generates both the XOR (Sum) and the AND (Carry) simultaneously, minimizing quantum cost and hardware overhead.

### 3. The Fredkin Multiplexer Tree (Phases 4, 5, & 6)
The math is routed to the output pins using cascading **Fredkin (CSWAP) Gates**. 
*   **Phase 6 (The Final Gatekeeper):** Based on the decoded instructions, this stage routes the selected math out to the Result LEDs. 
*   **The Garbage Collector:** Crucially, the math that was *not* selected is safely shunted to cross-wired "Garbage" output paths. This perfectly simulates the 3-bit CSWAP truth table without destroying a single bit of unselected data.

### 4. Bijection Buffers (Phase 7)
To satisfy the mathematical law of 1-to-1 Bijective Mapping, an 8-input circuit must have exactly 8 outputs.
Since the math result only utilizes 3 LEDs, the remaining 5 LEDs act as **Bijection Buffers**. Using logical AND gates tied to VCC, the circuit acts as a digital signal booster, passing 5 of the raw input states directly to the outputs. This provides a physical receipt to the observer, proving the entire 8-bit universe of input information was perfectly conserved.

---

## 🛠️ Testing & Pinout (Little Endian)

The circuit is purely combinational. Set the input switches (`ui[0-7]`) and observe the output LEDs (`uo[0-7]`). 

### Input Mapping
* **ui[0]:** Input A0 (LSB of A)
* **ui[1]:** Input A1 (MSB of A)
* **ui[2]:** Input B0 (LSB of B)
* **ui[3]:** Input B1 (MSB of B)
* **ui[4]:** Mode Selection bit 0 (M0)
* **ui[5]:** Mode Selection bit 1 (M1)
* **ui[6]:** Security Key bit 0 (K0)
* **ui[7]:** Security Key bit 1 (K1)

### Output Mapping
* **uo[0]:** Result 0 (R0)
* **uo[1]:** Result 1 (R1)
* **uo[2]:** Carry Out (Cout) - *Active during ADD only*
* **uo[3] to uo[7]:** Bijective Garbage (Mirrors M0, M1, A0, B0, B1)

### Mode Selection Table (Requires Key K=10)

| M1 (ui[5]) | M0 (ui[4]) | Operation | Logic Description |
|------------|------------|-----------|-------------------|
| OFF (0)    | OFF (0)    | ADD       | Result = A + B (with Carry) |
| OFF (0)    | ON (1)     | XOR       | Result = A ⊕ B |
| ON (1)     | OFF (0)    | SHIFT      | Result = Swap A (A0 to R1, A1 to R0) |
| ON (1)     | ON (1)     | PASS      | Result = A (Bypasses ALU) |

**Note:** In a constrained 2-bit Reversible architecture, a logical left shift acts functionally as a swap to prevent destroying the MSB, maintaining perfect bijection.

### Full Test Script (Valid Key: K0=OFF, K1=ON)

1. **ADD Test (1+2=3):** Set Switches to `[ON, OFF, OFF, ON, OFF, OFF, OFF, ON]`. 
   - *Expected LEDs:* `uo[0]` & `uo[1]` are ON (Result=3), `uo[2]` is OFF.
2. **XOR Test (3 XOR 1 = 2):** Set Switches to `[ON, ON, ON, OFF, ON, OFF, OFF, ON]`.
   - *Expected LEDs:* `uo[0]` is OFF, `uo[1]` is ON.
3. **SHIFT/SWAP Test (A=1 becomes 2):** Set Switches to `[ON, OFF, OFF, OFF, OFF, ON, OFF, ON]`.
   - *Expected LEDs:* `uo[0]` is OFF, `uo[1]` is ON (A0 moved to R1 position).
4. **PASS Test (A=2 stays 2):** Set Switches to `[OFF, ON, OFF, OFF, ON, ON, OFF, ON]`.
   - *Expected LEDs:* `uo[0]` is OFF, `uo[1]` is ON (Input A passes through).
5. **LOCK Test (Sabotaged ADD):** Set Switches to `[ON, ON, ON, OFF, OFF, OFF, OFF, OFF]` (Key is wrong).
   - *Expected LEDs:* Circuit sabotages ADD and performs XOR instead. `uo[1]` turns ON instead of `uo[2]`.
