#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">27/03/2026</span></p>

[[AD index]] (2)

**Author:** Claudio Bernardoni

**Hands-on:**![[Hands-on 27-03.png]]
**Point 1:**

We can see the adjacency list of a node $v\in V$ as an hash map from nodes to integers, where for every node, it returns the number of edges from $v$ to that node. Then, a graph would be another hash map, one that, given a node, it returns its adjacency hash map.

Now, the randomized min-cut algorithm proceeds as follows:
``` Python
	# G is the hash map that represents the graph
	# G.length returns the number of nodes O(1)
	# G.nodes returns a set containing the nodes O(1)
	# G.edges returns a set containing the pairs of nodes that are connected O(1)
	# G.get(node) returns the adjacency hash map of the node passed O(1)
	# G.remove(node) removes the passed node from the hash map O(1)
	# rand is a function that returns a random element from a set O(1)
	def RandMinCut(Graph G) {
		if (G.length <= 1) {
			return -1;
		}
		while (G.length>2) {
			(u, v) = rand(G.edges);
			EdgeContraction(G, u, v);
		}
		(u, v) = G.nodes # the only remaining two nodes
		return (G.get(u)).get(v)
	}
```

We assume that the graph we want to find the min-cut of is given in the previously mentioned structure. Therefore, `G` is a hash map from nodes to hash maps of nodes to integers. Accessing the `length`, `nodes`and  `edges` fields is $O(1)$. The `get`and `remove` methods are $O(1)$ because we are working with hash maps. Even the function that selects an item randomly is assumed to be $O(1)$. The algorithm takes a graph and keeps contracting nodes until only two nodes remain; once that happens, it returns the number of nodes that connect the two. It is clear that this algorithm has a cost of $O(n)$ times the cost of the `EdgeContraction` function.

The  `EdgeContraction` function can be implemented this way:
``` Python
	# A Node u is a hashmap from nodes to integers
	# u.get(node) returns the number of edges with the node passed O(1)
	# u.remove(node) removes the passed node from the adjacency list O(1)
	# u.put(node, n_edges) puts the node passed in the adjacency list with the value passed O(1)
	def EdgeContraction (Graph G, Node u, Node v) {
		uv = Node(); # creating a new node
		G.edges.remove((u,v)); # removing the connection from the two nodes
		
		for (Node k in G.nodes) {
			if (k!=u and k!=v) {
				n_edges = k.get(u)+k.get(v) # the number of edges between uv and k
				k.put(uv, n_edges); # putting the new node in k adjacency list
				uv.put(k, n_edges); # putting k in the new node adjacency list
				k.remove(u); # removing u from k adjacency list
				k.remove(v); # removing v from k adjacency list
				
				if (n_edges > 0) { # if u or v were connected to k 
					G.edges.add((uv, k)) # uv will also be connected to k
				}
				G.edges.remove((u,k)); # removing the eventual connection of (u,k) 
				G.edges.remove((v,k)); # removing the eventual connection of (v,k)
			}
		}
		
		G.nodes.remove(u);
		G.nodes.remove(v);
		G.nodes.add(uv);
	}
```
The function creates the new node and removes the connection from the nodes that need to be contracted. Then, for every node different from those that need to be contracted, it does 2 things:
- Updates the adjacency list of the current node. It puts the value for the new node in the adjacency list and removes the old nodes from it.
- Updates the edges of the graph. It adds the connections to the new node if there are any, and removes the connections to the old ones, if there were any.

Once every node has been updated, the old ones are eliminated and the new one added.

This procedure has cost $O(n)$ since it only uses $O(1)$ operation for $n$ times. This means that the total cost of the procedure is $O(n^2)$.

**Point 2:**

Multigraphs are, in a way, just graphs with weighted nodes. This is because the number of edges from two nodes can be viewed as a single edge with weight the number of edges it had. We can also perform the opposite, building a multigraph with as many edges from two nodes as the weight the original edge was labeled with. This allows to interchangeably go from multigraphs to weighted graphs and vice versa.

This means that, given a weighted graph, we can build its multigraph and use the previous algorithm. Consequently, the probability of success of this algorithm is the same as the one described before, which was established to be $\frac{2}{n(n-1)}$.

**Point 3:**

It is easy to define a new function `RandMinCut2`that is just like the previous `RandMinCut` function but stops at half the original size of the graph. Then we could perform this new algorithm $4$ times, obtaining that we have a cost of $O(4\cdot n\cdot\frac{n}{2})=O(n^2)$ for each call, which also creates $4$ new subproblems of size $\frac{n}{2}$. Therefore the recursive relation of the time complexity is the following:$$T(n)=4\,T\left(\frac{n}{2}\right)+n^2$$Using the Master's theorem, we notice that $n^2=\Theta(n^{\log_2 4})$, meaning that:$$T(n)=\Theta(n^{\log_2 4}\cdot\log(n))=\Theta(n^2\cdot\log(n))$$This increase in the time complexity is not a surprise, as we are increasing the number of subproblems by a factor of $4$ at each iteration, while keeping the size of the problem asymptotically the same as the one of the previous iteration, i.e. $O(\frac{n}{2})=O(n)$.
