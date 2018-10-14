# ITEM 1: CONSIDER STATIC FACTORY METHODS INSTEAD OF CONSTRUCTORS
  Reasons:
    * unlike constructors, they have names
    * they are not required to create a new object each time they’re invoked
    * they can return an object of any subtype of their return type
    * class of the returned object can vary from call to call as a function of the input parameters
    * class of the returned object need not exist when the class containing the method is written
    * main limitation of providing only static factory methods is that classes without public or protected constructors cannot be subclassed
    * second shortcoming of static factory methods is that they are hard for programmers to find

# ITEM 2: CONSIDER A BUILDER WHEN FACED WITH MANY CONSTRUCTOR PARAMETERS
  Reasons:
    * readability
    * cleaner api
    * also applicable to class hierarchy

# ITEM 3: ENFORCE THE SINGLETON PROPERTY WITH A PRIVATE CONSTRUCTOR OR AN ENUM TYPE
  Reasons:
    * Making a class a singleton can make it difficult to test its clients

# ITEM 4: ENFORCE NONINSTANTIABILITY WITH A PRIVATE CONSTRUCTOR 
    * If you want to group static methods / fields, eg. Math, Arrays; or to group factories, eg Collections
    * Side effect is this idion prevents class from being subclassed

# ITEM 5: PREFER DEPENDENCY INJECTION TO HARDWIRING RESOURCES
    * Enhance flexibility, reuseability and testability

# ITEM 6: AVOID CREATING UNNECESSARY OBJECTS
    * Can impact performance and clarity of your code
    * conter point to this item is defensive object copying
    * eg. new String("abc"), auto-boxing, keySet on a map object

# ITEM 7: ELIMINATE OBSOLETE OBJECT REFERENCES
    * Whenever a class manages it's own memory then there is a risk of memory leak
    * Non type checked objects (global) that are never released can cause memory leaks
    * Another example is cache. If the value dies as soon as key dies then make it a weak hash map
    * Third source of memory leaks is listeners and other callbacks

# ITEM 8: AVOID FINALIZERS AND CLEANERS
    * As per the spec there is no guarantee these will be executed promptly, (it's a function of GC Algo) if at all.
    * Never do anything time critical in these blocks
    * Uncaught exception during finalization is ignored which corrupts state
    * Finalizers have serious security problem. Finalizer attacks
    * Solution: Implement Auto-Closable and invoke close when no longer needed. Try with resources.
    * Finalizers and cleaner can be used as safety net but think long and hard before pursuing this some underlying lib might already have done that.

# ITEM 9: PREFER TRY-WITH-RESOURCES TO TRY-FINALLY
    * Finally would shadow the original exception, which makes debugging hard
    * try with resources keeps the original exception and supresses the later one. supressed exception is still printed and can be accessed programatically
    * implement auto closable interface if creating a class that works with resources

# ITEM 10: OBEY THE GENERAL CONTRACT WHEN OVERRIDING EQUALS
    * Properties: Idempotent (Reflexive), Commutative (Symmetric), Transitive, Consistent, Non-null compares with null
    * There is no way to extend an instantiable class and add a value component while preserving the equals contract
    * getClass violates Liskov substitution principle
    * Beware of some classes in Java (eg timestamp and date since equals impl for timestamp voilates symmetry)
    * do not write an equals method that depends on unreliable resources, this will cause Consistency violation (in Java platform URL relies on IP address so it causes a lot of problems)
    * Ideal equals method: reference check with == (idempotentcy), instanceof op to check non-instanciable type (non-nullity and symmetry), cast to correct type, check each "significant" field matches
    * primitives, except float and double, can be compared by ==. Float.compare(float, float) and Double.compare(double, double) (do not use equals for Float and Double, due to boxing). Array.equals() for array comparisions.
    * Focus on short circuit technique, for better equals performace

# ITEM 11: ALWAYS OVERRIDE HASHCODE WHEN YOU OVERRIDE EQUALS
    * Contract: Same hashcode if nothing changes, equals true for two objects must has same hashcode and not neccessary to have different hascode for distinct obects (although having it different would improve performance)
    * Look at book for recipie to achieve this
    * Objects.hash() is a good quality hash function but slower due to boxing and array creation for var args
    * Its not a bad idea to cache hashcode for immutable classes (on createtion or lazily, careful when lazily doing it for thread safety)

# ITEM 12: ALWAYS OVERRIDE TOSTRING
    * Always helpful for debugging and visualizing details via assert, string concat op, print, etc
    * Recommended to provide format of returned string for values classes
    * provide programmatic access to the information contained in the value returned by toString

# ITEM 13: OVERRIDE CLONE JUDICIOUSLY
    * class should first call super.clone then fix fields the needs fixing
    * any mutable objects should be deep copied and replaced in the clone's reference
    * if class only contains primitive fields or references to immutable objects then no fixing would be needed (exception: fields representing serial number or unique id would still need to be fixed)
    * copy construction or copy factory would be more desirable than dealing with complexity of cloning
      * no demand for unenforceable adherence to thinly documented conventions
      * no conflict with the proper use of final fields
      * no unnecessary checked exceptions
      * no casts required
      * can take an argument whose type is an interface implemented by the class, eg constructing a treeset from hashset (new TreeSet(someHashSet)), clone cannot do that
    * new implimentations should avoid clonable, its best to use copy constructor/factory; only arrays are best copied with the clone method

# ITEM 14: CONSIDER IMPLEMENTING COMPARABLE
    * enables interoperate with all of the many generic algo and collection impl
    * general contract: x.compareTo(y) must throw an exception iff y.compareTo(x) does, transitivity, strongly recommended to have x.compareTo(y)==0 == x.equals(y) relation hold (since hashset, hashmap, collection, etc will work same as treeset, treemap, etc)
    * java8 fluent comparator can be slower (ref for book was 10% slower)
    * whenever using compare or compareTo methods, avoid using < or > operators since there is a possibility of interger overflow. Always use static compare methods or comparator construction methods in Comparator interface

# ITEM 15: MINIMIZE THE ACCESSIBILITY OF CLASSES AND MEMBERS
    * reduce accessibility of program elements as much as possible (within reason)
    * Ensure that objects referenced by public static final fields are immutable
    * Instance fields of public classes should rarely be public. such public fields are generally not thread safe
    * never have a public static final array field, or an accessor that returns such a field, this causes security holes
      * alternative1 make the public array private and add a public immutable list
      * make the array private and add a public method that returns a copy of a private array
    * best to avoid modules unless there is a compelling need, since it adds 2 more access level so we'd need to rearrange classes and packages. They are mostly advisory in nature for a typical java programmer

# ITEM 16: IN PUBLIC CLASSES, USE ACCESSOR METHODS, NOT PUBLIC FIELDS
    * public classes shouldn't expose mutable fields. It is less harmful, though still questionable, for public classes to expose immutable fields.
    * its okay for package-private or private nested classes to expose fields, whether mutable or immutable

# ITEM 17: MINIMIZE MUTABILITY
    * Don’t provide methods that modify the object’s state
    * Ensure that the class can’t be extended. Make all fields final. Make all fields private. Ensure exclusive access to any mutable components.
    * Immutable objects are inherently thread-safe; they require no synchronization.
    * Immutable objects can share their internals; are great building blocks for other objects; provide atomicity for free.
    * The major disadvantage of immutable classes is that they require a separate object for each distinct value; but companion class can help (public example StringBuilder / StringBuffer for String, similarly BigInteger has a package private companion class)
    * Constructors should create fully initialized objects with all of their invariants established

# ITEM 18: FAVOR COMPOSITION OVER INHERITANCE
    * Inheritance relies on super class and it's underlying implementation which the super class is free to change in subsequent releases (in a way it defies encapsulation)
    * Create wrapper classes that use decorator patter which composes over a contract / interface and forwards / delegates / proxies the class through to the underlying composed object.
    * One caveat is that wrapper classes are not suited for use in callback frameworks, wherein objects pass self-references to other objects for subsequent invocations (“callbacks”).

# ITEM 19: DESIGN AND DOCUMENT FOR INHERITANCE OR ELSE PROHIBIT IT
    * the class must document its self-use of overridable methods. This unfortunately violates encapsulation principle by exposing the internals but thats the nature of inheritance
    * The only way to test a class designed for inheritance is to write subclasses
    * Constructors along with clonable and serializable methods must not invoke overridable methods
    * eliminate the class’s self-use of overridable methods entirely

# ITEM 20: PREFER INTERFACES TO ABSTRACT CLASSES
    * Existing classes can easily be retrofitted to implement a new interface
    * Interfaces are ideal for defining mixins
    * Interfaces allow for the construction of nonhierarchical type frameworks
    * consider providing default impl for interface method
    * restrictions on interfaces typically mandate that a skeletal implementation take the form of an abstract class (interfaces cannot provide default impl for Object class methods).

# ITEM 21: DESIGN INTERFACES FOR POSTERITY
    *  it is not always possible to write a default method that maintains all invariants of every conceivable implementation.
    * In the presence of default methods, existing implementations of an interface may compile without error or warning but fail at runtime.

# ITEM 22: USE INTERFACES ONLY TO DEFINE TYPES
    * interfaces should be used only to define types. They should not be used merely to export constants. Note: use underscore in digits to make constants readable

# ITEM 23: PREFER CLASS HIERARCHIES TO TAGGED CLASSES
    * tagged classes are verbose, error-prone, and inefficient

# ITEM 24: FAVOR STATIC MEMBER CLASSES OVER NONSTATIC
    * If each instance of a member class needs a reference to its enclosing instance, make it nonstatic; otherwise, make it static

# ITEM 25: LIMIT SOURCE FILES TO A SINGLE TOP-LEVEL CLASS
    * Never put multiple top-level classes or interfaces in a single source file since behavior of the program can be affected by the order in which the source files are passed to the compiler.

# ITEM 26: DON’T USE RAW TYPES
    * If you use raw types, you lose all the safety and expressiveness benefits of generics. raw types are generic types without type parameter eg. List instead of List<E>
    * using raw types can lead to exceptions at runtime, so don’t use them
    * unbounded wildcard type, eg for the generic type Set<E> is Set<?> (read “set of some type”), is the most general parameterized Set type, capable of holding any set. you can’t put any element (other than null) into a Collection<?>, similarly you cannot assume anything about the type of the objects that you get out.
    * You must use raw types in class literals, eg List.class, String[].class, and int.class are all legal, but List<String>.class and List<?>.class are not
    * because of type erasure it is illegal to use the instanceof operator on parameterized types other than unbounded wildcard types
    * Set<Object> is a parameterized type representing a set that can contain objects of any type, Set<?> is a wildcard type representing a set that can contain only objects of some unknown type, and Set is a raw type, which opts out of the generic type system

# ITEM 27: ELIMINATE UNCHECKED WARNINGS
    * Only use SuppressWarnings annotation when you're sure that the code is type safe. Always use the SuppressWarnings annotation on the smallest scope possible
    * Every time you use a @SuppressWarnings("unchecked") annotation, add a comment saying why it is safe to do so

# ITEM 28: PREFER LISTS TO ARRAYS
    * arrays are covariant, which means simply that if Sub is a subtype of Super, then the array type Sub[] is a subtype of the array type Super[]. Generics, by contrast, are invariant: for any two distinct types Type1 and Type2, List<Type1> is neither a subtype nor a supertype of List<Type2>
    * arrays are reified, which means that arrays know and enforce their element type at runtime. Generics, by contrast, are implemented by erasure
    * As a consequence, arrays provide runtime type safety but not compile-time type safety, and vice versa for generics
    * As a rule, arrays and generics don’t mix well
    * If you find yourself mixing them and getting compile-time errors or warnings, your first impulse should be to replace the arrays with lists

# ITEM 29: FAVOR GENERIC TYPES
    * generic types are safer and easier to use than types that require casts in client code

# ITEM 30: FAVOR GENERIC METHODS
    * as above
    * see examples for: generic singleton factory and recursive type bound

# ITEM 31: USE BOUNDED WILDCARDS TO INCREASE API FLEXIBILITY
    * For maximum flexibility, use wildcard types on input parameters that represent producers or consumers. Mnemonic: PECS stands for producer-extends, consumer-super. Also called Get and Put Principle.
    * Do not use bounded wildcard types as return types. Rather than providing additional flexibility for your users, it would force them to use wildcard types in client code
    * if a type parameter appears only once in a method declaration, replace it with a wildcard. If it’s an unbounded type parameter, replace it with an unbounded wildcard; if it’s a bounded type parameter, replace it with a bounded wildcard.
    * all comparables and comparators are consumers

# ITEM 32: COMBINE GENERICS AND VARARGS JUDICIOUSLY
    * the SafeVarargs annotation constitutes a promise by the author of a method that it is typesafe
    * if the varargs parameter array is used only to transmit a variable number of arguments from the caller to the method then the method is safe (i.e. never store anything into varargs array or allow a reference to the array to escape)
    * SafeVarargs annotation is legal only on methods that can’t be overridden (Java 8: static methods and final instance methods, Java 9: private instance methods as well)

# ITEM 33: CONSIDER TYPESAFE HETEROGENEOUS CONTAINERS
    * the normal use of generics, exemplified by the collections APIs, restricts you to a fixed number of type parameters per container. You can get around this restriction by placing the type parameter on the key rather than the container. You can use Class objects as keys for such typesafe heterogeneous containers. A Class object used in this fashion is called a type token.
    * Ref book for details and examples.

# ITEM 34: USE ENUMS INSTEAD OF INT CONSTANTS
    * Programs that use int enums are brittle. Because int enums are constant variables, their int values are compiled into the clients that use them
    * Enums are classes that export one instance for each enumeration constant via a public static final field. They are effectively final, by virtue of having no accessible constructors. They provide compile time safety.
    * To associate data with enum constants, declare instance fields and write a constructor that takes the data and stores it in the fields. Enums are by their nature immutable, so all fields should be final.
    * to associate a different behavior with each enum constant: declare an abstract method in the enum type, and override it with a concrete method for each constant in a constant-specific class body. Such methods are known as constant-specific method implementations. These can also be combined with conspecific data, which essentially means passing it via constructor and storing in private final field.
    * Enum types have an automatically generated valueOf(String) method that translates a constant’s name into the constant itself. If you override the toString method in an enum type, consider writing a fromString method to translate the custom string representation back to the corresponding enum (see book for eg)
    * disadvantage of constant-specific method implementations is that they make it harder to share code among enum constants, see book for eg (overtime pay eg)
    *  Switches on enums are good for augmenting enum types with constant-specific behavior (eg from book: if inverse api was not available on operation)

# ITEM 35: USE INSTANCE FIELDS INSTEAD OF ORDINALS
    * Never derive a value associated with an enum from its ordinal; store it in an instance field instead

# ITEM 36: USE ENUMSET INSTEAD OF BIT FIELDS
    *  just because an enumerated type will be used in sets, there is no reason to represent it with bit fields

# ITEM 37: USE ENUMMAP INSTEAD OF ORDINAL INDEXING
    * when you access an array that is indexed by an enum’s ordinal, it is your responsibility to use the correct int value (no typesafety); If you use the wrong value, the program will silently do the wrong thing or throw an ArrayIndexOutOfBoundsException
    * EnumMap provides the same performance as ordinal indexing but it also provides typesafety
    * It is rarely appropriate to use ordinals to index into arrays: use EnumMap instead. If the relationship you are representing is multidimensional, use EnumMap<..., EnumMap<...>> (ref book for eg)

# ITEM 38: EMULATE EXTENSIBLE ENUMS WITH INTERFACES
    * while you cannot write an extensible enum type, you can emulate it by writing an interface to accompany a basic enum type that implements the interface
    * ref to book for 2 examples. Essentially if the client code passes class then it can use to get values of the enum but if they pass Collection<Interface> then all the enums, base and extented, can be provided; in such a case you forgo the use of EnumSet and EnumMap

# ITEM 39: PREFER ANNOTATIONS TO NAMING PATTERNS
    * There is simply no reason to use naming patterns when you can use annotations instead. See book for examples of annotation.
    * Its useful to know how annotations work but most of the time you shouldn't need to write a new one (unless of course you're writing a framework)

# ITEM 40: CONSISTENTLY USE THE OVERRIDE ANNOTATION
    * use the Override annotation on every method declaration that you believe to override a superclass declaration

# ITEM 41: USE MARKER INTERFACES TO DEFINE TYPES
    * marker annotations DO NOT make marker interfaces obsolete; marker interfaces define a type that is implemented by instances of the marked class; marker annotations do not
    * Another advantage of marker interfaces over marker annotations is that they can be targeted more precisely
    * The chief advantage of marker annotations over marker interfaces is that they are part of the larger annotation facility. Therefore, marker annotations allow for consistency in annotation-based frameworks.
    * If you find yourself writing a marker annotation type whose target is ElementType.TYPE, take the time to figure out whether it really should be an annotation type or whether a marker interface would be more appropriate

# ITEM 42: PREFER LAMBDAS TO ANONYMOUS CLASSES
    * lambdas lack names and documentation; if a computation isn’t self-explanatory, or exceeds a few lines, don’t put it in a lambda. One line is ideal for a lambda, and three lines is a reasonable maximum.
    * you should rarely, if ever, serialize a lambda and anonymous class
    * Don’t use anonymous classes for function objects unless you have to create instances of types that aren’t functional interfaces

# ITEM 43: PREFER METHOD REFERENCES TO LAMBDAS
    * Where method references are shorter and clearer, use them; where they aren’t, stick with lambdas

# ITEM 44: FAVOR THE USE OF STANDARD FUNCTIONAL INTERFACES
    * If one of the standard functional interfaces does the job, you should generally use it in preference to a purpose-built functional interface; since the API will be simpler and will provide significant interoperability benefits, as many of the standard functional interfaces provide useful default methods
    * 6 base functional interfaces: Operator (Unary and Binary), Function, Predicate, Consumer, Supplier. Rest are derived from these. All Functional Interfaces work on Object Reference (eg: String::toLowerCase)
    * 3 variants for each of above 6, for primitives (int, long, double). Only Function functional interface is parameterized by return type (eg: LongFunction<int[]> which return int[]), others are very straight forward.
    * 9 variants of Function interface where result type is primitive. If both the source and result types are primitive, prefix Function with SrcToResult (eg: LongToIntFunction (6 variants)); If the source is a primitive and the result is an object reference, prefix Function with <Src>ToObj (eg: DoubleToObjFunction (3 variants))
    * There are two-argument versions of the three basic functional interfaces for which it makes sense to have them: BiPredicate<T,U>, BiFunction<T,U,R>, and BiConsumer<T,U>. BiFunction variants returning the three relevant primitive types (eg: ToIntBiFunction<T,U>); two-argument variants of Consumer that take one object reference and one primitive type (eg: ObjDoubleConsumer<T>).
    * BooleanSupplier interface, a variant of Supplier that returns boolean values.
    * Don’t be tempted to use basic functional interfaces with boxed primitives instead of primitive functional interfaces, due to performance reasons
    * Try to use the provided functional interfaces, but there are times when you have to write your own functionalo interface. Consider the following requirements when writing your own: It will be commonly used and could benefit from a descriptive name, It has a strong contract associated with it, It would benefit from custom default methods.
    * Always annotate your functional interfaces with the @FunctionalInterface annotation.
    * APIs overloaded by functional interface argument in the same position will get ambiguious, so mindful of that and avoid such a practice (eg: submit method of ExecutorService it accepts Callable<T> or a Runnable<T>)

# ITEM 45: USE STREAMS JUDICIOUSLY
    * Overusing streams makes programs hard to read and maintain
    * Using helper methods is important for readability in stream pipelines
    * refrain from using streams to process char values
    * stream limitations: read final / effectively final variables in scope (closure) and cannot modify local variables, due to callback nature control flow statems don't work eg break, continue, return, throw checked exceptions
    * stream are great for: map, filter, reduce, collect, search
    * If you’re not sure whether a task is better served by streams or iteration, try both and see which works better

# ITEM 46: PREFER SIDE-EFFECT-FREE FUNCTIONS IN STREAMS
    * The forEach operation should be used only to report the result of a stream computation, not to perform the computation
    * Collectors most common: toList, toSet, toCollection. Map collectors: if no collision toMap(keyMapper, valMapper), on collision toMap(keyMapper, valMapper, mergeFn) mergeFn egs: last wrt wins or aggregator like multiply or max, on collision with prefered map toMap(keyMapper, valMapper, mergeFn, mapFactory) mapFactory egs: TreeMap, EnumMap etc.
    * Collectors grouping: groupingBy(classifierFn) -> Map<class, List<E>>, groupingBy(classifierFn, collector) -> Map<class, ?> eg collector: toSet or toCollection(colFactory) or counting(), groupingBy(classifierFn, mapFactory, collector). partitionBy is similar to groupingBy just that it takes a predicate instead of classifierFn.
    * collectors returned by the counting method are intended only for use as downstream collectors; there is never a reason to say collect(counting()). similarly for summing, averaging, and summarizing.
    * finally Collection joining works on charSeq and has 2 overloaded forms. as the name suggests it joins charSeq (aka strings) with a delimeter, the other form has prefix and suffix for the stream and prints as any collection toString()

# ITEM 47: PREFER COLLECTION TO STREAM AS A RETURN TYPE
    * Collection or an appropriate subtype is generally the best return type for a public, sequence-returning method
    * If the contents of the sequence aren’t predetermined before iteration takes place, return a stream or iterable, whichever feels more natural. If you choose, you can return both using two separate methods.
    * There is no interoperablitiy between streams and iterables so if you return one and client wants to use the other then the client will need to write adapters. Also note that these adapter are 2.3 times slower (stream to iterable from book) where as a purpose-built collection is about 1.4 times as fast as our stream-based implementation.

# ITEM 48: USE CAUTION WHEN MAKING STREAMS PARALLEL
    * Do not parallelize stream pipelines indiscriminately
    * parallelizing a pipeline is unlikely to increase its performance if the source is from Stream.iterate, or the intermediate operation limit is used.
    * performance gains from parallelism are best on streams over ArrayList, HashMap, HashSet, and ConcurrentHashMap instances; arrays; int ranges; and long ranges. Locality-of-reference turns out to be critically important for parallelizing bulk operations: without it, threads spend much of their time idle, waiting for data to be transferred from memory into the processor’s cache
    * The best terminal operations for parallelism are reductions, where all of the elements emerging from the pipeline are combined using one of Stream’s reduce methods, or prepackaged reductions such as min, max, count, and sum, or short-circuiting operations anyMatch, allMatch, and noneMatch. Stream’s collect method, which are known as mutable reductions are not good candidates.
    * Not only can parallelizing a stream lead to poor performance, including liveness failures; it can lead to incorrect results and unpredictable behavior
    * A proper usage might still not offset the costs associated with parallelism. As a very rough estimate, the number of elements in the stream times the number of lines of code executed per element should be at least a hundred thousand.
    * all parallel stream pipelines in a program run in a common fork-join pool. A single misbehaving pipeline can harm the performance of others in unrelated parts of the system.
    *  it is possible to achieve near-linear speedup in the number of processor cores simply by adding a parallel call to a stream pipeline. Certain domains, such as machine learning and data processing, are particularly amenable to these speedups.

# ITEM 49: CHECK PARAMETERS FOR VALIDITY
    * Frist few lines of any method should be checking validity of parameters to fail fast and closer to the error, failure atomicity and consistent state of a class/program 
    * Utilities such as Objects.requireNonNull, checkFromIndexSize, checkFromToIndex, and checkIndex etc were added to aid in some boilerplate checks
    * public and protected should document the exception / error cases with @Throws annotation in javadoc. you can use asserts for private method but they are only applicable if the program starts with -ea switch
    * Checkout book for examples. Only exception to not doing a validity check is if it's really expensive or the process of operation does the implied validity check.

# ITEM 50: MAKE DEFENSIVE COPIES WHEN NEEDED
    * if a class has mutable components that it gets from or returns to its clients, the class must defensively copy these components
    * If the cost of the copy would be prohibitive and the class trusts its clients not to modify the components inappropriately, then the defensive copy may be replaced by documentation outlining the client’s responsibility not to modify the affected components
    * Refer examples from book

# ITEM 51: DESIGN METHOD SIGNATURES CAREFULLY
    * Choose method names carefully. names that are understandable and consistent with other names in the same package. look to the Java library APIs for guidance.
    * Don’t go overboard in providing convenience methods. When in doubt, leave it out.
    * Avoid long parameter lists. Several ways of achieving this: break into multiple methods (eg List where instead of find first or last they opted for subList, indexOf, lastIndexOf methods which are more powerful and flexible), helper classes to hold groups of parameters, Builder pattern
    * For parameter types, favor interfaces over classes. By using a class instead of an interface, you restrict your client to a particular implementation and force an unnecessary and potentially expensive copy operation if the input data happens to exist in some other form.
    * Prefer two-element enum types to boolean parameters. Enums provide greater flexibility and readability.
