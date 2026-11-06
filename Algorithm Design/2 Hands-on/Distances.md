#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">20/03/2026</span></p>

[[AD index]] (2)

**Author:** Claudio Bernardoni

**Hands-on:** 
![[Hands-on 20-03.png]]
**Point 1:**

First of all, let us notice that the expected value of the estimator is not affected by the choice of $k$. Let $\tilde{N}_{i,k}$ be the estimator of the number of pairs of nodes at distance $i$ with a sampling of $k$ nodes; then, for any $k$, using linearity of expectation, we have that: $$\mathbb{E}[\tilde{N}_{i,k}]=\mathbb{E}\left[\frac{\sum\limits_{u\in\ U}|N_i(u)|}{k(n-1)}\right]=\frac{\mathbb{E}\left[\sum\limits_{u\in\ U}|N_i(u)|\right]}{k(n-1)}=\frac{\sum\limits_{u\in\ U}\mathbb{E}\left[|N_i(u)|\right]}{k(n-1)}$$Now, since the sampling is done uniformly at random, each sampled node of $U$ behaves like a random node of $V$, so we get that:$$\frac{\sum\limits_{u\in\ U}\mathbb{E}\left[|N_i(u)|\right]}{k(n-1)}=\frac{k\cdot\mathbb{E}\left[|N_i(u)|\right]}{k(n-1)}=\frac{\mathbb{E}\left[|N_i(u)|\right]}{(n-1)}=\mathbb{E}\left[\frac{|N_i(u)|}{(n-1)}\right]=\mathbb{E}[\tilde{N}_{i,1}]$$
Now that we know that $\mathbb{E}[\tilde{N}_{i}]=\mathbb{E}[\tilde{N}_{i,1}]$, we can implicitly consider $k=1$. By definition of expected value, we can consider every possible value of $\tilde{N}_{i}$, multiplied by the probability of it happening. The value of $\tilde{N}_i$ depends on the sampling of the items, which, from the previous considerations, consists in just extracting one node. Therefore, by considering $U=\{u_1,\dots,u_n\}$, we get that: $$\mathbb{E}[\tilde{N}_i]=\sum_{j=1}^{n}\left(\frac{|N_i(u_j)|}{n-1}\cdot\mathbb{P}(u_j\textit{ being extracted})\right)$$Since every node has the same likelihood of being extracted, $\mathbb{P}(u_j\textit{ being extracted})=\frac{1}{n}$, thus we get that:$$\mathbb{E}[\tilde{N}_i]=\frac{\sum\limits_{j=1}^{n}|N_i(u_j)|
}{n(n-1)}=N_i$$
**Point 2:**

The diameter is, by definition, the maximum eccentricity $e(v)$ of all the nodes $v\in V$; therefore, if we were to sample a set $U$ from $V$, we could just calculate the eccentricity of the sampled nodes and take the maximum of them. Our estimator $\tilde{D}$ of the diameter would be:$$\tilde{D}=\underset{u\in U}{max}\{e(u)\}$$
**Point 3:**

Intuitively, the expected value will be an average, which is always smaller or equal than the maximum, hinting us to the fact that this estimator is biased.

A possible counterexample could be this graph (each node is labelled with its eccentricity):
![[Diagram1 20-03.svg]]
Clearly the diameter is $D=6$; however, the average is $\mathbb{E}[\tilde{D}]=4.42$. If we were to add many more nodes in the middle, all connected to the centermost node of the line, the average would converge to 4, making the approximation even worse. If we were to increase the size of the line, then we diameter would increase by 2 for every new node added at the extremities, but the eccentricity of the cluster in the middle, to which the average will converge, would just increase by 1. This means that, by increasing the number in the cluster in the middle, we stabilize the average to the value of their eccentricity; while, by increasing the nodes at the extremities, we make the average about half of the real diameter.

Intuitively, this happens because the nodes in the cluster are so many that it is highly likely to sample just them, and so the estimator $\tilde{D}$ would take their eccentricity as a guess for the diameter, which is the worst possible value it could take. In this graph we have $\tilde{D}\approx\frac{D}{2}$, which is the worst possible value for it since any eccentricity cannot be lower than half the diameter.