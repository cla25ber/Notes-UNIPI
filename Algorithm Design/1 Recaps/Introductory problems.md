#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">17/02/2026 & 18/02/2026</span></p>

[[AD index]] (1)

### Secretary problem

*Secretaries will be substituted with headphones because of political correctness*

Let us consider $n$ headphones, I want to select the best. My algorithm is to try one by one and buy it if it is better than the one I currently have, how many times I am expected to buy a pair of headphones?
Let us consider the following indicator variable:$$\begin{align}
    X_{i} = \begin{cases}
            1 \quad\text{if the $i\textit{-th}$ headphone is bought}\\
            0 \quad\text{otherwise}
           \end{cases}
\end{align}$$Then, by linearity of expectation, the expected number of purchases will be: $$\mathbb{E}\left[\sum\limits_{i=1}^{n}X_i\right]=\sum_{i=1}^{n}\mathbb{E}[X_i]$$If I have seen $i$ headphones, the probability of buying the $i\textit{-th}$ is the probability of it being the best among those $i$, which, since they are placed randomly, has probability $\frac{1}{i}$. This means that $\mathbb{E}[X_i]=0\cdot\frac{i-1}{i}+1\cdot\frac{1}{i}=\frac{1}{i}$. Therefore, the expected number of purchases is:$$\sum_{i=1}^{n}\mathbb{E}[X_i]=\sum_{i=1}^{n}\frac{1}{i}=O(\ln(n))$$
#### Random permutations

An adversary might place the headphones in such a way that we need to buy all of them, so we might want to randomize the input before we start the algorithm. To do so, the Fisher-Yates algorithm permutes it in $O(n)$ time, with every configuration possible equally random.
```Python
	def shuffle(X)
		n = X.length
		for i in [1,...,n]
			j = random(i, n)
			swap(i, j)
```

This is able to create every possible configuration each with equal probability, i.e. $\frac{1}{n!}$.

### Randomized Quicksort

In randomized Quicksort the pivot is chosen at random, how many comparison can we expect to be performed in total?
Let us consider the following indicator variable where we consider the numbers already ordered:$$\begin{align}
    X_{i,j} = \begin{cases}
            1 \quad\text{if $i$ has been compared with $j$}\\
            0 \quad\text{otherwise}
           \end{cases}
\end{align}$$Then, the number of total comparisons will be $\sum\limits_{i<j}\mathbb{E}[X_{i,j}]$.
This is because every couple of items is compared at most once, and that happens only if one of the two is a pivot. Therefore, we can see that at each iteration when the pivot gets selected we have 3 different cases:
- The pivot is selected between $i$ and $j$: then the two will end up in different branches of Quicksort and will never be compare, so $X_{i,j}=0$.
- The pivot selected is $i$ or $j$: then they will be compared, and so $X_{i,j}=1$.
- The pivot selected is outside of the interval between $i$ and $j$: in that case it is still possible to compare $i$ and $j$ and the previous two cases are applied.

Between $i$ and $j$ included, assuming $j$ is bigger than $i$, there are $j-i+1$ elements, but only $2$ of them cause the random variable to assume the value $1$ ($i$ and $j$ exactly). This means that:$$\mathbb{E}[X_{i,j}]=\frac{2}{j-i+1}$$Therefore we have an expected number of comparisons of:$$\sum\limits_{i=1}^n\sum\limits_{j=i+1}^n\frac{2}{j-i+1}=2\sum_{i=1}^n\sum_{k=1}^{n-i}\frac{1}{k+1}\leq2\sum_{i=1}^n\sum_{k=1}^{n-i}\frac{1}{k}\leq2\sum_{i=1}^n\log(n-i)=2n\log(n-i)=O(n\log(n))$$
### Karp-Rabin fingerprint

Let us consider 2 sequences of $n$ bits $S$ and $S'$, we want to check whether they are the same sequence efficiently. Instead of comparing the strings, we hash them and compare the hash. Hashing can have a cost of $O(n)$, but then is done just once for every string, then from that moment on, the comparison will be efficient.
To do so, we consider a range $[2,\dots,\tau]$, with $\tau$ chosen depending on the input size; then, we choose a random prime $p\in[2,\dots,\tau]$ that will be used to calculate the fingerprint $F(S)=S\,\%\,p$. In general, we say that a prime $p$ is bad for $k_1\neq k_2$ when $k_1\,\%\,p=k_2\,\%\,p$. We can calculate the probability of error in the following way: $\mathbb{P}(\mathit{error})=\mathbb{P}(k_1\,\%\,p=k_2\,\%\,p)$. Given $k_1$ and $k_2$, we have to consider the number of bad primes for them given the total number of primes in the interval. The latter can be over-approximated as $\frac{\tau}{\ln(\tau)}$ using the prime number theorem.
Regarding the number of bad primes, let us consider $D=|k_1-k_2|$, if $p$ divides both $k_1$ and $k_2$, then it must also divide $D$. Let us factorize $D$ as $p_1^{e_1}\cdot\dots\cdot p_r^{e_r}$; then, all of those prime will be bad primes, meaning that we will have $r$ bad primes. Since $k_1,k_2\in[0,2^n]$, then $D\in[0,\dots,2^n]$, which means that $r<n$, otherwise we would have that $D>2^n$. Consequently:$$\mathbb{P}(\mathit{error})\leq\frac{n}{\frac{\tau}{\ln(\tau)}}$$ This mans that if we take a $\tau$ such that $\frac{\tau}{\ln(\tau)}=n^{c+1}$ with $c>1$, what we get is that the probability of error of the Karp-Rabin fingerprint will be less than: $\frac{1}{n^c}$, meaning that, with high probability, we will not have an error. This is a Monte Carlo algorithm.

<span style="color:rgb(0, 112, 192)">Monte Carlo algorithms:</span> fast but probability of error.
<span style="color:rgb(0, 112, 192)">Las Vegas algorithms:</span> correct but probability of being slow.

#### Rolling hash

Let us consider a text $T$ of length $n$, and a pattern $P$ of length $m$. Searching the pattern $P$ inside of $T$ takes $O(n\cdot m)$ time if we brute force it.
We can find the pattern more efficiently by considering $|\Sigma|=\sigma$ the size of the alphabet, then we map every letter of the alphabet to the set $[0,\cdots,\sigma-1]$ through a function $f$. Once we have done that, we can turn every string $A=a_1a_2\dots a_n$ into a number by performing:$$\sum_{i=1}^nf(a_i)\cdot\sigma^{i}$$This value will be called the fingerprint of the string. To find all the occurrences of a pattern, we then take its fingerprint and we calculate the fingerprint of every substring of the same length of the pattern at any position. A naïve algorithm to do so would still cost $O(n\cdot m)$; for this reason, what we do is rolling hash. When we have calculated the fingerprint of a substring, we subtract its biggest element, multiply by $\sigma$ all the remaining ones, and add the new smallest by considering the next character in the string. This allows us to calculate the hash of the next substring from the previous one in constant time, meaning that now the cost of the algorithm is $O(n+m)$.