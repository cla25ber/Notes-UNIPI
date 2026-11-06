#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">13/03/2026</span></p>

[[AD index]] (2)

**Author:** Claudio Bernardoni

**Hands-on:** 
![[Hands-on 13-03.png]]

**Point 1:**

Let the graph $G=(V,E)$ have the nodes $V=\{u_{1},\dots,u_{n}\}$. 

From the definition of bubble of distance $i$, it immediately follows that: $$B_{i}(u_{j})=\bigcup_{x=1}^{i}N_{x}(u_{j})$$Consequently, the following property also holds: $$B_{i+1}(u_{j})=B_{i}(u_{j})\cup N_{i+1}(u_{j})$$
Using the first property, we can see that the cost of computing the bubble at distance $i$ of a node is the same cost as computing all the neighborhoods at distance $\leq i$. Since the cost of calculating a neighborhood can be upper-bounded by $O(n+m)$ (the algorithm explores at most $n$ nodes and $m$ edges), we can say that calculating the bubble at distance $i$ has cost at most $O(i(n+m))$, which, by assuming $m\geq n$, can be rewritten as $O(m\cdot i)$. Since this procedure is repeated for all nodes, the final complexity can be upper-bounded by $O(n\cdot m\cdot i)$.

This procedure is not efficient, in fact one could calculate directly the $i\textit{-th}$ neighborhood using a variation of a BFS search, which saves all the nodes explored and stops iterating when a distance of $i$ has been reached. This procedure is clearly faster than a normal BFS, which means that the usual upper-bound $O(n+m)$ of the standard BFS is also valid for this one. This is then repeated for every node, obtaining a complexity of $O(n(n+m))$, which becomes $O(n\cdot m)$, using the same assumption as before.

Each bubble describes all the nodes at distance at most $i$, this means that its size can be upper-bounded by $O(n)$, and since there are $n$ bubbles, we get a total space complexity of $O(n^{2})$.

**Point 2:**

Given a node $u$ and its $i\textit{-th}$ neighborhood $N_{i}(u)=\{v_{1}\dots,v_{j}\}$,  we notice that the following property holds:$$B_{i}(u)=\bigcup_{x=1}^{j}B_{i-1}(v_{x})$$This is because $(i-1)\textit{-th}$ bubble of a neighbor indicates all the nodes reachable with $i$ steps by going into that neighbor, so the union of all of them shows the entire $i\textit{-th}$ bubble of $u$.

This is very useful because it means that we can use the sketches of the $(i-1)\textit{-th}$ bubbles of the neighbors to calculate the sketch of the $i\textit{-th}$ bubble of a node, by just taking the $k$ smallest elements from its neighbors, which has cost $O(k\cdot |N_{i}(u)|)=O(|N_{i}(u)|)$, since $k=O(1)$.
However, this is the operation for just one node, calculating the sketch of the $i\textit{-th}$ bubble of every node, using the handshaking lemma, would take: $$\sum_{j=1}^{n}|N_{i}(u)|=2m$$This means that calculating the $i\textit{-th}$ bubble for every node has cost $O(m)$. Since to calculate all the $i\textit{-th}$ bubbles we need all the $(i-1)\textit{-th}$ bubbles, this procedure must be performed for all the distances up to $i$, meaning that we end up with a cost of $O(m\cdot i)$. Clearly, this inductive algorithm must start with a base case, which is that of calculating for every node $u$ its first bubble $B_{1}(u)$, which consists of the $k\textit{-th}$ smallest hashes of the adjacency list of $u$.

At the $j\textit{-th}$ iteration, this algorithm keeps at most all the sketches of the $(j-1)\textit{-th}$ bubbles needed to calculate the sketches of the $j\textit{-th}$ bubbles. As soon as the $(j+1)\textit{-th}$ iteration starts, the sketches of the $(j-1)\textit{-th}$ bubbles can be discarded, as now we will need to create the sketches of the $(j+1)\textit{-th}$ bubbles using the $j\textit{-th}$ bubbles.
For this reason, at most we will be using two set of bubble sketches. Since there are $n$ sketches, each with $k$ elements, then we get a total space occupancy of $O(2\cdot n\cdot k)=O(n)$. 

**Point 3:**

From the previous properties we know that, for any node $u$, $B_{i}(u_{j})=B_{i-1}(u_{j})\cup N_{i}(u_{j})$, so this means that:$$|B_{i}(u)|=|B_{i-1}(u)|+|N_{i}(u)|-|B_{i-1}(u)\cap N_{i}(u)|$$Since the $i\textit{-th}$ neighbor is clearly disjoint from the $(i-1)\textit{-th}$ bubble $|B_{i-1}(u)\cap N_{i}(u)|=0$. Consequently:$$|N_{i}(u)|=|B_{i}(u)|-|B_{i-1}(u)|$$Let us call $\tilde{a}$ and $\tilde{b}$ respectively the estimators for $|B_{i}(u)|$ and $|B_{i-1}(u)|$ given by the bottom-$k$ sketches, then we can approximate $|N_{i}(u)|$ as:$$\tilde{N}_{i}(u)=\tilde{a}-\tilde{b}$$
Let us now calculate the error of this estimator: $$\left| |N_{i}(u)| -\tilde{N}_{i}(u)\right|=\left| |B_{i}(u)| -|B_{i-1}(u)| -(\tilde{a}-\tilde{b})\right|=\left| |B_{i}(u)|-\tilde{a} -\left(|B_{i-1}(u)| -\tilde{b}\right)\right|$$Using the triangular inequality we get: $$\left| |B_{i}(u)|-\tilde{a} -\left(|B_{i-1}(u)| -\tilde{b}\right)\right|\leq \left| |B_{i}(u)| -\tilde{a}\right|+\left| |B_{i-1}(u)| -\tilde{b}\right|$$Let us now call $\epsilon_{a}$ and $\epsilon_{b}$ respectively the relative errors of $\tilde{a}$ and $\tilde{b}$, and also $\epsilon=\mathit{max}\{\epsilon_{a},\epsilon_{b}\}$, then: $$\left| |B_{i}(u)| -\tilde{a}\right|+\left| |B_{i-1}(u)| -\tilde{b}\right|=\epsilon_{a}\cdot |B_{i}(u)|+\epsilon_{b}\cdot |B_{i-1}(u)|\leq \epsilon\cdot (|B_{i}(u)|+|B_{i-1}(u)|)$$
Let us call $e$ the relative error of $|N_{i}(u)|$, then we get that: $$e\cdot |N_{i}(u)|=\left| |N_{i}(u)| -\tilde{N}_{i}(u)\right|\leq \epsilon\cdot (|B_{i}(u)|+|B_{i-1}(u)|)$$ Thus: $$e\leq\frac{\epsilon\cdot (|B_{i}(u)|+|B_{i-1}(u)|)}{|N_{i}(u)|}=\frac{\epsilon\cdot (|B_{i}(u)|+|B_{i-1}(u)|)}{|B_{i}(u)|-|B_{i-1}(u)|}$$ 
**Point 4:**

If we have a graph where for a certain node, the $i\textit{-th}$ bubble is really big, but only slightly bigger than the $(i-1)\textit{-th}$, then the denominator of the previous fraction is negligible, but the numerator is not, resulting in an approximation with a really big error.