#algorithm_design #computer_science
<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">17/04/2026</span></p>

[[AD index]] (2)

**Author:** Claudio Bernardoni

**Hands-on:** 
![[Hands-on 17-04.png|501]]

**Point 1:**
Instead of enumerating every possible subsets of items to put in the knapsack, we consider just those whose total weight does not exceed the capacity $W$ of the knapsack. To do so, a value $k$ is passed in the function's parameters to keep track of the weight of the items added up to that moment:
```pseudo
	\begin{algorithm}
		\caption{Enumerate valid solutions}
		\begin{algorithmic}
			\State \textbf{global: } $(w,W)$
			\Function{Enum2}{$S, X, i, k$}
				\If{$i \leq n$}
					\If{$k+w[i]\leq W$}
						\State \Call{Enum2}{$S\cup\{i\}, X, i+1, k+w[i]$}
					\EndIf
					\State \Call{Enum2}{$S, X\cup\{i\}, i+1, k$}
				\Else
					\State \textbf{output} $S$
				\EndIf
			\EndFunction
			\State \Call{Enum2}{$\emptyset, \emptyset, 1, 0$} //first call
		\end{algorithmic}
	\end{algorithm}
```
Just like before we perform two recursive calls: one for adding the element to the knapsack, and the other for not adding it. However, in this algorithm, before adding the element we check whether adding it would exceed the maximum capacity $W$, and if it does, the branch in which it is not added is the only one explored.

**Point 2:**
Each node is a recursive call and (taken singularly, without considering its subsequent calls) has a cost of $O(1)$ since it does only simple operations and then performs the next recursive calls.
In the case where every recursive call is performed, we will have a complete tree of choices, meaning that we will have $O(2^n)$ leaves (and so solutions to the knapsack problem) and $O(2\cdot2^n)$ nodes (function call). This means that, in this specific case, the cost per solution would be $\frac{O(2\cdot2^n)}{O(2^n)}=O(2)=O(1)$, and so it would be optimal. This does not surprise us, if there is an exponential number of solution, it is not bad to use an exponential amount of time.
On the other hand, if there is only one solution, i.e. to not take anything, we will have an unary tree with $O(1)$ leaves, but $O(n)$ nodes, since that is the height of the tree, The cost per solution would then be: $\frac{O(n)}{O(1)}=O(n)$. This is clearly a worsening of the cost per solution and is also the worst case.
It can be showed that up to $o(n)$ number of leaves the algorithm is not optimal (has a cost bigger than $O(1)$ per solution), but after $O(n)$ leaves it has a cost of $O(1)$ per solution. Intuitively, we can show that if we have $f(n)$ leaves, then for those leaves we will have to add at least around $2f(n)$ total nodes, so, if $f(n)=o(n)$, then the total number of nodes added will still be overtaken by the number of nodes of the unary tree $O(n)$, and thus the cost per solution would be: $$\frac{O(n)}{f(n)}=\frac{O(n)}{o(n)}=\omega(1)$$
$\\$
**Point 3:**
We can stop to store solutions in the leaves of the recursion call tree and instead keep them on the node. To do so, instead of printing a solution only after collecting the element present in it, $S$, and those that are not present, $X$, we just keep $S$ and output it at every recursive call. Doing this and keeping the same code as before will not work since if two consecutive elements are not added, $S$ remains the same and it gets outputted twice. For this reason, instead of choosing whether to take an element or not, we iterate on all the elements and recurse by adding them if they do not exceed the capacity $W$ of the knapsack.
```pseudo
	\begin{algorithm}
		\caption{Enumerate valid solutions}
		\begin{algorithmic}
			\State \textbf{global: } $(w,W)$
			\Function{Enum3}{$S, i, k$}
				\State \textbf{output} $S$
				\For{$j\in[i,\dots,n]$}
					\If{$k+w[j]\leq W$}
						\State \Call{Enum3}{$S\cup\{j\}, j, k+w[j]$}
					\EndIf
				\EndFor
			\EndFunction
			\State \Call{Enum3}{$\emptyset, 1, 0$} //first call
		\end{algorithmic}
	\end{algorithm}
```
To see that this algorithm generates every possible solution, let us consider the first iteration. If the first element is added, then the recursive step will consider every solution where the first item is included. The second iteration might add the second element and will recurse without the first one, meaning that it will find all the solutions containing the second element but not the first. The third iteration will find every solution containing the third element but not the first two, and so on.
This can be seen in the following diagram, where every possible enumeration is shown:
![[Diagram1 17-04.svg]]

**Point 4:**
Now, every node has a cost of $n-i-1$, where $i$ is the position at which it is called. This is because, without considering the subsequent recursive calls, it performs that amount of iterations. In general we cannot say that the total cost of a node, without considering subsequent recursive calls, is the amount of edges that exit through it, because even if an item is not taken (and so its edge is not in the graph), an iteration for it was wasted.
This means that, if as before we consider the instance where no element is taken, we will still waste $O(n)$ time iterating through the elements of the array, meaning that the cost per solution is still not optimal.

**Point 5:**

We consider a different version of the previous algorithm where we pre-process the elements and we sort them:
```pseudo
	\begin{algorithm}
		\caption{Enumerate valid solutions}
		\begin{algorithmic}
			\State \textbf{global: } $(w,W)$
			\State $w'\leftarrow $\Call{Sort}{$w$} //sorts by increasing weight
			\State \textbf{global: } $(w')$
			\Function{Enum4}{$S, i, k$}
				\State \textbf{output} $S$
				\For{$j\in[i,\dots,n]$}
					\If{$k+w'[j]\leq W$}
						\State \Call{Enum3}{$S\cup\{j\}, j, k+w'[j]$}
					\Else
						\Break
					\EndIf
				\EndFor
			\EndFunction
			\State \Call{Enum4}{$\emptyset, 1, 0$} //first call
		\end{algorithmic}
	\end{algorithm}
```
Now, since the items are ordered by lightest to heaviest, if one does not fit into the knapsack, then we are sure that no successive one will also fit, meaning that we can interrupt the cycle and save time.
With this algorithm, there is an actual correspondence between the cost of the algorithm and the number of edges in the tree. This is because we do not waste time for edges that are not actually present in the tree. Thanks to this property, if there are $f(n)$ nodes in the graph, I will have used $O(f(n))$ time to enumerate them, leading to having always $O(1)$ cost per solution.