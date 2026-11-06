#advanced_programming #computer_science

[[AP index]] (1)

<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">01/10/2025 & 02/10/2025</span></p>

### Names

In programming languages names are used to identify and associate resources or concepts with a string of character, which, pragmatically, has to be significant with what it represents.

There are two types of names, those dependent on the programming languages, like the name of primitive types, constructs, and so on. Instead, the other type is decided by the programmer, like identifiers for variables, functions, but even new composite types.

It is always important to remember that a name is associated with what it represents but it is not the concept itself, in fact it is possible to have different names for the same object but also the same name for different objects (at distinct times).

### Environment

The environment is a function that associates every name to the denotable object it is representing. Given the nature of functions, it does not surprise us that multiple name can refer to the same object. However, it is also possible for the same name to refer to multiple objects; this happens because the environment keeps the most recent association, but as soon as we exit the fragment where that binding is valid, which is determined by the scoping rules of the language, it immediately goes back to the previous binding.

### Scoping

Scoping rules determine how the environment changes when entering new blocks or functions, in particular we can identify two types of scoping:

<span style="color:rgb(255, 0, 0)"><span style="color:rgb(255, 0, 0)">Static scope</span>:</span> the environment can be solely determined by the syntactical structure of the program, so it can be completely determined by the compiler. It consists of the following rules:
   - Declarations define the local environment of a block, but not those inside an nested block (which define the environment of the nested block).
  - If a name is used inside a block, its associated object is searched in the current block environment, if not present it is searched in the block immediately before, and so on until the most external block of the program. If the name is neither there, it is checked if it is a predefined name of the language, if it is not even that, then there is an error.
  - A block can have a name (e.g. a function), then that name is present in the immediate previous block (note that this is not a problem for recursion).

<span style="color:rgb(255, 0, 0)">Dynamic scope:</span> just like before, if a variable is not found in a local environment, we search the immediate previous one; however, in dynamic scoping the order of the blocks is different. Instead of a syntactical one, i.e. the previous block is the one where the current one is nested, we have a temporal one, so, the previous block is the last one that has been entered by the program. Since it depends on execution, it cannot be determined statically, hence the "dynamic" in its name.

Here is an example where static scoping is different from dynamic scoping:
```C
	int x = 0;
	
	void f() {
		print(x);
	}
	
	void g() {
		int x = 1;
		f();
	}
	
	g();
```

With static scoping, the $g$ is called, $1$ is saved in $x$, then procedure $f$ is called which prints $x$. Since in the function $f$ there is no association for $x$, we go to the syntactic previous block, which is that of the external block, so $0$ is printed. Meanwhile, in dynamic scoping, the previous block is the previous temporal one, and since before $f$ the execution was in the block of $g$, $1$ is printed.

Generally speaking, implementing dynamic scope is easier since it is more flexible.

Depending on the programming language, some rules can be added. For example, in Pascal, there is also the rule that names must first be declared before being used, which consequently prohibits mutual recursion and recursive types.

```Pascal
type list = ^element
type element = record
	value: int
	next: list
	end
```

This recursive type definition is allowed just because Pascal relaxes the previous rule for recursive type definition regarding pointers (^ in Pascal).

##### Scoping with high-order functions

Scoping rules when considering high-order functions, i.e. functions that can take other functions as parameters or return them as result, complicate even further. Let us consider the following example:
```C
	int x = 1;
	
	int f(int y) {
		return x+y;
	}
	
	int g(int h(int n)) {
		int x = 2;
		return h(3)+x;
	}
	
	{
		int x = 4;
		print(g(f));
	}
```

What should the function $g$ return? We can see two different cases:
- <span style="color:rgb(255, 0, 0)">Deep binding:</span> the environment of a function used as a parameter in another function is defined as soon as there is the binding with the parameter. In the example above, the environment for $h$ is created when $g$ is called, so when we are binding the function $f$ with its parameter $h$.
- <span style="color:rgb(255, 0, 0)">Shallow binding:</span> the environment of the function is created when needed, so when the high-order function calls it. In the example above the environment for $h$ is created when it is called inside $g$.

Here is what is printed by $g(f)$ given every possible combination:
<table>  
  <tr>  
    <th></th>  
    <th>Deep binding</th>  
    <th>Shallow binding</th>  
  </tr>  
  <tr>  
    <th>Static scope</th>  
    <td>Since we are using deep binding, the environment for h is created when the high-order function is called. Since there is no declaration of x inside of f, it is searched in the external block due to static scoping, thus making f return 1+y.This means that after the execution of g, the printed value is <b><span style="color:rgb(0, 176, 240)">6</span></b>.</td>  
    <td>In this case the result would still be <b><span style="color:rgb(0, 176, 240)">6</span></b> because static binding still assigns x the value of the external block of f. The environment is determined statically.</td>  
  </tr>  
  <tr>  
    <th>Dynamic scope</th>  
    <td>Since we are using deep binding, the environment for h is created when g is called. Since inside of f we have no definition of x, it is searched inside the last used environment, the one of the block where g is called. Thus, due to dynamic scope, we get that f returns 4+y, so the final result is <b><span style="color:rgb(0, 176, 240)">9</span></b>.  </td>  
    <td>Since we are using shallow binding, the environment for h is created when it is called inside of g. Again, we have no definition of x inside of f, so we search in the last used environment, but this time it is the body of g, so we get that f returns 2+y and thus the final printed result is <b><span style="color:rgb(0, 176, 240)">7</span></b>.</td> 
</tr></table>

Implementing shallow binding is generally easier; when I call the function I access its body and there I perform the binding to the variable, so it is no more difficult than dynamic scope. On the other hand, deep binding needs to bind the variable inside the function before it is called, so before its body is accessed, so auxiliary data structures are necessary.

### Memory management

The more we climb the stairs of abstraction, the more memory management becomes difficult. In languages close to the hardware, memory management is quite simple, whenever something new is declared it remains there up until the end of the computation. If we introduce blocks we have to delete the variables once we exit it, which can be implemented easily considering that it is a static property. The problem comes when we have to allocate memory depending on the execution; for example, if we introduce recursion to the language, we do not know statically how many times we have to allocate memory for the procedures activated, so memory has to be allocated dynamically. Recursion is not a big problem, for it we can just create a <span style="color:rgb(255, 0, 0)">stack</span> that keeps track of the memory allocated; a LIFO data structure is enough because one procedure cannot end before the ones it has called. However, a stack is not enough when the language has the power to operate with memory directly, i.e. when using pointer. If we have the power to allocate and deallocate whenever we like, it is not necessarily true that a slot of memory allocated before another will be deleted after it, so we need to use other data structures, like a <span style="color:rgb(255, 0, 0)">heap</span>.

Notably, the memory of a program can be divided into 4 parts: stack, heap, data and text.

![[Memory of a program.png]]

The <span style="color:rgb(0, 176, 80)">text</span> part represents the memory occupied by the program itself; instead, <span style="color:rgb(0, 176, 80)">data</span> is the part about constants, global variables and all the other usage of the memory that can be managed statically.
The <span style="color:rgb(0, 176, 80)">stack</span> and the <span style="color:rgb(0, 176, 80)">heap</span> memory allocation depends on run-time, in fact they can grow as much as they want, as long as they do not surpass the amount of memory that the program was given.

#### Stack

The stack keeps activation records, which consist in all the information that is needed to make the procedure run correctly. It consists of the following:
![[Stack frame.png]]
- <span style="color:rgb(112, 48, 160)">Parameters</span><span style="color:rgb(112, 48, 160)">:</span> stores the parameters passed to the procedure.
- <span style="color:rgb(112, 48, 160)">Address for result:</span> stores the value returned by the procedure (if it returns a value).
- <span style="color:rgb(112, 48, 160)">Local variables:</span> stores the variables declared inside the procedure. The size of this section depends on the amount of variables declared, it is decided at compile time.
- <span style="color:rgb(112, 48, 160)">Intermediate results:</span> stores the intermediate results of operations. For example, if we have something like $\texttt{b:=(a-x)/(a+x);}$ then it is possible that the compiler might want to store the intermediate results of $\texttt{a-x}$ and $\texttt{a+x}$. This is done because they might be saved in registers before performing the division and other factors that depend on the architecture and the compiler.
- <span style="color:rgb(112, 48, 160)">Return address:</span> the address where the control was taken over by the execution of the function; i.e., the value the PC should hold after the end of the procedure.
- <span style="color:rgb(112, 48, 160)">Dynamic chain pointer:</span> stores the pointer to the previous activation record. Since they might need to hold different quantities of information, their size is not always the same, so we need to know when the previous one starts to avoid deleting it.
- <span style="color:rgb(112, 48, 160)">Static chain pointer:</span> it is needed only if static scoping is implemented. It stores the pointer to the syntactically last stack, i.e., the stack previous external stack needed if static scoping is used. If dynamic scoping is implemented, this role is taken by the dynamic chain pointer.

When a procedure is called a new frame (activation record) is added on the stack, then the current PC is saved in the return address and then is changed to start the execution of the procedure. The other fields of the activation record are then filled, and then the code of the procedure executed.
Once a procedure ends, the PC is set back to that of the caller, the variable that holds the return value is given the value stored in the address for result, and then the frame is deallocated from the stack.

Even blocks work with a stack; in particular, their activation records only contain the dynamic chain pointer, the local variables and the intermediate results. Blocks do not need a static chain pointer because, differently from procedures, they are executed as soon as they are introduced, meaning that their dynamic chain pointer coincides with their static chain pointer.

Static scoping is implemented by the following the static chain pointer until I find the variable I need; dynamic scoping is the same but with the dynamic chain pointer.

#### Heap

As we already said, the possibility of allocating memory makes the stack not enough for handling memory. The heap, which has nothing to do with the heap data structure, consists in a free list, i.e. a list of locations and pointers to the next free locations. 
![[Heap free list.png]]
The free list start with every location pointing to the next one, but as soon as execution gets involved, memory gets allocated and deallocated, thus scrambling the pointers around. The free list is usually implemented in one of two ways:

- <span style="color:rgb(255, 0, 0)">Fixed-size blocks:</span> every block is of the same size. It is easy to manage but sometimes it can be too restrictive; for example, in the case of languages where there are unbounded arrays, and thus their size is known at execution time.
- <span style="color:rgb(255, 0, 0)">Variable-length blocks:</span> the list starts as just one giant block, when some memory is allocated a block of that specific size is created, and so the list gets divided into 2 blocks. When memory is again allocated we create another block, and so the heap gets further divided into block of variable size. This implementation clearly creates problems of internal fragmentation (if the size is known only at runtime I have to guess how much memory will be needed), but more importantly the problem of external fragmentation. Usually, to avoid external fragmentation, after reaching the end of the free list, a compaction procedure is called; clearly, this undermines the efficiency of memory allocation.

### Garbage collection

The "garbage collector" is a part of many programming languages that aims to delete all the unused memory that is still on the heap. It does so in 2 phases: first it identifies which objects are still alive and which are not used anymore, secondly it deletes the latter.

Garbage collectors tend to be extremely conservative in identifying unused object, first of all because deleting something that might still be in use would be extremely dangerous behavior, and also because identifying exactly what is still alive would be very costly and impact the efficiency of the program (it is also undecidable in the general case).

We can divide garbage collection techniques in two types: those that delete something as soon as they are sure it is not used anymore, and those that start searching for unused memory when it is about to end. An example of the first type is <span style="color:rgb(0, 176, 80)">reference counting</span>, while an example of the second is <span style="color:rgb(0, 176, 80)">mark and sweep</span>.

#### Reference counting

In reference counting, we keep track of how many active pointers we have to some memory. Every time new memory is allocated, its reference count is set to 1. If an assignment of the type $\texttt{p:=q}$ occurs, then the reference count of the object pointed by $\texttt{q}$ increases by 1, since now even $\texttt{p}$ points to it; instead, the reference count of the object pointed by $\texttt{p}$ decreases by 1 (if it was pointing to something). Whenever we exit a block and so the lifetime of a variable ends, if that variable represented a pointer, the reference counter of the object pointed by it decreases also by 1, since now we cannot use anymore that pointer. Once an object has a reference count of 0, it is deleted since it cannot be accessed anymore.

The most prominent problem of reference counting is circular pointers, where, despite not being able to be accessed by the program variables, there are memory areas that point to themselves in circles, and so they all have non-0 reference count. Furthermore, the cost of reference counting is not proportional to heap size but to memory instructions, since every time a pointer is created, changed location or deallocated we need to refresh its reference count. Despite these drawbacks, reference counting remains one of the simplest way to have a garbage collector.

#### Mark and sweep

Once the memory is almost full, the mark and sweep technique marks every location on the heap as "unused". Then, it takes every pointer that is in use at the moment and follows the location to which it points recursively (if a pointer points to something that has pointers then it also follows those), using BFS or DFS until it visited all the data structure, marking as "used" every location it encounters. Once it has finished with the pointers in use, it deallocates everything on the heap that is marked as "unused".

While it is true that this technique gets activated only when the memory is full, the performance of the program greatly decreases once the garbage collecting procedure is active.

#### References

- "Programming Languages: Principles and Paradigms" Chapter 4, 5, 7 and 8 - Maurizio Gabbrielli, Simone Martini