#algorithm_design #computer_science 

<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">20/02/2026</span></p>

[[AD index]] (2)

**Author:** Claudio Bernardoni

**Hands-on:**

![[Hands-on 20-02.png]]

**Proof:**

Let us define the same indicator variable of the randomized QuickSort: $$\begin{align}
    X_{i,j} = \begin{cases}
            1 \quad\text{if $i$ is compared with $j$}\\
            0 \quad\text{otherwise}
           \end{cases}
\end{align}$$First of all, we observe that, just like for randomized QuickSort, if $X_{i,j}=1$ then either $i$ or $j$ must have been chosen as the pivot. However, differently from QuickSort, for every $i$ we can identify 2 cases when $X_{i,j}=1$. The first case is when $i$ is not the pivot, this means that, from the previous property, $j$ was the pivot; consequently, this means that in the Randomized Search Tree $i$ was not expanded and thus it was pushed down further in the tree, contributing to its depth.
Therefore, given the value $x_{i}$, its depth on the tree is exactly the number of comparison where $X_{i,j}=1$ but $j$ was the pivot. This can be expressed by a similar indicator variable:$$\begin{align}
    Y_{i,j} = \begin{cases}
            1 \quad\text{if $i$ is compared with $j$ and $j$ was the pivot}\\
            0 \quad\text{otherwise}
           \end{cases}
\end{align}$$so, given a certain value $x_{i}$, its depth is $\sum_{j=1}^{n}Y_{i,j}$ .

Then, the expected depth is $\mathbb{E}[\sum_{j=1}^{n}Y_{i,j}]=\sum_{j=1}^{n}\mathbb{E}[Y_{i,j}]$ by linearity of expectation. By the definition of expected value we get that $\mathbb{E}[Y_{i,j}]=\mathbb{P}(Y_{i,j}=1)$, which happens when $i$ and $j$ are compared but $j$ is the pivot.
![[Diagram1 20-02.png]]
Let us assume that $j>i$ (the opposite case is analogous). In the image we can see the whole array, with all the positions from $0$ to $n$, and the specific positions $i$ and $j$. To understand the probability of $Y_{i,j}$ assuming $1$, we need to understand the probability of the pivot selection; we identify the following cases:
- The pivot gets selected in the red area (before $i$ or after $j$): then this gives us no information, as it will mean that there is still potential for $i$ and $j$ to be compared; the pivot being chosen in this range does not affect the outcome of $Y_{i,j}$, so we consider the remaining $j-i+1$ possible position for picking the pivot.
- The pivot is chosen between $i$ and $j$ (both excluded): this means that $i$ and $j$ will end up in different sets in the tree and so will never be compared, meaning that it would cause $Y_{i,j}=0$ since it would violate the first condition ($i$ compared with $j$).
- The chosen pivot is $i$: in this case $i$ and $j$ are compared, but the second condition is violated ($j$ being the pivot), so $Y_{i,j}=0$.
- The chosen pivot is $j$: this is the only case where $Y_{i,j}=1$.

Given this analysis we can see that $\mathbb{P}(Y_{i,j}=1)=\frac{1}{j-i+1}$ since we have only one favorable case over all possible pivots.

Going back to the expected depth of node $x_{i}$, we get $\sum_{j=1}^{n}\mathbb{E}[Y_{i,j}]=\sum_{j=1}^{n}\frac{1}{|j-i|+1}$, the absolute value is due to the fact that before we considered the analysis only on $j>i$, but we can easily divide the summation in two parts, one when $j$ is smaller than $i$ and vice versa, obtaining: $\sum_{j=1}^{n}\frac{1}{|j-i|+1}=\sum_{j=1}^{i-1}\frac{1}{i-j+1}+\sum_{j=i+1}^{n}\frac{1}{j-i+1}$. We can perform a change of variable ($k=i-j$ for the first summation and $k=j-i$ for the second summation) and obtain $\sum_{k=1}^{i-1}\frac{1}{k+1}+\sum_{k=1}^{n-i}\frac{1}{k+1}$. Since for every $k$ it holds that$\frac{1}{k+1}\leq\frac{1}{k}$, and $i-1\leq n$ and also $n-i\leq n$, the summation can be upper-bounded by $\sum_{k=1}^{n}\frac{1}{k}+\sum_{k=1}^{n}\frac{1}{k}=2\cdot \sum_{k=1}^{n}\frac{1}{k}$. That is an harmonic series, so we get the commonly known upper-bound: $2\cdot \sum_{k=1}^{n}\frac{1}{k}\leq 2\ln(n)$.

Analyzing the expected size of the subtree for $x_{i}$ consists in calculating how may nodes are under it, so how many nodes have been compared with $x_{i}$ when it was selected pivot. The analysis is parallel to that of the depth, with the difference that the indicator variable takes into account when $i$ has been pivot: $$\begin{align}
    Z_{i,j} = \begin{cases}
            1 \quad\text{if $i$ is compared with $j$ and $i$ was the pivot}\\
            0 \quad\text{otherwise}
           \end{cases}
\end{align}$$ The size of the subtree from node $x_{i}$ is clearly $\sum_{j=1}^{n}Z_{i,j}$, so the analysis continues like the previous one, with the exception that now the "winning" case is when $i$ is the pivot, not $j$.
![[Diagram2 20-02.png]]
So, just like before, we get $\sum_{j=1}^{n}\mathbb{E}[Z_{i,j}]=\sum_{j=1}^{n}\frac{1}{|j-i|+1}$ , and thus that it is upper-bounded by $2\ln(n)$.