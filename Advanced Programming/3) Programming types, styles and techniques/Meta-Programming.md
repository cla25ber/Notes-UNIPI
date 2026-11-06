#advanced_programming #computer_science

[[AP index]] (3)

<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">22/10/2025 & 23/10/2025</span></p>

Contrary to ordinary programming where a program manipulates values, in meta-programming a program manipulates code, even its own.

The first example of meta-programming that might come to mind is a compiler; however, they are commonly not considered such, this is because they are external to the language itself. Even though a compiler takes code as input and generates other code, it is considered a tool of the language, the manipulation of code happens outside of the language; while, for the most common definition of meta-programming, the language itself must provide construct to generate and manipulate code.

An example of why meta-programming is useful is parametric polymorphism ([[Type systems]]). Many languages (like C++) implement parametric polymorphism using meta-programming. One can define a function with generics (templates in C++) and then, whenever it is needed with a certain type, a function specific to that type but with the same code is generated. This technique is called **monomorphization**.

##### Meta-programming as syntactic rewriting

The easiest form of meta-programming is syntactic rewriting, like macros in C.

```C
	#define unless(cond, body) \
	if (!cond) {body}
	
	unless(x==0, 1/x);
```

In this example the macro that was written gets substituted syntactically whenever it is later used. 
However, being just a syntactic substitution, it is very limiting, and also sometimes it can be dangerous:

```C
	#define square(x) x*x
	
	square(1+2); //produces 1+2*1+2
```

This macro applies syntactic substitution, but, due to the precedence of operators, the result is $5$ instead of $9$.
It can be debated whether this form of generating code can be considered meta-programming.

### Quotations

Syntactic rewriting is not good enough to generate code because code is not just a string, it has a specific structure: the abstract syntax tree (AST). 
Quotations allow the programmer to define expressions that are not compiled as part of the program, but are actually treated like objects that represent the expression in the language. It is like if we could declare an expression with the data structure used for the AST. Here is an example in F#:

```F#
	open Microsoft.FSharp.Quotations
	// A typed code quotation. (Just with one @)
	let expr : Expr<int> = <@ 1 + 1 @>
	// An untyped code quotation. (With two @) (Untyped is faster to traverse)
	let expr2 : Expr = <@@ 1 + 1 @@>
	
	//Splicing for typed code quotations
	<@ 1 + %expr @>
	//Splicing for untyped code quotations
	<@@ 1 + %%expr @@>
```

Splicing allows to build even larger expressions, starting from the already built ones. The difference between typed and untyped quotations is that the latter do not keep the information about the type, which makes the navigation of the AST faster.

The use of quotations allow for an easier analysis of code rather than just text. An example of the usage of quotations could be a way to have SQL queries in F# (more information on [[F sharp]]):

```F#
	let query : Expr = <@@ fun p -> p.Age > 18 @@>
```

This quotation is a function that returns true for the users who have more than 18 years. If it were just a string, it would be more difficult to analyze and turn it into a SQL query; however, with this format, it is far easier to manage. One could argue that we could just write an SQL query liker this:
```F#
	let query : String = "SELECT * FROM Users WERE Age > 'hello'"
```

However, as it can be possible to see, this raises some problems. First of all errors like "WERE" instead of "WHERE" and type errors like "Age > 'hello'" would only be detected at run-time, while, with typed quotations, they would be immediately detected. Also, with quotations, queries can be composed or even programmed, one could create a function that takes an integer and returns the quotation for the query of users being older than that age. This allows to systematically create queries, avoiding rewriting the same code.

This query system is integrated into C# through LINQ (Language-INtegrated Query). This is done using expression trees, which are almost the same thing as F# quotations. 


### Reflection

Up until now, meta-programming was a program generating another one; instead, reflection allows the program to inspect information about itself at run-time. The usefulness of reflection can immediately be seen by methods like getType, or getFields, which respectively return the type and an array of the fields of the object that called it. This allow for an easier serialization of objects (JSON).
Another example of reflection is when a class gets uploaded dynamically, for example as a plugin, so you find a method by name and run it. Clearly, the name will not be known at compile time, since that class will have not still been uploaded.
Another example of reflection are annotations, which are basically metadata that enriches the code with more information. By themselves, annotations do nothing, but when read by specific run-time tool they can be really useful. For example, JUnit uses them in Java to find all method with the ```@Test``` annotation and execute them. More information in [[Java]].

Generally, due to the dynamic nature of reflection, it is slower than regular programming, so it is advised to use it only when strong flexibility is required.

##### Reflection in .NET

Reflection in the .NET runtime is implemented through assemblies, which are the fundamental unit of deployment (more information in [[dotNET runtime]]). Assemblies are the building block of .NET applications, they contain one or more modules, thus containing their compiled code, the types defined inside them, and much more meta-information. Each assembly has a manifest, which is a reference to it; to use an assembly in an application you must reference its manifest. Once you have done that, all the accessible types, properties, methods of its namespace can be used as if they were part of your source code. Since assembly are loaded dynamically in the memory, reflection is the mechanism through you can access all the namespace of the assembly.

#### References

- C macros - <a href=https://gcc.gnu.org/onlinedocs/cpp/Macros.html>GNU compiler collection (GCC)</a>
- F# quotations - <a href=https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/code-quotations>Microsoft Learn</a>
- .NET assemblies - <a href=https://learn.microsoft.com/en-us/dotnet/standard/assembly>Microsoft Learn</a>
- .NET reflection - <a href=https://learn.microsoft.com/en-us/dotnet/fundamentals/reflection/overview>Microsoft Learn</a>