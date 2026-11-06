#advanced_programming #computer_science

[[AP index]] (3)

<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">05/11/2025 & 06/11/2025</span></p>

.NET is a free, open-source, cross-platform development platform for building different kinds of applications. It provides a runtime, a rich set of libraries, and development tools, creating a common ecosystem for supported programming languages.

At the application level, developers can use languages such as C# and F#. These languages have their own compilers, which can translate source code into a common intermediate representation called the <span style="color:rgb(0, 176, 80)"><i>Common Intermediate Language</i></span> (CIL), also known as Intermediate Language (IL). The resulting assemblies contain IL instructions and metadata describing types, methods, and other program elements.

The <span style="color:rgb(0, 176, 80)"><i>Common Language Runtime</i></span> (CLR) provides the execution environment for this managed code. In a typical .NET execution model, the runtime uses a Just-In-Time (JIT) compiler to translate IL into native machine code, which the processor can execute. The runtime also provides services such as garbage collection, type handling, and exception management.

This architecture is comparable to Java (more information on [[Java]]), where Java compilers typically translate source code into Java bytecode, which is executed by the Java Virtual Machine (JVM). In both ecosystems, multiple supported languages can target a common runtime and share libraries and runtime services. This facilitates interoperability and allows the platform to apply execution optimizations. Ultimately, the processor executes the native machine instructions, while the operating system provides the underlying services and resources that the application needs.

![[dotNET.svg]]

## Application level

Three languages are offered with continued support by Microsoft on the .NET platform: C#, F# and Visual Basic. 
- **C#** is the most general-purpose language of the .NET platform and is, mainly, an object-oriented language similar to Java, although it even incorporates features from functional programming languages. Similarly to C, it supports low-level features while offering more secure code than C.
- **F#** is a functional programming language that is able to blend the interoperability of the .NET platform with a pragmatic function-first approach. See [[F sharp]] for more specific information about the language itself.
- **Visual Basic** is generally deemed as an easily approachable language that favours clarity over brevity. It has a stable design where the practice of adding new syntax is generally avoided; for this reason, it is less integrated in the .NET platform than the previous two.

#### Class libraries

The strength of the .NET platform is its interoperability between different languages, which can be seen with class libraries. They are a collection of reusable code that defines classes, interfaces, methods and types that can be used by any of the supported languages in the .NET. This is possible because these libraries target the same runtime and type system. Therefore, when defining a new library, this is transformed into an <span style="color:rgb(255, 0, 0)">assembly</span>, which is then usable by any application. This is true even for user-defined libraries.

> [!NOTE]  
> **Assemblies** are the produced artifacts after the build level of the .NET platform. They are the building blocks of .NET applications and contain the compiled code and much more metadata.

## Build level

When writing code in C# or F#, the compiler translates it into CIL instructions, the intermediate representation of the .NET platform; then, many tools handle the different parts of building the application itself.

- The **Software Development Kit** (SDK) provides development tools and standard build infrastructure. The main development tools provided by the SDK are the .NET CLI and the .NET runtime. The former has the ```dotnet``` driver, which is used to execute command, e.g. ```dotnet new console``` creates a new console for a project, while ```dotnet fsi``` creates an interactive F# REPL.
- **MSBuild** coordinates the work needed to build the application. For instance, when running ```dotnet build``` it determines which tasks need to run, which files need to be recompiled and much more. It uses properties and targets to describe and execute the build process.
- **NuGet** is the package manager, it can download packages, resolve their dependencies, while also providing a way to publish the user's own libraries. 

## Artifact level

After the build level, assemblies are produced. They are the fundamental unit of deployment and security permissions for .NET applications. Assemblies are ```.exe```or ```.dll``` files, they can be divided into two part: the data, and the metadata, which is contained in the so-called *assembly manifest*. Some of the things defined in the assembly are the following:
- **Code**: compiled code that the CLR executes through an entry point (typically main).
- **Security boundary**: which permissions are requested and granted.
- **Type boundary**: it uniquely identifies all the types define inside of it, i.e. the assembly itself is part of the type identity, if two types with the same name are defined in different assemblies, they are distinguished by the assembly itself.
- **Deployment unit**: describes which assemblies are needed when the application first starts, and which can instead be retrieved on demand.

## Runtime level

A runtime is a collection of software services and mechanisms that support a program while it is running. For instance, C has a minimal one, providing ```malloc```, ```free```, ```printf``` and few others. Instead, Java and .NET have tried to build language-agnostic runtime, able to host all the languages that can compile to their intermediate bytecode. The main components that a modern runtime must provide are:
- **Type loader**: reads binary format and instantiates type structures.
- **JIT compiler**: a Just-In-Time compiler that is able to translate bytecode into native machine code.
- **Garbage collector**: automatic memory reclamation.
- **Memory manager**: performs stack manipulation and heap allocation.
- **Exception handling**: the ability to handle exceptions without unexpected behaviour.
- **Threading support**: manage threads and synchronization.

We will focus on 4 main components of the .NET runtime:
1) <span style="color:rgb(0, 112, 192)"><i>Assembly and type loading</i></span>: how the runtime locates the code and the types the application needs.
2) <span style="color:rgb(0, 112, 192)"><i>JIT and execution</i></span>: how the IL become executable machine code.
3) <span style="color:rgb(0, 112, 192)"><i>Types, exception and threading</i></span>: we consider some of the rules that govern the runtime by looking at the runtime type system, and how it handles exceptions and also threading.
4) <span style="color:rgb(0, 112, 192)"><i>Garbage collection</i></span>: how .NET manages memory.

### Assembly and type loading

As we already know, an assembly is a compiled unit containing both code and metadata. An application may depend on several assemblies, but before the code they store can be used, we need to look at their metadata, in particular to the dependencies to other assemblies. 

One assemblies are available, the runtime needs to understand the types defined in them. This is done through the use of **Method Tables** and **EEClasses**.

The <span style="color:rgb(0, 176, 80)">Method Table</span> (MT) is a runtime data structure that describes a type. Every object has a pointer to the MT of its type, which is how constructs such as ```GetType()``` and ```as``` work. As the name suggests, the Method Table holds all methods of an objects; in particular, for each methods it holds a pointer to the compiled code of that method. When a type is a sub-type of another, it inherits its methods; in the MT, the first methods are the ones of the parent type (eventually overridden), and then its own methods. Keeping this structure allows for upcasts to be simple, I just go to the same slot, which will contain the pointer to the same method but of the parent type. For this reason, we can easily see why multiple inheritance can be a problem, and so why it is restricted in .NET.

The MT is usually accessed multiple times during execution, this is why it is usually kept in cache and so it is as small as possible. Instead, all the metadata relative to a .NET classes needed to describe its type is stored in <span style="color:rgb(0, 176, 80)">EEClasses</span>. For instance, filed layouts, security information, generic parameters, are all examples of metadata stored in EEClasses.

After loading assemblies, types are loaded by creating EEClasses and MTs. Through dynamic loading it is also possible to load types from binaries that did not exists at runtime, as long as they implement an already existing interface.

### JIT and execution

There are two main ways in which the Just-In-Time compilation can happen:
- **Orchestrated JIT** (JVM style): the runtime environment manages the program as it runs, it decides which parts need to be optimized further.
- **Passive callback** (.NET style): the runtime calls the JIT through an interface, and asks it to compile methods when needed. This clearly performs less optimizations, but it also avoids the overhead of orchestration.

.NET uses mainly passive callbacks; however, these two methods are not antithetical. When creating a Method Table, every method is initialized with a stub, which is then backpatched when the method is used.

> [!NOTE]
> **Backpatching** is the act of updating a pointer to make its target address known.

### Types, exceptions and threading

The aim of this section is to show that a runtime's job goes beyond just executing code, it also handles important things such as, for example, a common type system, exceptions and threads.

#### Types

C# and F# have a very different syntax; however, they must be able to map their constructs into the same CIL, which, for example, means that they have to target the same type system.

The runtime thus decides to treat types in two different ways:
- **Reference types** (`classes`): they are allocated on the heap and they are not copied, just their reference is.
- **Value types** (`structs`): they are allocated on the stack and they are deep copied.

Intuitively, value types are the types for basic types (integers, booleans, ...), while reference types are for more complex data structures. However, in the .NET runtime they both inherit from `object`, whereas in Java only reference types do.

It is possible to transform any value type into a reference type through boxing, here is an example:
```C#
int x = 42;
object o = x; // boxes
int y = (int)o; // unboxes
```
Boxing wraps a value type into an heap allocated object; this is useful because it allows for value types to implement interfaces, and also for reflection, i.e. interacting with its own MT (see more in [[Meta-Programming]]). However, boxing is generally not suggested, since it can easily waste space on the heap, and create overhead for the garbage collector.

Another interesting thing about the .NET runtime type system is about how it treats generics. In Java they are simply ignored once the compilation phase ends, every generic occurrence is substituted by their explicitly cast counterpart. Instead in the .NET runtime uses reified generics, so type parameters also exist at runtime. For example, let us consider the polymorphic type `Stack<'T>`; when instantiated as `Stack<Int>` and `Stack<String>`, after compilation Java will just use the runtime type `Stack`, while in .NET they will separately use the runtime types  `Stack<Int>` and `Stack<String>`. This means that, for example, it is possible to use reflection to see what was the original construction type for each one of them, while in Java this is not possible.

#### Exceptions

When an exception is thrown, the .NET runtime proceeds in 2 phases:
1) *The search pass*: The .NET runtime creates the `System.Exception` object and tries to find in the stack a `catch` block for that specific exception, if none is found, a fatal error occurs and the program crashes. 
2) *The unwind pass*: If the exception is caught, every `finally` block of the frames higher than the one that caught the exceptions are executed, while also being eliminated. Then, execution is transferred to the frame that caught the exception and returns to normal.

#### Threading

The .NET runtime manages threads by wrapping OS threads into runtime objects, creating CLR threads. This makes it possible for the thread to be represented as an object, which means that it is typed, and therefore that it is accessible via reflection, with fields and methods like any other class. Another advantage is that it allows for cooperation with the garbage collector; in fact, it allows the JIT to put safe-points into the thread execution, where it will stop executing to let the garbage collector inspect their heap when it requires them to.

Another interesting aspect of threading in the .NET runtime, is that every object, potentially, has a sync block, which means that it can be used as a mutex. This is exactly what happens when `lock(obj)` is used in C#, allowing for lightweight synchronization without having to allocate a separate `Mutex` or `Semaphore` object.

### Garbage collector

The Garbage Collector (GC) is needed for objects allocated on the heap, so that the user does not have to remove them manually. This guarantees that no memory leaks are possible, since every object is deleted in the end; however, this takes power away from the programmer which is not allowed anymore to decide when objects get deleted, and also the GC computations cause overhead. An interesting fact about the allocator for the .NET runtime is that it performs memory zeroing, i.e. the portion of memory returned by the allocator has been cleared, effectively guaranteeing that no data leakage from dead objects is possible.

The .NET runtime divides objects into two kinds of objects: *small objects* (less than 85,000 bytes) and *big objects* (more than 85,000 bytes). The garbage collector divides small objects into three generations based on how often they are used:
- Gen 0: Newly generated objects.
- Gen 1: Objects that survived Gen 0 collection. It is kind of a buffer zone between Gen 1 and 2.
- Gen 2: Long-lived objects, i.e. infrequently collected objects.

Big objects are treated permanently as Gen 2 small objects, meaning that, heuristically, small objects die young, while big objects live longer. 

The GC of the .NET runtime uses similar algorithms to that explained in [[Environment and memory management]], for example *mark and sweep*. In particular, the .NET garbage collector marks reachable objects by traversing the object graph, then it simulate a compaction to see how the resulting compacted heap looks like to decides whether the compaction is effectively worth performing. If it is, then it is performed and backpatches all the references, otherwise it sweeps dead objects by indicating them as free and their portion of memory as re-usable (so it does not compact it).

One way the GC scan time is reduced is by using a write barrier, which, whenever a reference type object is used, sets a bit in the card table to record that a reference in that region has been modified.  A card table is a bitmask where each bit represents a region of the heap. This allows the GC to only check those regions that have been effectively modified; however, this comes at a cost of executing more instructions, i.e. those for setting the bit in the card table.

As we mentioned earlier in the Threading Section, the GC collaborates with threads. It mainly works using a "Stop-the-world" mechanism, where `EESuspendAll` and the GC waits for every thread to reach a safe point; then, every object is examined with exclusive access by the GC and, after finishing its work, `EERestartAll` is called to resume execution.

#### References

- The .NET platform - <a href=https://learn.microsoft.com/en-us/dotnet/core/introduction>Microsoft Learn</a> (Introduction)
- .NET languages - <a href=https://learn.microsoft.com/en-us/dotnet/fundamentals/languages>Microsoft Learn</a> (Application level)
- .NET class libraries - <a href=https://learn.microsoft.com/en-us/dotnet/standard/class-libraries>Microsoft Learn</a> (Application level)
- The .NET SDK - <a href=https://learn.microsoft.com/en-us/dotnet/core/sdk>Microsoft Learn</a> (Build level)
- .NET assemblies - <a href=https://learn.microsoft.com/en-us/dotnet/standard/assembly>Microsoft Learn</a> (Artifact level)

All references are to Microsoft Learn; however, I would highly suggest to find other sources because Microsoft provides really bad documentation with incomprehensible jargon and infinite links each pointing at each other, ultimately not explaining anything. 