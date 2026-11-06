#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">13/05/2026 & 15/05/2026</span></p>

[[AD index]] (1)

### Chordal graphs

A graph is said to be chordal if every cycle has a chord (a shortcut), thus there is no induced cycle longer than $3$.

A property about chordal graphs is that they always have a <span style="color:rgb(0, 112, 192)">simplicial node</span>, i.e. a node whose neighbors form a clique.

**Proof:**
Let us assume $G=(V,E)$ to be a chordal graph without any simplicial vertex. Let us consider a node $x\in V$ and two nodes $y,z\in N(x)$ such that $(y,z)\notin E$. Such nodes must exists, otherwise it would mean that all the neighbors of $x$ are connected and therefore $x$ would be a simplicial node. Also, there is no node outside of $N(x)$ other than $x$ connected to both $y$ and $z$, otherwise we would have a cycle of $4$ nodes. This means that $y$ and $z$ are separated, no nodes that are connected to $y$ (except for those passing through $N(x)$) will be connected to those of $z$. Now we can apply the same reasoning to $y$ and $z$, which considers at least a new node different from the previously mentioned ones, which consequently creates one different from the previous ones, and so on.... Meaning that for $G$ not to be chordal we would need infinite nodes.
$\square$

If $G=(V,E)$ is a chordal graph, then for any $v\in V$, $G\backslash\{v\}$ is chordal. This is because we cannot create cycles by removing nodes.

This means that we can see if a graph is chordal by iteratively finding its simplicial vertex and remove it, since the removal creates a chordal graph, it will have another simplicial vertex.

Given the definition of a simplicial node $v$, it is clear that $\{v\}\cup N(v)$ forms a maximal clique, since if there could be a node connected to them all that expands the clique, then it would be a node not present in $\{v\}\cup N(v)$ but present in $N(v)$ since it would need to be connected to $v$. From this fact, we understand that there are at most $n$ maximal cliques in a chordal graph, in contrast with the exponential $3^{\frac{n}{3}}$ in a generic graph. Also, the algorithm for checking if a graph is chordal, gives us a way to find all maximal cliques.

#### Treewidth

Let us consider a generic graph $G=(V,E)$, then we can turn it into a <span style="color:rgb(0, 176, 80)">tree decomposition</span>, i.e. a tree $T$ for which the following properties holds:
- Every node of $T$ is a subset of $V$.
- For any $v\in V$, if we consider only the nodes of $T$ containing $v$, they form a sub-tree of $T$.
- For any $(v,w)\in E$, there exists a node in $T$ that contains both $v$ and $w$.

The trivial decomposition is the one with a single node containing $V$. We will consider the minimal decomposition, i.e. the decomposition whose maximal node size is the smallest possible. Let us call $t$ that value, the treewidth is $t-1$. Generally, it is NP-hard to compute the treewidth of a graph.

The power of a tree decomposition is that it allows to treat the graph as if it were a tree, each bag in fact acts as a separator, meaning that if such nodes are removed from the graph, the remaining ones are separated. This means that the structure of the graph gets simplified, and problems that were exponential before, given a tree decomposition, they can be parametrized into being of the form $\text{poly}(n)\cdot\text{exp}(t)$. 
