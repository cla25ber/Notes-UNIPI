#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">17/03/2026 & 18/03/2026</span></p>

[[AD index]] (1)

Given a graph $G=(V,E)$ with $n=|V|$ nodes and $m=|E|$ edges, the distance between two nodes $d(u,v)$ with $u\neq v$ is the minimal path between the two nodes. If we consider a completely connected graph, then the average distance is:$$\frac{\sum\limits_{u,v\in V}d(u,v)}{n(n-1)}$$This is because there are $n(n-1)$ possible combinations of 2 nodes, so the sum of the distances divided by that gives the average distance.


#### Diameter

The diameter of a graph is the maximum distance between any two nodes:$$D=\underset{u,v\in V}{max}\{d(u,v)\}$$The definition of diameter can also be given in terms of eccentricity; the eccentricity of a node is $$e(u)=\underset{v\in V}{max}\{d(u,v)\}$$Thus, the diameter is:$$D=\underset{u\in V}{max}\{e(u)\}$$
Calculating the eccentricity of a node costs $O(n+m)$ since we will be performing a BFS; to calculate the diameter we would be doing that with every node, which gets us to $O(n(n+m))$. We say that a graph is sparse when $m\approx n$, which means that the cost of calculating the diameter would be $O(n^{2})$, which is a lot in big networks with a huge number of nodes. The fact is that, if we want to calculate exactly the diameter, we cannot go any lower than that, and this is linked to the Strong Exponential Time Hypothesis (SETH).

### SETH

Let us take a Boolean formula in CNF (Conjunctive Normal Form) with $n'$ variables. Then, the number of possible assignments is $2^{n'}$; the objective is to find one that satisfies the formula. This is the famous SAT problem, which is known to be NP-complete, since in the worst case we need to generate all possible assignments $O(2^{n'})$ and check them $O(\mathit{poly}(n'))$, for a total cost of $O(2^{n'}\cdot\mathit{poly}(n'))=\tilde{O}(2^{n'})$. The problem is interesting when the number of clauses $f$ is $o(2^{\frac{n'}{c}})$ for any constant $c$, otherwise the problem will be polynomial in the input.
According to SETH, no algorithm can solve SAT in $\tilde{O}(2^{\gamma\cdot n'})$ for any $0<\gamma<1$ unless P=NP.

It is possible to show, with a specific graph construction, that determining if the diameter of a graph is $2$ or $3$ cannot be calculated in $O(n^{2-\epsilon})$ time for any $\epsilon>0$ unless SETH is false. Since the problem of determining if the diameter of a graph is 2 or 3 is easier than determining its diameter, this result would obviously impact also the more general problem.

#### Graph construction for SETH

Let us consider a formula $F$ in CNF with a number of clauses $|F|=f=o(2^{\frac{n'}{c}})$ and a number of variables $n'$. The graph consists of $f$ nodes in the middle, one for every clause, and one node for every possible assignment. These nodes are split into two groups, one for every possible assignments of the variables $x_{1},\dots,x_{\frac{n'}{2}}$ (the <span style="color:rgb(255, 64, 192)">pink</span> ones on the left), and one for every possible assignment of the remaining variables $x_{\frac{n'}{2}+1},\dots,x_{n'}$ (the <span style="color:rgb(0, 176, 80)">green</span> ones in the right). Also, two dummy nodes (in <span style="color:rgb(0, 176, 240)">blue</span>) are added. Therefore, the total number of nodes is $n=2^{\frac{n'}{2}}+2^{\frac{n'}{2}}+f+2=O(2^{\frac{n'}{2}})$.

Then we are going to add some construction edges (in <span style="color:rgb(255, 0, 0)">red</span> in the picture): every "left assignment node" is connected with the "left dummy node", every "right assignment" node is connected with the "right dummy" node, and every "clause node" is connected with both "dummy nodes". Also, there is one edge connecting the "dummy nodes". The number of construction edges is $2^{\frac{n'}{2}}+2^{\frac{n'}{2}}+f+f+1=O(2^{\frac{n'}{2}})$.

![[Graph construction for SETH.png]]

In the following table we are going to list the distances between every type of node:

|                                                                     | <span style="color:rgb(255, 64, 192)">Left assignment</span> | Clauses | <span style="color:rgb(0, 176, 80)">Right assignment</span> | <span style="color:rgb(0, 176, 240)">Dummy left</span> | <span style="color:rgb(0, 176, 240)">Dummy right</span> |
| ------------------------------------------------------------------- | :----------------------------------------------------------- | ------- | ----------------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------- |
| <span style="color:rgb(255, 64, 192)"><b>Left Assignment</b></span> | 2                                                            | 2       | 3                                                           | 1                                                      | 2                                                       |
| **Clauses**                                                         | 2                                                            | 2       | 2                                                           | 1                                                      | 1                                                       |
| <span style="color:rgb(0, 176, 80)"><b>Right Assignment</b></span>  | 3                                                            | 2       | 2                                                           | 2                                                      | 1                                                       |
| **<span style="color:rgb(0, 176, 240)">Dummy left</span>**          | 1                                                            | 1       | 2                                                           | 0                                                      | 1                                                       |
| **<span style="color:rgb(0, 176, 240)">Dummy right</span>**         | 2                                                            | 1       | 1                                                           | 1                                                      | 0                                                       |
So the diameter of the graph is 3, since to reach a "left assignment node" from a "right assignment node" (or vice versa) we need 3 hops (passing through the "dummy nodes"). However if we were to create a shortcut, then the diameter would be 2, for this reason we add other edges to the graph.

We create new connection, every "left assignment node" and "right assignment node" gets connected with the clauses it does not satisfy. For example, given the variables $x_1,x_2,x_3,x_4$, and the clause $f_x=(x_1\lor x_2\lor \overline{x_3})$, would be considered satisfied by the "half-assignment" $\{x_1\to0,x_2\to 1\}$ but would not by $\{x_1\to0,x_2\to 0\}$ (so in this case the node would be connected with the clause).

Therefore, for $F$ to be unsatisfiable, it means that every possible assignment does not satisfy at least one clause. In this case it would mean that, fixing the "left-assignment", through that clause it is connected to all other "right-assignments" (and vice versa). Since through a clause every "left-assignment" would be connected to very other "right assignment", the diameter would then be 2; however, if there is even one assignment that satisfies $F$, then we would have two nodes non connected through any clause and therefore at distance 3, which means that:
- $F$ is satisfiable $\iff$ the diameter of the constructed graph is 3
- $F$ is unsatisfiable $\iff$ the diameter of the constructed graph is 2

Thanks to the *sparsification lemma*, we are able to say that the constructed graph is sparse, i.e. $m=O(n)=O(2^{\frac{n'}{2}})$, and consequently we know that its diameter can be calculated in $O(n^{2})$ time. Now, suppose that we have an algorithm can determine if the diameter of a graph is 3 or 2 is $O(n^{2-\epsilon})$ time for $\epsilon>0$; then we would get an algorithm that is able to solve SAT in time complexity: $$O\left((2^{\frac{n'}{2}})^{2-\epsilon}\right)=O\left(2^{n'-\frac{\epsilon\cdot n'}{2}}\right)=O\left(2^{n'(1-\frac{\epsilon}{2})}\right)=O\left(2^{\gamma\cdot n'}\right)$$Since $\gamma=1-\frac{\epsilon}{2}<1$, SETH would be false.

### 2-sweep

Since calculating the exact diameter cannot be done in time under than $O(n^2)$, then we could approximate it using the 2-sweep technique. We select one random node $u$ and calculate its maximum distance $d_1$, and the node in which it ends up, $v$. Then, we calculate the maximum distance from $v$ into $w$, which is $d_2$. Since $d_1$ is the maximum distance from $u$, we know that any node is at distance at most $2d_1$, that is because we could always go from node 1 to node 2 passing from $u$, Therefore, by definition of diameter we know that $d_{1}\leq D\leq 2d_1$ and $d_{2}\leq D\leq 2d_2$, which means that $\mathit{min}\{d_{1},d_2\}\leq D\leq \mathit{max}\{2d_1, 2d_2\}$. This is the 2-sweep, but we could have even more distances to get a better approximation.

In theory, it is possible to create graphs in which the 2-sweep techniques does not perform well, but in practice, since those configurations do not appear often, it is really used and powerful.

### Sampling

Another way to approximate the diameter is through sampling. We sample some nodes $U=\{u_{1},\dots,u_{k}\}\subseteq V$ and calculate the maximum distance of those nodes, taking the maximum of those as the diameter. Clearly, we want to keep $k$ as low as possible to avoid computations, but also we want to minimize the error emerging from this method. This algorithm has cost $O(k\, m)$. 