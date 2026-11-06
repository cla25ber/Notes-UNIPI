#advanced_programming #computer_science

[[AP index]] (2)

<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">15/10/2025 & 20/11/2025</span></p>

# Memory management

CPython, the standard implementation of Python, uses a garbage collector with *reference counting* and *mark and sweep* to decide which portion of memory to collect (more information about both in [[Environment and memory management]]). This means that in Python there are no dangling pointers or double frees, as object live as long as there is a reference to them, and memory is freed automatically.

A problem that many languages implementing reference counting encounter have, is having to deal with multi-threaded programs. Since almost every operation has to update a reference counter, multi-threaded program could create race conditions where every time synchronization is needed. However, building a lock for each object or making the update operation for reference counting atomic would add a big overhead to the operations, even single-threaded ones. Since Python's philosophy is to be a language as simple as possible for the programmer, explicitly handling locks would be antithetical its aim as a language.

A solution for this problem is found in the **GIL** (Global Interpreter Lock), which is a coarse-grained lock of the entire Python interpreter. This means that only one thread at a time can use the Python interpreter, clearly affecting negatively multi-threaded programs, which consists in just a CPU at a time making computation. This can still be useful for I/O-based computation, where threads mainly have to wait, but for CPU-intensive computation there is basically no advantage with respect to single-threaded programming.

This solution is the simplest one, as it also avoid having to build synchronized data structures and other synchronization mechanisms; however, it loses almost every advantages derived from multi-threaded programming. For this reason, many Python implementation have tried to get rid of the GIL; however, doing so would cause many backward compatibility issues with already existing libraries of Python. A way to circumvent the problem is to use, instead of multi-threading, multi-processing. Since every process has its own interpreter, the GIL would not be a problem; nevertheless, the overhead caused by using processes instead of threads would be significant.

# Functions

In Python, functions are objects of the `Function` class. They are first-class objects with the possibility of higher-order functions; furthermore, parameters are passed by object reference.

## Decorators

Decorators are objects that are used to modify functions. They take a function and return one that behaves the same with some additional actions. For instance, in this example some text is printed before and after the function:
```Python
def decorator(func):
	def wrapper(): # defines the function with additional behaviour
		print("Calling a function.")
		func() # calls the effective function
		print("Function execution has ended.")
	return wrapper # returns the inner function
```

Other examples of decorators:
```Python
# executes a function twice
def do_twice(func):
	def wrapper(*args):
		func(*args) # *args is an unknown number of arguments
		func(*args)
	return wrapper
	
# calculates the running time of a function
def timer(func):
	def wrapper(*args):
		start = time.perf_counter()
		func(*args)
		end = time.perf_counter()
		runtime = end - start
		print("Finished in {runtime:.4f} seconds.")
	return wrapper	
```
Another use of decorators is, for example, debugging.

## Namespaces

A namespace is a mapping from names to objects in memory, implemented as a dictionary in Python. There are many types of namespaces in Python:
- **Built-in:** already defined names (like `print`). They are created at the compiler start-up.
- **Global:** names defined at the module level. They are created when the module is read.
- **Local:** names defined inside of functions. They are created when the function is executed, and deleted when it is removed from the stack.
- **Class:** names defined inside of classes. 

A name is searched in the current local namespace going outward. First, in the active function or class, then in any other enclosing function, then in the module, and finally it is searched in the built-in namespace. When defining a variable, the keyword `nonlocal` can be used to write it in the enclosing namespace, while `global` to write it in the global namespace.

# Object Oriented Programming

The main ingredients of OOP are:
- **Encapsulation:** the ability to divide the code into a public and a private interface.
- **Inheritance:** the ability to create specialized sub-classes of parent classes.
- **Polymorphism:**  the ability to *override* inherited behaviour.

When a class is defined, it creates a new namespace. When instances of that class are created, they create a new namespace enclosed in the one of the class. For instance, the following code creates the following namespaces:
```Python
class Point:
	x = 0
	y = 0
	def str(): # stringify
		return "Point in (" + ((str) Point.x) + "," + ((str) Point.y) + ")"

p1 = Point()
p2 = Point()
p1.x = 3
Point.y = 5
```
![[Namespaces.svg]]

So, both `p1.y` and `p2.y` will have value `5`, but `p1.x` will have `3` while `p2.x` will have `0`.

In a class it is also possible to define instance methods, i.e. methods that take as an implicit parameter the class instance itself. The implicit parameter is usually denoted with `self`.
```Python
class Foo
	def fun(par_0, par_1, ..., par_n): statements

obj = Foo()
obj.fun(arg_1,...,arg_n)
# is syntactic sugar for
obj.__class__.fun(obj,arg_1,...,arg_n)
```
Here the `fun` method is a instance-specific method of `obj`. However, it is just syntactic sugar for calling the method in the class' namespace and passing `obj` as an argument. `__class__` is the attribute through which the class of an instance is obtained. The constructor is defined through the method `__init__`.

## Inheritance

When a class is extended, its namespace is enclosed in the one of the parent. However, in the case of multiple inheritance, it uses the MRO (Method Resolution Order) to determine which classes' namespace has the priority. This is done with a deterministic procedure (the C3 linearization algorithm), and also determines the `super` of a class that had multiple inheritance. This also solves the *diamond problem*, i.e. the problem of determining  from which parent class a certain method should be derived from.

## Encapsulation

Python does not have proper encapsulation, as it lacks the `private` keyword; conventionally, private variables' names start with an underscore. However, sometimes private members are useful to avoid clashes with members with the same name defined in a superclass; this problem is mitigated using <span style="color:rgb(0, 112, 192)"><i>name mangling</i></span>.
In Python, members starting with two underscores are mangled into an underscore followed by the class name and then the previously defined name. E.g. `__spam` is transformed into `_classname__spam`, where `classname` is the name of the class in which it is defined. This allows to avoid name clashes and also makes members less accessible; however, name mangling is not foolproof, and also worsen readability.

#### References

- Lecture Notes 15/10/2025 - Andrea Corradini
- Python GIL - <a href=https://realpython.com/python-gil>Real Python</a> 
- Lecture Notes 20/11/2025 - Andrea Corradini