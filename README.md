# Formally Reasoning About and Generating Optimizations for LLVM IR

John Regehr, Zhengyang Liu, Nuno P. Lopes

## Introduction

- What are we trying to do here?
  - several projects in different stages
  - making Alive2 as complete and definitive as possible
  - making the LLVM implementation as formally grounded as possible [example of ongoing discussion](https://github.com/llvm/llvm-project/issues/49839)
  - making it unnecessary to create optimizers by hand
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
- unrolling loops [C to LLVM](https://gcc.godbolt.org/z/573Mr3d7e) [verifying LLVM](https://alive2.llvm.org/ce/z/4QTCQ_)

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
  - this motivated the addition of various forms of "deferred undefined behavior" to LLVM IR

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
  - even with a coherent set of rules, compiler devs have a very hard time with undef

- [undef can be transformed into an arbitrary value](https://alive2.llvm.org/ce/z/ZVUpXz)
- this means that sequential code ends up having many behaviors
- [undef requires reasoning about arbitrary subsets of values](https://alive2.llvm.org/ce/z/DYNQiv)
- powerset behavior leads to extraordinarily difficult solver queries
- [each SSA use of undef might yield a different value](https://alive2.llvm.org/ce/z/GLtJgB)
  - an optimization can reduce the number of uses of a value that might be undef
  - an optimization can keep the number of uses of a maybe-undef value the same
  - an optimization can almost never increase the number of uses of a maybe-undef value

- now we're ready to introduce refinement!
- a transformation is a refinement if, forall program configurations, the transformed code has a subset of the original code's behaviors
- when it's a proper subset we call it a non-trivial refinement
- non-trivial refinement is ubiquitous, we have to deal with it
- checking refinement is the primary function of Alive2
- refinement checks in the presence of undef put some stress on the
  solver's quantifier elimination parts
- Z3 still needs a lot of work here, until this happens we get timeouts on some apparently very simple queries

- we've seen undef and immediate undefined behavior
- these two concepts ended up being insufficient to justify all of the
  optimizations that the LLVM developers wanted to write, so one more
  kind of undefined behavior was added
- in C and C++, a + b > a and b > 0 always have the same result, because signed overflow is undefined
- [the analogous optimization in LLVM didn't work](https://alive2.llvm.org/ce/z/wPtEnP)
  - this can't be fixed by making signed overflow return undef
  - it can be fixed by immediate UB on signed overflow, but that creates too many other problems
- solution is a "poison" value that is outside the normal space of values
- for most operations, "op x, poison" [evaluates to poison](https://alive2.llvm.org/ce/z/8NFkSZ)
- this solves this problem, but poison and undef are a persistent source of problems for compiler developers, who have an easy time forgetting about one or the other of them
- [poison refines to undef but not the other way around](https://alive2.llvm.org/ce/z/3-S3NT)

- we proposed a "freeze" instruction that basically makes both undef and poison pick a stable value
- we got the LLVM community to accept it
- we implemented it (this was mainly Juneyoung Lee's work)
- we're trying to get rid of undef entirely, but this is a long and difficult project
  - many real-world engineering constraints due to LLVM's wide adoption

## Memory

- not going into details about this since it's pretty difficult stuff and most of the work was done outside of my group, but we had a PLDI paper a few years ago and Nuno + Juneyoung [have a newer one at CAV '21](https://web.ist.utl.pt/nuno.lopes/pubs.php?id=alive2-mem-cav21)
- but it pretty much just works [C to LLVM](https://gcc.godbolt.org/z/ooP5jEao3) and [LLVM to LLVM](https://alive2.llvm.org/ce/z/emHFcp) and [better LLVM to LLVM](https://alive2.llvm.org/ce/z/9jEVvb)

## Some Bugs

- [list of bugs](https://github.com/AliveToolkit/alive2/blob/master/BugList.md)

## Translation Validation for the AArch64 backend

- [blog post](https://blog.regehr.org/archives/2265)

## A Current Project: Continuous Fuzzing for LLVM

- [we wrote a mutation engine for LLVM IR](https://blog.regehr.org/archives/2148)
- we want to take a strong machine and have it...
  - build a new LLVM each day
  - spend the rest of the day performing mutation-based fuzzing
  - test the middle-end optimizer and also some backends
  - I wrote a cache to avoid redundant queries
  - we'll reduce bug triggers using llvm-reduce
- the current roadblock is triaging the outputs of this process  

# Minotaur: A superoptimizer focusing on vectorized x86-64 code

- Zhengyang presents!
