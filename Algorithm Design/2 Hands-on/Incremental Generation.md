#computer_science #algorithm_design 
<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">24/04/2026</span></p>

**Author:** Claudio Bernardoni

[[AD index]] (2)

**Hands-on:** 
![[Hands-on 24-04.png]]

**Point 1:**
Starting from the previous solutions $\mathit{SOL}_{i-1}$, we can find the maximal cliques that include vertex $v_i$.
The algorithm starts by iterating on all the previous solutions; then, it can be divided into two parts: first, it iterates on all the nodes of the previous maximal clique $C$, putting all the elements connected to $v_i$ into a set $S$. Then, based on the resulting $S$, we identify 3 cases:
1) $S$ has the same cardinality as $C$, which means that $v_i$ is connected to all the nodes of the clique $C$. Thus there exist a node $v_i$ for which $C$ alone is not a maximal clique anymore, for this reason, to the new solutions $\mathit{SOL}_i$ we add the clique composed by $C$ and $v_i$, i.e. $\{C\cup v_i\}$.
2) $S$ is not connected with any nodes of $C$. Then, the clique $C$ remains maximal and thus is added as it is to $\mathit{SOL}_i$. In this case, a counter $c$ is also incremented, this is because in the end it will be useful to see how many times the node $v_i$ was not connected to anything.
3) $S$ is connected to some nodes in $C$ but not all of them, i.e. $S\subset C$. This means that the previous clique $C$ is still maximal; however, even $\{S\cup\{v_i\}\}$ is a clique. Before adding it, we check whether it is maximal, this is because it could be part of a bigger clique, and thus we need to check if there is a node in $G_i$ that makes the clique not maximal.
Once all cliques in $\mathit{SOL}_{i-1}$ have been checked, we verify whether the counter $c$ is equal to the cardinality of $\mathit{SOL}_{i-1}$, i.e. whether $v_i$ was connected with any nodes in $G_{i-1}$. If it is, then $v_i$ is an isolated node in $G_i$, which means that it is its own clique, and thus $\{v_i\}$ is added to the solution.
This is the just described algorithm:
```pseudo
	\begin{algorithm}
		\caption{Compute maximal cliques of $G_i$ starting from $\mathit{SOL}_{i-1}$}
		\begin{algorithmic}
			\State \textbf{global: } $G(V,E)$
			\Function{MaximalClique}{$\mathit{SOL}_{i-1}, v_i$}
				\State $c\leftarrow0$
				\ForAll{$C\in\mathit{SOL}_{i-1}$}
					\State $S\leftarrow\emptyset$
					\ForAll{$u_j\in C$}
						\If{$E[j][i]=1$}
							\State $S\leftarrow S\cup u_j$
                        \EndIf
	                \EndFor
	                \If{$|S|=|C|$}
		                \State $\mathit{SOL}_i\leftarrow\mathit{SOL}_i\cup\{C\cup\{v_i\}\}$
		            \Elif{$|S|=0$}
			            \State $\mathit{SOL}_i\leftarrow\mathit{SOL}_i\cup\{C\}$
	                    \State $c\leftarrow c+1$
                    \Else
	                    \If{\Call{Maximal}{$S, C, i$}}
		                    \State $\mathit{SOL}_i\leftarrow\mathit{SOL}_i\cup\{S\cup\{v_i\}\}$
		                \EndIf
		                \State $\mathit{SOL}_i\leftarrow\mathit{SOL}_i\cup\{C\}$
                    \EndIf
                \EndFor
                \If{$c=|\mathit{SOL}_{i-1}|$}
	                \State $\mathit{SOL}_i\leftarrow\mathit{SOL}_i\cup\{v_i\}$
                \EndIf
                \Return $\mathit{SOL}_i$
			\EndFunction
		\end{algorithmic}
	\end{algorithm}
```

Checking if a clique is maximal can be done as follows:
```pseudo
	\begin{algorithm}
		\caption{Check if a clique is maximal in the graph}
		\begin{algorithmic}
			\Function{Maximal}{$S, C, i$}
			\State $\mathit{T}\leftarrow\{v_1,\dots,v_{i-1}\}\backslash C$
				\ForAll{$u_j\in\mathit{T}$}
					\State $\mathit{q}\leftarrow0$
					\ForAll{$w_k\in S$}
						\If{$E[j][k]=0$}
							\Break
                        \EndIf
                        \State $\mathit{q}\leftarrow\mathit{q}+1$
                    \EndFor
                    \If{$q=|S|$}
	                    \Return $\mathit{false}$
                    \EndIf
                \EndFor
                \Return $\mathit{true}$
			\EndFunction
		\end{algorithmic}
	\end{algorithm}
```
Basically, we search among all the nodes in $G_{i-1}$, minus those already checked $C$, if there is a node that is connected to all of those in $S$. If such node exists, it means that the clique $S$ is not maximal and should not be added to the solutions $\mathit{SOL}_i$, and thus $\mathit{false}$ is returned. Instead, if such node is not found, $\mathit{true}$ is returned and the click $S$ is added to the solutions.

**Point 2:**
Let us consider first the cost of the $\mathtt{MaximalClique}$ algorithm at a generic iteration $i$. First, we iterate on all the cliques in $\mathit{SOL}_{i-1}$, which are $N_{i-1}$. Then the algorithm is divided into two phases: in the first we check the connections of the nodes in the clique taken into considerations with $v_i$, creating the set $S$. The size of the clique is at most $n$, which means that this procedure has cost $O(n)$. In the second phase, based on the constructed set $S$, we have 3 cases:
1) $|S|=|C|$, the clique added is $C$ with $v_i$. This has cost $O(1)$.
2) $|S|=0$, the clique $C$ gets added. This has cost $O(1)$.
3) $S\subset C$, we have to check if $S$ is maximal and then decide to add just $C$ or even the clique $S\cup\{v_i\}$; these last step only have cost $O(1)$. Checking if a clique is maximal takes all the $i\leq n$ nodes and has to check if they are all connected to the nodes of $S$. Since $|S|\leq n$, the $\mathtt{Maximal}$ procedure has cost $O(n^2)$, so the total cost of this case is $O(n^2)+O(1)=O(n^2)$.
In the worst case, we always end up in the third case, meaning that the second phase has cost $O(n)+O(n^2)=O(n^2)$. The total cost of the procedure is then $O(N_{i-1}\cdot n^2)$.

Now, the procedure $\mathtt{MaximalClique}$ is called from $i=1$ to $n$, which means that the total cost of the original algorithm is: $O\left(\sum\limits_{i=1}^{n}N_{i-1}\cdot n^2\right)$.

We can see that at every iteration $|\mathit{SOL}_i|\geq|\mathit{SOL}_{i-1}|$, in fact we can see this by checking every case of the algorithm:
1) In the first case, a clique in $\mathit{SOL}_{i-1}$ is transformed into one of $\mathit{SOL}_{i}$, so the number of cliques remains the same.
2) In the second case, the same happens, so the number of cliques remains the same.
3) In the third case, we either add 1 or 2 cliques, depending if $S$ is maximal, so the number of cliques in $\mathit{SOL}_{i}$ either increases or remains the same.

This means that $\forall i\in\{1,\dots,n\}.N\geq N_{i}$, and thus we get that:$$\sum\limits_{i=1}^{n}N_{i-1}\cdot n^2\leq \sum\limits_{i=1}^{n}N\cdot n^2=N\cdot n^3$$Therefore, we can say that the cost of the algorithm is $O(N\cdot n^3)$.

**Point 3:**
Since we output the result with all the cliques just at the end, the delay is terrible, since it is the entire cost of the algorithm, i.e. $O(N\cdot n^3)$.

**Point 4:**
In order to improve the delay, we need to perform more scattered outputs throughout the computation, which means that, once we are sure that what we found is a maximal clique, we need to output it.
To do so, let us consider the following procedure $\mathtt{MaximalClique2}$:
```pseudo
	\begin{algorithm}
		\caption{Find all cliques of a graph G=(V,E)}
		\begin{algorithmic}
			\Function{MaximalClique2}{$G$}
				\ForAll{$e\in E$}
					\State $e.\mathit{clique}\leftarrow\mathit{false}$
				\EndFor
				\ForAll{$v\in V$}
					\State \Call{FindClique}{$v, \{v\}$}
	            \EndFor
			\EndFunction
		\end{algorithmic}
	\end{algorithm}
```
First of all, we initialize every edge with $\mathit{false}$ in its $\mathit{clique}$ field. This means that no edge has yet been used to form a clique. Once that is done, we iterate on every node and find every maximal clique where it is contained using the $\mathtt{FindClique}$ procedure. Clearly, we will output every maximal clique contained in the graph; however, we will be sure of not outputting the same clique twice using the $\mathit{clique}$ field of the edges.
Given set $S$ of nodes that form a clique, and $v$ the last of those nodes that was added, the $\mathtt{FindClique}$ procedure searches other nodes to add in $N(v)$, i.e. the neighborhood of $v$. If the clique is not maximal, then there must exist another node connected to them all, which consequently means that it is also connected to $v$, and for this reason we search in its neighborhood.
For node in the neighborhood, we check whether it is connected to all nodes in S. If so, the node is added and we recursively continue the search with the enlarged clique $S\cup\{u\}$; otherwise, if no node can be added, we output $S$.
This is algorithm just described:
```pseudo
	\begin{algorithm}
		\caption{Starting from a node, it finds one of its maximal cliques}
		\begin{algorithmic}
			\State $\textbf{global: }\ G=(V,E)$
			\State $\mathit{found}\leftarrow\mathit{false}$
			\Function{FindClique}{$v, S$}
				\ForAll{$u\in N(v)$}
					\If{$\neg(v,u).clique$}
						\State $c\leftarrow 0$
						\ForAll{$w\in S$}
							\If{$(u,w)\in E$}
								\State $c\leftarrow c+1$
                            \EndIf
                        \EndFor
                        \If{$c=|S|$}
	                        \ForAll{$w\in S$}
		                        \State $(u,w).\mathit{clique} \leftarrow \mathit{true}$
	                        \EndFor
	                        \State $\mathit{found}\leftarrow\mathit{true}$
	                        \State \Call{FindClique}{$u, S\cup\{u\}$}
                        \EndIf
                    \EndIf
	            \EndFor
	            \If{$\neg\mathit{found}$}
			        \State $\mathbf{output}\ S$
                \EndIf
			\EndFunction
		\end{algorithmic}
	\end{algorithm}
```
To avoid exploring the same clique multiple times, we mark edges as used using the $\mathit{clique}$ field. Before considering a node, we check whether the edge connecting it to $v$ has already been marked. If it has, we skip it since this indicates that the clique containing this edge has already been explored through a different path. If such edge is shared among multiple cliques, then the node it connects to will be found via a different node, and the clique will still be found.
Finally, the $\mathit{found}$ variable is used to avoid printing every sub-clique of a maximal clique.

Regarding the delay of $\mathtt{MaximalClique2}$ procedure, it completely depends on that of $\mathtt{FindClique}$. First we perform a loop in the neighborhood of $v$, which we can upper-bound with $n$. Then, we cycle in $S$, which, being a subset of $V$, we can also upper-bound with $n$. Finally, we will find a "dead-end" after at most $n$ recursive step, this is because each time we recurse we add one element, and since there are at most $n$, we cannot have more than that.
Therefore, despite the algorithm being exponential in the worst case, the delay is $O(n^3)$.

