#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">06/03/2026</span></p>

[[AD index]] (2)

**Author:** Claudio Bernardoni

**Hands-on:** 
![[Hands-on 06-03.png]]
**Proof:**

Given $x\in U$, if $B_{S}[h(x)]=0$ then we can be sure that $x$ was not present in $S$, since its bit would be 1; instead, if $B_{S}[h(x)]=1$, we cannot be sure about the presence of $x$ in $S$ because that bit might have been lit by another value in $S$.

Given $x\not\in S$, let us calculate the probability of error, i.e. the probability that its bit is still 1. $$\mathbb{P}(B_{S}[h(x)]=1)\ =\ 1-\mathbb{P}(B_{S}[h(x)]=0)$$However, for the opposite event to occur, every value $y\in S$ must not have its hash value equal to that of $x$, so: $$1-\mathbb{P}(B_{S}[h(x)]=0)\ =\ 1-\mathbb{P}(\forall y\in S.h(y)\neq h(x))$$Given the fact that they are $n$ independent events, we get that: $$1-\mathbb{P}(\forall y\in S.h(y)\neq h(x))\ =\ 1-\mathbb{P}(h(y)\neq h(x))^{n}\ =\ 1-(1-\mathbb{P}(h(y)=h(x)))^{n}$$Now, since $h\in\mathcal{H}$ is universal, the probability of two keys having the same hash value is $\frac{1}{m}$, so: $$1-(1-\mathbb{P}(h(y)=h(x)))^{n}\ =\ 1-\left(1-\frac{1}{m}\right)^{n}$$ 
Let us consider the indicator variable:
$$\begin{align}
    X_{i} = \begin{cases}
            1 \quad\text{if the $i$\textit{-th} bit of $B_{S}$ is 1}\\
            0 \quad\text{otherwise}
           \end{cases}
\end{align}$$
Then, clearly, the random variable $X=\sum_{i=0}^{m-1}X_{i}$ counts the number of 1s in $B_{S}$. Let us now calculate the expected number of 1s in $B_{S}$, i.e. $\mathbb{E}[X]=\mathbb{E}[\sum_{i=0}^{m-1}X_{i}]=\sum_{i=0}^{m-1}\mathbb{E}[X_{i}]$ by linearity of expectation.
By definition of expected value $\mathbb{E}[X_{i}]\ =\ \mathbb{P}(X_{i}=1)$; this happens if there exists a $y\in S$ such that $h(y)=i$, so: $$\mathbb{E}[X_{i}]\ =\ \mathbb{P}(\exists y\in S.h(y)=i)\ =\ 1-\mathbb{P}(\forall y\in S.h(y)\neq i)$$Using the previous result, we get that: $$\mathbb{E}[X_{i}]\ =\ 1-\left(1-\frac{1}{m}\right)^{n}$$Therefore, going back to the expected number of 1s in $B_{S}$: $$\mathbb{E}[X]\ =\ \sum_{i=0}^{m-1}1-\left(1-\frac{1}{m}\right)^{n}\ =\ m\cdot\left(1-\left(1-\frac{1}{m}\right)^{n}\right)$$
Since $S$ has been lost, we do not know the exact value of $n$; nonetheless, we can estimate it. We know that $n\leq\frac{m}{c}$, but also that it must be bigger than the number of 1s present in $B_{S}$, since we need at least that number of keys to lit that many bits.
Given these considerations, it is clear that from $n_{1}\leq n\leq \frac{m}{c}$, where $n_{1}$ is the exact number of 1s in $B_{S}$, we can derive that: $$m\cdot\left(1-\left(1-\frac{1}{m}\right)^{n_{1}}\right)\ \leq\ m\cdot\left(1-\left(1-\frac{1}{m}\right)^{n}\right)\ \leq\ m\cdot\left(1-\left(1-\frac{1}{m}\right)^{\frac{m}{c}}\right)$$

Having $B_{S}$, we can count the exact number of 1s $n_{1}$, which would be approximately the expected value of the number of 1s: $\mathbb{E}[X]\approx n_{1}$. So we can say that: $$n_{1}\ =\ m\cdot\left(1-\left(1-\frac{1}{m}\right)^{n}\right)$$Which means that: $$1-\frac{n_{1}}{m}\ =\ \left(1-\frac{1}{m}\right)^{n}$$And so: $$n\ =\ \frac{\ln(\frac{m-n_{1}}{m})}{\ln(\frac{m-1}{m})}$$
