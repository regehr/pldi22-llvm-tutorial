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

- undefined behavior is awful in user-facing languages
- however, in an IR it is a useful tool for exposing optimization opportunities
- IR-level UB has little to do with the source language!
- several kinds of UB in LLVM, it's kind of complicated

- "immediate UB" in LLVM is the same as in C and C++
  - program loses all meaning when you perform this sort of operation
  - it is reserved for things with serious consequences
    - divide by zero, since it traps on many architectures
    - OOB memory operations, since these result in corrupted storage
  - immediate UB [inhibits speculation](https://alive2.llvm.org/ce/z/wyArce)
    - IR-level speculation is pretty important, shows up in a lot of places e.g. LICM
  - this motivated the addition of various forms of "bounded undefined behavior" to LLVM IR

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
  - working out a coherent set of rules for undef was hard
    - for example, what happens when you branch on undef?

- [undef can be transformed into an arbitrary value](https://alive2.llvm.org/ce/z/ZVUpXz)
- this means that sequential code ends up having many behaviors
- [undef requires reasoning about arbitrary subsets of values](https://alive2.llvm.org/ce/z/DYNQiv)
- powerset behavior leads to extraordinarily difficult solver queries

- now we're ready to introduce refinement!
- a transformation is a refinement if, forall program configurations, the transformed code has a subset of the original code's behaviors
- when it's a proper subset we call it a non-trivial refinement
- non-trivial refinement is ubiquitous, we have to deal with it
- checking refinement is the primary function of Alive2
- refinement checks in the presence of undef put some stress on the
  solver's quantifier elimination parts
- Z3 still needs a lot of work here, until this happens we get timeouts on some apparently very simple queries

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

- not going into details about this since it's pretty difficult stuff and most of the work was done outside of my group, but we had a PLDI paper a few years ago and Nuno + Juneyoung [have a newer one at CAV '21](https://web.ist.utl.pt/nuno.lopes/pubs.php?id=alive2-mem-cav21)
- but it pretty much just works [C to LLVM](https://gcc.godbolt.org/z/ooP5jEao3) and [LLVM to LLVM](https://alive2.llvm.org/ce/z/emHFcp) and [better LLVM to LLVM](https://alive2.llvm.org/ce/z/9jEVvb)

## Some Bugs

- [list of bugs](https://github.com/AliveToolkit/alive2/blob/master/BugList.md)

## Translation Validation for a Backend

- [blog post with list of bugs](https://blog.regehr.org/archives/2265)

# Minotaur: A superoptimizer focusing on vectorized x86-64 code

- Zhengyang presents!
