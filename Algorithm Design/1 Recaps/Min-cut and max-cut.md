#algorithm_design  #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">24/03/2026 & 25/03/2026</span></p>

[[AD index]] (1)
### Min-cut

Given a multigraph $G=(V,E)$ with $n=|V|$ vertices and $m=|E|$ edges (multigraph allow for repeated edges between nodes), a cut is a pair of nodes sets $C=(V_1,V_2)$ such that $V_1\cap V_2=\emptyset$ and $V_1\cup V_2=V$, so a disjunct full division of the nodes.
From a cut $C=(V_1,V_2)$, we can derive a cut set: $$E(C)=\{(u,v)\ |\ (u\in V_1\land v\in V_2) \lor (v\in V_1\land u\in V_2)\}$$We say that a cut $C$ is a min-cut if $|E(C)|\leq|E(C')|$ for any other cut $C'$.

Let $(u,v)\in E$, edge contraction creates a new graph $G'=(V',E')$ where $V'=(V\backslash \{u,v\})\cup\{uv\}$, with $uv$ being a new single node, and $E'=E\backslash\{(u,v)\in E\}$ and also every node of the type $(u,z)$ or $(v,z)$ is replaced with $(uv,z)$.
![[Edge contraction.svg]]
Performing an edge contraction to the nodes $u$ and $v$ is like claiming that $u$ and $v$ go into the same partition in the cut. What happens if we choose a random sequence of edge contractions? Will we end up with something close to the real solution?

Suppose that for $G=(V,E)$ the min-cut is $C=(V_1,V_2)$, then our algorithm will not find the min-cut if it chooses an edge $(u,v)\in E(C)$. Let $k=|E(C)|$, then this means that we will have at most $k$ bad edges, i.e. edges for which the contraction will not produce the cut $C$.

Let us consider the probability of choosing a bad edge for the contraction:$$\mathbb{P}[\{(u,v)\in E\}\textit{ is a bad edge}]\leq\frac{k}{m}$$Then, by the handshaking lemma:$$2m=\sum_{v\in V}|N(v)|\leq n\cdot k$$This is because every node must have degree bigger than $k$, otherwise the min-cut would be to just isolate that node. This means that:$$\frac{k}{m}\leq\frac{2k}{n\cdot k}=\frac{2}{n}$$So the probability of choosing a bad edge is less than $\frac{2}{n}$.

Let $\mathbb{P}(n)$ be the probability of the algorithm to find the min-cut of a graph with $n$ nodes by guessing edge contractions, then $\mathbb{P}(2)=1$ because, with just two nodes, we already know which is the min-cut. However, for a generic $n$, $\mathbb{P}(n)\geq(1-\frac{2}{n})\cdot\mathbb{P}(n-1)$. Expanding the recursive relation we get:$$\mathbb{P}(n)\geq\frac{n-2}{n}\cdot\frac{n-3}{n-1}\cdot\ ...\ \cdot\frac{1}{3}=\frac{(n-2)!}{\frac{n!}{2}}=\frac{2}{n(n-1)}$$ This means that, if we repeat this algorithm $N$ times, and we take the minimal cut that we find, the probability of getting the wrong one is:$$\mathbb{P}[\textit{wrong min-cut}]\leq\left(1-\frac{2}{n(n-1)}\right)^{N}$$Consequently, the probability of success for very large $n$ is:$$\mathbb{P}[\textit{correct min-cut}]\geq1-\left(1-\frac{2}{n(n-1)}\right)^{N}\approx1-e^{-\frac{2N}{n(n-1)}}$$By setting $e^{-\frac{2N}{n(n-1)}}=\frac{1}{n^c}$ with $c>1$ a constant, we get that the probability of success is $1-\frac{1}{c}$, so highly likely.
However, in order to be able to have this, we have to perform $N=\Theta\left(\frac{n(n-1)}{2}\cdot c\cdot \ln(n)\right)$ experiments.

### Max-cut

While finding the minimum cut is relatively easy, finding the maximum cut is not, a sit is NP-hard.
We explore 2 approaches to the max-cut problem:
1) Greedy
2) Randomized

##### Greedy

We want to find a cut $E(C)$, so we start with $S=\{\}$ and take one by one every node $u\in V$. You compare the number of edges that go from $u$ to $S$ and from $u$ to $\bar{S}$, if there are more that go into $S$ then it is put into $\bar{S}$, otherwise into $S$. In the end the cut will be $C=(S,\bar{S})$.

The value found by this algorithm is:$$|E(C)|\geq\frac{\sum\limits_{u\in V}\frac{d_u}{2}}{2}$$Where $d_u$ is the degree of $u$. This is because for every node $u$, there are more edges that traverse the cut than those that do not (otherwise that node would have been put in the complementary set); however we have to divide this value by $2$ otherwise each edge would be counted twice. This means that we can lower bound the value obtained by the cut with half the degree of each node. By summing all the degrees we get twice the number of edges (each edge is counted twice for the two nodes it is connected to), and thus:$$\frac{\sum\limits_{u\in V}\frac{d_u}{2}}{2}=\frac{\frac{2|E|}{2}}{2}=\frac{|E|}{2}$$By definition of optimal solution $\text{OPT}\geq|E(C)|$, where $\text{OPT}$ is the maximum value. Clearly, since we cannot exceed all the possible edges, $\text{OPT}\leq|E|$, which consequently means that for any cut $C$:$$\frac{OPT}{|E(C)|}\leq\frac{|E|}{|E(C)|}\leq\frac{|E|}{\frac{|E|}{2}}=2$$This result shows us that the local search method will find a solution that, in the worst case, is at least half of the optimal solution.

##### Randomized

Here we start with $S=\{\}$ and extract one random node $u\in V$ and with probability $\frac{1}{2}$ decide whether to put it in $S$ or $\bar{S}$, until no other nodes in $V$ remain. Now, given an edge $(u,v)\in E$, there is probability $\frac{1}{2}$ of it ending up in $E(C)$ where $C=(S,\bar{S})$. This is because there are 4 possible cases:
1) $u$ and $v$ are in $S$
2) $u$ and $v$ are in $\bar{S}$
3) $u$ is in $S$ and $v$ in $\bar{S}$
4) $u$ is in $\bar{S}$ and $v$ in $S$
These cases are equally random, and in the first two we would have $(u,v)\notin E(C)$ while or the other two $(u,v)\in E(C)$.

Let us consider the random variable:$$\begin{align}
    X_{i,j} = \begin{cases}
            1 \quad\text{if $(i,j)\in E(C)$}\\
            0 \quad\text{otherwise}
           \end{cases}
\end{align}$$Then, from the previous analysis, we have that $\mathbb{E}[X_{i,j}]=\frac{1}{2}$. By how we defined the random variable, it is clear that:$$|E(C)|=\sum_{(i,j)\in E}X_{i,j}$$Therefore:$$\mathbb{E}[|E(C)|]=\sum_{(i,j)\in E}\mathbb{E}[X_{i,j}]=\frac{|E|}{2}$$This means that, as before, we expect to have a cut of size around $\frac{|E|}{2}$.