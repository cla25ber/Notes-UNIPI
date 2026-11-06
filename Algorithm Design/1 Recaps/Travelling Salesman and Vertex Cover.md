#algorithm_design  #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">14/04/2026 & 15/04/2026</span></p>

[[AD index]] (1)

We can have many types of combinatorial problems:
- <span style="color:rgb(0, 176, 240)">Decision problems</span>: Does a solution exist?
- <span style="color:rgb(0, 176, 240)">Search problems</span>: Find a solution.
- <span style="color:rgb(0, 176, 240)">Optimization problems</span>: Find the best solution (according to a measure).
- <span style="color:rgb(0, 176, 240)">Counting problems</span>: Find the number of solutions.
- <span style="color:rgb(0, 176, 240)">Enumeration Problems</span>: Enumerate all solutions.

### Inapproximability of TSP

Let us consider now an instance of an optimization problem: the Travelling Salesman Problem (TSP) with weights on the edges of the graph, the objective is to not only find a cycle that passes through all the nodes just once, but also to find the one with the smallest cost (the sum of the weights).

Finding an r-approximation in polynomial time would imply that P=NP, which means that finding an r-approximation for this problem is as difficult as solving the original problem.

**Proof:**
Given a graph $G=(V,E)$, suppose we have an algorithm $A_r$ to calculate an r-approximation of the optimal solution $\mathit{OPT}$ in polynomial time. Then, we could consider a clique composed of the nodes present in $G$ that has edges labelled in the following way:$$\begin{align}
    D_{i,j} = \begin{cases}
            1 \quad\quad\qquad\ \ \ \text{if $(i,j)\in E$}\\
            1+r\cdot|V| \quad\text{otherwise}
           \end{cases}
\end{align}$$Then, if we executed $A_r$ on this graph, we would get a cost $C$ in polynomial time, for which, by definition of r-approximation, holds that $C\leq r\cdot\mathit{OPT}$.

If the original graph $G$ has an Hamiltonian path, then in the constructed graph $\mathit{OPT}=|V|$ since it is possible to pass through all the nodes using only those that cost $1$. However, we know that $C\leq r\cdot\mathit{OPT}=r\cdot|V|$, which means that $A_r$ also finds a solution that uses only edges of weight $1$, and thus $G$ having an Hamiltonian path implies that $C\leq r\cdot|V|$.
If the original graph $G$ does not have an Hamiltonian path, then even the optimal solution must have used at least one costly edge, and thus $\mathit{OPT}\geq1+r|V|+|V|-1=(r+1)|V|>r\cdot |V|$. Since by definition of optimal solution $C\geq\mathit{OPT}$, then $C>r\cdot|V|$. This means that $G$ not having an Hamiltonian path implies that $C>r\cdot|V|$.
Finally, putting these two things together, we have that $G$ has an Hamiltonian path iff  $C\leq r\cdot|V|$; this means that we could solve TSP in polynomial time, and thus that P=NP.
$\square$ 

#### An approximation of a specific case of TSP

If we know that we operate on a graph that is a clique and is metric, i.e. the triangular inequality holds, then we can compute the MST (Minimum Spanning Tree), i.e. the a tree that covers all the nodes and has the minimum cost, in polynomial time. Once we have the MST, we can perform a pre-order visit of it (starting from any node). Once we have the list of node visited, that is the path we should follow, and it will be a 2-approximation.

Let us consider the optimal Hamiltonian cycle of cost $\mathit{OPT}$, if we remove an edge from it, what we get is a spanning tree, and the optimal solution will have a bigger cost than it because we removed an edge. This means that $\mathit{OPT}\geq\mathit{LB}$, where $\mathit{LB}$ is the cost of the MST, this is because any spanning tree has a bigger cost than the MST.  

Here is an example of a graph and its MST in yellow:
![[MST example.png]]
Starting from M, the pre-order visit finds: M-G-O-N-P-.... Then the path we follow will be the one found by the visit. If we could only pass through the spanning tree, to visit every node we would pass through every edge of it twice, meaning that we would pay $2\cdot\mathit{LB}$. 
Let us now consider the Hamiltonian cycle we will use (the one given by the pre-order visit). In the example we go from M to G, and then to O and then to N, using only MST edges until there. Now we skip directly from N to P to avoid passing through O again, so this means that, thanks to the triangular inequality, $d(N,P)\leq d(N,O)+d(O,P)$. From this, we get that this path we follow will always be smaller or equal than passing only from the MST (the yellow path), and thus, calling $S$ the cost of our solution, $S\leq2\cdot\mathit{LB}$, and consequently: $\frac{S}{\mathit{LB}}\leq2$.
Now, using what we have seen so far: $\frac{S}{\mathit{OPT}}\leq\frac{S}{\mathit{LB}}\leq2$, and thus this algorithm produces a 2-approximation of the optimal solution in the TSP.

### Vertex cover

Given a graph $G=(V,E)$, the Vertex Cover (VC) problem aims to find the smallest subset $S\subseteq V$ such that $\forall(u,v)\in E.(u\in S)\lor(v\in S)$. A simple algorithm for this problem is the following:
```Python
	# G=(V,E) is the graph
	S = {}
	for (u,v) in E
		if (not(u in S) and not(v in S))
			S = S + {u,v} 
```
The optimal solution must at least choose one of the two nodes in the edge chosen, so, if $S$ is the solution given by this algorithm, $\mathit{OPT}\geq\frac{|S|}{2}$, which means that $S$ is a 2-approximation.
VC is NP-complete.

###### Set Cover

Let $U$ be a universe set and $C\subseteq2^U$ a collection of sets of elements of the universe. The Set Cover (SC) problem aims to find a sub-collection $C'\subseteq C$ such that it covers the entire universe: $\bigcup\limits_{S\in C'}S=U$.
A greedy algorithm that chooses the set containing the biggest amount of elements not yet present in the union of sets in $C'$ can obtain a $O(\lg(n))$-approximation.

###### Independent Set

Given a graph $G=(V,E)$, the Independent Set (IS) problem aims to find the biggest subset $S\subseteq V$ such that $\forall u,v\in S.(u,v)\notin E$. This problem can be seen as the dual of VC, because if $S'$ is the smallest vertex cover, then $S=V\backslash S'$ is the biggest independent set.

### Kernelization of VC

We know that VC is NP-complete and thus can be solved in exponential time, but we can still get a better exponential time, something along the lines of $O(f(k)\cdot\mathit{poly}(n))$, where $k$ is an added parameter of the problem. For example, in the case of VC we could have that the problem would be to find a vertex cover of size at most $k$ ($k$ is not necessarily linked to other parameters like $n=|V|$ or $m=|E|$).

One solution to this problem could be to check every subset $S\subseteq V$ such that $|S|\leq k$ and see if they are a vertex cover; however, even if checking if a set is a VC can be done in $O(\mathit{poly}(n))$ time, we would still have $n\choose k$ possible subsets, meaning that the total time would then be $O({n\choose{k}} \cdot \mathit{poly}(n))=O(n^k\cdot\mathit{poly}(n))$, which would still be exponential in $n$.

Kernelization is reducing the problem into a much more manageable size using necessary conditions. For example, these are the rules for a kernelization of VC:
1) If $v$ is an isolated node in $G$ then we solve VC($G-\{v\}$, $k$).
2) If $v$ has a degree strictly greater than $k$, then it must belong in the VC (because it has a maximum of size $k$), so we need to solve VC($G-\{v\}$, $k-1$).
Once rules 1 and 2 cannot be applied anymore, if $k<0$ then there is no solution since that would mean that we necessarily need more than $k$ nodes to have a VC. Also, let us consider $k$ to be the number at which the algorithm ended up when it could not perform anymore rules 1 and 2; then if $G$ has more than $k^2+k$ nodes or more than $k^2$ edges, the algorithm also fails, as there is no possible vertex cover of size at most $k$.

**Proof edges:**
Since we could not apply anymore rule 2, we know that every node must have degree less or equal than $k$, and since we want a vertex cover of at most size $k$, this means that there must be at maximum $k^2$ edges. If there were more it would be impossible to cover them all.
$\square$

**Proof nodes:**
Let $V_k$ be the nodes remaining after not being able to apply 1 or 2 anymore. Then, the number of nodes in $V_k$ are the number of nodes that will end up in the VC, which is at most $k$, plus the number of nodes not taken into the VC. The number of nodes not taken must all be connected to at least one node in the solution $S$, but since every node has at most degree $k$ for the same reason as before, this means that there are at most $|S|\cdot k=k^2$ nodes not taken into the solution. Therefore, if there are at most $k$ nodes taken and at most $k^2$ nodes not taken, then $|V_k|\leq k^2+k$.
$\square$
Once we have a graph with at most $k+k^2$ nodes and $k^2$ edges we can brute force it with an exponential cost on $k^2$, while the kernelization has polynomial cost on $n$, obtaining a general cost of $O(k^k+\mathit{poly}(n))$. This happens assuming that we already know what is the perfect $k$, if we do not, then we just try every $k$ starting from $1$ up to the actual $k$. This means that the cost can be upper-bounded by $O(k\cdot(k^k+\mathit{poly}(n)))$. 

##### Another kernelization for VC

Another possible kernelization for the VC problem could be the following:
Every time I encounter a node I have to decide whether to keep it or not:
- <span style="color:rgb(0, 176, 80)">Keep it</span>: then we call VC($G-\{v\}$, $k-1$).
- <span style="color:rgb(0, 176, 80)">Discard it</span>: then we have to keep all of the other nodes connected to it, otherwise those edges will not be take, so we call VC($G-N(v)$, $k-|N(v)|$).

This means that when choosing about $v$ with a VC of $k$ descend $L(k)$ leaves, where $L(1)=1$ and $L(k)=L(k-1)+L(k-N(u))\leq L(k)=L(k-1)+L(k-2)$ since we can assume the neighborhood to be at least of size $2$. This Fibonacci structure gives us $L(k)\leq O(\phi^k)$, where $\phi$ is the golden ratio, which means that the total cost will be $O(\phi^k\cdot\mathit{poly}(n))$.