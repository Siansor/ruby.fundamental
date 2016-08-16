# Fundamental programming with ruby examples

This is repo keeps examples with description modern principles, patterns.

## Solid

In computer programming, SOLID (single responsibility, open-closed, Liskov substitution, interface segregation and dependency inversion) is a mnemonic acronym introduced by Michael Feathers for the "first five principles" named by Robert C. Martin in the early 2000s that stands for five basic principles of object-oriented programming and design. The intention is that these principles, when applied together, will make it more likely that a programmer will create a system that is easy to maintain and extend over time. The principles of SOLID are guidelines that can be applied while working on software to remove code smells by causing the programmer to refactor the software's source code until it is both legible and extensible. It is part of an overall strategy of agile and Adaptive Software Development.

####SRP - Single responsibility principle
A class should have only a single responsibility.
 
Every class should have a single responsibility, and that responsibility should be entirely encapsulated.
All its services should be narrowly aligned with that responsibility, this embrace the high cohesion.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/solid/single_responsibility.rb)
     
####OCP - Open/closed principle
Software entities should be open for extension, but closed for modification.
That is, such an entity can allow its behaviour to be extended without modifying its source code.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/solid/open_close.rb)
 
####LSP - Liskov substitution principle
Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.

It states that, in a computer program, if S is a subtype of T, then objects of type T may be replaced with objects of type S (i.e., objects of type S may substitute objects of type T) without altering any of the desirable properties of that program (correctness, task performed, etc.)

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/solid/liskov_substitution.rb)

####ISP - Interface segregation principle
Many client-specific interfaces are better than one general-purpose interface.

States that no client should be forced to depend on methods it does not use.
ISP splits interfaces which are very large into smaller and more specific ones so that clients will only have to know about the methods that are of interest to them. Such shrunken interfaces are also called role interfaces. 
ISP is intended to keep a system decoupled and thus easier to refactor, change, and redeploy.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/solid/interface_segregation.rb)

####DIP - Dependency inversion principle
One should Depend upon Abstractions, Do not depend upon concretions.

Refers to a specific form of decoupling software modules. When following this principle, the conventional dependency relationships established from high-level, policy-setting modules to low-level, dependency modules are inverted (i.e. reversed), thus rendering high-level modules independent of the low-level module implementation details. 

####Code and articles were taken from resources:

* [https://gist.github.com/khusnetdinov/9d8f50fdcaab197871b31578f2e14d5d] (https://gist.github.com/khusnetdinov/9d8f50fdcaab197871b31578f2e14d5d) 

* [https://robots.thoughtbot.com/back-to-basics-solid] (https://robots.thoughtbot.com/back-to-basics-solid) 

* [https://subvisual.co/blog/posts/19-solid-principles-in-ruby] (https://subvisual.co/blog/posts/19-solid-principles-in-ruby) 

* [http://blog.siyelo.com/solid-principles-in-ruby/] (http://blog.siyelo.com/solid-principles-in-ruby/)

## Threads

Note about parallelism and concurrency: The primary difference between using processes versus threads is the way that memory is handled. At a high level, processes copy memory, while threads share memory. This makes process spawning slower than thread spawning, and leads to processes consuming more resources once running. Overall, threads incur less overhead than processes. This Thread API is a Ruby API. I've hinted that the different Ruby implementations have different underlying threading behaviours.

#### Green Thread

Ruby 1.9 replaced green threads with native threads. However, the GIL is still preventing parallelism. That being said, concurrency has been improved through better scheduling. The new scheduler makes context-switch decisions more efficient, by essentially moving them to a separate native thread, known as the timer thread.

#### GIL - Global Interpreter Lock

MRI has a global interpreter lock (GIL). It's a lock around the execution of Ruby code. This means that in a multi-threaded context, only one thread can execute Ruby code at any one time.

So if you have 8 threads busily working on a 8-core machine, only one thread and one core will be busy at any given time. The GIL exists to protect Ruby internals from race conditions that could corrupt data. There are caveats and optimizations, but this is the gist.

This has some very important implications for MRI. The biggest implication is that Ruby code will never run in parallel on MRI. The GIL prevents it.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/threads/gil.rb)

#### Mutex - Mutual Execution

Mutexes provide a mechanism for multiple threads to synchronize access to a critical portion of code. In other words, they help bring some order, and some guarantees, to the world of multi-threaded chaos.

The name 'mutex' is shorthand for 'mutual exclusion.' If you wrap some section of your code with a mutex, you guarantee that no two threads can enter that section at the same time.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/threads/mutex.rb)

#### Fibers

Fibers are primitives for implementing light weight cooperative concurrency in Ruby. Basically they are a means of creating code blocks that can be paused and resumed, much like threads. The main difference is that they are never preempted and that the scheduling must be done by the programmer and not the VM. As opposed to other stackless light weight concurrency models, each fiber comes with a small 4KB stack. This enables the fiber to be paused from deeply nested function calls within the fiber block.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/threads/fibers.rb)

#### Rails - Thread-safety in the framework

The problem with this is that there is no simple way to say with absolute certainty whether an app as a whole is thread-safe.

* Global variables are global. This means that they are shared between threads. If you weren’t convinced about not using global variables by now, here’s another reason to never touch them. If you really want to share something globally across an app, you are more than likely better served by a constant (but see below), anyway.

* Class variables. For the purpose of a discussion about threads, class variables are not much different from global variables. They are shared across threads just the same way.The problem isn’t so much about using class variables, but about mutating them. And if you are not going to mutate a class variable, in many cases a constant is again a better choice.

* Class instance variables. But maybe you’ve read that you should always use class instance variables instead of class variables in Ruby. Well, maybe you should, but they are just as problematic for threaded programs as class variables.

* Memoization by itself is not a thread safety issue. However, it can easily become one for a couple of reasons:

  - It is often used to store data in class variables or class instance variables (see the previous points).
  - The ||= operator is in fact two operations, so there is a potential context switch happening in the middle of it, causing a race condition between threads.
  - It would be easy to dismiss memoization as the cause of the problem, and tell people just to avoid class variables and class instance variables. However, the issue is more complex than that.
  - In this issue, Evan Phoenix squashes a really tricky race condition bug in the Rails codebase caused by calling super in a memoization function. So even though you would only be using instance variables, you might end up with race conditions with memoization.

  Make sure memoization makes sense and a difference in your case. In many cases Rails actually caches the result anyway, so that you are not saving a whole lot if any resources with your memoization method.
  Don’t memoize to class variables or class instance variables. If you need to memoize something on the class level, use thread local variables (Thread.current[:baz]) instead. Be aware, though, that it is still kind of a global variable. So while it’s thread-safe, it still might not be good coding practice.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/threads/rails.rb)

#### Code and articles were taken from resources:

* [http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil] (http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil)

* [https://en.wikipedia.org/wiki/Global_interpreter_lock] (https://en.wikipedia.org/wiki/Global_interpreter_lock)

* [http://www.csinaction.com/2014/10/10/multithreading-in-the-mri-ruby-interpreter/] (http://www.csinaction.com/2014/10/10/multithreading-in-the-mri-ruby-interpreter/)

## Design patterns

### Creational pattern

In software engineering, creational design patterns are design patterns that deal with object creation mechanisms, trying to create objects in a manner suitable to the situation. The basic form of object creation could result in design problems or in added complexity to the design. Creational design patterns solve this problem by somehow controlling this object creation. Creational design patterns are composed of two dominant ideas. One is encapsulating knowledge about which concrete classes the system uses. Another is hiding how instances of these concrete classes are created and combined.

[Read wiki] (https://en.wikipedia.org/wiki/Creational_pattern)

#### Abstract factory pattern

[See example] ()

#### Builder pattern

[See example] ()

#### Factory pattern

[See example] ()

#### Prototype pattern

[See example] ()

#### Singleton pattern

[See example] ()

#### Not covered patterns:

* [Lazy initialization] (https://en.wikipedia.org/wiki/Lazy_initialization)

* [Multiton] (https://en.wikipedia.org/wiki/Multiton_pattern)

* [Object pool] (https://en.wikipedia.org/wiki/Object_pool_pattern)

* [Resource acquisition is initialization] (https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization)

* [Utility] (https://en.wikipedia.org/wiki/Utility_pattern)

### Structural pattern

In software engineering, structural design patterns are design patterns that ease the design by identifying a simple way to realize relationships between entities.

[Read wiki] (https://en.wikipedia.org/wiki/Structural_pattern)

#### Adapter pattern

[See example] ()

#### Bridge pattern

[See example] ()

#### Composite pattern

[See example] ()

#### Decorator pattern

[See example] ()

#### Facade pattern

[See example] ()

#### Flyweight pattern

[See example] ()

#### Proxy pattern

[See example] ()

#### Not covered patterns:

* [Annotated Callback] (http://c2.com/cgi/wiki?AnnotatedCallback)

* [Data Bus] (http://c2.com/cgi/wiki?DataBusPattern)

* [Role Object] (http://c2.com/cgi/wiki?RoleObjectPattern)

### Behavioral pattern 

In software engineering, behavioral design patterns are design patterns that identify common communication patterns between objects and realize these patterns. By doing so, these patterns increase flexibility in carrying out this communication.

[Read wiki] (https://en.wikipedia.org/wiki/Behavioral_pattern)

#### Chain of responsibility pattern

[See example] ()

#### Command pattern

[See example] ()

#### Interpreter patter

[See example] ()

#### Iterator patter

[See example] ()

#### Mediator patter

[See example] ()

#### Momento pattern

[See example] ()

#### Observer pattern

[See example] ()

#### State pattern

[See example] ()

#### Strategy patter

[See example] ()

#### Template method pattern

[See example] ()

#### Visitor patter

[See example] ()

#### Not covered patterns:

* [Hierarchical visitor] (http://c2.com/cgi/wiki?HierarchicalVisitorPattern)

####Code and articles were taken from resources:

* [https://ru.wikipedia.org/wiki/Design_Patterns] (https://ru.wikipedia.org/wiki/Design_Patterns)

* [https://en.wikipedia.org/wiki/Creational_pattern] (https://en.wikipedia.org/wiki/Creational_pattern)

* [https://en.wikipedia.org/wiki/Structural_pattern] (https://en.wikipedia.org/wiki/Structural_pattern)

* [https://en.wikipedia.org/wiki/Behavioral_pattern] (https://en.wikipedia.org/wiki/Behavioral_pattern)

* [http://c2.com/cgi/wiki?CategoryPattern] (http://c2.com/cgi/wiki?CategoryPattern)


