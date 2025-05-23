[//]: # (title: Reflection)

_Reflection_ is a set of language and library features that allows you to introspect the structure of your program at runtime.
Functions and properties are first-class citizens in Kotlin, and the ability to introspect them (for example, learning the name or
the type of a property or function at runtime) is essential when using a functional or reactive style.

> Kotlin/JS provides limited support for reflection features. [Learn more about reflection in Kotlin/JS](js-reflection.md).
>
{style="note"}

## JVM dependency

On the JVM platform, the Kotlin compiler distribution includes the runtime component required for using the reflection features as a separate
artifact, `kotlin-reflect.jar`. This is done to reduce the required size of the runtime
library for applications that do not use reflection features.

To use reflection in a Gradle or Maven project, add the dependency on `kotlin-reflect`:

* In Gradle:

    <tabs group="build-script">
    <tab title="Kotlin" group-key="kotlin">

    ```kotlin
    dependencies {
        implementation(kotlin("reflect"))
    }
    ```

    </tab>
    <tab title="Groovy" group-key="groovy">
    
    ```groovy
    dependencies {
        implementation "org.jetbrains.kotlin:kotlin-reflect:%kotlinVersion%"
    }
    ```

    </tab>
    </tabs>

* In Maven:
    
    ```xml
    <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-reflect</artifactId>
        </dependency>
    </dependencies>
    ```

If you don't use Gradle or Maven, make sure you have `kotlin-reflect.jar` in the classpath of your project.
In other supported cases (IntelliJ IDEA projects that use the command-line compiler or Ant),
it is added by default. In the command-line compiler and Ant, you can use the `-no-reflect` compiler option to exclude
`kotlin-reflect.jar` from the classpath.

## Class references

The most basic reflection feature is getting the runtime reference to a Kotlin class. To obtain the reference to a
statically known Kotlin class, you can use the _class literal_ syntax:

```kotlin
val c = MyClass::class
```

The reference is a [KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html) type value.

>On JVM: a Kotlin class reference is not the same as a Java class reference. To obtain a Java class reference,
>use the `.java` property on a `KClass` instance.
>
{style="note"}

### Bound class references

You can get the reference to the class of a specific object with the same `::class` syntax by using the object as a receiver:

```kotlin
val widget: Widget = ...
assert(widget is GoodWidget) { "Bad widget: ${widget::class.qualifiedName}" }
```

You will obtain the reference to the exact class of an object, for example, `GoodWidget` or `BadWidget`,
regardless of the type of the receiver expression (`Widget`).

## Callable references

References to functions, properties, and constructors can
also be called or used as instances of [function types](lambdas.md#function-types).

The common supertype for all callable references is [`KCallable<out R>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-callable/index.html),
where `R` is the return value type. It is the property type for properties, and the constructed type for constructors.

### Function references

When you have a named function declared as below, you can call it directly (`isOdd(5)`):

```kotlin
fun isOdd(x: Int) = x % 2 != 0
```

Alternatively, you can use the function as a function type value, that is, pass it
to another function. To do so, use the `::` operator:

```kotlin
fun isOdd(x: Int) = x % 2 != 0

fun main() {
//sampleStart
    val numbers = listOf(1, 2, 3)
    println(numbers.filter(::isOdd))
//sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3"}

Here `::isOdd` is a value of function type `(Int) -> Boolean`.

Function references belong to one of the [`KFunction<out R>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-function/index.html)
subtypes, depending on the parameter count. For instance, `KFunction3<T1, T2, T3, R>`.

`::` can be used with overloaded functions when the expected type is known from the context.
For example:

```kotlin
fun main() {
//sampleStart
    fun isOdd(x: Int) = x % 2 != 0
    fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"
    
    val numbers = listOf(1, 2, 3)
    println(numbers.filter(::isOdd)) // refers to isOdd(x: Int)
//sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3"}

Alternatively, you can provide the necessary context by storing the method reference in a variable with an explicitly specified type:

```kotlin
val predicate: (String) -> Boolean = ::isOdd   // refers to isOdd(x: String)
```

If you need to use a member of a class or an extension function, it needs to be qualified: `String::toCharArray`.

Even if you initialize a variable with a reference to an extension function, the inferred function type will
have no receiver, but it will have an additional parameter accepting a receiver object. To have a function type
with a receiver instead, specify the type explicitly:

```kotlin
val isEmptyStringList: List<String>.() -> Boolean = List<String>::isEmpty
```

#### Example: function composition

Consider the following function:

```kotlin
fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C {
    return { x -> f(g(x)) }
}
```

It returns a composition of two functions passed to it: `compose(f, g) = f(g(*))`.
You can apply this function to callable references:

```kotlin
fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C {
    return { x -> f(g(x)) }
}

fun isOdd(x: Int) = x % 2 != 0

fun main() {
//sampleStart
    fun length(s: String) = s.length
    
    val oddLength = compose(::isOdd, ::length)
    val strings = listOf("a", "ab", "abc")
    
    println(strings.filter(oddLength))
//sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3"}

### Property references

To access properties as first-class objects in Kotlin, use the `::` operator:

```kotlin
val x = 1

fun main() {
    println(::x.get())
    println(::x.name) 
}
```

The expression `::x` evaluates to a `KProperty0<Int>` type property object. You can read its
value using `get()` or retrieve the property name using the `name` property. For more information, see
the [docs on the `KProperty` class](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-property/index.html).

For a mutable property such as `var y = 1`, `::y` returns a value with the [`KMutableProperty0<Int>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-mutable-property/index.html) type
which has a `set()` method:

```kotlin
var y = 1

fun main() {
    ::y.set(2)
    println(y)
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3"}

A property reference can be used where a function with a single generic parameter is expected:

```kotlin
fun main() {
//sampleStart
    val strs = listOf("a", "bc", "def")
    println(strs.map(String::length))
//sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3"}

To access a property that is a member of a class, qualify it as follows:

```kotlin
fun main() {
//sampleStart
    class A(val p: Int)
    val prop = A::p
    println(prop.get(A(1)))
//sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3"}

For an extension property:

```kotlin
val String.lastChar: Char
    get() = this[length - 1]

fun main() {
    println(String::lastChar.get("abc"))
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3"}

### Interoperability with Java reflection

On the JVM platform, the standard library contains extensions for reflection classes that provide a mapping to and from Java
reflection objects (see package `kotlin.reflect.jvm`).
For example, to find a backing field or a Java method that serves as a getter for a Kotlin property, you can write something like this:

```kotlin
import kotlin.reflect.jvm.*
 
class A(val p: Int)
 
fun main() {
    println(A::p.javaGetter) // prints "public final int A.getP()"
    println(A::p.javaField)  // prints "private final int A.p"
}
```

To get the Kotlin class that corresponds to a Java class, use the `.kotlin` extension property:

```kotlin
fun getKClass(o: Any): KClass<Any> = o.javaClass.kotlin
```

### Constructor references

Constructors can be referenced just like methods and properties. You can use them wherever the program expects a function type object
that takes the same parameters as the constructor and returns an object of the appropriate type.
Constructors are referenced by using the `::` operator and adding the class name. Consider the following function
that expects a function parameter with no parameters and return type `Foo`:

```kotlin
class Foo

fun function(factory: () -> Foo) {
    val x: Foo = factory()
}
```

Using `::Foo`, the zero-argument constructor of the class `Foo`, you can call it like this:

```kotlin
function(::Foo)
```

Callable references to constructors are typed as one of the
[`KFunction<out R>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-function/index.html) subtypes
depending on the parameter count.

### Bound function and property references

You can refer to an instance method of a particular object:

```kotlin
fun main() {
//sampleStart
    val numberRegex = "\\d+".toRegex()
    println(numberRegex.matches("29"))
     
    val isNumber = numberRegex::matches
    println(isNumber("29"))
//sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3"}

Instead of calling the method `matches` directly, the example uses a reference to it.
Such a reference is bound to its receiver.
It can be called directly (like in the example above) or used whenever a function type expression is expected:

```kotlin
fun main() {
//sampleStart
    val numberRegex = "\\d+".toRegex()
    val strings = listOf("abc", "124", "a70")
    println(strings.filter(numberRegex::matches))
//sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3"}

Compare the types of the bound and the unbound references.
The bound callable reference has its receiver "attached" to it, so the type of the receiver is no longer a parameter:

```kotlin
val isNumber: (CharSequence) -> Boolean = numberRegex::matches

val matches: (Regex, CharSequence) -> Boolean = Regex::matches
```

A property reference can be bound as well:

```kotlin
fun main() {
//sampleStart
    val prop = "abc"::length
    println(prop.get())
//sampleEnd
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3"}

You don't need to specify `this` as the receiver: `this::foo` and `::foo` are equivalent.

### Bound constructor references

A bound callable reference to a constructor of an [inner class](nested-classes.md#inner-classes) can
be obtained by providing an instance of the outer class:

```kotlin
class Outer {
    inner class Inner
}

val o = Outer()
val boundInnerCtor = o::Inner
```
