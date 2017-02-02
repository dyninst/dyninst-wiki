# Instruction Descriptions
The descriptions for the instructions in the Intel instruction set are defined
in a struct in arch-x86.h. The struct can be summarized as follows:

 1. **id (Entry ID)**
  - This is used to identify the instruction. Multiple rows can have the same ID if they have the same instruction mnemonic. This ID should map with an entry in the `dyn_hash_map which` stores the actual string for the instruction
mnemonics.
 2. **otable (Next Opcode Table)**
  - This is used during the decoding process. It the identifier for the next table that should be used in the decoding process. This is explained more in `ia32_decode_opcode`.
 3. **tabidx (Opcode Table Index)**
  - This is also used during the decoding process and specifies the index for the
next table that should be used. Depending on the table, this value is ignored
and other logic is used to make the decision as to which table index should
be used next.
 4. **hasModRM (Whether or not this instruction has a ModR/M byte)**
  - This designates whether or not the instruction being described has a ModR/M byte or not. NOTE: This MUST be set to true for instructions that have operands that use the ModR/M byte even if you specify it has operands that use the
ModR/M byte.
 5. **operands\[3\] (Instruction Operands)**
  - This is an array of descriptors for the first 3 operands. Please look at the Intel manual to see which addressing modes and operand sizes are available. We follow the same format as the Intel manual except for a couple of very
rare cases.
 6. **legacyType (Legacy information)**
  - This is generally used for dataflow analysis and other semantic information. You shouldn't have to mess with this assuming that intel doesn't add any new instructions that are capable of changing the RIP/EIP like jumps, calls, ret, ect.
 7. **opsema (Operand Read/Write Semantics)**
  - This describes which operands and read and written. Search arch-x86.h for 'operand semantic' and you should be able to find the options for this.
 8. **impl_dec (Implicit Operand Description)**
  - This is a mask that should be used to mark implicit operands. If an operand should not be printed in AT&T syntax, then you should mask it here.

# The Decoding Process

The main decoding function is `ia32_decode`, which calls a bunch of helper functions
that are for the most part extremely well commented. The overall flow of the
decoding process is like this:

## 1. Decode any prefixes

This step is really easy. When we first look at an instruction, we have
to decode which bytes are prefix bytes. Usually, there is only one prefix,
however some instructions can have 2 or more prefixes. All VEX instructions
can only have one prefix: The VEX2, VEX3 or EVEX prefix.

## 2. Decode the opcode and determine the instruction description

In this step we start at some initial decoding table. The starting table
is determined by which prefixes are present. If there are no prefixes present,
we start in the `oneByteMap`. We can go through several tables in the decoding
process and the logic for each table is a bit different.

## 3. Decode operands

In this step we look at the description we got from step 2. The operand
descriptions in the `ia32_entry` entry are used to determine what to look for
and how long the final instruction length should be.
The main objective of the decoding process is to get the length of the instruction
correct. Because x86 has variable length instructions, getting the instruction
length wrong will mess up the decoding for the instructions that follow.

# Decoding Tables

## One Byte Map (oneByteMap)

## Two Byte Map (twoByteMap)

## Three Byte Maps (threeByteMap, threeByteMap2)

## SSE Maps (sseMap, sseMapBis, sseMapTer)

## Group Map (groupMap)

## SSE Group Map (sseGroupMap)

## VEX W Map (vexWMap)