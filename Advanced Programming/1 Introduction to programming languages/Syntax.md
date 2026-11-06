#advanced_programming #computer_science

[[AP index]] (1)

<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">25/09/2025</span><span style="color:rgb(0, 176, 80)"> & 29/09/2025</span></p>

### Grammars

To define a programming language we need to define its <span style="color:rgb(255, 192, 0)">syntax</span>, <span style="color:rgb(255, 192, 0)">semantics</span> and <span style="color:rgb(255, 192, 0)">pragmatics</span>.
The <span style="color:rgb(255, 192, 0)">syntax</span> defines well-formed programs, and thus determines which program can be accepted and which are not.
The <span style="color:rgb(255, 192, 0)">semantics</span> gives meaning to well-formed programs, i.e. how their execution should behave.
The <span style="color:rgb(255, 192, 0)">pragmatics</span> is concerned with how the programming language is used in practice.

Given an alphabet $\mathcal{\Sigma}$, a language $\mathcal{L}\subseteq\mathcal{\Sigma^{*}}$ determines which string are allowed and which are not. To define such language it is possible to use a grammar, i.e. a 4-tuple $\mathcal{G}=(N, T, P, S)$, where:
- $N$ is the finite set of non-terminal symbols.
- $T$ is the finite set of terminal symbols.
- $S\in N$ is the staring non-terminal symbol.
- $P$ is a finite set of productions of the form $u\to v$ where $u\in(N\cup T)^{*}N(N\cup T)^{*}$ and $v\in(N\cup T)^{*}$, which defines how derivations can be obtained.

The grammar $\mathcal{G}$ just defined is at its highest freedom since the production can be formed by any kind of grammar string; by restricting its expressivity we define new kinds of grammars:
- **Regular grammars**: $u\in N$ and $v\in T^{*}N$ for right linear, otherwise $v\in NT^{*}$ for left linear.
- **Context-free grammars**: $u\in N$ and $v\in(N\cup T)^{*}$.
- **Context-sensitive grammars**: $u\in(N\cup T)^{*}N(N\cup T)^{*}$ and $v\in(N\cup T)^{*}$ but $|v|\geq|u|$.
- **Unrestricted grammars**: $u\in(N\cup T)^{*}N(N\cup T)^{*}$ and $v\in(N\cup T)^{*}$, no restrictions.

These types of grammars form the Chomsky hierarchy:

![[Chomsky Hierarchy.png]]

### Compilation

Compilation can be divided into 2 main phases, a <span style="color:rgb(0, 255, 42)"></span><span style="color:rgb(0, 255, 42)">front-end</span> one and a <span style="color:rgb(0, 255, 42)"><span style="color:rgb(0, 255, 42)"></span>back-end</span> one. The front-end phase takes the source code and elaborates it through a <span style="color:rgb(0, 176, 240)">scanner</span> to see if all the tokens are written correctly, and then a <span style="color:rgb(0, 176, 240)">parser</span> to see if the tokens follow the rules of the grammar. The semantic analysis ensures that certain semantic properties are enforced in the program, e.g. type safety, and then the intermediate code is generated. The back-end phase performs optimization to the code and, finally, generate the object code.

![[Compilation.svg]]

### Scanner

A program is just a string of characters, but a different number of character form different tokens; so, the scanning phase has as its main goal to transform a sequence of character into a sequence of tokens. A list of tokens is preferable than a list of characters because it simplifies the job of the parser to create the parse tree efficiently, and also improves the portability of the language, for example with non-standard symbols (UTF8).

Usually, the language that defines single tokens is regular, meaning that it can be recognized by a DFA (or a RE). For example, it is not true that any sequence of digits is a number, it must respect the following RE: $0 + (1+...+9)(0+...+9)^{*}$. Another example could be that of the accepted identifiers: $(\_+a+...+z+A+...+Z)^{*}(0+...+9)^{*}$.

So the scanner takes character after character and checks whether the number or the identifier or any other token respects its regular language.

### Parser

Once we achieved a sequence of tokens, we need to check whether they respect the grammar of the entire language, which is typically a context-free grammar. There are algorithms that in $O(n^{3})$ parse the list of tokens into a parse tree; however, for large programs, this becomes extremely inefficient. For this reason, we will focus on 2 different classes of context-free grammars called: LL (Left to right, Leftmost derivation) and LR (Left to right, Rightmost derivation); the first one is used for Top-Down parsing, so the parse tree is built from the root to the leaves, while the second one for Bottom-Up parsing, which builds the parse tree from the leaves to the root.

### Top-down parsing

The simplest Top-Down parses algorithm is <span style="color:rgb(0, 176, 240)">Recursive Descent</span>, which defines a procedure for every non-terminal symbol which tries every possible production and fails if none work. So, the algorithms starts with $S$, i.e. the starting non-terminal symbol, and continuously calls the procedure of the leftmost non-terminal symbol that it finds. The algorithm proceeds like a depth-first enumeration of all possibilities with backtracking if necessary, PROLOG style, and fails if all recursive calls have failed. This algorithm clearly grows exponentially at every recursive call, making it extremely inefficient.

Given the weaknesses of Recursive Descent, one could use <span style="color:rgb(0, 176, 240)">Predictive Parsing</span>, which, instead of blindly trying all the possible procedures of the grammar, it tries to predict which is the right production to take by considering the lookahead tokens of the input.

One of the problems that these kinds of algorithms encounter is <span style="color:rgb(255, 0, 0)">left-recursion</span>, i.e. the possibility of having $A\Rightarrow^{+}A\alpha$, where $A\in N$ is a non-terminal symbol, $\alpha$ a generic grammar string, and $\Rightarrow^{+}$ is the transitive closure of production application. If we find ourselves in that position, it means that the execution might not even end, as to expand the non-terminal $A$ we would need to expand the non-terminal $A$ (infinite recursion).

Another problem of top-down parser is <span style="color:rgb(255, 0, 0)">common prefixes</span>. Suppose that we have the following productions: $C\ \to\ \mathit{if}\ B\ \mathit{then}\ C\ |\ \mathit{if}\ B\ \mathit{then}\ C\ \mathit{else}\ C$, then with just one lookahead token we cannot distinguish the two, we would just see the keyword $\mathit{if}$. 

##### Removing left-recursion

There are procedures that eliminate left-recursion, we consider two cases:
- Direct left-recursion, in this case we have productions of the form: $$A\ \to\ A\alpha_{1}\ |\ \dots\ |\ A\alpha_{n}\ |\ \beta_{1}\ |\ \dots\ |\ \beta_{m}$$ so just one production step is needed to left-recurse. $A\in N$ is a non-terminal symbol, while $\alpha_{1},\dots,\alpha_{n}$ and $\beta_{1},\dots,\beta_{m}$ are just grammar string, with the $\beta_{i}$ not starting with $A$. Then, the following transformation gets applied: $$A\ \to\ \beta_{1}A'\ |\ \dots\ |\ \beta_{m}A'\qquad\qquad A'\ \to\ \alpha_{1}A'\ |\ \dots\ |\ \alpha_{n}A'\ |\ \epsilon$$ so, from $A$ we can only get to strings that do not start with $A$, while the other productions are moved to $A'$ which, being a new symbol, surely does not left-recurse.
- Indirect left-recursion, in this case left-recursion can be obtained with more than one step. Given every non-terminal symbol $A_{1},\dots,A_{n}\in N$, we take $i$ from $1$ to $n$ and perform the following:
  1) For $j$ from $1$ to $i-1$ let us consider all productions of the form $A_{i}\ \to\ A_{j}\gamma$ where $\gamma$ is a grammar string.
  2) Create the productions $A_{i}\ \to\ \delta_{1}\gamma\ |\ \dots\ |\ \delta_{m}\gamma$ where $A_{j}\ \to\ \delta_{1}\ |\ \dots\ |\ \delta_{m}$ 
  Once that is done, we eliminate direct left-recursion as shown before and proceed with $i+1$.

  Since this procedure is difficult to understand, let us start from the first production $A_{1}$ and see what happens. Since there are no $j<1$ we just remove direct left-recursion from $A_{1}$. Now we go to $A_{2}$, there we substitute every starting $A_{1}$ with the productions we obtained before, so that if it was possible to left-recurse on $A_{2}$ from $A_{1}$ now it would be direct and we could eliminate it. Then it proceeds for $A_{3}$ and so on. The procedure works because at $A_{i}$ we are sure that the grammar up to $i-1$ does not left-recurse.

##### Removing common prefixes

To remove common prefixes we use a technique called left factoring. Imagine having a grammar $$A\ \to\ \alpha\beta_{1}\ |\ \dots\ |\ \alpha\beta_{n}\ |\ \gamma_{1}\ |\ \dots\ |\ \gamma_{m}$$
where $\alpha, \beta_{1}, \dots, \beta_{n}, \gamma_{1}, \dots, \gamma_{m}$ are all grammar strings, and the $\gamma_{i}$ do not start with $\alpha$; then, we can transform it into the following grammar: $$A\ \to\ \alpha A'\ |\ \gamma_{1}\ |\ \dots\ |\ \gamma_{m}\qquad\qquad A'\ \to\ \beta_{1}\ |\ \dots\ |\ \beta_{n}$$ This clearly maintains the language expressed by the grammar and also removes the problem of common prefixes, as now knowing the next token allows to know which production to take.

#### LL-grammars

As we said, we will concentrate on a subset of all context-free grammars, that of LL-grammars; in order to define them, we must first define $\mathit{FIRST}$ and $\mathit{FOLLOW}$ sets.
Given a grammar string $\alpha$, we define $\mathit{FIRST}(\alpha)\subseteq T$ to be the set of the firsts terminal tokens generated by $\alpha$, i.e. $\mathit{FIRST}(\alpha)=\{c\in T\ |\ \alpha\Rightarrow^{*}c\beta\ \}$. Instead, given $A\in N$, $\mathit{FOLLOW}(A)\subseteq T$ identifies every terminal token that can follow $A$, i.e. $\mathit{FOLLOW}(A)=\{c\in T\ |\ S\Rightarrow^{+} \alpha Ac\beta\ \}$, so we collect all terminal symbols $c$ that, from the starting symbol $S$, can generate a grammar string where $A$ is followed by $c$.

Given a non left-recursive grammar $\mathcal{G}$, for each collection of productions $A\ \to\ \alpha_{1}\ |\ \dots\ |\ \alpha_{n}$ , we say that it is a LL-grammar when:
- $\forall i,j.\mathit{FIRST}(\alpha_{i})\cap\mathit{FIRST}(a_{j})=\emptyset$ .
- $\alpha_{i}\Rightarrow^{*}\epsilon$ implies that $(\forall j\neq i.\alpha_{j}\not\Rightarrow^{*}\epsilon)$ and $(\forall j\neq i.\mathit{FIRST}(\alpha_{j})\cap\mathit{FOLLOW}(A)=\emptyset)$ .

We can see by these conditions, that LL-grammars are used for predictive parsing, in fact they aim to uniquely identify the production to take when considering a lookahead token. The first condition tells us that the firsts tokens identified by one production have to be disjoint by those identified by another; this allow us to uniquely identify which production to take given our token. The second condition tells us two things: the first is that if a production can end up in $\epsilon$, then no other can, as this allows us to uniquely identify which one to take in the case we want to just eliminate $A$; the second one tells us that if a production can end up in $\epsilon$, then the firsts terminal symbols reached by all other productions must not contain anything that is also in the follow set of $A$, this is because if we were to have a $t\in\mathit{FIRST}(\alpha_{j})$ and $t\in\mathit{FOLLOW}(A)$ then we saw $t$ in the lookahead token we would not know if it is there because it is followed by $A$ (and so we should just eliminate $A$ using the $\epsilon$ production), or if it is generated by $\alpha_{j}$.

These concepts can be better understood with some examples:

![[LL examples.png]] 
1) The first grammar exposed is not LL because it is left-recursive, the predictive parsing algorithm might diverge.
2) The second grammar is not LL because if we encounter an $a$ at $S$ we do not know uniquely which production to take (first condition).
3) The third grammar is not LL because if we finished the tokens at $R$, then we cannot uniquely tell which production to take (second condition, first conjunct).
4) The fourth grammar is not LL because if we encounter an $a$ at $R$ then we do not know if it must be derived by applying $R\ \to\ S$ and then $S\ \to\ \underline{a}Ra$, making the $a$ we saw the one underlined, or it could be the case that before landing in $R$ we performed $S\ \to\ aR\underline{a}$ and so the $a$ we are seeing is the one underlined, making the production to take $R\ \to\ \epsilon$ (second condition, second conjunct).

Once we have established how LL-grammar work, the predictive parsing algorithm is trivial: we create a procedure for every non-terminal symbol where we can uniquely determine what to do based on the lookahead token. Thus, given $A\ \to\ \alpha_{1}\ |\ \dots\ |\ \alpha_{n}$ the procedure for $A$ checks whether the lookahead token is in $\mathit{FIRST}(\alpha_{i})$ for every $i$ until one fires. If none fire, then we check if the lookahead token is in $\mathit{FOLLOW}(A)$, if it is then we take the unique production $A\ \to\ \alpha_{j}$ such that $\alpha_{j}\Rightarrow^{*}\epsilon$ , otherwise, if it is not in the follow set or we do not have such production, we return $\mathtt{fail}$.

#### Table-driven top-down parsing

A variant of predictive descent that does not use recursion is table-driven top-down parsing, which also requires a LL-grammar, but uses a table to memorize the information about which production to uniquely take based on the input of the lookahead token.
Here is an example of how the table is constructed from the $\mathit{FIRST}$ and $\mathit{FOLLOW}$ sets:
![[Table-driven top-down parsing.png]]

Suppose we receive $\mathbf{id}+\mathbf{id}\$$, where $\$$ is the ending token, then starting from $E$, the lookahead token is $\mathbf{id}$ so from the table we realize that the production to take is $E\ \to\ TE_{R}$ . Since the lookahead token has not changed we take the production $T\ \to\ FT_{R}$ and obtain $FT_{R}E_{R}$ . Then $F\ \to\ \mathbf{id}$ so we get $\mathbf{id}\ T_{R}E_{R}$ which changes the lookahead token into $+$ , so from the table we know to use production $T_{R}\ \to\ \epsilon$ and thus obtain $\mathbf{id}\ E_{R}$, and so on....

When we encounter a token that the LL-grammar cannot uniquely identify (so we $\mathtt{fail}$ in the procedure), there are many strategies that we can follow:
- **Panic mode**: the error is reported and we discard every token of the input until we find a synchronizing token (like { or ; ) that allows us to restart the parsing phase.
- **Phrase-level recovery**: pretend that the error token is not there or substitute it with a meaningful one.
- **Error productions**: Augment the grammar with productions specifically for handling common errors. 

Panic mode is the most common, since you just need to prepare error messages for it. Phrase-level recovery can be very good, but sometimes the recovery may be not intuitive.

#### References

- Lecture Notes 25/09/2025 - Andrea Corradini
- "Programming Language Pragmatics" Chapter 2 - Michael L. Scott