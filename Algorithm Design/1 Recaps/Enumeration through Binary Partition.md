#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">21/04/2026 & 22/04/2026</span></p>

[[AD index]] (1)

## Graph enumeration

Given a graph $G=(V,E)$, graph enumeration consists in generating all the sub-graphs of $G$ that satisfy a certain property. Let us consider that $G$ has $n$ nodes, and we have $N$ sub-graphs that result from the generation, then we say that the output is polynomial if it is polynomial in $n$ and $N$, e.g. $O(n^3N)$. We also consider another factor, the <span style="color:rgb(0, 176, 80)">delay</span>, which is the time in the worst case between outputting a solution and outputting the next.

One of the immediate solutions is to use binary partition, so we take a node $a$ and recursively generate all solutions that contain $a\in V$ and that does not. Here is the pseudocode:
![[Enumeration pseudocode.png]]
We iterate on all the nodes, and we recurse by adding each element; after all solutions containing that element have been outputted, all solutions without it are considered.

We would like to stop the enumeration if we know that we are in a recursion tree branch that will not lead to any new solutions. This is called the <span style="color:rgb(0, 176, 80)">extension problem</span>, which, unfortunately, is NP-complete for most maximal solution problems.

#### Maximal cliques

A maximal clique is a clique that cannot be expanded with any more nodes. This means that a clique $C\subseteq V$ is maximal if $\nexists a\in V$ such that $C\cup\{a\}$ is a clique.
We can solve the problem of finding all maximal cliques in a graph by using Bron-Kerbosch (BK) algorithm, which essentially enumerates all possible solutions but with a little twist. It takes 3 parameters: $R$, $P$, and $X$; $R$ are elements that MUST be included, $P$ are elements that CAN be included, while $X$ are elements that CANNOT be included. The algorithm then proceeds as follows:
```pseudo
	\begin{algorithm}
		\caption{Bron-Kerbosch algorithm}
		\begin{algorithmic}
		\State \textbf{global: }$G=(V,E)$
			\Function{BK}{$R, P, X$}
				\If{$P=\emptyset \land X=\emptyset$}
					\State \textbf{output: }$R$
                \EndIf
                \ForAll{$v\in P$}
	                \State \Call{BK}{$R\cup\{v\}, P\cap N(v), X\cap N(v)$}
	                \State $P\gets P\backslash\{v\}$
	                \State $X\gets X\cup\{v\}$
                \EndFor
			\EndFunction
			\State \Call{BK}{$\emptyset, V, \emptyset$} //first call
		\end{algorithmic}
	\end{algorithm}
```
So, first we check whether $R$ is a maximal clique, we do so by checking if there are not any more nodes to add ($P$ is empty) but also there are not any more node to exclude ($X$ is empty), this last condition ensures that all cliques found are maximal (if a node kept appearing as to be excluded, it would mean that it is in the neighborhood of every selected node, and thus it does not form a maximal clique).

This algorithm is still $O(2^n)$, and also the recursive tree has many branches where the leaves have no solution (many branches where $P$ is empty but $X$ is not). In order to have a better time, we could use a lemma:
For any $v\in V$, each maximal clique $C$ either contains $v$ or a non-neighbor of $v$.
This lemma is actually pretty simple to prove, if a clique contained only nodes that are not $v$ and are not non-neighbors of $v$, then it would mean that it only contains neighbors of $v$, which would mean that it is not maximal since $v$ would be a valid candidate for the clique.

We can use this fact when iterating the nodes in $P$, this is because we choose a random node $u\in P$  to be the pivot and can actually iterate on the set $P\backslash N(u)$ since we will find eventually a node to add to the clique (if it was not already maximal). This allows us to perform less recursive calls, and actually Tomita et al. algorithm does this by selecting as pivot the node $u$ that maximizes $P\cap N(u)$, meaning that it will minimize $P\backslash N(u)$ and thus will perform less recursive calls. The aforementioned algorithm runs in $O(3^{\frac{n}{3}})=O(1.442^n)$, which is better than BK.

#### Branch and reduce

Branch and reduce is a technique used to enumerate solutions that consists in using rules to reduce the instance of the problem into an equivalent but smaller one. These rules are applied until a trivial case is hit, or not anymore rules can be applied and then, instead of applying brute-force like in kernelization, branching is applied, where the algorithm branches into sub-problems each corresponding to having taken a different decision. This is not to be confused with brute-force, the decision we take for branching are still dictated by forced logical choice, for example by avoiding impossible configuration or recomputing solutions.