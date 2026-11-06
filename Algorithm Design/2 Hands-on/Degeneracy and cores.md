#computer_science #algorithm_design 
<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">12/05/2026</span></p>

**Author:** Claudio Bernardoni

[[AD index]] (2)

**Hands-on:**
![[Hands-on 12-05.png]]

**Point 1:**
As it was stated, to calculate a degeneracy ordering we can iteratively remove the node with the smallest degree, so we could perform an algorithm as following:
```pseudo
	\begin{algorithm}
		\caption{Compute a degeneracy ordering of the graph $G=(V,E)$}
		\begin{algorithmic}
			\Function{DegeneracyOrdering1}{G}
				\If{$|V|>0$}
					\State $u\leftarrow V[0]$
					\ForAll{$v\in V$}
						\If{$v.\mathit{degree}<u.degree$}
							\State $u\leftarrow v$
						\EndIf
		            \EndFor
		            \State $\textbf{output:}\ v$
		            \State \Call{DegeneracyOrdering1}{$G\backslash v$}
		        \EndIf
		        \Return
			\EndFunction
		\end{algorithmic}
	\end{algorithm}
```
This algorithm computes the node with the minimum degree in $O(n)$ time, then recurses $n$ other times, one for each node, obtaining a total cost of $O(n^2)$.

We can actually compute the degeneracy list in a better time, we just need to do some pre-processing first. We build an array of buckets from $0$ to $n-1$, where $n=|V|$, and in bucket $i$ we will put all the nodes with degree $i$. Furthermore, in this structure, every bucket will have a $\mathtt{next}$ and a $\mathtt{prev}$ pointer respectively to the next and previous non-empty bucket; also, a $\mathtt{head}$ pointer will be available, which will point to the first non-empty bucket. Finally, we will also keep an array that maintains at position $i$ a pointer to the position in the bucket where node $i$ is present.

This is the algorithm that build the aforementioned structure:
```pseudo
	\begin{algorithm}
		\caption{Build the bucket structure for $G=(V,E)$, where $|V|=n$}
		\begin{algorithmic}
			\Function{BuildBuckets}{G}
				  
\State $\mathit{buckets} \gets \mathbf{new}\ \mathit{array}[0 \dots n-1]$ \Comment{Buckets indexed by degree$\qquad\qquad\qquad\qquad\qquad\qquad\qquad$} 
  
\For{$d \gets 0$ to $n-1$}  
\State $\mathit{buckets}[d] \gets \mathbf{new}\ \mathit{bucket}$  
\EndFor
                \State \\  
\State $\mathit{pos} \gets \mathbf{new}\ \mathit{array}[0 \dots n-1]$  \Comment{Pointer to the position of each vertex inside its bucket$\qquad\qquad$}
  
\State $\mathit{head} \gets \mathtt{null}$  
\State $\mathit{minDegree} \gets n$
                  
\ForAll{$v \in V$}  \Comment{Insert vertices into the proper bucket$\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad$}
  
\State $d \gets v.\mathit{degree}$  
  
\State $\ell \gets \mathit{buckets}[d].\mathit{insert}(v)$  
  
\State $\mathit{pos}[v.\mathit{id}] \gets \ell$  
  
\If{$d < \mathit{minDegree}$}  
\State $\mathit{minDegree} \gets d$  
\State $\mathit{head} \gets \mathit{buckets}[d]$  
\EndIf  
  
\EndFor
\State \\
                  
\State $\mathit{prevBucket} \gets \mathit{head}$  
  
\For{$d \gets \mathit{minDegree}+1$ to $n-1$}  \Comment{Link non-empty buckets$\qquad\qquad\qquad\qquad\qquad\qquad\qquad$}
  
\If{$\mathit{buckets}[d].\mathit{not\_empty}()$}  
  
\State $\mathit{prevBucket}.\mathit{next} \gets \mathit{buckets}[d]$  
  
\State $\mathit{buckets}[d].\mathit{prev} \gets \mathit{prevBucket}$  
  
\State $\mathit{prevBucket} \gets \mathit{buckets}[d]$  
  
\EndIf  
  
\EndFor  
  
\Return $\{\mathit{buckets}, \mathit{pos}, \mathit{head}\}$
			\EndFunction
		\end{algorithmic}
	\end{algorithm}
```
We can see that this auxiliary structure keeps an array $\mathtt{p}$ of integers ($O(n)$ space occupancy), a pointer $\mathtt{head}$ ($O(1)$ space), and an array $\mathtt{a}$ of $n$ buckets, that contain at most $n$ elements ($O(n)$ space); this means that the total space occupancy is $O(n)$.

Regarding the time needed to build this structure, we can divide the algorithm into 3 parts:
1) The first $\mathtt{for}$ loop just initializes the structure, so it is $O(n)$ time.
2) The second $\mathtt{for}$ inserts all the nodes in the buckets and initializes the array of their position, so $O(n)$ time.
3) The third $\mathtt{for}$ creates the connections between the buckets by iterating on the array that keeps them, so it is also $O(n)$ time.
Therefore, the total cost in time of the pre-processing is $O(n)$.

Now that the structure has been built, the algorithm for calculating the degeneracy ordering just keeps extracting the head. However, in order to have a correct degeneracy ordering, whenever we extract a node we must update all of its neighbors by lowering their degree by $1$. This is the algorithm to do so:
```pseudo
\begin{algorithm}  
\caption{Extract the current minimum-degree vertex}  
\begin{algorithmic}  
  
\Function{ExtractHead}{$s, G$}  
  
\State $b \gets s.\mathit{head}$  
  
  
\State $v \gets b.\mathit{extractElement}()$  \Comment{Remove a vertex from bucket b, the first non-empty$\qquad\qquad\qquad\qquad$}
  
\State $s.\mathit{positions}[v.\mathit{id}] \gets \mathtt{null}$  
  
  
\If{$b.\mathit{empty}$}  \Comment{If the bucket became empty, advance the head$\qquad\qquad\qquad\qquad\qquad\qquad\qquad$}
\If{$b.\mathit{next}\neq\mathtt{null}$}
	\State $b.\mathit{next}.\mathit{prev}\gets \mathtt{null}$  
\EndIf
\State $s.\mathit{head} \gets b.\mathit{next}$
\EndIf  
\State \\
\ForAll{$u \in$ \Call{Neighbors}{$v, G$}} \Comment{Update all remaining neighbors$\qquad\qquad\qquad\qquad\qquad\qquad\quad$}  
    
\If{$s.\mathit{positions}[u.\mathit{id}] \neq \mathtt{null}$}  \Comment{Ignore vertices already extracted$\qquad\qquad\qquad\qquad\qquad\quad$}
  
\State $\ell \gets s.\mathit{positions}[u.\mathit{id}]$  
  
\State $b_u \gets \ell.\mathit{bucket}$ \Comment{The bucket number in which u is present$\qquad\qquad\qquad\qquad\qquad\qquad\qquad$} 
\State $b\_curr \gets s.\mathit{buckets}[b_u]$
\State $b\_new \gets s.\mathit{buckets}[b_u-1]$
\State $b\_curr.\mathit{remove}(\ell)$ \Comment{Elimination is immediate because of the direct pointer$\qquad\qquad\qquad$}
\If{$b\_new.\mathit{empty}()$} \Comment{If a non-empty bucket is created we update the links$\qquad\qquad\qquad$}
	\State $b\_curr.\mathit{prev}.\mathit{next}\gets b\_new$
	\State $b\_new.\mathit{prev}\gets b\_curr.\mathit{prev}$
	\State $b\_curr.\mathit{prev}\gets b\_new$
	\State $b\_new.\mathit{next}\gets b\_curr$
\EndIf 
    
\State $\ell' \gets b\_new.\mathit{insert}(u)$
  
\State $s.\mathit{positions}[u.\mathit{id}] \gets \ell'$ \Comment{Update the position of u$\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad$}  
\If{$(s.\mathit{head} \neq \mathtt{null}) \land (b_u-1 < s.\mathit{head}.\mathit{position})$}  
\State $s.\mathit{head} \gets b\_new$ \Comment{If the newly created bucket is behind head, head gets updated$\qquad\qquad $}
\EndIf  
  
\EndIf  
  
\EndFor  
  
\Return $v$  
  
\EndFunction  
  
\end{algorithmic}  
\end{algorithm}
```
All operations performed by the algorithm are in constant time $O(1)$, except for the $\mathtt{for}$ loop which iterates on all the neighbors, making the algorithm cost $O(N(v))$, where $v$ is the extracted node.

Now, as we said before, the algorithm for calculating a degeneracy order just keeps extracting elements:
```pseudo
	\begin{algorithm}
		\caption{Compute a degeneracy ordering of the graph $G=(V,E)$}
		\begin{algorithmic}
			\Function{DegeneracyOrdering2}{G}
				\State $s\leftarrow\ $\Call{BuildBuckets}{$G$}
				\While{$s.\mathit{not\_empty()}$}
					\State $v\gets\ $\Call{ExtractHead}{$s,G$}
					\State $\textbf{output:}\ v$
                \EndWhile
		        \Return
			\EndFunction
		\end{algorithmic}
	\end{algorithm}
```
Without considering the pre-processing of $\mathtt{BuildBuckets}$, the procedure $\mathtt{ExtractHead}$ has a cost of $O(N(v))$, and, since we iterate on all the nodes, thanks to the handshaking lemma we get a total cost of:$$O\left(\sum_{v\in V}N(v)\right)=O(2\, m)=O(m)$$
**Point 2:**
Given an ordering of the nodes $\{v_1,\dots,v_n\}$, we define the degeneracy of a node $v_i$ to be: $d(v_i)=|N(v_i)\cap\{v_{i+1},\dots,v_n\}|$, so the number of nodes after itself in the order that it is connected to.

Let us consider the degeneracy ordering, then the set $S=\{v_i,\dots,v_n\}$, where $v_i$ is the first node with degeneracy $k$, represents the $k$-core of the graph. This is because any node $v_j$, with $j>i$, has at least $k$ other connections with nodes in $S$, if that were not the case, it would mean that, at the time of extracting a node with $\{v_i,\dots,v_n\}$ remaining, that node would have been extracted, since it would have a degree smaller than that of $v_i$, and so we would have $j<i$. We can also be sure that $S$ cannot be further extended because if a node $v_h$, with $h<i$, could be part of the $k$-core, then this would mean $v_h$ would have degeneracy at least $k$, and so $v_i$ would not anymore be the first node with degeneracy $k$.

Now, using the algorithms in *Point 1*, we can create the degeneracy list in $O(m)$ time and $O(n)$ space. We use a slight modification of the algorithms, where we keep and initialize an array $d[0,\dots,n-1]$, that keeps in position $i$ the first node with degeneracy at least $i$. Then, to find the whole $k$-core, we just iterate the degeneracy ordering from that position, obtaining an algorithm that, without considering the pre-processing, calculates the $k$-core in $O(|S|)$ time.

**Point 3:**
Let us consider the degeneracy ordering $\{v_1,\dots,v_n\}$, then, if we take a $v_i$, we can calculate $\mathtt{CLIQUES}(\{v_{i+1},\dots,v_n\},\{v_i\},\{v_1,\dots,v_{i-1}\})$, so we find all the maximal cliques with the nodes after $v_i$ in the order, ignoring all the previous ones. We iterate this procedure for every node in the order; we are sure to output only maximal cliques, because that is the output of the procedure $\mathtt{CLIQUES}$. We are also sure that we output all cliques because we iterate the procedure on every node, which means that, since every clique contains at least one $v_i$, we output all cliques. Finally we are also sure that this procedure does not output duplicates because if we find a clique when iterating for $v_i$, then it cannot appear afterwards because $v_i$ will be in he unusable nodes $X$, and it cannot have appeared before because that would mean that there exists a node $v_j$ with $j<i$ that allowed the clique to generate, but that would mean that such $v_j$ is also present in the clique and thus it could not have been discovered at $v_i$.

Now that we are sure that the algorithm is correct we can see that what we can actually run is the following call:$$\mathtt{CLIQUES}(\{v_{i+1},\dots,v_n\}\cap N(v_i),\{v_i\},\{v_1,\dots,v_{i-1}\}\cap N(v_i))$$This is because the nodes outside of the neighborhood of $v_i$ are not considered anyway by the $\mathtt{CLIQUES}$ procedure.
Now, from *Point 2* we have that $|P|=|\{v_1,\dots,v_{i-1}\}\cap N(v_i)|=d(v_i)$, but since we know that every degeneracy is smaller or equal than $d$, we get that $|P|\leq d$. Since the complexity the $\mathtt{CLIQUES}$ procedure is $O(3^{\frac{|P|}{3}})$, we now get $O(3^{\frac{d}{3}})$. By calling $f(d)=3^{\frac{d}{3}}$, we get that the total complexity of the algorithm, iterating on all the nodes is $O(n\cdot f(d))$, and thus is an FTP algorithm.
