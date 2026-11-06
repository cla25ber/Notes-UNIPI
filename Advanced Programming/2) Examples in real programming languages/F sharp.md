#advanced_programming #computer_science

[[AP index]] (2)

<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">16/10/2025 & 20/10/2025</span></p>
F# is a member of the ML (Meta Language) family, like OCaml, with which it shares lots of its syntax and constructs. It is compiled and expression-oriented and variable are immutable unless defined otherwise with the ```mutable``` keyword.

Like many functional programming languages, it has pattern matching:
```F#
let rec sum_list ls acc =
	match ls with
	| [] -> acc
	| x :: lst -> sum_list lst (acc+x)
;;
```
Pattern matching must be exhaustive.

An interesting construct is the forward pipe operator, which allows to apply functions by providing the argument before the function, allowing for more elegant syntax of multiple function applications:
```F#
let (|>) x f = f x ;;

let result =
    [1..10] // list comprehension
    |> List.map (fun x -> x * x)
    |> List.filter (fun x -> x % 2 = 0)
    |> List.sum
;;
```
F# also supports parametric polymorphism, while also supporting constraints on types, similarly to type classes in Haskell (more in [[Haskell]]).
```F#
let max (x: 'a) (y: 'a) : 'a when 'a:comparison
	if x>y then x else y
;; 
```
Although the function being defined on possibly every type, it is also being limited to those types that supports comparison.

F# mainly uses eager evaluation, so, differently from Haskell, when defining a list, all of its values are constructed and held in memory. This can clealry become a problem when the list is really big or even infinite. Having said that, F# still supports infinite lists, but does it through the use of <span style="color:rgb(0, 112, 192)"><b>sequences</b></span>. Instead of representing an infinite number of expressions, a sequence represents a computation that can yield such data:
```F#
let naturals =
	seq {                   // imperative programming style of code
        let mutable i = 0   // mutable variable
        while true do       // infinite loop, will not be executed endlessly
            yield i         // returns the current value
            i <- i + 1
    }
;;

let first1000 = Seq.take 1000 naturals  // only takes the first 1000 naturals
```
The sequence has type ```seq<'T>```, where ```T``` is the type yielded. When a value is returned from the sequence, executions stops, then, if other values are needed, execution resumes from where was interrupted. Another interesting function is ```Seq.unfold```:
```F#
let powersOfTwo =
    Seq.unfold (fun state -> Some(state, state * 2)) 1
;;

powersOfTwo |> Seq.take 8 |> Seq.toList ;; // -> [1; 2; 4; 8; 16; 32; 64; 128]
```
It takes a function that takes a value and produces a pair with the value and the next value it has to produce, or it returns ```None```, determining when it stops. If ```None``` is not returned, like in the previous example, then the sequence created never ends. 

### I/O in F\# and computational expressions

The strength of functional programming is that the purity of the language, i.e. the lack of side-effects, as this allows for trivially testable, parallelable and composable programs. However, all meaningful programs must, in some way or another, perform non-pure actions, such as opening a socket, writing a file, ... .

The solution to this problem was to distinguish pure from non-pure functions using the type system: a impure function does not return just some value while performing side-effects in its body, it returns a "recipe" of IO. The structure that achieves this is a **Monad** (more on [[Haskell]]), although they are called <span style="color:rgb(0, 112, 192)"><b>computational expressions</b></span> in F#.

A monad value just describes an effect and is on itself a pure value, i.e. its existence does not affect the purity of the program. However, when runtime happens, the effect it describes is actually executed. This allows to distinguish functions that use monadic values, i.e. functions that perform side-effects, from pure one on the level of the type system.

A monad is a type constructor ```M<'a>``` that encapsulates a value of type ```'a``` into the monadic context. It must also implement a *return* function of type ```'a -> M<'a>``` which, given a pure value, it lifts it to the monadic context, and it must also provide a *bind* ```(>>=)``` function of type ```M<'a> -> (a -> M<'b>) -> <'b>```, which is able to compose monadic values. 

This is how  sequences in F# implement monads:
```F#
// return for sequences: wrap one value in a sequence
let return x = seq { yield x }

// bind for sequences: map and flatten
let bind (m: seq<'a>) (f: 'a -> seq<'b>) : seq<'b> =
    seq { for x in m do yield! f x }
```
This implementation is similar to that of the list monad of Haskell.

When instantiating a monad, we have to respect 3 monad laws:
- **Left identity:** ```return x >>= f = f x``` 
- **Right identity:** ```m >>= return = m``` 
- **Associativity:** ```(x >>= f) >>= g = x >>= (\x -> f x >>= g)``` 

These laws are required for the monad to behave as intended; however, it is not the compiler that enforces them, as proving they hold is an undecidable problem. 

#### Computational expressions

Let us consider the following example:
```F#
// Three sequenced operations written as raw bind calls
bind (getUser userId) (fun user ->
    bind (getPermissions user) (fun perms ->
        bind (loadDocument docId) (fun doc ->
            return (renderPage user perms doc)
        )
    )
)
```
In this example the bind operator is used to compose the *maybe monad*; however, F# allows to hide the bind operator behind syntactic sugar, in particular through the use of <span style="color:rgb(0, 112, 192)"><b>computational expressions</b></span>. We can see that the typical syntax of computational expression needs a builder, and then curly brackets which surround the computation under that builder; for instance, the previous example can be re-written as follows:
```F#
maybe {
	let! user = getUser userId
	let! perms = getPermissions user
	let! doc = loadDocument docId
	return renderPage user perms doc
}
```
The ```let``` keyword is different from just ```let!``` because the former describes binding a variable to an expression, while the latter performs a computation and binds its result with a variable. This means that if we were to use ```let``` inside the maybe block, we would be binding a variable to the recipe of the computation, i.e. the variable would represent the unexecuted computation.

These keywords are only available in monadic composition, another example would be the ```do!```keyword that just performs the computation and ignores its result, just like the *then* operator ```>>``` (see more in [[Haskell]]). Other keywords are: ```return```which wraps a value in  a monad, ```return!``` which is used when the value to return is already a monad and we don't want to wrap it again (we cannot avoid to write it like we would in Haskell), ```yield```which is used when we want to produce multiple result, like we did for sequences, and ```yield!``` which returns all the results of a computation and flattens them.

This sugaring process creates an imperative-like syntax for computations, while the compiler keeps seeing them as just series of *bind* applications.

### The Async monad

Usually, in synchronous programming, when a function is called, everything is blocked until we get the result of said function. However, in asynchronous programming, actions that do not require the CPU (like waiting for input or downloding a file) are left to finish, while something else is being computed. This is a bit different from parallelism, where multiple executions are performed at the same time using multiple CPUs or using one but interleaved.

If these operations were synchronous, execution would be blocked until we get a result; let us consider this JavaScript example:
```javascript
const user = getUser();
console.log(user);

doSomethingElse(); //does not use user
```
We would not be able to advance the computation because we need user right away. For this reason, asynchronous programming uses *callbacks*, which consist in passing all the operations that are needed on the waited result to the function itself, so that it can do them when the result arrives:
```javascript
const user = getUser2((user => console.log(user))); //takes a function that will be executed on the waited result

doSomethingElse(); //does not use user
```
Now, while we are waitin for the result, we can asynchronously do ```doSomethingElse```, and when the result arrives, we know what to do because we will have passed all the actions that need to be done on that value through a function. This is called **Continuation Passing Style** (CPS), where a function (the continuation) is passed to be applied to the resutl of some computation.

The problem with the callback approach is that it can create many functions nested inside the other, especially when the result of an asynchronous operation is used to perform another one. This is the typical problem that monad try to solve; for this reason, F# introduced the async monad, i.e. the async builder block, which allows to have asynchronous computations while keeping a synchronous-looking code.

```F#
open System
open System.IO

// Perform an asynchronous read of a file using 'async'
let printTotalFileBytesUsingAsync (path: string) =
    async {
        let! bytes = File.ReadAllBytesAsync(path) |> Async.AwaitTask
        let fileName = Path.GetFileName(path)
        printfn $"File {fileName} has %d{bytes.Length} bytes"
    }

[<EntryPoint>]
let main argv =
    printTotalFileBytesUsingAsync "path-to-file.txt"
    |> Async.RunSynchronously

	0
```
First, we define an asynchronous function, ```printTotalFileBytesUsingAsync```, which in fact begins with an async block. We call a function that returns the number of bytes read asynchronously. This function is part of the .NET ecosystem (more on [[dotNET runtime]]), meaning that it returns a ```Task<byte[]>```, i.e. the type of the .NET standard task abstraction; in order to get the specific F# ```Async,byte[]>```, we pass this value to ```Async.AwaitTask``` function. 
When calling the asynchronous function, we pass it to ```Async.RunSynchronously```, which wraps the asynchronous part around a synchronous bubble. This is done because most programs are typically synchronous; therefore, with this bubble we are creating a sort of environment where it is okay for the program to run asynchronously.

##### Javascript Promises

JavaScript solved the callback problem using <span style="color:rgb(0, 112, 192)"><b>Promises</b></span>, which are an object that act as a proxy to a value that could be still unknown whe the promise is created. They can be in one of three possible states:
- *Pending*: this is the initial state, the computation has not ended yet.
- *Fulfilled*: the operation has been completed successfully.
- *Rejected*: the operation has failed.
Promises are actually just monads in disguise; for example, the ```.then``` method is a kind of *bind* operator, it allows for concatenation of promises, it takes two functions, the first to be applied in the case of a successful execution, the other in the case of an erroneous execution. Instead, the ```.resolve``` method is just like a *return* operator.

An example of the use of premises can be seen here:
```JavaScript
myPromise
  .then(handleFulfilledA)
  .then(handleFulfilledB)
  .then(handleFulfilledC)
  .catch(handleRejectedAny);
```
We use the ```.then``` operator to concatenate asynchronous actions, without specifying what to do in the case of an error because that is done with the ```.catch``` method.

### Computational expressions as a DSL mechanism

The ML (Meta-Language) family of languages, aimed to create, as the name suggests, a language that could manipulate other language. The objective was to create a language that provided one syntax but multiples semantics, which could be defined to create a **Domain-Specific Language** (DSL).

In that regard, using computational expressions allows the use of a specific syntax (for example ```!let```, ```return```, ...) to provide multiple semantics, that depend on the builder block used, e.g. *maybe*, *async*, ... . This allows for creating multiple DSLs, which comes at the price of determining the semantics of a block. Everybody knows, without ambiguity, what ```let``` does in a programming language; however, when we are in a DSL, the semantics depends on the language that we built. For instance, it is not straightforward to understand what ```let!``` does in an unknown computational expression.

#### References

- F# documentation - <a href=https://learn.microsoft.com/en-us/dotnet/fsharp>Microsoft Learn</a>
- F# computational expressions - <a href=https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/computation-expressions>Microsoft Learn</a> 
- F# asynchronous programming - <a href=https://learn.microsoft.com/en-us/dotnet/fsharp/tutorials/async>Microsoft Learn</a> 
- JavaScript Promises - <a href=https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise>Mozilla</a>
