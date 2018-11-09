# Dyninst Support for the ARMv8 (64 bit)
Status as of the Dyninst 10.0 release (November 9, 2018)
## Overview
Many of the Dyninst component APIs already support ARMv8 binaries, including InstructionAPI, ParseAPI, PatchAPI, SymtabAPI, DataflowAPI, and StackwalkAPI. While the code generation part of Dyninst is still under development, a subset of dynamic instrumentation functionality is now available. Note that Dyninst currently does not support static binary rewriting on ARMv8.
 
On ARMv8, a user can use Dyninst to perform a variety of tasks, including tracing function calls, monitoring function parameters and return values, analyzing memory accesses, and analyzing program control flow and loops. To better determine whether Dyninst can support a specific task on ARMv8, a user can break down the task into four capability dimensions provided by Dyninst. The four dimensions are the analysis of a binary, specifying instrumentation locations, generating instrumentation, and controlling a running process.  

The binary analysis capabilities have been fully ported to ARMv8. A user can use Dyninst to analyze the control flow and data flow of a program, extract loop structures, and analyze stack variables layout.

The capabilities of specifying instrumentation points have been fully ported to ARMv8. A user can instrument at any program location, including functions, basic blocks, loops, and individual instructions. 

Instrumentation generation is under development and the current focus of the ARMv8 port. Two typical ways of generating instrumentation are (1) using the code snippet data structures provided by Dynisnt, and (2) writing instrumentation in source code, compiling the instrumentation into a stand-alone library, and using Dyninst code snippet to make function calls to the instrumentation library. The support for Dyninst code snippet data structures are in progress. Current supported snippets include making function calls, accessing variables, performing arithmetic operations, and performing control-flow operations such as if-then-else. Note that Dyninst on ARMv8 supports instrumentation that makes function calls, so method (2) for generating instrumentation is supported. Notable unsupported snippets include memory access address snippets (for memory tracing) and the while-loop snippet.

The capabilities of process control are fully supported. A user can stop/resume processes and threads, read/write processes’ memory, and monitor system events delivered to the processes (such as fork, exec, and breakpoints). 

## Technical Details
Dynamic instrumentation requires two key functionalities: code relocation and code generation. Code relocation, not to be confused with the relocation done by the loader, is the basic task done by Dyninst to allow instrumentation and code modification. With it, entire functions are relocated (copied and moved) to a different address space without changing the program behavior. After relocating a function, it can be instrumented by adding code snippets (code generation) to those relocated functions. 

The code relocation is done behind the scenes by Dyninst and the user does not need to worry about it. This functionality is fully implemented for ARMv8.

To instrument functions, the user makes calls to Dyninst to (bpatch class) insert snippets. These snippets, represented by subclasses of BPatch_Snippet, comprising multiple types of expressions such as branch, boolean, arithmetic, variable, and function call. For each of these expressions, Dyninst code generation will emit instructions.

Currently, Dyninst can insert snippets to retrieve variables and local variables references, get variables addresses, dereference pointers, make call to functions with up to eight parameters, perform conditional operations, arithmetic operations, pointer operations, assign constant and variable values, insert conditional branches, reference parameters passed to functions, reference functions return values, and perform sequenced expressions. 

Boolean operations comprise comparisons between two operands, including <, ≤, >, ≥, =, and ≠.

The arithmetic operations implemented are: +, −, ×, ÷, and sign change.. Pointers operations available include dereferencing and address-of.

## The following snippets are supported and will be released in Dyninst 10.0

### BPatch_boolExpr 
(BPatch_relOp op, const BPatch_snippet &lOperand, const BPatch_snippet &rOperand)
Define a relational snippet. The available operators are:

| Operator  | Description |
|-----------| ------------|
|BPatch_lt  | Return lOperand < rOperand |
|BPatch_eq  | Return lOperand == rOperand|
|BPatch_gt  | Return lOperand > rOperand |
|BPatch_le  | Return lOperand <= rOperand|
|BPatch_ne  | Return lOperand != rOperand|
|BPatch_ge  | Return lOperand >= rOperand|
|BPatch_and | Return lOperand && rOperand (Boolean and)|
|BPatch_or  | Return lOperand \|\| rOperand (Boolean or) |


### BPatch_arithExpr 
(BPatch_binOp op, const BPatch_snippet &lOperand, const BPatch_snippet &rOperand)
Perform the required binary operation. The available binary operators are:

| Operator | Description |
|----------|-------------|
| BPatch_assign | assign the value of rOperand to lOperand |
| BPatch_plus   | add lOperand and rOperand |
| BPatch_minus  | subtract rOperand from lOperand |
| BPatch_divide | divide rOperand by lOperand |
| BPatch_times  | multiply rOperand by lOperand |
| BPatch_ref    | Array reference of the form lOperand[rOperand] |
| BPatch_seq    | Define a sequence of two expressions (similar to comma in C) |


### BPatch_arithExpr 
(BPatch_unOp, const BPatch_snippet &operand)
Define a snippet consisting of a unary operator. The unary operators are:

| Operator  | Description |
|-----------|-------------|
|BPatch_negate | Returns the negation of an integer |
|BPatch_addr | Returns a pointer to a BPatch_variableExpr |
|BPatch_deref| Dereferences a pointer |

### BPatch_constExpr 

Define a constant snippet of the primitive data types, including int, unsigned int, long int.

### BPatch_funcCallExpr 
(const BPatch_function& func, const std::vector<BPatch_snippet*> &args)

Define a call to a function. Currently maximum of 8 arguments.

### BPatch_ifExpr 
(const BPatch_boolExpr &conditional, const BPatch_snippet &tClause)

This constructor creates an if statement. The first argument, conditional, should be a Boolean expression that will be evaluated to decide which clause should be executed. The second argument, tClause, is the snippet to execute if the conditional evaluates to true. Else-if statements not implemented.

### BPatch_nullExpr() 

Define a null snippet. This snippet contains no executable statements.

### BPatch_paramExpr
(int paramNum)

This constructor creates an expression whose value is a parameter being passed to a function. ParamNum specifies the number of the parameter to return, starting at 0. Since the contents of parameters may change during subroutine execution, this snippet type is only valid at points that are entries to subroutines, or when inserted at a call point with the when parameter set to BPatch_callBefore.

### BPatch_retExpr() 

This snippet results in an expression that evaluates to the return value of a subroutine. This snippet type is only valid at BPatch_exit points, or at a call point with the when parameter set to BPatch_callAfter.

### BPatch_sequence
(const std::vector<BPatch_snippet*> &items)

Define a sequence of snippets. The passed snippets will be executed in the order in which they appear in items.

### BPatch_variableExpr 
1. (char *in_name, BPatch_addressSpace *in_addSpace, AddressSpace *as, AstNodePtr ast_wrapper_, BPatch_type *type, void* in_address)
2. (BPatch_addressSpace *in_addSpace, AddressSpace *as, void *in_address, int in_register, BPatch_type *type, BPatch_storageClass storage = BPatch_storageAddr, BPatch_point *scp = NULL)
3. (BPatch_addressSpace *in_addSpace, AddressSpace *as, BPatch_localVar *lv, BPatch_type *type, BPatch_point *scp)
4. (BPatch_addressSpace *in_addSpace, AddressSpace *ll_addSpace, int_variable *iv, BPatch_type *type)

Define a variable snippet of the appropriate type. The first constructor is used to get function pointers; the second is used to get forked copies of variable expression, used by malloc; the third is used for local variables; and the last is used by BPatch_addressSpace::findOrCreateVariable().
