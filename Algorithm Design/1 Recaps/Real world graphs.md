#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">05/05/2026</span></p>

[[AD index]] (1)

Real world graphs can be really big, up to billions of links, but they are not just random, everything is not described by $n$. For this reason, we also define:
- $\Delta$ = maximum degree
- $d$ = degeneracy
- $t$ = trussness

##### Degeneracy

Degeneracy is the minimum $d$ such that there exists an ordering of the nodes $\{v_1,\dots,v_n\}$ where every node has a degree $\leq d$ in the residual graph (i.e. the nodes after itself). The ordering that causes the minimal degeneracy is called degeneracy ordering.

Generally, in real world graphs, $n\gg\Delta\gg d$, which means that if we are able to bound an algorithm by $\Delta$ or $d$ we would get a great speed up in efficiency.

Degeneracy and many other measures can be used to calculate cliques efficiently. Let us consider again the pseudocode for the reverse search algorithm for maximal cliques:
```pseudo
	\begin{algorithm}
		\caption{Enumerating maximal cliques using reverse search}
		\begin{algorithmic}
			\Function{Enum}{$P$}
				\ForAll{$v\in V\backslash P$}
					\State $S\leftarrow\ $\Call{complete}{$(P_{\leq v}\cap N(v))\cup\{v\}$}
	                \If{$P=\ $\Call{parent}{$S$}$\ \land\ v=P.index(S)$}
			            \State \Call{Enum}{S}
	                \EndIf
                \EndFor
			\EndFunction
		\end{algorithmic}
	\end{algorithm}
```
This algorithm had a cost of $O(N\cdot n^3\lg(n))$. Let us consider the degeneracy order as the order chosen for the nodes. When choosing the node $v\in V\backslash P$, the choice actually makes sense if it is neighboring $v$, otherwise we start the $\mathtt{complete}$ with just $v$. For the same reason as before, $v$ must be a forward neighbor in the order for some node of $P$, otherwise we would still perform the $\mathtt{complete}$ on just $v$. This means that the $v$ I am actually searching is a forward neighbor of some node in $P$, meaning that we have $O(|P|\cdot d)$ possibilities, after all the degeneracy $d$ indicates the maximal number of forward neighbors a node can have. By calling $q$ the maximal clique size we actually get a bound of $O(q\cdot d)$.
The $\mathtt{complete}$ had a complexity of $O(n^2)$ because every node had at most $n$ neighbors and we could add at most $n$ nodes. However, we can actually see that a node has at most $\Delta$ neighbors and the maximum number of nodes that can be added is actually $q$, meaning that $\mathtt{complete}$ has a cost of $O(q\cdot \Delta)$.
As before, the $\mathtt{parent}$ check costs $O(\lg(n))$, giving us a cost per solution of $O(q^2\cdot\Delta\cdot d\cdot\lg(n))$, for a total cost of $O(N\cdot q^2\cdot\Delta\cdot d\cdot\lg(n))$.

An alternative to cliques are $k$-plex, i.e. cliques where $k-1$ edges can be missing. For instance, any $1$-plex is a clique. $k$-plex portray better real noisy data, where cliques can be found with difficulties.

##### Trussness

Trussness is another measure on graphs similar to degeneracy. Trussness is the minimum $t$ such that there exists an ordering of the edges $\{e_1,\dots,e_m\}$ where every edge forms at most $t$ triangles with the residual graph (i.e. the edges after itself).