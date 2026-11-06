#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">24/02/2026 & 25/02/2026</span></p>

[[AD index]] (1)

### Cuckoo hashing

Let us consider two hash functions $h_1,h_2:U\to[m]$, where $[m]$ denotes the set $\{0,\dots,m-1\}$. In cuckoo hashing, when trying to hash an element $x\in U$, we try first to it it in position $h_1(x)$, and if that is already taken, we go to $h_2(x)$. If both are already occupied, then one is kicked out and we have to re-hash that element, hoping that one of its two spot are free. Clearly, this hashing method is not perfect because if we get two cycles, one for $h_1(x)$ and one for $h_2(x)$, it means that we cannot place all the items with the current hash functions. However, with this setup, lookup and deletion are $O(1)$, while insertion is $O(1)$ expected time, so it is really fast.

To see that insertion is expected $O(1)$ time, let us consider a set $S\subseteq U$ with $|S|=n$ of keys to insert. We will build a multi-graph  $G=(V,E)$ (more than one edge per pair of nodes), where $V=[m]$, and $E=\{(h_1(x),h_2(x))\ |\ x\in S\}$. Let us consider also the case where $m>2\,c\,n$, so we have a table bigger than $2\,n$ by a constant factor $c>2$.
When adding element $x\in S$ to the table there are 3 possible cases:
1) The element is added because $h_1(x)$ or $h_2(x)$ were free.
2) Both $h_1(x)$ or $h_2(x)$ are occupied but we find in $l$ steps a free spot to satisfy every element.
3) We are not able to find a spot to fit all the elements, i.e. we end up in the previously discussed bad case. To see that we ended up in the bad case we can check if we have moved elements more than 2 times the number of elements already inserted.

In case 1 we have insertion with time $O(1)$, so we will focus on the other 2 cases.

In case 2 we consider the following lemma: $$\mathbb{P}(\text{path}_{l}(i,j))\leq\frac{1}{c^l\,m}$$**Proof:**

**Base case:** $l=1$,

$\mathbb{P}(\text{path}_{l}(i,j))=\mathbb{P}(\exists k.E_1\lor E_2)$, where $E_1 \triangleq h_1(k)=i\land h_2(k)=j$ and $E_2 \triangleq h_1(k)=j \land h_2(k)=i$. Using the union-bound, we can upper-bound the previous probability with: $$\mathbb{P}(\exists k.E_1\lor E_2)\leq\sum_{k\in S}\mathbb{P}(E_1\lor E_2)\leq\sum_{k\in S}\mathbb{P}(E_1)+\mathbb{P}(E_2)$$and since they are equally likely events: $$\sum_{k\in S}\mathbb{P}(E_1)+\mathbb{P}(E_2)=\sum_{k\in S}2\,\mathbb{P}(E_1)$$Since the two hash functions are independent :$$\sum_{k\in S}2\,\mathbb{P}(h_1(k)=i\land h_2(k)=j)=\sum_{k\in S}2\,\mathbb{P}(h_1(k)=i)\cdot\mathbb{P}( h_2(k)=j)$$and since every spot in the table is equally likely to be chosen by the function we have that:$$\sum_{k\in S}2\,\mathbb{P}(h_1(k)=i)\cdot\mathbb{P}( h_2(k)=j)=\sum_{k\in S}2\frac{1}{m^2}=\frac{2\,n}{m}\cdot\frac{1}{m}$$We know that $m>2\,c\,n$, therefore:$$\frac{2\,n}{m}\cdot\frac{1}{m}\leq\frac{2\,n}{m}\cdot\frac{1}{2\,c\,n}=\frac{1}{c\,m}$$
**Inductive step:** We assume that $\mathbb{P}(\text{path of length }l-1)\leq\frac{1}{c^{l-1}\,m}$. 

$\mathbb{P}(\text{path}_l(i,j))=\mathbb{P}(\exists z.\text{path}_{l-1}(i,z)\land \text{path}_1(z,j))$. Using the union-bound we get that:$$\mathbb{P}(\exists z.\text{path}_{l-1}(i,z)\land \text{path}_1(z,j))\leq\sum_{z\in\{1,\dots,m\}}\mathbb{P}(\text{path}_{l-1}(i,z)\land \text{path}_1(z,j))$$Since the two paths are independent events:$$\sum_{z\in\{1,\dots,m\}}\mathbb{P}(\text{path}_{l-1}(i,z)\land \text{path}_1(z,j))=\sum_{z\in\{1,\dots,m\}}\mathbb{P}(\text{path}_{l-1}(i,z))\cdot \mathbb{P}(\text{path}_1(z,j))$$Then, using the inductive hypothesis:$$\sum_{z\in\{1,\dots,m\}}\mathbb{P}(\text{path}_{l-1}(i,z))\cdot \mathbb{P}(\text{path}_1(z,j))\leq\sum_{z\in\{1,\dots,m\}}\frac{1}{c^{l-1}\,m}\cdot\frac{1}{c\,m}=m\cdot\frac{1}{c^l\,m^2}=\frac{1}{c^l\,m}$$
$\square$

The insertion has cost $O(1+l)$, but the expected length of $l$ can be upper-bounded by $\sum_{l\geq1}l\cdot\frac{1}{c^l\,m}$ since we sum all possible lengths of the path multiplied by the probability of it being that length. Using some algebra, we can upper-bound such value with $O(1+\frac{1}{m})=O(1)$, so insertion is performed in constant time.

In case 3 we have to consider the possibility of having two cycles:$$\mathbb{P}(\text{having 2 cycles})\leq\mathbb{P}(\text{having 1 cycle})=\mathbb{P}(\exists i.\text{there is a cycle starting in }i)$$This is like an $\lor$ between all possible position of the tables, which means that it can be upper-bounded using the union-bound:$$\mathbb{P}(\exists i.\text{there is a cycle starting in }i)\leq\sum_{i=0}^{m-1}\mathbb{P}(\text{there is a cycle starting in }i)$$The cycle an be of any length:$$\sum_{i=0}^{m-1}\mathbb{P}(\text{there is a cycle starting in }i)=\sum_{i=0}^{m-1}\mathbb{P}(\exists l.\text{there is a path of length }l\text{ from }i\text{ to }i)$$This can also be upper-bounded with the union-bound:$$\sum_{i=0}^{m-1}\mathbb{P}(\exists l.\text{there is a path of length }l\text{ from }i\text{ to }i)\leq\sum_{i=0}^{m-1}\sum_{l\geq1}\mathbb{P}(\text{there is a path of length }l\text{ from }i\text{ to }i)$$We can use the approximations found in case 2:$$\sum_{i=0}^{m-1}\sum_{l\geq1}\mathbb{P}(\text{there is a path of length }l\text{ from }i\text{ to }i)\leq\sum_{i=0}^{m-1}\sum_{l\geq1}\frac{1}{c^l\,m}$$Now, using some algebraic steps:$$\sum_{i=0}^{m-1}\sum_{l\geq1}\frac{1}{c^l\,m}=\sum_{i=0}^{m-1}\frac{1}{m}\sum_{l\geq1}\frac{1}{c^l}=m\cdot\frac{1}{m}\cdot\sum_{l\geq1}\frac{1}{c^l}=\frac{1}{c-1}$$Now, as $c$ grows bigger, and consequently the size of the table, the smaller the probability of having cycles. Let us call $p=\frac{1}{c-1}$ this probability; if we end up in such bad case, then we have to rehash, i.e. select two new $h_1$ and $h_2$ and start adding again the elements of $S$. Rehashing has a cost of $O(n)$ since we might have to re-insert every element of $S$. Since the hash functions are selected randomly, the probability of having to rehash 1 time is $p$, 2 times is $p^2$, 3 times is $p^3$ and so on. The expected number of rehashes is then:$$\sum_{i\geq1}i\cdot p^i=O(1)$$This means that, considering that some bad cases could happen, they will be in constant order, which means that the final cost of insertion of $n$ elements will still be $O(n)$, meaning that insertion per element will be $O(1)$.

### Universal hashing

Given hash functions of the type $U\to[m]$, we say that a family $\mathcal{H}$ of hash functions is universal if given any $k_1,k_2\in U$ the number of $h\in\mathcal{H}$ such that $h_1(k)=h_2(k)$ is less than $\frac{|\mathcal{H}|}{m}$. Therefore, by choosing $h$ uniformly at random, we have that $\mathbb{P}(\text{collision})=\frac{\text{\#bad h}}{|\mathcal{H}|}\leq\frac{\frac{|\mathcal{H}|}{m}}{|\mathcal{H}|}=\frac{1}{m}$. 

An example of family of hash functions is $h_{ab}(k)=((ak+b)\mod p)\mod m$ with $a\in[1,\dots,p-1]$ and $b\in[0,\dots,p-1]$ and $p$ being a prime larger than any $k\in U$; giving us that $|\mathcal{H}|=p\cdot(p-1)$.   

Now we show that this hash function family is universal:

**Proof:**
Let us consider any distinct $x,y\in U$, then let us call $r\triangleq (ax+b)\mod p$ and $s\triangleq (ay+b)\mod p$. Let us now consider $r-s\equiv a(x-y)\mod p$, we know that, since $x\neq y$, $r-s\neq0\mod p$; therefore, this means that $x-y$ has an opposite element which we will call $(x-y)^{-1}$, and thus $a\equiv (r-s)(x-y)^{-1}\mod p$ which means that $a$ is uniquely determined. $b$ is also uniquely determined: $b\equiv r-ax\mod p$.
Therefore, there is a bijection between $(a,b)$ and $(r,s)$. We want to know when $r\equiv s\mod m$, and this happens when $s$ is $r$ or $r+m$ or $r+2m$ and so on. There are at most $\lceil\frac{p}{m}\rceil$ of such values, but since $r\neq s$ they are actually at most $\lceil\frac{p}{m}\rceil-1\leq\frac{p-1}{m}$.
So, for each element $r$, there are at most $\frac{p-1}{m}$ bad partners for it, meaning that at most we will have $\frac{p(p-1)}{m}$ bad couples. Since the total number of couples was $p(p-1)$, we have that the probability of a collision is $\frac{1}{m}$.
$\square$

Universal hashing is very powerful, for example, in hashing with incidence lists, if the hash function used is from a universal hashing family, then the expected length of the incidence lists is $\frac{n}{m}$, which is optimal. 

**Proof:**
let us consider the following random variable:$$\begin{align}
    X_{i,j} = \begin{cases}
            1 \quad\text{if $h(i)=h(j)$}\\
            0 \quad\text{otherwise}
           \end{cases}
\end{align}$$Then, being $h$ universal, $\mathbb{E}[X_{i,j}]=\frac{1}{m}$, thus the expected length length of a specific element $k$ is:$$\sum_{i\neq k}\mathbb{E}[X_{x,i}]\leq\sum_{i\neq k}\frac{1}{m}\leq\frac{n}{m}$$$\square$