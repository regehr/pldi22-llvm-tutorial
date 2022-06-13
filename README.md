# Formally Reasoning About and Generating Optimizations for LLVM IR

John Regehr, Zhengyang Liu, Nuno P. Lopes

## Introduction

- What are we trying to do here?
- Nuno's role, John's role, Zhengyang's role
- parts of this talk

## Equivalence and Inequivalence

- [simple equivalence](https://alive2.llvm.org/ce/z/arJCEv)
  - instructions
  - basic blocks
  - terminators
  - explicit typing
  - implicit naming
- [simple inequivalence](https://alive2.llvm.org/ce/z/gtT2J4)
  - what does "value mismatch" mean?
  - reading counterexamples can be challenging
  - we're working on making them better
- [another simple example](https://alive2.llvm.org/ce/z/ZdEyvR)
- [simple example with overflow disallowed](https://alive2.llvm.org/ce/z/qhL7v9)
- [running specific opt passes](https://alive2.llvm.org/ce/z/4H2oo5)
- exercise: [udiv -> sdiv](https://alive2.llvm.org/ce/z/IsgIcg) and [answer](https://alive2.llvm.org/ce/z/LS6Hty)
- [changing control flow]()
- unrolling loops [C to LLVM](https://gcc.godbolt.org/z/bx3YMeEe8) [verifying LLVM](https://alive2.llvm.org/ce/z/KnE2Su)

## Undefined Behavior and Refinement

- several kinds of UB in LLVM, it's kind of complicated

- undefined behavior is awful in user-facing languages
- however, in an IR it is a useful tool for exposing optimization opportunities
- IR-level UB has little to do with the source language!

- undef stands for "any legal value for the type"
- so an i32 undef can produce any value from 0..2^32-1
- why is undef needed?
  - it's common to have paths where a variable gets no specific value
  - we can materialize any value we want, such as zero, but this has a cost
  - undef lets us not care about what value is there, we basically end up just
    using whatever value happens to be in a register or memory cell
  - there's still a cost, but it's in terms of increasing the complexity of reasoning
    about the compiler!
  - basically all optimizing compilers end up with this concept in one form or another
  - 

- "immediate UB" in LLVM is the same as in C and C++
  - program loses all meaning when you perform this sort of operation
  - it is reserved for things with serious consequences
    - divide by zero, since it traps on many architectures
    - OOB memory operations, since these result in corrupted storage
  - immediate UB [inhibits speculation](https://alive2.llvm.org/ce/z/wyArce)
    - IR-level speculation is pretty important, shows up in a lot of places e.g. LICM

  why is poison needed?
    examples from paper
  drop UB flags
  poison -> undef
  undef -> constant
  2*x ->XX x+x
  undef implies arbitrary subsets
    even/odd examples
  2^2^width undef values, yikes  
  freeze

## Memory

- [C to LLVM](https://gcc.godbolt.org/z/ooP5jEao3) and [LLVM to LLVM](https://alive2.llvm.org/ce/z/emHFcp) and [better LLVM to LLVM](https://alive2.llvm.org/ce/z/9jEVvb)

## Some Bugs

- [list of bugs](https://github.com/AliveToolkit/alive2/blob/master/BugList.md)

## Translation Validation for a Backend

- [blog post with list of bugs](https://blog.regehr.org/archives/2265)

# Minotaur: A superoptimizer focusing on vectorized x86-64 code

- Zhengyang presents!
