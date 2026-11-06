#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">03/03/2026 & 04/03/2026</span></p>

[[AD index]] (1)

### Bloom filter

Let us consider a set $S\subseteq U$ of $n$ keys, we want to check for membership on $S$ without having to iterate and compare all the elements, to do so we perform the non-deterministically procedure (that might have an error) given by the Bloom filter.
We choose $h_1,\dots,h_k:U\to[m]$ hash functions from a class $\mathcal{H}$ of universal hash functions. We also have a bit array $B$ of size $m$ all initialized with $0$. First, for every $x\in S$ we calculate $h_1(x),\dots,h_k(x)$, and we lit to $1$ every bit that is the result of a hash function; from this moment forward, we do not need anymore $S$ to check for membership.
When we want to check if an element $y$ is in $S$, we just check all the bits $B[h_1(y)],\dots,B[h_k(y)]$ and see if all of them are lit:
- If there is even one that is not lit, then we can be sure that $y\notin S$.
- If all of them are lit we cannot be sure whether $y$ is in $S$ or not, since it could be a false positive given by other elements lighting those bits.

Now, we want to calculate the probability of error. Let us fix a certain hash function $h_i$, the probability of a bit being lit $\mathbb{P}(B[q]=1)$ is the probability of an element $x$ such that $h_i(x)=q$, and since all $h_i$ are universal, that is $\frac{1}{m}$, meaning that the probability of it not being lit is $1-\frac{1}{m}$.
Noe, we have $k$ of such functions, so the probability of a bit not being lit is the probability of no function lighting that position, and since they are all independent functions, that is $(1-\frac{1}{m})^k$.
We have checked a position considering an element, but we have to consider all of them, meaning that the total error probability is $(1-\frac{1}{m})^{k\cdot n}$.

### Min hash

The Bloom filter checks efficiently (even though with error) the membership in a set, but we can also check for similarity of two sets. Given $A$ and $B$, we use Jaccard similarity to check for similarity:$$J(A,B)=\frac{|A\cap B|}{|A\cup B|}\in[0,1]$$If we iterate on all the elements, this takes $O(|A|+|B|)$ time. Instead, we consider a permutation $\pi_i$ of the elements of the universe $U$, and $\text{min}_i(S)$ the minimum of the set $S$ according to permutation $\pi_i$. We can use these with the following indicator variable:$$\begin{align}
    X_{i} = \begin{cases}
            1 \quad\text{if $\text{min}_i(A)=\text{min}_i(B)$}\\
            0 \quad\text{otherwise}
           \end{cases}
\end{align}$$Clearly, $\mathbb{E}[X_i]=\mathbb{P}(\text{min}_i(A)=\text{min}_i(B))=\frac{|A\cap B|}{|A\cup B|}=J(A,B)$, meaning that it is an approximator of Jaccard similarity. 

However, using permutation yields some problems: first, permuting $U$ is very expensive, and also, even though $X_i$ has good expected value, it has terrible variance (extracting an element that is from $A\cap B$ results in a 100% similarity). For this reason, we instead use hash functions, specifically a family of hash functions $U\to[|U|]$ that permutes $U$ (i.e. $a<b$ iff. $h(a)<h(b)$). Now permuting is far less costly, however, to estimate Jaccard similarity we still have to repeat $k$ experiments, and the average is the correct approximation of $J(A,B)$.