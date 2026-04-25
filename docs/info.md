<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->

## How it works

The 2-Bit Logic Locked ALU is a processor core that was builty purely by using reversibe logic gates such as Toffoli, Fredkin and Peres gates. This design is strictly to 1-to-1 or bijective, ensuring that essentially no information is lost or destroyed during computation. The entire architecture can be organized into 5 main phases:

1. Authentication: First, a Toffoli-gate network validates a 2-bit security key
2. Decoding: Then, a decoder identifies the requested instruction(ADD, XOR, SHIFT, PASS)
3. Arithmetic: A network of Peres gates is used to simultaneously calculate the 2-bit addition(with the carry) as well as bitwise XOR and bit-rotation(SHIFT).
4. Decision Making (Fredkin Routing Tree): A three stage tree of Fredkin gates (Controlled Swap) acts as a decision maker. It is used to route the desired mathematical results to the output pins while also preserving the original operands and mode to ensure reversibility.
5. Hardware Obfuscation Lock: Based on the output of the first step, the ouput is protected by a final Fredkin lock stage. If provided an incorrect key, the circuit swaps the result pins R0 and R1(rather than simple inversion to reduce reverse engineering attacks) and inverts the carry bit. This will allow the circuit to appear functional while providing incorrect outputs to unauthorized users.

## How to test
(Keep in mind that the switches will follow Little Endian format i.e. for any sequence of two bits, the Least Significant Bit (LSB) comes first followed by the Most Significant Bit (MSB) (or just generally the binary must be read from right to left/down to up for a set of bits). For example the Security Key is 10 in binary, thus Switch 7 would be OFF(0) and Switch 8 would be ON(1). Hence 01 ------> 


The project can be tested using an 8-pin switch for input and a 7-segment display for output. Once provided the correct key (Switch 7 OFF, Switch 8 ON), the output on display acts as a unique 'fingerprint' of the input, resulting in a reversible design.
1. Set the Security Key
   Switch 7: OFF, Switch 8: ON (Binary: 10)
If the key is incorrect, the results will be scrambled

3. Select Operation (Switches 5,6)



