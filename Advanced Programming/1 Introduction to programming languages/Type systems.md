#advanced_programming #computer_science

[[AP index]] (1)

<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">27/10/2025</span></p>

Every machine stores data as just bits; however, if the type of such data can be identified, it is possible to prevent errors that might occur, like calling something that is not a function or adding integers to strings. Every language, in one way or another, implements a type system to avoid incurring in the aforementioned errors, and they are implemented in two ways:
- <span style="color:rgb(255, 0, 0)">Static typing:</span> if a type error occurs in the program, it is known before execution at compile time, and must be solved to continue. This means that the type system further restricts the number of acceptable programs, beyond what the grammar would normally do.
- <span style="color:rgb(255, 0, 0)">Dynamic typing:</span> type errors in the program come out during execution. Clearly, this kind of typing is less safe; however, it is far more flexible (if I assigned the variable $x$ an integer and then I want to assign it a string there is no problem). Also, dynamic typing does not annoy the programmer with warnings about type errors that, given the nature of the code, the programmer is sure they will never happen.

Languages can also be more or less strict about enforcing type rules, for example some languages (like C) might allow implicit casting of variable in specific cases (like when summing an integer to a float). For this reason, programming languages can also be divided into two other categories:
- <span style="color:rgb(255, 0, 0)">Strongly typed:</span> programming languages are very strict regarding their type rules, every casting might be explicit and designed. Clearly, this is the safest option.
- <span style="color:rgb(255, 0, 0)">Weakly typed:</span> allow type rules to be broken in specific occasions, making the code more flexible and sometimes also easier to program.

The characteristics of a language being static/dynamically typed or strongly/weakly typed are orthogonal, here is an example of every possible combination:
<table>  
  <tr>  
    <th></th>  
    <th>Static typing</th>  
    <th>Dynamic typing</th>  
  </tr>  
  <tr>  
    <th>Strongly typed</th>  
    <td><span style="color:rgb(0, 112, 192)">Haskell</span>, <span style="color:rgb(0, 112, 192)">Java</span>, <span style="color:rgb(0, 112, 192)">Rust</span>, <span style="color:rgb(0, 112, 192)">OCaml</span>, <span style="color:rgb(0, 112, 192)">F#</span>, <span style="color:rgb(0, 112, 192)">Typescript</span></td>  
    <td><span style="color:rgb(0, 112, 192)">Python</span></td>  
  </tr>  
  <tr>  
    <th>Weakly typed</th>  
    <td><span style="color:rgb(0, 112, 192)">C</span></td>  
    <td><span style="color:rgb(0, 112, 192)">Javascript</span></td> 
</tr></table>

#### Extending the type system

Every programming languages has primitive types, e.g. $\texttt{int}$, $\texttt{bool}$, ...; however, the programmer, generally, has the ability to construct new types:

- *Records* ($\texttt{struct}$ in C) are the cartesian product of other types. $\alpha\times\beta$
- *Union* are the union of other types. $\alpha\ |\ \beta$ 
- *Arrays* are a function from $[1,n]$ to the type they store. $[1,n]\to\alpha$
- *Sets* and *Lists* store a certain type. $\alpha\texttt{ set}$ or $\beta\texttt{ list}$  
- *Pointers* location to data of a certain type. $\alpha\texttt{ ref}$ (in OCaml) or $\alpha*$ (in C)

#### Type inference

Many languages (like Haskell or OCaml) have type inference; i.e., the programmer does not need to annotate the type of every variable since it is inferred by the language.
To do this, every variable is assigned the most general type it can have; meaning that, if a variable is an integer, then it is assigned that, but if no other information is known, we assign it a generic type $\alpha$. We can collect many constraints about a variable depending on how it is used across the program. Once they have all been collected, the unification algorithm is performed to get the most general type of every variable.

The fact that we can end up with some variables being typed generically, foreshadows the concept of **polymorphism**.
```OCaml
	let f x = x;;
```
This OCaml function is the identity function, i.e., given $x$ it returns $x$. Given how it is defined, we have no constraints on the type of $x$, which can be $\alpha$, meaning that the function $f$ will have type $\alpha\to\alpha$. This means that $f$ can work on any type.


## Polymorphism

Polymorphism is when a variable can assume multiple forms, i.e. types. There are many kinds of polymorphism; below a small diagram showing some classes of polymorphism.

![[Polymorphism.svg]]

- <span style="color:rgb(0, 176, 80)">Ad hoc polymorphism:</span> a function denotes different algorithms, one for every type it works.
- <span style="color:rgb(0, 176, 80)">Universal polymorphism:</span> there is only one algorithm that is universal for all types.

#### Overloading (Ad hoc)

The same name identifies multiple functions, differing just from the types they handle.

```C
	int sqr(int x) {return x*x;}
	double sqr(double x) {return x*x;}
```

In this example the name $\texttt{sqr}$ has been overloaded to handle multiple types. When $\texttt{sqr}$ will be called, the function to use will be decided based on the value that is passed to it. Overloading is intrinsically present in lots of programming languages, arithmetic operations ($+$, $-$, $\times$, ...) can usually handle multiple numeric types ($\texttt{int}$, $\texttt{double}$, $\texttt{float}$, ...).

#### Coercion (Universal)

Coercion consists in converting a type into another one automatically. This allows for multiple types operation, for example
```javascript
	"5" + 2
```
produces the string "52", because for the operation to work, one of the types is automatically converted into an acceptable one. Weakly typed languages are usually associated with coercion.

#### Inclusion (Universal)

Inclusion, also known as inheritance, works on the concept of subtyping. According to the Liskov substitution principle, we say that a type $A$ is a subtype of $B$, i.e. $A<:B$, if every time something of type $B$ is needed, a data of type $A$ can be used in its place. This polymorphism allows a function defined for a certain type to work for every of its subtypes.

Let $A<:B$, then it is natural to wonder whether composite types using $A$ and $B$ still maintain the subtyping relationship. Let $C(\cdot)$ be a way to compose types, the answer depends on $C$:
- We say that $C$ is <b><span style="color:rgb(146, 208, 80)">covariant</span></b> if $A<:B\implies C(A)<:C(B)$, meaning that the subtyping relation gets preserved by the type constructor $C$. Usually, read-only type constructors are covariance. For example, if $C$ is an immutable list, then $\mathit{Cat}<:\mathit{Pet}\implies \mathit{ImmList}(\mathit{Cat})<:\mathit{ImmList}(\mathit{Pet})$. The $\mathit{Optional}$ type constructor is also covariant.
- We say that $C$ is <b><span style="color:rgb(146, 208, 80)">contravariant</span></b> if $A<:B\implies C(B)<:C(A)$, meaning that the subtyping relation gets inverted by the type constructor $C$. Usually, type constructor that modify data are contravariant, for example $\mathit{Cat}<:\mathit{Pet}\implies \mathit{Action}(\mathit{Pet})<:\mathit{Action}(\mathit{Cat})$, because an action of pets can always be performed as an action on cats.
- We say that $C$ is <b><span style="color:rgb(146, 208, 80)">invariant</span></b> if $A<:B\not\Rightarrow C(A)<:C(B)$ and $A<:B\not\Rightarrow C(B)<:C(A)$, meaning that no relation gets preserved by the type constructor $C$. This is the case when a single type constructor has covariant and contravariant characteristics, like a mutable list. Given $\mathit{Cat}<:\mathit{Pet}$, it is clear that it is not contravariant since $\mathit{MutList}(\mathit{Pet})<:\mathit{MutList}(\mathit{Cat})$ does not hold, this is because a list of pets cannot be used whenever there is a need for a list of cats. However, it is also not true that it covariant, since if $\mathit{MutList}(\mathit{Cat})<:\mathit{MutList}(\mathit{Pet})$ were to hold, we could use a list of cats whenever a list of pets is used, even when adding a $\mathit{Dog}<:\mathit{Pet}$ to a list of pets.

In Java, lists are mutable objects so they are invariant. Typically, in functional programming languages (like Haskell and OCaml) lists are immutable, so they are covariant.
Interestingly, a curious case is that of the type constructor for functions, in fact it is covariant in the result and contravariant in the parameters, meaning that if $A_{2}<:A_{1}$ and $B_{1}<:B_{2}$ then it holds that $A_{1}\to B_{1}<:A_{2}\to B_{2}$. The type of the parameter is contravariant because a function that handles pets can be substituted with one that handles cats. The type of the result is covariant because a function that returns pets can be substituted with one that returns cats.

#### Overriding (Ad hoc)

Overriding is when a method of a class gets redefined in a subclass.

```Java
	class A {
		public void m() {/* something */}
	}
	
	class B extends A {
		public void m() {/* something different */}
	}
```

In this Java example, the method $m$ of class $A$ is overridden by the method with the same name in class $B$. This means that depending on the class, and thus the type, of the caller, the method changes. This type of polymorphism is strictly related to inheritance, in fact, since $B<:A$, we are creating ad hoc polymorphism in the universal polymorphism of inclusion, where certain subtypes of $A$ will perform the same action differently.

#### Parametric (Universal)

Parametric polymorphism is when a function can have infinitely many types. This happens because instead of having a specific type, it has a parametric one, so any data that respects its form is accepted. If a function takes something of type $\alpha\ \mathit{ list}$, then any kind of data that fits into parameter $\alpha$ is accepted, so $\textit{int list}$, $(\mathit{bool\to int})\textit{ list}$ or even $((\textit{int list})\textit{ list}) \textit{ list}$ is accepted.
This kind of polymorphism can be implemented in two ways:
- <span style="color:rgb(255, 0, 0)">Explicit:</span> in this case the programmer needs to annotate the parameters, meaning that they are written directly in the code. An example of this is generics in Java or Typescript.
- <span style="color:rgb(255, 0, 0)">Implicit:</span> in this case the type is inferred, so the type of the function is not written in the code. Examples of implicit parametric polymorphism are OCaml and Haskell, which, thanks to type inference, can conclude the parameters of the functions by their code. In the OCaml example of the identity function, it was never needed to show the type.

#### References

- Lecture Notes 27/10/2025 - Andrea Corradini
- "Concepts in Programming Languages" Chapter 6 - John C. Mitchell