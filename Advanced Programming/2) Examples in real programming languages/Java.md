#advanced_programming #computer_science

[[AP index]] (2)

<p style="text-align:right;"><span style="color:rgb(255, 0, 0)">29/10/2025 & 17/11/2025 & 19/11/2025</span></p>

# Generics

Recalling the classification of polymorphism given in [[Type systems]], Java generics provide explicit parametric polymorphism. They were introduced in Java 5 to avoid using `Object` as type for creating polymorphic functions, as this could create problems with down-casting and `ClassCastException`. For instance, the following example creates a generic `Box` class that stores a value:
```Java
public class Box<T> {
	private T t;
	
	public void set(T t) {this.t = t; }
	public T get() {return this.t; }
}
```
Then, it can be instantiated in many types: `Box<Integer>`, `Box<String>`, ... .
It is also possible to create generic methods, for example, here the `Util` class creates a `compare` method for comparing pairs. `Pair` was a generic class that took two generic types to form a pair, so the method `compare` instantiates two generics, which are then used to define the type of `Pair`:
```Java
public class Pair<K, V> {

    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
}

public class Util {
    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
        return p1.getKey().equals(p2.getKey()) &&
               p1.getValue().equals(p2.getValue());
    }
}
```

## Type bounds

Sometimes allowing for any type inside a function is not the desired behaviour of the program; for this reason, it is possible to bound which types can be passed into a generic declaration with type bounds, which effectively require an interface to be implemented by the type for it to be used in that context. For example, we specify that the type must implement the `Comparable` interface in order for it to be used inside the method:
```Java
public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e > elem)
            ++count;
    return count;
}
```

## Inheritance and generics

Java already used inclusion universal polymorphism, which was based on the concept of *inheritance*. Introducing also generics caused a problem, if $\texttt{Integer}<:\texttt{Number}$, then is it true that $\texttt{List<Integer>}<:\texttt{List<Number>}$ ? For generic classes this may not be the case, i.e. they are **invariant** with respect to subtyping. Instead, if a class $\texttt{A}$ extends a class $\texttt{B}$, then $\texttt{A<C>}:<\texttt{B<C>}$ for any type $\texttt{C}$. Recalling what was said in [[Type systems]], we have to consider constructor that are **covariant**, **contravariant** and **invariant**, and $\texttt{List}$ is invariant; however, immutable lists, or read-only lists are covariant.

### The array covariance problem

Considering what we have said about inclusion polymorphism, one would expect arrays to be invariant in Java; however, they are actually covariant. This is because the introduction of generics, there was still the need of polymorphic methods, like for example a generic `sort` function which sorted elements of a generic array:
```Java
public static void sort(Object[] o);
```
Had covariance not been implemented for arrays, there would be the need to redefine a `sort` function whenever an object different from `Object` was used; however, sorting does not insert any new element in the array, so it could be used covariantly. When generic were introduced, they were implemented to be invariant, but there was already many Java Libraries relying on the covariance of arrays, meaning that changing also array to be invariant would have caused a massive compatibility problem with previous Java versions. This means that the following code compiles correctly, but throws a runtime exception:
```Java
Apple[] apples = new Apple[1];
Fruit[] fruits = apples; // LEGAL: Arrays are covariant.

fruits[0] = new Strawberry(); // COMPILES, BUT CRASHES AT RUNTIME!
// The JVM catches this at runtime and throws an ArrayStoreException.
```

This means that the JVM must check every array insertion at runtime. The JVM also performs **type erasure**, which consists in transforming every type parameter into `Object` or to their first bound after compilation. This was done to avoid backward compatibility problem, if information about generics is kept at the bytecode level, then it would mean creating machinery to determine how these new constructs interact with the rest of the code through assignment, method calls and so on. This meant that keeping information about generics would cause a clash with old bytecode, resulting in overhead due to the new rules about these constructs being introduced. Due to type erasure we get this:
```Java
List<String> lst1 = new ArrayList<String>();
List<Integer> lst2 = new ArrayList<Integer>();
lst1.getClass() == lst2.getClass()    // returns true
```

Type erasure also causes arrays of generics to not be supported in Java. Suppose we had `List<String>[]`, if we create a `List<Integer>` and try to add it to the previous one, we would get a compilation error, because their type is different and they are not related by subtyping. Instead, if we create an `Object[]` and set it equal to the array of lists of strings, then this wouldn't create any problem due to the covariance of array. However, if we then create the list of integers and assign it as the first value of the object array defined previously, this also wouldn't cause any problems because $\texttt{List<Integer>}<:\texttt{Object}$. One would think that the JVM, checking every array insertion at runtime, would detect this problem; however, since type erasure is applied, it would not be able to distinguish between `List<String>` and `List<Integer>`, which would both appear as `List`. This very same example is shown here:
```Java
List<String>[] lsa = new List<String>[10]; // illegal
Object[] oa = lsa; // OK by covariance of arrays
List<Integer> li = new ArrayList<Integer>();
li.add(new Integer(3));
oa[0] = li; // should throw ArrayStoreExeception,
// but JVM only sees “oa[0]:List = li:ArrayList”
String s = lsa[0].get(0); // type error !!
```

For this reason, arrays of generics are not supported, as they would break the strong type guarantee of Java.

### Use-site variance

When defining generics, many languages allow the user to specify whether they will be covariant or contravariant, for example in C# the `out` keyword used before a generic means that it will be covariant, while `in` contravariant. Similarly, Scala and OCaml use `+` for covariant and `-` for contravariant generics. This is **use-site variance**.

Instead, Java uses a different approach, using the a wildcard `?` to represent an unknown type. As a standalone operator, `?` can be used to represent unknown types used only once, for example:
```Java
Class<?> c = x.getClass();
```
Not knowing the class of the variable `x`, we use the wildcard.

Nevertheless, wildcards are used to represent use-site variance; for instance `? extends Type` denotes a subtype of `Type`, while `? super Type` denoted a supertype of `Type`. The former lets us express covariance, while the latter contravariance. Let us now consider the following example:
```Java
List<Apple> apples = new ArrayList<Apple>();
List<? extends Fruit> fruits = apples; //covariance
fruits.add(new Strawberry()); // compile-time error! OK
Fruits f = fruits.get(0); // OK
fruits.add(new Apple()); // compile-time error
```
When creating a `List<? extends Fruit> fruits`, we are saying that `fruits` can be a list of any type that is a subtype of `Fruit`, e.g. `List<Apple>`, `List<Strawberry>`, or even `List<Fruit>`, so we can assign `apples`to it because of covariance. On the other hand, when trying to modifying the list by adding a `Strawberry`, we get a compile-time error, because we don't know exactly what kind of list `List<? extends Fruit>` is, but exactly for this reason, even adding an `Apple` is not allowed.
Similarly, for contravariance, `List<? super Apples>` can represent, for example, `List<Apple>`, `List<Fruit>` but even `List<Object>`, and thus we cannot add a `Fruit`.

Due to generics and type erasure, we are not allowed to create arrays of generics, but there other limitations. For example, we cannot use primitive types in place of generics `List<int>` because it cannot be converted into a subtype of `Object` in bytecode. Furthermore, we cannot use an instantiated generic with `instanceof`, because we can't know its specific instantiated type at runtime due to type erasure.

# Lambda expressions and stream API

With Java 8, many functional constructs were introduced, like **lambda expressions**, i.e. anonymous functions, and **streams**, which consist in a sequence of elements supporting sequential and parallel operations. 

## Lambda expressions

The addition of lambda expressions in Java allowed for passing behaviours other than data. Here is an example of passing behaviour before and after Java 8:
```Java
// pre Java 8
button.addActionListener(new ActionListener() {
	public void actionPerformed(ActionEvent evt) {
		System.out.println("Handled by anonymous class listener");
	}
});
// An anonymous class is created and passed with the intended method inside

// Java 8
button.addActionListener(
e -> System.out.println("Handled by Lambda listener"));
// An anonymous function is passed
```

When a lambda expression is generated, it is converted by the compiler into an inner class implementing just that method; for instance `x -> System.out.println(x)` could be converted into:
```Java
public static void genName(Integer x) {
	System.out.println(x);
}
```

Then, functional interfaces, i.e. an interface that just has exactly one abstract method, are used to basically give a type to a lambda (Java lacks typical function types `->` of functional languages), which can then be used for type-checking.
This is because, when a lambda expression is passed, it must satisfy the functional interface of the target, i.e. of what it is passed to: here are some examples of Java-defined functional interfaces:
```Java
public interface Comparator<T> {  //java.util
	int compare(T o1, T o2);
}
public interface Runnable {       //java.lang
	void run();
}
public interface Consumer<T>{     //java.util.function
	void accept(T t)
}
public interface Callable<V> {    //java.util.concurrent
	V call() throws Exception;
}
```
Since lambda expressions need to satisfy the interface they supposedly implement, this allows to type-check them. 

Now, in modern JVM, not always a lambda expression is created whenever there is a lambda expression. Sometimes the compiler converts the lambda expression into a method containing the body of the lambda, and uses an `invokedynamic` instruction, which postpones the creation of the object at runtime. Basically, when the `invokedynamics` instruction is executed, it creates an object implementing the target functional interface, putting the method created for the lambda as the method of the interface. This allows to optimize performance.

#### Lambda expressions and variable capture

A lambda expression is basically a function that is passed to another function, which means that if some local variables are used inside of it, there could be some ambiguity on its behaviour. Suppose we have:
```Java
public class LVCExample {
	public static void main(String[] args) {
		List<Integer> intSeq = Arrays.asList(1,2,3);
		int var = 10; // must be [effectively] final
		intSeq.forEach(x -> System.out.println(x + var));
		var = 3; // this line does not compile
	}
}
```
Should the lambda expression add `10` or `3`? Since we don't know when a lambda expression will be executed, Java does not allow this behaviour, so the previous code does not compile. For this reason Java allows the use of local variables inside of lambdas if and only if they are final or effectively final, i.e. their value is not changed after the definition of the lambda. When a local variable is used, a copy is created for the lambda expression, so that, even if the stack where the variable and the lambda are defined is discarded, the lambda can still retrieve the supposed value of the variable. This means that changing the original variable would cause discrepancies with the copy that the lambda stored.

Instead, had it been an attribute of an object (for example `obj.var`), the situation would have been different. This is because `obj` is saved on the heap and not on the stack, so, copying `obj` for the lambda would just mean saving a reference to the object, and thus this would not cause discrepancies if one of its attributes were to change value.

### The problem with backward compatibility

We have now defined lambda expressions, and with them methods that take them as arguments, like `forEach` or, as we will se later, `stream` from the `Collection` interface. However, implementing these methods in already defined interfaces would have caused a problem with every already existing class that implemented them. One could think of adding these method as static; however, by doing this they would become methods of the class itself, not its instantiations, which would nullify any effort of introducing a method like `forEach`. If it were added as a static method, then it would have been called on the class, but `forEach` clearly only works in an instance of the class, iterating on the elements of the collection.

For this reason, **default methods** were also introduced, which effectively provide a default implementation of a method in case the instantiation does not provide it. To avoid ambiguity, Java gives strict resolution rules regarding which method gets used: methods defined in classes win over interfaces, and if two interfaces conflict, a compile error forces the programmer to choose one overriding the other.

## Streams

Streams take collections of objects and transform them into a flow of data to which a pipeline of operations is applied. It is functional in nature, in fact, it does not modify the original collection. Here is an example of a stream:
```Java
int sum = widgets.stream()   // widget is a Collection<Widget>
				  .filter(w -> w.getColor() == RED)
				  .mapToInt(w -> w.getWeight())
				  .sum();
```
Here we apply `stream` to a collection, creating a flow of data, and then we apply many operations to obtain a final result.

Streams consists of many intermediate operations, and a final one, like, for example, `findFirst`. This allows for laziness, in fact the collection, instead of being processed horizontally, i.e. layer by layer, is processed vertically, i.e. element by element, allowing for optimizations. For instance, if the final operation is `findFirst`, we can just stop as soon as we find the first element passing through the pipeline.

Streams can be generated by `Collection`, `Array`, `Files`, but also by generators. This allows for possibly infinite sequences, which, in principle, is not a problem if we use laziness to terminate operations before visiting all elements (this clearly does not work with terminal operations such as `sum`).

### Aggregation

Among the various terminal operation that can be used in with streams, there is also `reduce`, that acts as a `fold`. For instance, the previous stream example could be written this way using `reduce`:
```Java
int sum = widgets.stream()   // widget is a Collection<Widget>
				  .filter(w -> w.getColor() == RED)
				  .mapToInt(w -> w.getWeight())
				  .reduce(0, (a,b) -> a + b);
```
We provide the base case for 0 elements, i.e. `0`, and then what to do with more elements (addition). This will result in summing all numbers in the sequence.

Now, suppose that we want to concatenate strings from a list:
```Java
String concatenated = listOfStrings.stream()
									.reduce("", String::concat)
```
Since strings are immutable in Java, using `reduce` would result in creating a new string for every string that passes through the pipeline of the stream, which would be highly inefficient. To solve this issues, `collect` is used, which requires:
- **Supplier:** to create an empty container (analogous to the base case of `reduce`).
- **Accumulator:** to add one element to the container.
- **Combiner:** to merge two containers.

These three operations can be passed not only as three functions, but also as a `Collector` method, which encapsulates them. For instance, the string example of earlier would be written like this:
```Java
String concatenated = streamOfString.collect(Collectors.joining());
```
Here `Collectors.joining()` returns the `Collector` object needed for concatenating strings, which can then be used by a stream with `collect`.

It is also possible to perform stream operation in parallel thanks to `parallelStream()`.  Obviously, to avoid interference with computations, the source of the stream shall not be modified while the stream is flowing. Generally, thread-safety is guaranteed even if the starting collection is not-necessarily thread-safe, e.g. `ArrayList`.

##### Monads in Java

With the implementation of many functional features in Java, monadic behaviour was also introduced to express more easily computation and concatenation of computation. Le us consider for example the `Optional` and `Stream` classes. To make a monad a `return` and `bind` operations are needed, the are expressed in these classes through the methods `of`and `flatMap`.
The method `Optional<T> of (T value)` takes `value` and returns an optional with that value, while `<U> Optional <U> flatMap(Function<? super T, Optional<U>> mapper)` takes a function and applies it to the value of option (if present).
This works similarly as monads in Haskell (more in [[Haskell]]), and the same goes for `Stream`, which is similar to the list monad of Haskell.

# Reflection

Reflection is the ability of a program to manipulate as data something representing the state of the program during execution. As reflection, we can also have:
- **Introspection:** the ability to observe and reason about its own state.
- **Intercession:** the ability to modify its own execution state, or alter its meaning.
Both of these aspects require *reification*, i.e. turning abstract concepts into concrete things, to make encodings of the execution state as data (that the program can reflect upon). 

We can identify two types of reflection:
- **Structural reflection:** the ability of a language to provide reification of the program currently executed and its abstract data types.
- **Behavioural reflection:** the ability of a language to provide reification of its own semantics and the implementation of the run-time system.
The first provides ways to act upon the structure of the program, like it is possible to do with quotations (more in [[Meta-Programming]]), while the second one allows for changing the semantics of established constructs. For instance, using behavioural reflection, it could be possible to change what happens when a function is called, not a specific function, but every function.

Generally speaking, if possible, reflection should be avoided. First of all, it causes performance overhead; that's because when reflection is performed, it involves dynamically resolved structures, where obviously no optimization can be performed. Furthermore, reflection requires runtime permission to access itself, which may satisfy the requirements of the system. 

Reflection in Java can do introspection but not intercession, i.e. it cannot modify code. Thanks to reflection, it is possible to use methods such as `getClass()` or access the `class` field. Also, when obtaining a class, it is possible to use methods like `getName()` and `getSuperClass()`. This is the same use of reflection that was discussed in [[dotNET runtime]].

An interesting example is the `getMethod(String name, Class<?>... parameterTypes)` method, which takes a name and the type of its parameters, and returns the method itself (through a `Method` interface). It is an interesting example because, due to type erasure described earlier, when searching for a method that uses generic we should put `Object.class` as its type.
```Java
LinkedList<String> list = new LinkedList<String>();
Class c = list.getClass();

Method add = c.getMethod("add", String.class) // does not work because String is                                                  lost with type erasure
Method add = c.getMethod("add", Object.class) // works
```

## Annotations

Java provides modifiers like `static`, `final` or `public` to add metadata describing properties of program elements. However, sometimes it is useful to define new metadata without the need to change the language, and this is where **annotations** come into play. They are indicated by the `@` symbol, and are basically user-definable modifiers that the compiler (or any other program that reads code) handles each in different ways. For example:
- `@Override`: makes explicit that the annotated method overrides the correspondent method of its superclass.
- `@Deprecated`: the annotated element is deprecated. For example, this annotation can be used by IDEs to suggest the usage of another, non-deprecated, method.
- `@SuppressWarnings`: instructs the compiler to avoid issuing warnings for the annotated element.

Using the `getAnnotations()` method, it is also possible to retrieve annotations. This is what JUnit, the most famous Java testing framework, uses to see which method use the `@Test` annotation, and, therefore, to test them.

#### References

- Lecture Notes 29/10/2025 - Andrea Corradini
- Java Generics - <a href=https://docs.oracle.com/javase/tutorial/java/generics/index.html>Oracle, Java tutorial</a> 
- Lecture Notes 17/11/2025 - Andrea Corradini
- Java Stream - <a href=https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html>Oracle, Java documentation</a> 
- Lecture Notes 19/11/2025 - Andrea Corradini