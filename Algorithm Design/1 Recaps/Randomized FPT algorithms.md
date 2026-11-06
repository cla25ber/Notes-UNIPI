#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">06/05/2026</span></p>

[[AD index]] (1)

Here we consider randomized Fixed Parameter Tractable (FPT) algorithms.

### Color coding

Let us consider the problem of finding a $k$-path in a graph. Finding a walk (no repeating edges but repeating nodes are ok), is easy, but a path is not, this comes down to the fact that if we put $k=n$ we obtain the Hamiltonian path problem, a notorious NP-hard problem.

Let us consider now a random coloring of the graph, i.e. a function $X:V\to[k]$ where each color is assigned uniformly at random. We say that a subset $V'\subseteq V$ is colorful if every node in $V'$ is of a distinct color pairwise.
Now, we can say the following: a colorful walk of $k$ nodes is a $k$-path. This easily stems from the fact that if a walk is colorful, then it does not have repeated nodes (otherwise we would have the same color), and therefore it is also a $k$-path.
Also, a walk of $k$ nodes is a $k$-path $\iff$ there exists a random coloring $X$ such that the walk is colorful.
The $\implies$ arrow is easy, if a walk of $k$ node is also a $k$-path it means that all nodes are different, so we just need to find a coloring that assign them different colors.
The $\impliedby$ arrow is also easy as it directly stems from the previous proposition.

Consider a random coloring $X$, we choose $V'\subseteq V$ such that $|V'|=k$, then we want to check:$$\mathbb{P}(V'\text{ is colorful})=\frac{\#\text{colorings that make $V'$ colorful}}{\#\text{total colorings}}$$Clearly, $\#\text{total colorings}=k^n$, since every node can be assigned one of $k$ colors.
Also, $\#\text{colorings that make }V' \text{ colorful}=k!\cdot k^{n-k}$, since when assigning colors to the $k$-path we cannot reuse one already chosen, but for the remaining $n-k$ nodes we do not care. Therefore:$$\mathbb{P}(V'\text{ is colorful})=\frac{k!\cdot k^{n-k}}{k^n}=\frac{k!}{k^k}\geq\frac{1}{e^k}$$The power of this result comes from the fact that $n$ is not present, so the probability of success only depends on $k$. Therefore, if we try the randomized algorithm enough times we will get that eventually we will find a solution, and this eventually depends on $k$ not $n$, and thus as the graph increases, the number of trials remains the same. Clearly though, by increasing $k$, the number of trials needs to be increased too.

We can use the fact that given a node $u$ of color $X(u)$. If a $k$-path end in $u$, then we need to find a $k-1$-path that end in a node in $N(u)$. Finding a colorful path can be done using dynamic programming, the recurrence relation is the following:$$\begin{align}
    T[S][u] = \begin{cases}
            \mathit{true} \ \ \quad\qquad\qquad\qquad\text{if $S=\{X(u)\}$}\\
            \bigvee\limits_{v\in N(u)}T[S\backslash X(u)][v] \quad\text{if }X(u)\in S\land|S|\geq2\\
            \mathit{false} \ \ \quad\qquad\qquad\qquad\text{otherwise}
           \end{cases}
\end{align}$$
We have a table with $n$ columns (one per node), and $2^k-1$ rows, one per possible subset of the $k$ colors (except the empty set). A cell $(S,u)$ is true if there is a possible coloring with colors $S$ that ends in $u$; therefore, if $S$ is a singleton and $u$ is that specific color, then we have $\mathit{true}$. If $S$ is not a singleton, we see if there is a possible colorful path with colors $S\backslash X(u)$ that end in the neighbors of $u$. If none of the previous conditions are satisfied, then we return $\mathit{false}$.

Filling a cell can cost at most $O(|N(u)|)$ time, this is because we have to visit every neighbor of the node. Since we are doing this for all the nodes (all the columns) we get a time, using the handshaking lemma, of:$$O\left(\sum_{v\in V}|N(u)|\right)=O(m)$$Since there are $2^k-1$ rows, we get that this dynamic programming algorithm runs in $O(2^k\cdot m)$.

Now, this is for a specific coloring, if we select $e^k$ random colorings, then we still get a constant probability (the probability decreases by $e^k$ by increasing $k$, but we also increase the number of tries by $e^k$ when $k$ increases, effectively cancelling out the effect of increasing $k$). Time-wise, this obviously increases the cost to $O(e^k\cdot 2^k\cdot m)=O((2\,e)^k\cdot m)$, which is still FPT.

### Randomized separation

Let us consider a graph $G=(V,E)$, and $H=(V_H,E_H)$, we need to find a sub-graph $\hat{H}$ of $G$ that is isomorphic to $H$. Let $k=|V_H|$ and $\Delta$ the maximum degree of $G$. Suppose that a sub-graph $\hat{H}$ isomorphic to $H$ exists, then we could create the set $\Gamma=\{(x,y)\in(E\backslash\hat{E})|x\in\hat{V}\lor y\in\hat{V}\}$, basically the frontier edges of the isomorphism, i.e. those edges connected to a node of the isomorphism and a node outside of it. From this definition we get that $\Gamma$ and $\hat{E}$ are disjoint, and thus:$$|\Gamma|+|\hat{E}|\leq|\hat{V}|\cdot\Delta=|V_H|\cdot\Delta=k\cdot\Delta$$This is because the sum of the edges inside the isomorphism and on its frontier can be maximized by "assigning" to every node of the isomorphism the maximum degree. The number of nodes in the isomorphism is $k$, so we get an upper-bound of $k\cdot\Delta$.

Let us now consider a $2$-coloring of the edges $X:E\to\{R,B\}$, we want all the edges of $\Gamma$ to be one color, and all the edges of $\hat{E}$ to be the other, this way we can check whether all connected components of size $k$ are isomorphic to $H$. We can check connected components with a BFS, so $O(n+m)$, and we can check isomorphism with $O(k!\cdot k^2)$, this is because (if we brute-force it) we take a random ordering of the nodes, and check if their adjacency matrix coincide, that takes $k^2$ time, and there are $k!$ orderings of the nodes in total.
Therefore, we create a coloring, then check connected components, and then, for every connected component we check if they are isomorphic, so: $O(m)+O(n+m)+O(k!\cdot k^2\cdot n)=O(k!\cdot k^2\cdot n)$.

For a coloring to be successful, it must color $\Gamma$ and $\hat{E}$ as described before, thus we get that:$$\mathbb{P}(X\text{ is successful})=\frac{2}{2^{|\Gamma|+|\hat{E}|}}\geq\frac{1}{2^{k\cdot\Delta}}$$
This means that, in order to have a constant probability like before, we need to perform $2^{k\cdot\Delta}$ tries, meaning that the total cost of the algorithm becomes $O(2^{k\cdot\Delta}\cdot k!\cdot k^2\cdot n)$, which is FPT not on only $k$ but on $k$ and $\Delta$.