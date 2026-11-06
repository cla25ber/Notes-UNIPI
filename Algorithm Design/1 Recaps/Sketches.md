#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">10/03/2026 & 11/03/2026</span></p>

[[AD index]] (1)

### Sketches

We want to estimate the frequency of items from $0$ to $n-1$ in an unbounded stream; on request we can provide an estimate of the frequency of the items seen so far. We could keep an array $F$ where $F[i]$ contains the frequency of item $i$; however, we want to use as little space as possible, so we will use an approximation $\tilde{F}$ of the array $F$ with bounded error probability.
We will build an approximation array $\tilde{F}$ such that:$$F[i]\leq\tilde{F}[i]\leq F[i]+\epsilon\|F\|$$where $\|F\|=\sum_{i=0}^{n}F[i]$, so it is just the stream length.
Considering how we will build the array $\tilde{F}$, the first inequality $F[i]\leq\tilde{F}[i]$ is always true.
However, the second inequality $\tilde{F}[i]\leq F[i]+\epsilon\|F\|$ holds with probability $\delta$, meaning that we find ourselves in a FPTAS scenario, with $\epsilon$ being the accuracy, and $\delta$ being the probability of error.

Now, let us build the actual approximation, we will first build a table $T$ with $c=\frac{e}{\epsilon}$ columns and $r=\ln\delta^{-1}$ rows. Clearly, the space occupancy of the table is $O(r\cdot c)=O(\epsilon^{-1}\,\ln\delta^{-1})$. Each entry of the table is a counter initialized at $0$. Then, we independently choose $h_0,\dots,h_{r-1}:[n]\to[c]$ hash functions from an universal hash family $\mathcal{H}$.
When a new item $i$ is read, instead of adding $1$ to $F[i]$, we add $1$ to every counter at position $(0,h_0(i)),\dots,(r-1,h_{r-1}(i))$. Therefore, adding a new element has a cost of $O(r)$.
When we want to query the frequency of an item $i$, what we actually do is take an approximation by returning $\tilde{F}[i]=\text{min}\{T[0][h_0(i)],\,\dots,\,T[r-1][h_{r-1}(i)\}$. We return the minimum because it is the closest approximation to the real frequency.
What can happen is that if there are some $i\neq j$ such that there exists a $k$ for which $h_k(i)=h_k(j)$, we will have a collision:
![[Sketches collisions.png]]Like in the image above, some cells will be upgraded even for the other values, meaning that their count will not just reflect that of just $i$ or just $j$. Now it is clear that $F[i]\leq\tilde{F}[i]$ is true by construction.

Let us define the following indicator variable:$$\begin{align}
    I_{kij} = \begin{cases}
            1 \quad\text{if $h_k(i)=h_k(j)$}\\
            0 \quad\text{otherwise}
           \end{cases}
\end{align}$$This indicator variable tells whether item $i$ and $j$ share the same cell at row $k$.
Also let us define the following random variable:$$X_{ki}=\sum_{x\in[n]\backslash\{i\}}I_{kix}\cdot F[x]$$which counts the number of other items sharing the same cell of $i$ at row $k$.
Now, we can neatly express the value inside the cells of the table:$$T[k][h_k(i)]=F[i]+X_{ki}$$
Let us now calculate the expected value of the previous random variables:$$\mathbb{E}[I_{kij}]=\mathbb{P}(I_{kij}=1)=\mathbb{P}(h_k(i)=h_k(j))=\frac{1}{c}=\frac{\epsilon}{e}$$This is due to the fact that the hash functions are universal.
Now, for the other random variable:$$\mathbb{E}[X_{ki}]=\mathbb{E}\left[\sum_{x\in[n]\backslash\{i\}}I_{kix}\cdot F[x]\right]=\sum_{x\in[n]\backslash\{i\}}\mathbb{E}[I_{kix}]\cdot F[x]=\sum_{x\in[n]\backslash\{i\}}\frac{\epsilon}{e}\cdot F[x]\leq\frac{\epsilon}{e}\sum_{x\in[n]}F[x]=\frac{\epsilon}{e}\|F\|$$which consequently means that $e\cdot\mathbb{E}[X_{ki}]\leq\epsilon\cdot\|F\|$.

Now, let us first recall Markov's inequality:$$\mathbb{P}[X>a]\leq\frac{\mathbb{E}[X]}{a}$$Then, let us calculate the probability of error of the other inequality $\tilde{F}[i]> F[i]+\epsilon\|F\|$. Since $\tilde{F}[i]$ is a minimum operation, it must hold for any hash function, i.e. for any $k\in[r]$, that $T[k][h_k(i)]\geq\tilde{F}[i]$, and thus that $T[k][h_k(i)]=F[i]+X_{ki}>F[i]+\epsilon\|F\|$, and therefore $X_{ki}>\epsilon\|F\|$.
$$\mathbb{P}(\tilde{F}[i]> F[i]+\epsilon\|F\|)=\mathbb{P}(\forall k\in[r].X_{ki}>\epsilon\|F\|)=\prod_{k\in[r]}\mathbb{P}(X_{ki}>\epsilon\|F\|)$$Now, since $e\cdot\mathbb{E}[X_{ki}]\leq\epsilon\cdot\|F\|$, then $\mathbb{P}(X_{ki}>\epsilon\|F\|)\leq\mathbb{P}(X_{ki}>e\cdot\mathbb{E}[X_{ki}])$, so:$$\prod_{k\in[r]}\mathbb{P}(X_{ki}>\epsilon\|F\|)\leq\prod_{k\in[r]}\mathbb{P}(X_{ki}>e\cdot\mathbb{E}[X_{ki}])$$Here we can use Markov's inequality:$$\prod_{k\in[r]}\mathbb{P}(X_{ki}>e\cdot\mathbb{E}[X_{ki}])\leq\prod_{k\in[r]}\frac{\mathbb{E}[X_{ki}]}{e\cdot\mathbb{E}[X_{ki}]}=\prod_{k\in[r]}\frac{1}{e}=\left(\frac{1}{e}\right)^r=e^{-\ln(\delta^{-1})}=\delta$$Meaning that the probability of error can be upper-bounded by $\delta$.

### Sketches of sets

We want a compact (lossy) representation of sets to allow for some useful operation (like Jaccard); so, given a set $A\subseteq U$, we want to create a sketch $S(A)\subseteq A$ that must support the following operations:
- **INIT**: build $S(A)$ from $A$.
- **UPDATE**: given an element $u$ to add, get $S(A\cup\{u\})$ from $A$.
- **UNION**: get $S(A\cup B)$ from $S(A)$ and $S(B)$.
- **SIZE**: estimate $|A|$ from $S(A)$.

Like in min-hash, we are going to use hash functions as permutations.

#### k-min

We randomly choose $h_1,\dots,h_k:U\to(0,1]$ hash functions of a universal hash function family $\mathcal{H}$. They map each element of the universe to $\frac{1}{|U|},\dots,\frac{|U|}{|U|}$, e.g. the order $abcde$ gives the following map: $h(a)=0.2, h(b)=0.4, h(c)=0.6, h(d)=0.8, h(e)=1$. 

- **INIT**: When creating the sketch from set $A$, we take the minimum of each function, i.e. we create the set $\{a_1\dots, a_k\}$ composed of $a_i=\text{min}(h_i(A))$
- **UPDATE**: when a new element $u$ gets inserted, it is checked whether it will substitute other elements, so the new set will be that composed of $a_i'=\text{min}(h_i(a_i), h_i(u))$.
- **UNION**: we must keep the smallest element of the two sets $A$ and $B$, so the new set will be that composed of $a_i'=\text{min}(h_i(a_i),h_i(b_i))$
- **SIZE**: to calculate an approximation of the size we consider the following formula:$$|A|\approx\frac{k}{\sum\limits_{i\in[k]}h_i(a_i)}-1$$
As an example, let us consider $U=\{a,b,c,d,e\}$ with $A=\{a,c,d,e\}$ and $B=\{b,c,d\}$. Let us consider the following hash functions: $k=3: h_1=abcde, h_2=ecdba, h_3=bdaec$. Then, we get:
- $S(A)=[a,e,d]$ - $|A|\approx\frac{3}{0.8}-1=2.75$
- $S(B)=[b,c,b]$  - $|B|\approx\frac{3}{1}-1=2$
- $S(A\cup B)=[a,e,b]$ - $|A\cup B|\approx\frac{3}{0.6}-1=4$

#### Bottom-k

Now let us consider a single hash function $h:U\to(0,1]$ that permutes the elements of $U$ the same way as before, i.e. by mapping them to $\frac{1}{|U|},\dots,\frac{|U|}{|U|}$. Now, as a sketch of $A$, we take the $k$ smallest elements according to $h$, so we get:
- **INIT**: $k$ smallest elements of $A$.
- **UPDATE**: if $u$ is smaller than the biggest of the $k$ smallest elements, the biggest is kicked out, otherwise $u$ is not considered.
- **UNION**: take the $k$ smallest elements of the set of the $k$ smallest elements of $A$ and of $B$.
- **SIZE**: as an approximation we take the following formula:$$\frac{k-1}{h(a_k)}$$ where $a_k$ is the biggest among the smallest $k$ elements.

As an example, let us consider the same universe as before with $A=\{a,c,d,e\}$ and $B=\{b,c,d,e\}$, and the hash functions: $h=abcde$ with $k=2$. Then, we get:
- $S(A)=[a,c]$ - $|A|\approx\frac{2-1}{0.6}=1.666$
- $S(B)=[b,c]$  - $|B|\approx\frac{2-1}{0.6}=1.666$
- $S(A\cup B)=[a,b]$ - $|A\cup B|\approx\frac{2-1}{0.4}=1.5$

Even though bottom-k sketches tend to give worse approximations, it is notable that $S(S(A)\cup S(B))=S(A\cup B)$, this allows for a better estimation of Jaccard similarity starting from just $S(A)$ and $S(B)$.
We can estimate it with:$$\tilde{J}(A,B)=\frac{|S(A\cup B)\cap S(A)\cup S(B)|}{k}$$Let us consider the following division of sets:
![[Set intersection.png]]
Then, if we extract $k$ random items, we expect $k\cdot\frac{y}{g+y+r}$ of them to be from $A\cap B$. We notice that $J(A,B)=\frac{y}{g+y+r}$, which means that extracting $k$ random items from $A\cup B$, i.e. $S(A\cup B)$, and then estimating how many of them are both in $A$ and $B$ consists in the previously described procedure, and therefore:$$\tilde{J}(A,B)=\frac{|S(A\cup B)\cap S(A)\cup S(B)|}{k}\approx\frac{k\cdot\frac{y}{g+y+r}}{k}=J(A,B)$$
#### Threshold-t sketches

We consider again a single hash function $h:U\to(0,1]$ that permutes the elements of $U$ the same way as before, i.e. by mapping them to $\frac{1}{|U|},\dots,\frac{|U|}{|U|}$; however, we also consider a threshold value $t\in(0,1]$ that influences the operations in the following way:
- **INIT**: $S(A)=\{a\in A | h(a)\leq t\}$.
- **UPDATE**: $S(A)\cup\{u\}$ if $h(u)\leq t$, $S(A)$ otherwise.
- **UNION**: $S(A\cup B)=S(A)\cup S(B)$.
- **SIZE**: $\frac{S(A)}{t}$

Now we sort of have a choice in the elements that get added to the sketch, but we do not have anymore any clear bound in the size of the sketch.