This wiki captures a high-level description of what the new ROSE semantics framework is like and what needs to be done to port a new architecture to it.

First, a diagram capturing the high-level structure of the framework:
![](https://github.com/dyninst/dyninst/blob/arm64/feature/semantics/common/docs/rose_structure.png?raw=true)

Details about each part can be found at the beginning of the `dataflowAPI/rose/semantics/BaseSemantics2.h` file. This file also contains the base class definitions for each of the above components. To explain this briefly, we have two main parts in the framework - the dispatcher and the semantics domain. The dispatcher is responsible for actually emulating the execution of each instruction in a given architecture (and thus also has several utility methods required for the emulation of any instruction). The semantic domain dictates what this emulation actually achieves. It has 3 sub-parts. During the emulation of an instruction, it is values of this value type that are modified and stored in the state (either registers or memory). The RISC operators perform the manipulation on instances of the value types: the block of code for the emulation of each instruction in the dispatcher is thus a sequence of calls to the RISC operator functions and dispatcher utility methods (which themselves call RISC operator functions).

The dispatcher holds a reference a RISC operators object and also has access to the architecture's registers through a `RegisterDictionary` (one for each architecture). Instances of each of the above components hold references to instances of the other components, directly or indirectly.
***
`SymEvalSemantics` is the semantic domain used in Dyninst and was defined when ARM semantics were implemented. It's definitions are contained in `SymEvalSemantics.h`. The value type is essentially a wrapper around `Dyninst::AST` that was used in the old semantics. The RISC operators thus do CRUD operations on these ASTs. Both the value type and RISC operators can be used across architectures without modification. The `State`, `RegisterState` and `MemorySate` definitions could change for each architecture, but the structure should be similar to that for ARM. The important thing in `RegisterState` is to implement the `wrap` method which converts ROSE registers to Dyninst registers. Once all of the state types are defined, one can use the framework in the following way:
```
const RegisterDictionary *reg_dict = RegisterDictionary::dictionary_<architecture>();

BaseSemantics::SValuePtr protoval = SymEvalSemantics::SValue::instance(1, 0);
BaseSemantics::RegisterStatePtr registerState = SymEvalSemantics::RegisterState<arch>::instance(protoval, reg_dict);
BaseSemantics::MemoryStatePtr memoryState = SymEvalSemantics::MemoryState<arch>::instance(protoval, protoval);
BaseSemantics::StatePtr state = SymEvalSemantics::State<arch>::instance(res, addr, insn->getArch(), insn, registerState, memoryState);
BaseSemantics::RiscOperatorsPtr ops = SymEvalSemantics::RiscOperators<arch>::instance(state);
```
`<arch>` is to be replaced with the name of the actual architecture (as defined in the headers). Note that although `RiscOperators` is defined with the `<arch>` currently, it can be refactored to be platform-independent, as described above.