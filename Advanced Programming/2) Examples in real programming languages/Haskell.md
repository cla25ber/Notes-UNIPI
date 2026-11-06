#advanced_programming #computer_science

[[AP index]] (2)

<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">30/10/2025 & 10/11/2025</span></p>

## Polymorphism and type classes

Many functions are defined to be polymorphic, e.g. the $\texttt{length}$ function, the $\texttt{member}$ function and the $\texttt{sort}$ function:
```Haskell
length :: [a] -> Int
member :: [a] -> a -> Bool
sort :: [a] -> [a]
```

However, some of them are not actually "fully" polymorphic. For example, the $\texttt{member}$ function is polymorphic only on those types that support equality (determining function equality is undecidable, therefore, in programming languages they are typically incomparable), while the $\texttt{sort}$ function only on those that support an ordering.

One solution could be the one adopted first by Miranda (functional programming language that later inspired Haskell), which is to allow polymorphism of any type, and then throw a run-time error whenever an incomparable type was compared. SML (Standard ML) instead identifies comparable types by using $\texttt{eqtype}$ types, identified by being written as ```"a``` instead of ```'a```. 

Haskell generalized the idea of $\texttt{eqtype}$, creating <span style="color:rgb(0, 112, 192)"><b>Type Classes</b></span>: a construct that determine which types have which characteristics. For example, under the $\texttt{Eq}$ type class fall all those types that support equality, under $\texttt{Ord}$ all of those that can support ordering, and so on.

```Haskell
class Eq a where
	(==) :: a -> a -> Bool
	(/=) :: a -> a -> Bool
```

For instance, the declaration of the $\texttt{Eq}$ type class can be instantiated for any type $\texttt{a}$ as long as an equality function $\texttt{==}$ and an inequality function $\texttt{/=}$ get defined on it. We could create our own type class:
```Haskell
class YesNo a where
	yn :: a -> Bool
	
instance YesNo Int where
	yn 0 = False
	yn _ = True
	
instance YesNo [a] where
	yn [] = False
	yn _ = True
	
instance YesNo Bool where
	yn = id -- identity function
```

The $\texttt{YesNo}$ type class takes can be defined on all those types $\texttt{a}$ where a $\texttt{yn}$ function get defined; the $\texttt{yn}$ function takes an element of that type and returns a boolean. For how we have instantiated it, it defines falsy values on various types, allowing them to be used as booleans. For example, for integers $0$ can be used as false, while every other number would be considered true.

We can add these restrictions on types in the function's type:
```Haskell
length :: [a] -> Int
member :: Eq a => [a] -> a -> Bool -- type a must be of class Eq
sort :: Ord a => [a] -> [a] -- type a must be of class Ord
```
So, $\texttt{member}$ only accepts comparable types, $\texttt{sort}$ only accept orderable types, while $\texttt{length}$ is truly polymorphic.

When defining a new type (which is done with the $\texttt{data}$ keyword in Haskell), we can ask the compiler to instantiate it to a specific type class using the $\texttt{deriving}$ keyword:
```Haskell
data TrafficLight = Red | Yellow | Green
	deriving (Eq, Ord, Show)
```

#### Compiling type classes

Let us consider the following type class, with an instance on $\texttt{Int}$ and the $\texttt{square}$ function:
```Haskell
class Num a where -- class for types that can be used with arithmetic operations
	(+) :: a -> a -> a
	(*) :: a -> a -> a
	-- etc

instance Num Int where
	a+b = intPlus a b -- where intPlus is a primitive
	a*b = intTimes a b -- where intTimes is a primitive
	-- etc

square :: Num a => a -> a
square x = x*x
```

First, the type class is converted into a data type:
```Haskell
data NumDict a = MkNumDict (a -> a -> a) (a -> a -> a) -- ...

getPlus :: NumDict a -> (a -> a -> a)
getPlus (MkNumDict p t) = p

getTimes :: NumDict a -> (a -> a -> a)
getTimes (MkNumDict p t) = t
```
The type class is turned into a dictionary of operations (in our specific case two binary operations: + and \*); it also creates helper functions, i.e. $\texttt{getPlus}$ and $\texttt{getTimes}$, to extract the operations.  Then, the instance is translated to:
```Haskell
intDict :: NumDict Int
intDict = MkNumDict intPlus intTimes
```
The dictionary is instantiated for the Int type, providing the primitive operations for addition and multiplication. Finally, the $\texttt{square}$ function is compiled as follows:
```Haskell
square :: NumDict a -> a -> a
square d x = 
	let times = getTimes d in
	times x x
```
So, we get a new parameter, the dictionary that tells us the multiplication operation that we should use. This avoids the code blow-up that would result from creating one function for each type (like e should have done in languages like C). While it is true that we still have to create dictionaries for every type ($\texttt{NumDict Int}$, $\texttt{NumDict Float}$, ...), the space it takes is far less than the one for replicating the bodies of functions. Furthermore, if a function were to be polymorphic on more parameters, we would have to create one not only for every type, but one for every combination, resulting in an even bigger blow-up, potentially even an exponential blow-up.

It is important to note that even integer literals are overloaded in Haskell, for example, the type of $1$ is $\texttt{Num a => a}$. Therefore, rather than an addition between a $\texttt{Char}$ and an $\texttt{Int}$ failing because it is not possible in the language, it fails because there is no instance for $\texttt{Num Char}$. Consequently, if we were to define it, it would be possible to sum integers with characters.

#### Type inference with type classes

A classic type inference algorithm parses the code and assigns types to each node of the parse tree; however, in Haskell, we also need to infer the constraints, so we have to infer $\texttt{T}$ and $\texttt{Q}$ to get the qualified type $\texttt{Q => T}$. For instance, we can consider the following example to see how an algorithm like the one we described could operate:
```Haskell
example z xs =
	case xs of
		[] -> False
		(y:ys) -> y>z || (y==z && ys==[z])
```
Clearly the type of he function is $\texttt{a -> [a] -> Bool}$, and that is the $\texttt{T}$ part. However, we can see that, since we perform $\texttt{y>z}$, $\texttt{y==z}$ and $\texttt{ys==[z]}$, that  $\texttt{Q}$ will be composed of the following constraints: $\texttt{(Ord a, Eq a, Eq [a])}$.  Nevertheless, these constraints simplify each other; for instance, if we have $\texttt{Eq a}$, then we can remove $\texttt{Eq [a]}$, since to compare lists we just need to compare their items. Furthermore, if we have $\texttt{Ord a}$, we can remove $\texttt{Eq a}$, since being able to order items implies that we can compare them, and thus the only constraint remaining is $\texttt{Ord a}$.

## Type constructors

We can think of type constructors as functions over types: given a basic predicate, we get another basic predicate. For example, we can have the list type constructor $\texttt{[]}$, which, given a basic type $\texttt{a}$, returns the basic type $\texttt{[a]}$; or others like $\texttt{Maybe}$.

Just like we had type classes for basic types, we can have <span style="color:rgb(0, 112, 192)"><b>Constructor Classes</b></span> (or Type Constructor Classes) for type constructor. An example is the $\texttt{Functor}$ constructor class, which is defined as follows:
```Haskell
class Functor f where
	fmap :: (a -> b) -> f a -> f b
```
Basically, it is like a $\texttt{map}$ function which, instead of operating on just lists, it operates on any type constructor. So, if we think of type constructor as boxes, this $\texttt{fmap}$ is applied on what is inside it without modifying the actual box. Here are the instance of it for type constructors like $\texttt{[]}$ and $\texttt{Maybe}$:
```Haskell
instance Functor [] where
	fmap :: (a -> b) -> [a] -> [b]
	fmap f [] = []
	fmap f (x:ls) = (f x):(fmap f ls)
	
instance Functor Maybe where
	fmap :: (a -> b) -> Maybe a -> Maybe b
	fmap f (Just x) = Just (f x)
	fmap f Nothing = Nothing
```

### Monads

Let us consider the following example:
```Haskell
father :: Person -> Maybe Person -- given a person returns their father (if present in the DB)
mother :: Person -> Maybe Person -- same but for the mother
```
If we wanted to find both grandfathers of a person we would have to write:
```Haskell
bothGrandfathers :: Person -> Maybe (Person, Person)
bothGrandfathers p =
	case father p of
		Nothing -> Nothing
		Just dad ->
			case father dad of 
				Nothing -> Nothing
				Just grandfather1 ->
					case mother p of
						Nothing -> Nothing
						Just mom ->
							case father mom of
								Nothing -> Nothing
								Just grandfather2 -> 
									Just (grandfather1, grandfather2)
```
We can see that the generic piece of code that perpetuates $\texttt{Nothing}$ if it was also received, and does something else otherwise, can actually be written as the <span style="color:rgb(0, 112, 192)"><i>bind</i></span> operator, which has the following type:
```Haskell
(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b -- bind operator
(>>=) m f =
	case m of
		Nothing -> Nothing
		Just x -> f x 
		
return :: a -> Maybe a -- return operator
return x = Just x
		
-- rewriting the bothGrandfathers function using bind
bothGrandfathers2 p =
	father p >>= 
		(\dad -> father dad >>=
			(\grandfather1 -> mother p >>=
				(\mom -> father mom >>=
					\grandfather2 -> return (grandFather1, grandfather2)
				)))

-- rewriting again the bothGrandfathers function using the do construct
bothGrandfathers3 p = do { -- do is syntactic sugar for bind operator >>=
	dad <- father p;
	grandfather1 <- father dad;
	mom <- mother p;
	grandfather2 <- father mom;
	return (grandfather1, grandfather2);
}
```
A monad generalizes this concept not only for the $\texttt{Maybe}$ but for any type constructor:
```Haskell
class Monad m where
	return :: a -> m a -- return
	(>>=) :: m a -> (a -> m b) -> m b -- bind
	(>>) :: m a -> m b -> m b -- then
	-- and more things
```
It is difficult to grasp what a monad does because it differs depending on the type constructor it receives. Generally speaking, we can see monads as describing some kind of computation. For example, with $\texttt{Maybe}$ we have seen that it allowed to model partial functions. Instead, with $\texttt{[]}$, we can see that it models non-deterministic computations:
```Haskell
instance Monad [] where
	return :: a -> [a]
	return x = [x]
	
	(>>=) ls f = concat (map f ls) -- concat flattens lists
	
	-- more stuff to instantiate the class
```
So the bind operator takes a list, which represent the possible inputs, and a function, which represent the computation that is going to be performed, and generates all the possible outputs by applying it to every input and concatenating all the resulting lists, i.e. all the resulting outputs.

In the monad definition, we saw the bind $\texttt{>>=}$ operator but also the <span style="color:rgb(0, 112, 192)"><i>then</i></span> $\texttt{>>}$ operator. While $\texttt{x >>= f}$ means "run x and then apply f to its result", $\texttt{x >> y}$ means "run x, discard its result and then run y". In the $\texttt{Monad Maybe}$ case this can become useful when checking if the computation was successful, for example:
```Haskell
checkLength :: String -> Maybe () 
checkLength password = 
	if length password >= 8 
		then Just () 
		else Nothing

checkDigit :: String -> Maybe ()
checkDigit password = 
	if any isDigit password 
		then Just () 
		else Nothing 

checkUppercase :: String -> Maybe () 
checkUppercase password = 
	if any isUpper password 
		then Just () 
		else Nothing
		
validatePassword :: String -> Maybe ()
validatePassword password =
	checkLength password >> checkDigit password >> checkUppercase password
```
Here every function we use just returns $\texttt{Maybe()}$ when succeeding, so it does not give any useful information more than that. As a result, we can use the then $\texttt{>>}$ operator instead of $\texttt{>>=}$.

### I/O in Haskell

The main gig of functional programming is the use of pure functions and, therefore, the Church-Rosser theorem (also called Confluence theorem) which states that the order in which expressions are evaluated does not matter, as the result will always be the same.

Unfortunately, as soon as we stop considering just pure functions and introduce functions with side-effects, the Church-Rosser theorem stops holding. Let us consider the following example (written in a generic programming language):
```
	print "Hello" ; print "World"
```
If it still were to hold, executing ```print "World"```before ```print "Hello"``` would not be different that the opposite; however, one prints ``` HelloWorld``` while the other ```WorldHello```.

Furthermore, a key difference is the possibility to substitute expression:
```
	let f () = print "Hi"; 5
```
This functions prints ```Hi``` and then returns $5$. If $\texttt{f}$ were to be a pure mathematical function, we could substitute the expression ```5+5``` with ```f() + f()```, but that is not possible due to $\texttt{f}$ having side-effects.

Functional languages like OCaml use the $\texttt{;}$ operand to determine which expression to evaluate before another (sequential composition), effectively introducing imperative constructs into the language. The problem with this approach is that it does not solve the substitution problem, and these "impure" functions are not treated like the pure ones.

Haskell takes a different approach, it adds these imperative feature while keeping the "purity" of the language. It does so by, first, defining what $\texttt{IO}$ is:
```Haskell
	type IO t = World -> (t, World)
```
So, something of type $\texttt{IO t}$ is an action that, when performed, may do some input/output, before delivering a result of type $\texttt{t}$. Therefore, given a state of the world, after performing the action we will get the result (of type $\texttt{t}$) and a new state of the world.

$\texttt{IO}$ is defined to be a type constructor, but in order to enforce the sequential composition needed for actual input/output operations we are going to use monads, in particular a $\texttt{Monad IO}$.

```Haskell
	instance Monad IO where
	return :: a -> IO a
	return x = (\w -> (a,w))
	
	(>>=) :: IO a -> (a -> IO b) -> IO b
	(>>=) m k = (\w -> case m w of (r, w') -> (k r) w')
	-- generates a function that takes a w :: World
	-- takes the current m :: IO a and gives it the w :: World
	-- this generates a (r, w') :: (a, World)
	-- we then perform the function on the result (k r) which returns an IO b
	-- we give w' :: World to the IO b and generates a (b, World)
	-- TLDR (>==) m k generates an unnamed function which takes a World and returns a (b, World), which means that it returns a IO b
```

Now it is clear that in order to perform an IO action we need the $\texttt{World}$ from the previous one, we cannot proceed without it, effectively creating sequential composition.

The strength of this approach comes from the fact that, despite many operands being able to return IO actions, $\texttt{>>=}$ is the only operator that can actually take one, which means that it is also the only one that can combine them, forcing sequentiality. Unfortunately, this implementation also comes with some drawbacks. Suppose that we have a configuration file that we read at the start of the program; after reading it we will have an $\texttt{IO String}$, this means that we have to write the entire program considering the IO monad, which complicates greatly the code, or we use a function of type $\texttt{IO String -> String}$. However, such function does not exist and cannot exist, because we would be able to exit the "IO realm" even when executing side-effects. For example, if ```runIO :: IO String -> String``` existed, then we could write something like:
```Haskell
name :: String
name = runIO getLine
```
So the variable name would have type ```String``` suggesting that it is pure, while actually performing an IO action. 

In Haskell there is actually such a function:
```Haskell
unsafePerformIO :: IO a -> a
```
However, as the name suggests, its use is discouraged. This is because using it breaks the soundness of the proof system and requires the promise to the compiler that the timing of this operation relative to all other operations does not matter.

#### Monad laws

When defining a monad instance, the programmer is expected to respect the following laws, called "monad laws": 
```Haskell
return x >>= f  = f x                     -- left identity
m >>= return    = m                       -- right identity
(x >>= f) >>= g = x >>= (\v -> f v >>= g) -- associativity
```

The first monad (left identity) law states that if return turns a value into a monad, and then immediately bind it to $\texttt{f}$, it is the same as just applying $\texttt{f}$ to the pure initial value. This is because bind extracts the value from the constructor and applies the function, so we just get that it is put and then taken out.

The second monad (right identity) law states that if the function to be applied when binding is return, then nothing changes. This is because bind extracts the value from the constructor and then applies return which puts it again inside it, effectively resulting in the identity.

The third monad law (associativity) states that binding is associative.

These laws are not enforced by Haskell because that would require Haskell to be a theorem prover. 

#### References

- Lecture Notes 30/10/2025 - Andrea Corradini
- Lecture Notes 10/11/2025 - Andrea Corradini
- Haskell Monads - <a href=https://mvanier.livejournal.com/3917.html>Yet Another Monad Tutorial (Part 1)</a> 