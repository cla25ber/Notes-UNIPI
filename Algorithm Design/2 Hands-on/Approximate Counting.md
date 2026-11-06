#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">10/04/2026</span></p>

[[AD index]] (2)

**Author:** Claudio Bernardoni

**Hands-on:** 
![[Hands-on 10-04.png]]

**Point 1:**

The complexity in time of the dynamic programming algorithm for the knapsack problem is $O(n\, W)$, where $n$ is the number of weights and $W$ the capacity of the knapsack. Therefore, since the input has been transformed into $(\tilde{w},\tilde{W})$, where $\tilde{w}$ is the new vector of the weights and $\tilde{W}$ the new capacity of the knapsack, performing the dynamic programming algorithm costs $O(|\tilde{w}|\cdot\tilde{W})$. The transformation is a downscaling by $k$, so the length of the array remains unchanged, i.e. $|\tilde{w}|=n$, while, by construction, $\tilde{W}=n^2$. This means that the running time for calculating the solution of this transformation is $O(n^3)$.

$\\$

**Point 2:**

To prove that $|F|\leq|\tilde{F}|$ it is possible to prove that $F\subseteq\tilde{F}$. Let us consider $S\in F$; then, by definition of solution for the knapsack problem, it holds that:$$\sum_{i\in S}w_{i}\leq W$$which means that it also holds that:$$\sum_{i\in S}\frac{w_{i}}{k}\leq\frac{W}{k}=\tilde{W}$$Now, since by definition of the floor $\lfloor\cdot\rfloor$ function, it is true that: $$\tilde{w}_{i}=\left\lfloor\frac{w_i}{k}\right\rfloor\leq\frac{w_i}{k}$$and consequently:$$\sum_{i\in S}\tilde{w}_i\leq\sum_{i\in S}\frac{w_{i}}{k}$$Finally, by transitivity, we obtain that:$$\sum_{i\in S}\tilde{w}_i\leq\tilde{W}$$meaning that also $S\in\tilde{F}$, and thus $F\subseteq\tilde{F}$.


**Point 3:**

Let us consider a solution $S$ such that $S\in \tilde{F}\backslash F$, meaning that $S\in\tilde{F}$ but $S\notin F$. This means that for $S$ the following holds:$$\sum_{i\in S}\tilde{w}_{i}\leq\tilde{W}\quad\quad\quad\sum_{i\in S}w_{i}>W$$
Let $e$ be the index of the element with the maximum weight inside of $S$, then we know that:$$W<\sum_{i\in S}w_{i}\leq\sum_{i\in S}w_{e}=n\,w_e$$This is because we can maximize every weight with the maximal one; which consequently means that $w_e>\frac{W}{n}$.

Now, let's suppose by absurd that not even by removing $e$ from solution $S$ we are able to obtain a solution valid for the original problem, i.e. $(S\backslash\{e\})\notin F$. This would mean that:$$\sum_{i\in S\backslash\{e\}}w_{i}>W$$which implies that, by adding $w_e$, we get:$$\sum_{i\in S}w_{i}>W+w_e>W+\frac{W}{n}$$which used the previous property about $w_e$ that we found. We can then divide by $k$ and obtain:$$\sum_{i\in S}\frac{w_{i}}{k}>\frac{W}{k}+\frac{W}{k\,n}=\tilde{W}+\frac{n^2}{n}=\tilde{W}+n$$which was simplified using the fact that by construction $\tilde{W}=\frac{W}{k}=n^2$. Since $S$ is a valid solution for the simplified problem, i.e. $S\in\tilde{F}$, we know that:$$\tilde{W}\geq\sum_{i\in S}\tilde{w}_i=\sum_{i\in S}\left\lfloor\frac{w_i}{k}\right\rfloor$$which consequently means that:$$\sum_{i\in S}\frac{w_{i}}{k}>\tilde{W}+n\geq\sum_{i\in S}\left\lfloor\frac{w_i}{k}\right\rfloor+n$$Therefore:$$\sum_{i\in S}\left(\frac{w_{i}}{k}-\left\lfloor\frac{w_i}{k}\right\rfloor\right)>n$$However, it is true that:$$0<\frac{w_{i}}{k}-\left\lfloor\frac{w_i}{k}\right\rfloor<1$$which means that it is impossible for the sum of $n$ numbers between $0$ and $1$ to be over $n$. This means that the initial hypothesis, i.e. removing the biggest weight $w_e$ will not make the solution $S$ satisfy the original problem, is false.

**Point 4:**

For $|\tilde{F}|$ to be a $(n+1)$ approximation of $|F|$, it means that $\frac{|\tilde{F}|}{|F|}\leq n+1$. Consequently, this would mean that for every element of $F$, we can associate at most $n+1$ elements of $\tilde{F}$.

![[Diagram1 10-04.svg]]

From Point 2, we know that $F\subseteq\tilde{F}$, which means that every exact solution is also included in $\tilde{F}$. Therefore, we can trace an exact one-to-one correspondence with that subset of $\tilde{F}$, as shown in the figure.

Then, we consider the following construction: for every exact solution of $\tilde{F}$, we consider $n$ other expansions by adding to them the $i\textit{-th}$ weight, i.e. for an exact solution $S\in F\cap\tilde{F}$, we consider every other possible $S_i=S\cup\{i\}$; these sets are at most $n$ for each solution, even tough we could say that they are certainly less considering that if $i\in S$ then $S_i=S$.

What we claim is that this construction completely covers $\tilde{F}$, meaning that all solution expressible through the scaled down problem are covered by considering all the exact solution and adding one element to them. Let us consider a solution $S'\in\tilde{F}$ that was obtained by adding two elements from an exact one; then, from Point 3, we know that by removing the biggest element $e$, we obtain a solution that $S=S'\backslash\{e\}$ such that $S\in F$. However, this means that $S'=S_e$, and thus can be obtained by adding only one element.

This means that we are covering completely $\tilde{F}$, and we are doing so by associating every element of $F$ only their exact counterpart in $\tilde{F}$, and their $n$ counterparts with one weight added. This means that every element of $F$ has associated at most $n+1$ elements and thus $|\tilde{F}|\leq |F|\,(n-1)$.

**Point 5:**

Let us sample solutions from $\tilde{F}$ and consider the following random variable:$$\begin{align}
    X_{j} = \begin{cases}
            1 \quad\text{if solution $j$ also works for $F$}\\
            0 \quad\text{otherwise}
           \end{cases}
\end{align}$$Let us repeat the sampling $h$ times, marking solutions already extracted to avoid repetitions. Then, by the law of large numbers, for an enough big $h$ we would get that the average of the experiments tends to the expected value of the random variable:$$\frac{\sum\limits_{j=1}^{h}X_j}{h}=\mathbb{E}[X_j]$$By definition of expected value we have that:$$\mathbb{E}[X_j]=1\cdot\mathbb{P}(\textit{solution $j$ working for $F$})=\frac{|F|}{|\tilde{F}|}$$This is because from Point 2 we know that $F\subseteq\tilde{F}$, which means that if we sample a random item from $\tilde{F}$ the probability of it belonging also to $F$ is $\frac{|F|}{|\tilde{F}|}$. This means that, by calling $s=\sum_{j=1}^{h}X_j$ the number of successes, we can estimate the size of $F$ to be:$$|F|=\frac{s\cdot|\tilde{F}|}{h}$$