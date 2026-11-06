#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">27/02/2026</span></p>

[[AD index]] (2)

**Author:** Claudio Bernardoni

**Hands-on:** ![[Hands-on 27-02.png]]
**Proof:**

Given an alphabet $\Sigma$ of size $\sigma=|\Sigma|$, we create a bijection $t:\Sigma\to[\sigma]$ (where $[\sigma]$=$\{0,\dots,\sigma-1\}$) that translates every symbol of the alphabet into a number from $0$ to $\sigma-1$.
Now that we have a numerical value associated with every symbol of the alphabet, we can also associate every string with a number, in particular a string $S$ of length $n$ will be associated with its fingerprint $F_{S}=\sum_{i=0}^{n-1}t(S[i])\cdot\sigma^{i}$ .

The problem with this fingerprint is that, if $a\in\Sigma$ is the element such that $t(a)=0$, then we get that for every string $S$, $F_{S}=F_{aS}=F_{aaS}=\dots$ which would make the fingerprint not unique anymore. To solve this problem we consider the alphabet $\$\cup\Sigma=\Sigma_{\$}$, where $\$\not\in\Sigma$ is a metacharacter that cannot be used (since strings remain on alphabet $\Sigma$), and we consider as the new translation bijection $t_{\$}:\Sigma_{\$}\to[\sigma+1]$ such that: $$\begin{align}
    t_{\$}(x) = \begin{cases}
            0 \quad\quad\quad\quad\text{if $x=\$$}\\
            t(x)+1 \quad\text{otherwise}
           \end{cases}
\end{align}$$Now the fingerprint of a string $S$ is calculated as follows: $F_{S}=\sum_{i=0}^{n-1}t_{\$}(S[i])\cdot(\sigma+1)^{i}$ . The fingerprint is now unique since every string gets translated into a unique number in base $\sigma+1$; also, the complexity of calculating the fingerprint of a string is clearly $O(n)$, where $n$ is its length.

Given a string $S$, we are going to pre-process it by calculating the fingerprint of every suffix; normally, this would take $O(n^{2})$ time, but since one suffix is equal to its predecessor expect for the starting character, we can calculate it in a faster way. As we said, we can calculate the next suffix by removing the first character: $F_{S[i+1\cdots n]}= F_{S[i\cdots n]}-t_{\$}(S[i])\cdot(\sigma+1)^{n-i}$ , which means that after calculating the fingerprint $F_{S}$ of the whole string with $O(n)$ cost, we calculate every other suffix in constant time. Since there are $O(n)$ suffixes we get a total cost of $O(2n)=O(n)$ for the pre-processing.

If the string is long enough, the fingerprint of all the suffixes of the string can become rather costly on space; for this reason, they are stored using Karp-Rabin hashing with a big enough prime $p$ to minimize the probability of error, i.e. the probability of clashing. When defining the operations $\mathit{equal}_{S}$ and $\mathit{lce}_{S}$ we are going to assume that there was no error in Karp-Rabin hashing, and thus that every fingerprint when hashed remains unique. The hashed fingerprints are small enough to be considered $O(1)$ in space occupation, for a total of $O(n)$ space occupation.

##### $\mathit{equal}_{S}$ implementation

Let us consider first $\mathit{equal}_{S}(i,j,l)$; this returns true if the substrings $S[i\dots i+l]$ and $S[j\dots j+l]$ are equal. For simplicity, we will call the fingerprint of the suffix starting in position $i$ as $F_{i}$ instead of $F_{S[i\dots n]}$. Let us take the fingerprint $F_{i}$, by subtracting to it $F_{i+l+1}$ we get the remaining part of the string, i.e. the part from $i$ to $i+l$. However, this value is not the fingerprint $F_{S[i\dots i+l]}$, this is because it is not its representation in base $\sigma+1$, its smallest term, i.e. $t_{\$}(S[i+l])$, is multiplied by a factor of $(\sigma+1)^{n-(i+l)}$. For this reason we divide by that factor to obtain the actual representation of the string. We do the same for $F_{j}$ and thus obtain $F_{S[i\dots i+l]}$ and $F_{S[j\dots j+l]}$, so we can see if they are equal if their subtraction is equal to $0$. ![[Diagram1 27-02.png]]
We saw that this procedure works if we use the fingerprints; however, we said that we only stored the hashed value of the fingerprints. Fortunately, these operations are all sums and subtractions which the modulo $p$ operation does not mutate; even the division by $(\sigma+1)^{n-(i+l)}$ is not a problem because, since $p$ is a prime, every number has a multiplicative inverse, so instead of dividing by it we perform multiplication of its multiplicative inverse.

This function, being only sums, subtractions and modulo, is clearly constant time $O(1)$.

##### $\mathit{lce}_{S}$ implementation

Let us consider now $\mathit{lce}_{S}(i,j)$, which returns the number of characters up to which the strings starting from $i$ and $j$ are equal; i.e., if $\mathit{lce}_{S}(i,j)=l$, then $S[i\dots i+l]=S[j\dots j+l]$. We can implement it as a sort of binary search, where we start by performing $\mathit{equal}_{S}(i,j,n-j)$, which is the maximum length of the substring starting from $j$. If we get true, then the longest common extension is $n-j$, otherwise we continue but on half of the same length, $\mathit{equal}_{S}(i,j,(n-l)/2)$, continuing up until we get true as a result, because that would mean that we have found part of the longest common extension. Once we find that, we are in a situation where on the left we have that $\mathit{equal_{S}}$ returned true, while on the right it returned false, so we start searching in that segment as before.

This is clearly a binary search on $n-j\leq n$ elements, so we get a cost in time of $O(\log(n))$.

##### Universal hashing

If we substituted Karp-Rabin hashing with universal hashing dependent on two parameters $a\in\{1,\dots,p-1\}$ and $b\in\{0,\dots,p-1\}$ of the type $\mathcal{H}\ni h:U\to[m]$ where $U$ is the set of keys and $h_{a,b}(x)=(ax+b\mod p)\mod m$. We showed that the class of functions $\mathcal{H}$ is universal; i.e., given any $k_{1},k_{2}\in U$, $\mathbb{P}(h_{a,b}(k_{1})=h_{a,b}(k_{2}))\leq\frac{|\mathcal{H}|}{m}$ for any random $h_{a,b}\in\mathcal{H}$.

The generic formula for rolling hash is $F_{S[i\dots j]}=(F_{i}-F_{j+1})/(\sigma+1)^{n-j}$, so we would get that: $$h_{a,b}(F_{S[i\dots j]})=(a(F_{i}-F_{j+1})/(\sigma+1)^{n-j}) +b\mod p\mod m$$Focusing in the internal part, we see that $a(F_{i}-F_{j+1})=aF_{i}+b-aF_{j}-b=h_{a,b}(F_{i})-h_{a,b}(F_{j})$, meaning that to calculate the fingerprint of any substring we just need the hash of the suffixes, so rolling hash is preserved.

Regarding the new modulo operations, since $p$ is a prime it does not affect the rolling hash. The only problem could be that since $m$ is not necessarily a prime, even if additions and subtractions are not affected, we might not be able to perform the division by $(\sigma+1)^{n+j}$. This means that the only constraint we have on the choice of $m$ is that it is not a multiple of $\sigma+1$, i.e. $\mathit{GCD}(m,\sigma+1)=1$, that way we can be sure that it will have a multiplicative inverse and the rolling hash can still be performed.

In conclusion, using universal hashing allows us to still perform rolling hash.