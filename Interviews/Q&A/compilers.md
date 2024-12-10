# Compilers-Related Questions

## Heterogeneous Compilation

### How to prevent optimizations
We have a couple of options. 
- Write the code in ASM and mark it as voaltile to prevent code being optimized
- Mark any variables that should not be optimized away with `volatile`
- Add memory fences to prevent reordered memory access across the fence

## SSA
### Explain SSA

SSA, i.e. Satic Single Assignment, is one form of IR, used in Compilers.
The 2 main key characters of SSA is: 1. Any bindings can only be create once, no rebinding is allowed; 2. When there's data being merge on the control flow, use PHI node to indicate where does the data come from.

- Some relatively new compiler libraries like cranelift that don't have PHI nodes but instead use block parameters.

### Explain SSA Construction Algorithm
#### Basics
##### Dominator
- $N$ *dominates* $M$ iff any path from entry to $M$ on the CFG will pass by $N$, wrote $N \text{ dom } M$.
- $N$ *strictly dominates* $M$ iff $N \text{ dom } M \land N \neq M$, wrote $N \text{ sdom } M$
- $N$ *immediate dominates* $M$ iff $N \text{ dom } M \land \not\exists N' s.t. N \text{ dom } N' \land N' \text{ dom } M$, wrote $N \text{ idom } M$
##### Dominator Tree
A tree where $A$ is $B$'s parent iff $A\text{ idom } B$
##### Dominance Frontier
Intuitively, A's *Dominance Fronter* includes B if A can reach B but not all path from entry to B passes by A.
###### The *Dominance Frontier* of $N$ is a set
$$
DF(N) = \{ W | \exists V,\text{ a predecessor of }W, s.t. N \text{ dom } V \land N \cancel{\text{ sdom }} W\}
$$

##### Algorithm for Computing Dominance Frontier

![](
https://llvm-study-notes.readthedocs.io/en/latest/_images/1563330393644.png
)

##### Iterated Dominance Frontier

We extend Dominance Frontier to a set:

$$
DF(S) = \bigcup_{n \in S} DF(n)
$$

Iterated Dominace Frontier $DF^+(S)$ is the fixpoint of following equation: 

$$
DF^+(S) = DF(S \cup DF^+(S))
$$

#### Algorithm Phase I: Placing $\Phi$
![](https://llvm-study-notes.readthedocs.io/en/latest/_images/1563330823497.png)

#### Algorithm Phase II: Rename all variables
![](https://llvm-study-notes.readthedocs.io/en/latest/_images/1563330955034.png)

## TODO: 
https://sites.cs.ucsb.edu/~yufeiding/cs293s/slides/293S_06_SSA.pdf

## LLVM
### Explain the GEP instruction in LLVM
GEP, i.e. Get Element Pointer, is one instruction in under the LLVM IR Specification. It's used to retrive the pointer to the data inside a struct of nested C-arrays and C-structs. Here's its syntax:
```llvm
%ptr = getelementptr [type], [base pointer], [list of indices]
```
- This instruction doesn't access the underlying memory.
- Can be used to calculate the size of a struct type in a machine independent way.

Recommended Readings:
- https://llvm.org/docs/GetElementPtr.html
- https://mukulrathi.com/create-your-own-programming-language/llvm-ir-cpp-api-tutorial/#geps
### Explain the backend architecture of LLVM
#### Type Legalization
Purpose: replace any types unsupported by the target architecture with supported equivalence
##### Split
dividing large types into multiple supported small ones
##### Promote
replacing a small types into supported larger ones, pad if needed
##### Expand
rewrite unsupported type into an equivalence
- e.g. replace vector types with multiple scalars
#### Operation Legalization
Purpose: replace any instructions unsupported by the target architecture with supported equivalence
- e.g. replace floating point operations with software equivalence library calls on architecture that doesn't support floating point operations
#### Instruction Selection
The backend matches LLVM IR instructions to target-specific instructions. This is done using pattern-matching algorithms in frameworks like SelectionDAG or GlobalISel.
#### Instruction Scheduling
Instructions are reordered to minimize pipeline stalls, respect data dependencies, and optimize performance.
Causes for pipeline stalls: Data Hazards, Control Hazards, Structural Hazards, Cache Misses
The reason this is done before register allocation is that reordering of instructions affects register allocation.
#### Register Allocation
Maps virtual registers to physical registers while handling constraints like limited register availability.
##### Register allocation algorithms
- Linear scan
- Graph coloring
#### Peephole Optimization (Target Specific)
Performs small, localized optimizations on the generated instructions
#### Code Emission
The final machine instructions are serialized into an output format
- Assembly: Human-readable format.
- Object Files: Binary-encoded format for linking and execution.
#### Linking
The emitted object files can be linked together to produce an executable.
