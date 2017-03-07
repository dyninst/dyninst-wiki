For our next major release, Dyninst 10.0, we're planning on some more radical changes to our code base than usual; we're going to break backwards compatibility for a nontrivial number of interfaces, and we will be adding significant changes to our dependencies and to our libraries themselves. Below is a preliminary list of some of the Dyninst 10 features in development or discussion. Feedback is strongly encouraged.

# PLATFORM UPDATES

* Make Dyninst feature complete on ARMv8/64
* Make progress on ARM/32 implementation
* Add ppc64 instruction semantics
* Windows binary rewriter?
* Win64 support?

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
* Continue increasing thread-safety of Dyninst components?

# INFRASTRUCTURE

* Migrate from libdwarf to libdw
* Test suite improvements?
* Migrate to a standard logging library and streams in preference to printf-style?
