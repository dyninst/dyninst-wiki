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

Here is a quick reference for the flow of the decoding tables. Decoding can stop in any table, however it is completely dependent on the instruction. Some instructions will stop in the `oneByteMap`, others will continue on into the `twoByteMap`. Decoding also doesn't always start in the `oneByteMap`. Certain prefixes can change the starting table to the `twoByteMap` or one of the three byte maps. 

![Dyninst Decoding Reference](https://raw.githubusercontent.com/dyninst/dyninst/att_syntax/common/docs/decoding_diagram.png)

## One Byte Map (oneByteMap)

This is generally the first table in the decoding process. The row selected
here is just based on the current byte we are looking at in the instruction.
This table contains a lot of the really basic and most common x86
instructions.

## Two Byte Map (twoByteMap)

This table is for two byte instructions. The index for this table is
always the current byte. Decoding can continue into the group map, the
sse map or one of the three byte maps. It can also continue into the
sse/vex multiplexing table if there is an SSE and VEX version of the
same instruction.

## Three Byte Maps (threeByteMap, threeByteMap2)

Just like the two byte map, these tables are for three byte instructions. Unlike the two byte map, decoding from `threeByteMap` can progress into the `sseMapBis` table if it's an SSE instruction or into the `sseMapBisMult` table if the current instruction has both an SSE and VEX version.

`threeByteMap2` is just like `threeByteMap` but instead of progressing into `sseMapBis` when the current instruction is an SSE instruction, decoding can progress into the `sseMapTer` table. It can also progress into the `sseMapTerMult` table if there is an SSE and VEX version of the instruction.

## SSE Maps (sseMap, sseMapBis, sseMapTer)

These tables hold SSE instructions. Each of these tables have multiple entries per row. The correct entry is selected based off of the SSE prefix that was decoded during the first stage of decoding. If there is no SSE prefix present, then entry 0 is normally selected.

## Group Map (groupMap)

This is one of the more complicated tables. Each row in this table has multiple entries. The row that is selected is based off of the previous table the decoder was in. The current byte is broken down as a ModR/M byte. Here is a quick overview of how a ModR/M byte is broken down:

```
+-----+-------+-------+
| 8 7 | 6 5 4 | 3 2 1 |
+-----+-------+-------+
 Mod   Reg     R/M
```

We only care about the `Reg` part of the ModR/M byte. This value is used as the indexer for this table. For example, if you have a byte `0x3F`:

```
Value in Hex: 0x3F
Value in Bin: 0 0 1 1 1 1 1 1
Reg bits:         1 1 1
Reg value: 7
```

Therefore we can see that we should choose entry 7 in this row.

## Group Map 2 (groupMap2)

This table is very similar to `groupMap`. Each row in this table actually has 2 sub rows which contain 8 entries. The row that is selected is based off of the previous table the decoder was in. The current byte is broken down as a ModR/M byte. Here is a quick overview of how a ModR/M byte is broken down:

```
+-----+-------+-------+
| 8 7 | 6 5 4 | 3 2 1 |
+-----+-------+-------+
 Mod   Reg     R/M
```

We care about two things now: the `Mod` and the `Reg`. If the `Mod` is all 1's, then we will use the 2nd sub row. Otherwise we will use the first sub row (sub rows here contain 8 entries each). Then the entry is selected based off of the `Reg` value, just like the `groupMap` table. Here is our example again of using `0x3F` as our ModR/M


```
Value in Hex: 0x3F
Value in Bin: 0 0 1 1 1 1 1 1
Reg bits:         1 1 1
Mod bits:     0 0
Reg value: 7
Mod value: 0
```

Therefore we will use the first sub row, and select the 7th entry in that sub row. If `Mod` would have been all 1's, we would have selected the second sub row. **Note: all values of mod should map to the first sub row except for 3 (0b11)**

## Pre-SSE VEX Multiplexing Table (sseVexMult)

The purpose of this table is to allow our decoder to differentiate between looking at an SSE instruction or a VEX instruction before we enter the SSE tables. For certain instructions, there are SSE and VEX versions of the same instruction. Having a VEX prefix on the instruction instead of an SSE prefix means that we have to make a different decoding decision.

Each row in this table has multiple entries. The entry that is selected is based on the prefix of the instruction. If the instruction has only an SSE prefix, entry 0th is selected. If the instruction has a VEX2 prefix, then the 1st entry is selected. If the instruction has a VEX3 prefix, then the 2nd entry is selected. Finally, if the instruction has an EVEX prefix, the 3rd entry is selected.

## SSE Multiplexer Tables (sseMapMult, sseMapBisMult, sseMapTerMult)

We are allowed to have a VEX instruction that has a decoding path that passes through one of the SSE tables (sseMap, sseMapBis, sseMapTer). However, a VEX instruction cannot end it's decoding in one of the SSE tables. Therefore when the decoder exits one of the SSE tables when it's decoding a VEX instruction, it will enter one of the SSE multiplexer tables in order to get the correct decoding for the VEX prefix that is on the current instruction.

This table has 3 entries per row. The 0th entry specifies the entry that should be used for VEX2 instructions. The 1st entry specifies the entry that should be used for VEX3 instructions. Finallly, the last entry specifies the entry that should be used for EVEX instructions.

## SSE Group Map (sseGroupMap)

This is the table that typically follows after the Group Map table. This table holds the SSE version of the instructions in the Group Map. The format of the names is as follows:

```
+---+----+-----+-----+-----+
| G | XX | SSE | XXX | (X) |
+---+----+-----+-----+-----+
      |           |     +-> If this is a B, then the Mod value was 3
      |           +-------> This is the value of the Reg value in binary
      +-------------------> This is the group number.
```

Each row in this table contains two entries. The first entry should be used if this instruction has no prefix or any prefix except `0x66`. The second entry should be used if the instruction has an `0x66` prefix. 

## VEX W Map (vexWMap)

This table is for complex VEX instructions. Each row contains two entries. The first entry should be selected if the W bit in the VEX prefix is a 0. The second entry should be selected if the W bit in the prefix is a 1.