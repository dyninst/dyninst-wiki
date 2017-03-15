For our next major release, Dyninst 10.0, we're planning on some more radical changes to our code base than usual; we're going to break backwards compatibility for a nontrivial number of interfaces, and we will be adding significant changes to our dependencies and to our libraries themselves. Below is a preliminary list of some of the Dyninst 10 features in development or discussion. Feedback is strongly encouraged.

# PLATFORM UPDATES

* Make Dyninst feature complete on ARMv8/64
* Make progress on ARM/32 implementation
* Add ppc64 instruction semantics and semantics-based jump table analysis
* Windows binary rewriter?
* Win64 support?
* CUbin support (level and form TBD)
  - Symtab
    * understand appropriate sections as containing sub-Symtabs
    * associate debug info correctly
    * ensure that GPU architectures are handled appropriately
  - ParseAPI
    * build CFGs given either instructionAPI support or an appropriate block-level API for direct CFG construction
  - InstructionAPI
    * architectures TBD

# API CHANGES

* Liberal replacement of raw pointers with shared pointers
* Replace asserts with exceptions or local error handling (everywhere)
* Replace unlikely error return/outparam interfaces with return value/exception?
* Deduplication of data using multi-index containers
* API will use C++11 standard
* Make each Dyninst library self-contained from a linking perspective
* Replace BPatch-level classes that serve as pass-throughs to component classes with direct use of the component classes?

# CROSS-PLATFORM SUPPORT

* Allow Windows builds to analyze ELF files?
* Cross-platform binary rewriting?
* ProcControl over network as a first step to cross-platform dynamic instrumentation?

# MULTITHREADING SUPPORT

* Add option to parallelize various analyses in Dyninst
  - DWARF parsing
  - CFG parsing/construction
  - other?
* Add option to use Cilk for multithreading
* Continue increasing thread-safety of Dyninst components:
  - Fix any outstanding issues in read access to Symtab/ParseAPI data structures
  - Add thread safety to other components for read access?
  - Add thread safety to Symtab/ParseAPI for write access?

# INFRASTRUCTURE

* Migrate from libdwarf to libdw
* Test suite improvements?
  - Integrate as submodule
  - Better use of prolog for efficient coverage
  - Meaningful names on tests
  - ParseAPI test cases
* Logging improvements?
  - Use a standard logging library?
  - Start migrating from printf-style to stream-style?
* Serialization of CFGs to/from graphviz format?
* More rational structure of code generation?
  - Generate all instruction sequences through Emitters
  - Abstract away BaseTramp into allocate/spill/restore, recursion guard, call setup?