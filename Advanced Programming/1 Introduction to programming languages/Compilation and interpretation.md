#advanced_programming #computer_science

[[AP index]] (1)

<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">15/09/2025 & 18/09/2025 & 24/09/2025</span></p>
### Abstract machines

Given a programming language $\mathcal{L}$, an abstract machine $\mathcal{M_{L}}$ is the set of algorithms and data structures that allow the memorization and execution of programs written in $\mathcal{L}$. 
An abstract machine can represent an actual physical machine (e.g. hardware implementation) or even just a mathematical model (e.g. Turing machine).
### Interpreter

The abstract machine is equipped with an interpreter to perform the "fetch and execute" cycle on programs of $\mathcal{L}$.  This interpreter must be able to perform these 3 kinds of operations:
1) Elaboration of primitive data (logical operations, arithmetic operations, ...).
2) Control of execution flow (which instructions must be executed at a given point).
3) Memory management (data transfers, memory allocations, ...).

Given an abstract machine $\mathcal{M_{L}}$ , the language "understood" by the interpreter is called machine language, as it is the language at the lowest level of the machine. For simplicity we consider $\mathcal{L}$ the machine language of $\mathcal{M_{L}}$ .
Generally speaking, a language is called "low level" the closer it is to the machine language, while "high level" language are those that, through abstraction mechanisms, allow the use of constructs independently of the physical characteristics of the machine.

Let us consider a language $\mathcal{L}$ , if we were to implement it in a machine $\mathcal{M_{L_{0}}}$ we would need some kind of "translation" from $\mathcal{L}$ to $\mathcal{L_{0}}$ . This "translation" could be made by using an interpreter for the language $\mathcal{L}$ written in $\mathcal{L_{0}}$ , so it would be a function:
$$\mathcal{I_{L}^{L_{0}}}:(\mathit{Prog}^{\mathcal{L}}\times \mathcal{D})\to\mathcal{D}$$
Where $\mathit{Prog}^{\mathcal{L}}$ is the set of all programs writable in $\mathcal{L}$ and $\mathcal{D}$ is the set of all possible inputs. From a purely mathematical point of view, $\mathcal{I_{L}^{L_{0}}}(\mathit{P}^{\mathcal{L}}, D)=P^{\mathcal{L}}(D)$ for all inputs $D\in\mathcal{D}$.

### Compiler

Another way to encode language $\mathcal{L}$ into $\mathcal{M_{L_{0}}}$ is by compilation, in this case $\mathcal{L}$ would be called *source language*, while $\mathcal{L_{0}}$ *object language*. The compiler would be a function:
$$\mathcal{C_{L,L_{0}}}:\mathit{Prog}^\mathcal{L}\to\mathit{Prog}^\mathcal{L_{0}}$$
So, given a program $P^{L}$, compilation does not execute the program (in fact it does not even take any input), it creates a new program $\mathcal{C_{L,L_{0}}}(P^{\mathcal{L}})=Pc^{L_{0}}$, where, for all inputs $D\in\mathcal{D}$, it holds that $Pc^{L_{0}}(D)=P^{\mathcal{L}}(D)$.

### Compilation vs Interpretation

The main disadvantage of <span style="color:rgb(0, 112, 192)">interpretation</span> is <span style="color:rgb(255, 0, 0)">efficiency</span>, in fact, other than the time used for execution, we have to decode every instruction in real time as the program is being executed, even repeated ones, i.e., the body of a loop is interpreted every time despite remaining unchanged. On the other hand, an interpreter is far more <span style="color:rgb(0, 176, 80)">flexible</span>, as it allows a much more direct interaction with the source program (for example bug finding is easier); furthermore, developing an interpreter is generally easier than developing a compiler.
The advantages, and disadvantages, of <span style="color:rgb(0, 112, 192)">compilation</span> are specular to that of interpretation, it is generally more <span style="color:rgb(0, 176, 80)">efficient</span> to compile a program since it removes the overhead of decoding (and also compilation might perform optimizations to the source code); but the <span style="color:rgb(255, 0, 0)">flexibility</span> is lacking.

There are hybrid cases where both compilation and interpretation is performed, for example in Java where $\texttt{javac}$ compiles the code into $\textit{bytecode}$, and then the JVM basically interprets it and executes it. It is pretty common in practice to have an intermediate language $\mathcal{L_{i}}$ between the two languages so that we have a compiler $\mathcal{C_{L,L_{i}}}$ and an interpreter $\mathcal{I_{L_{i}}^{L_{0}}}$. This is typically done because a pure compilation approach would require certain instructions to be translated with hundreds of lines of code (printf in C), those are left to be interpreted; while, to make the interpreter the most efficient as possible, $\mathcal{L_{i}}$ and $\mathcal{L}$ tend to be not distant as languages.

An important thing to note is also that a "compiled language" or "interpreted language" technically does not exists since it is not a characteristic of the language but rather of its implementation; a language is typically compiled/interpreted, meaning that its most common form of distribution is the one where it is compiled/interpreted.

#### References

- "Programming Languages: Principles and Paradigms" Chapter 1 - Maurizio Gabbrielli, Simone Martini