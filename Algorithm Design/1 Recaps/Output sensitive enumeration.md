#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">28/04/2026 & 29/04/2026</span></p>

[[AD index]] (1)

### Solution graph

We consider a super-graph (also called solution graph), where the vertices are maximal solutions while the edges small tweaks that transform one maximal solution into another. If we are able to build such graph, then enumeration would be just a graph visit with DFS. We don't use BFS because we would need to keep a frontier of solutions, but since there may be exponential solutions, the frontier would occupy exponential space.

#### Input Restricted Problem

Suppose we already know a solution $S$, we pick a vertex $v\notin S$, an we search for maximal solutions that do include $v$ and are "similar" to $S$. The set found might not be maximal, for this reason we call a procedure $\mathtt{complete}$ which adds nodes until the set found is actually a maximal solution. If there are a polynomial number of such solutions, then they can be listed efficiently and thus enumeration becomes much more efficient.

An interesting property about IRP is that it has output polynomial time $\iff$ the general problem has output polynomial time.

**Proof:**
$\impliedby$ is pretty easy, if the general problem has output polynomial time, then the IRP, which is a specific instance of the general problem, necessarily has output polynomial time.
$\implies$ If the IRP has output polynomial time, then it means that it has a cost of $O(N_R^c\cdot n^c)$, where $c$ is a constant and $N_R$ are the solutions of the IRP before maximalization. For each solution given by the IRP¨, we might have to maximize it (it might not be maximal), which means that we get a cost of $O(N_R^c\cdot n^{c+1})$, since we add at most $n$ nodes per IRP result. Clearly $N_R\leq N$, since $N_R$ are the solutions reachable from some other solutions $S$, while $N$ is all the solutions; therefore we get a cost of $O(N^c\cdot n^{c+1})$. Since we perform the IRP at most at every solution we end up in, the total cost for calculating all solutions is $O(N^{c+1}\cdot n^{c+1})$, which is still output polynomial.
$\square$

In order for IRP to work, we must be able to find a path in the super-graph between any two solutions; this happens if the problem is at least *strongly accessible*:![[Accessibility set.png]]
- **Hereditary:** Subset of solutions are solutions. For example: <span style="color:rgb(0, 176, 80)">feasible knapsack</span>, <span style="color:rgb(0, 176, 80)">independent sets</span>, <span style="color:rgb(0, 176, 80)">cliques</span>.
- **Connected Hereditary:** Connected subsets are solutions. For example: <span style="color:rgb(0, 176, 80)">finding trees</span>, <span style="color:rgb(0, 176, 80)">finding paths</span>.
- **Commutable:** Given two solutions $S, S'$, if there exists a solution that is a subset of $S\cap S'$ and there exists a solution to which $S\cup S'$ is a subset, then $S\cup S'$ is a solution. For example, if a problem is hereditary, then the fact that $S\cup S'$ is a subset of a solutions makes $S\cup S'$ also a solution.
- **Strongly Accessible:** We can get a maximal solution by starting from a solution and adding $1$ element at a time. For example: <span style="color:rgb(0, 176, 80)">maximal cliques</span>, <span style="color:rgb(0, 176, 80)">maximal independent set</span>.
- **Accessible:** We can get solutions by starting from one and removing one element at a time. For example: <span style="color:rgb(0, 176, 80)">dense subgraphs</span>.

>[!note]
	>*Small excursus:*
	>The dense subgraph problem aims to find a subset $S$ of a graph where the density, defined as $\frac{\#\text{edges}}{\#\text{possible edges}}=\frac{\#\text{edges}}{|S|\choose2}$, must be at least $t$. This problem is Accessible because if we remove the node with the smallest degree, in the previous fraction the numerator decreases less than the denominator, which means that such value increases and if it was above $t$ before, it will be above $t$ even afterwards.

As an example, let us take the case of cliques, which is quite nice for IRP. Suppose we have solution $S$, let us pick $v\notin S$, then the new maximal solution must contain $v$, and be "similar" to $S$, so we take the members of $S$ that are actually connected to $v$, i.e. $S\cap N(v)$. We then take as a candidate the set $(S\cap N(v))\cup\{v\}$, we check if it is maximal and if it is not we just add vertices until it is.

Now that we know how to build neighbors, searching solutions is clearly a DFS. The only thing is that we have to keep a dictionary of the already visited nodes to avoid visiting them again. Since there might be an exponential number of solutions, the aforementioned dictionary might occupy exponential space.

#### Reverse Search

The problem with the super-graph is that we need to keep a dictionary to traverse it; so, instead of having a graph, we have a tree, where we will be sure of not incurring in the same node twice, and thus the DFS does not need to keep a dictionary, improving space occupancy.
In order to get a tree, we must establish a parent for each solution (except the root), so that if from solution $S$ I get solution $S'$ but $S$ is not the father, it is ignored and another solution is searched.

This means that we need a procedure $\mathtt{parent}(S)$ that is able to deterministically assert the father of solution $S$.

We define $\mathtt{complete}(X)$ to iteratively add the minimum addible vertex to $X$ until it is maximal, which means that now it is deterministic and produces the smallest solution according to an order. This means that the root of the solution tree will be $\mathtt{complete}(\emptyset)$.

Let $S=\{v_1,\dots,v_k\}$ be a solution where the nodes are ordered according to the previously mentioned order and $k$ is the number of nodes in the solution; then we define $S_i=\{v_1,\dots,v_i\}$ to be the subset of nodes of $S$ up to $i\leq k$. When getting a new solution $S$ of size $k$, I can check its parent by performing $\mathtt{complete}(S_{k-1})$, this will yield a solution. If the solution found is still $S$, it means that we have not removed enough nodes to get the parent solution, so we proceed with $k-2$. We proceed until at position $i$, $\mathtt{complete}(S_i)$ will give $S'\neq S$, that is the parent.
So, $\mathtt{parent}(S)=\mathtt{complete}(S_i)$ where $i$ is the highest index for which $\mathtt{complete}(S_i)\neq S$.

The $\mathtt{parent}$ procedure can be implemented as a binary search. We start in the middle, if the result of $\mathtt{complete}$ is the same solution, we recurse on the left, otherwise we recurse on the right. This in done until we find the turning point, where the parent solution will be, yielding a cost of $O(\lg(n))$.

Going back to the example of maximal cliques, for each solution ($N$ total solutions), I check for every node $v$ outside of it ($O(n)$ nodes), if a "similar" solution including $v$ can be found (calling $\mathtt{complete}$, $O(n^2)$), and then we check if we are allowed to access it (we are its parent, calling $\mathtt{parent}$, $O(\lg(n))$). Thus, the final cost of this algorithm is $O(N\cdot n^3\lg(n))$, with a polynomial space occupancy instead of a exponential one. This is the algorithm:
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
As it was explained, a node not in the solution is searched $v\in V\backslash P$. We then try to complete to clique the set $(P_{\leq v}\cap N(v))\cup\{v\}$, we take the "similar" set $P_{\leq v}\cap N(v)$ and add $v$ to it. We consider $P_{\leq v}$, i.e. the nodes in $P$ that come before $v$ in the order, instead of just $P$ because we want to generate the first clique in order, so by also taking bigger nodes we would mess up such order.


Still, the delay can be bad, the fact that we want to avoid visiting solution, either because they are already in the dictionary (super-graph) or because we are not its parent (reverse search super-tree), causes us to lose time between outputs, worsening the delay. After discovering a solution, we might need to backtrack to the root of the super-tree, thus paying $O(n^3\lg(n))$ for every node. The worst possible number of nodes traversed is $N-1$ because I might have generated all solutions except for one, therefore the delay in the worst case is $O(N\cdot n^3\lg(n))$.

We can improve the delay with a technique called **alternative output**, where, if the depth of the tree is even, we output as soon as we find the solution, otherwise, if the depth is odd, we output after having found all the solutions in the sub-tree starting at that node. With this, I will always have some solutions to output whenever I am going downward and whenever I am going upward. This makes the delay $O(2n^3\lg(n))=O(n^3\lg(n))$.