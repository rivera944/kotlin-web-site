[//]: # (title: Intermediate: Extension functions)

<tldr>
    <p><img src="icon-1.svg" width="20" alt="First step" /> <strong>Extension functions</strong><br />
        <img src="icon-2-todo.svg" width="20" alt="Second step" /> <a href="kotlin-tour-intermediate-functions-part-2.md">Scope functions</a><br />
        <img src="icon-3-todo.svg" width="20" alt="Third step" /> <a href="kotlin-tour-intermediate-functions-part-3.md">Lambda expressions with receiver</a><br />
        <img src="icon-4-todo.svg" width="20" alt="Fourth step" /> <a href="kotlin-tour-intermediate-classes-part-1.md">Classes and interfaces</a><br />
        <img src="icon-5-todo.svg" width="20" alt="Fifth step" /> <a href="kotlin-tour-intermediate-classes-part-2.md">Objects</a><br />
        <img src="icon-6-todo.svg" width="20" alt="Sixth step" /> <a href="kotlin-tour-intermediate-classes-part-3.md">Open and special classes</a><br />
        <img src="icon-7-todo.svg" width="20" alt="Seventh step" /> <a href="kotlin-tour-intermediate-properties.md">Properties</a><br />
        <img src="icon-8-todo.svg" width="20" alt="Eighth step" /> <a href="kotlin-tour-intermediate-null-safety.md">Null safety</a><br />
        <img src="icon-7-todo.svg" width="20" alt="Seventh step" /> <a href="kotlin-tour-intermediate-libraries-and-apis.md">Libraries and APIs</a></p>
</tldr>

In this chapter, you'll explore special Kotlin functions that make your code more concise and readable, 
and learn how they can help you use efficient design patterns to take your projects to the next level.

## Extension functions

In software development, you often need to modify the behavior of a program without altering the original source code. 
For example, in your project, you might want to add extra functionality to a class from a third-party library.

Extension functions allow you to extend a class with additional functionality. You call extension functions as if they 
are member functions of a class.

Before introducing the syntax for extension functions, you need to understand the terms **receiver type** and 
**receiver object**.

The receiver object is what the function is called on. In other words, the receiver is where or with whom the information is shared.

![An example of sender and receiver](receiver-highlight.png){width="500"}

In this example, the `main()` function calls the `.first()` function. The `.first()` function is called **on** the `readOnlyShapes`
variable, so the `readOnlyShapes` variable is the receiver.

The receiver object has a **type** so that the compiler understands when the function can be used.

To declare an extension function, write the name of the class that you want to extend followed by a `.` and the name of
your function. Continue with the rest of the function declaration, including its arguments and return type.

In the following example:

* `String` is the extended class, also known as the receiver type.
* The `.bold()` extension function's return type is `String`.
* An instance of `String` is the receiver object.
* The receiver object is accessed inside the body by the [keyword](keyword-reference.md): `this`.
* A string template (`$`) is used to access the value of `this`.
* The `.bold()` extension function takes a string and returns it in a `<b>` HTML element for bold text.

```kotlin
fun String.bold(): String = "<b>$this</b>"

fun main() {
    // "hello" is the receiver object
    println("hello".bold())
    // <b>hello</b>
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-extension-function"}

## Extension-oriented design

Extension functions can be defined anywhere, enabling you to create extension-oriented designs. Such designs separate 
core functionality from useful but non-essential features, making your code easier to read and maintain.

A good example is the [`HttpClient`](https://api.ktor.io/ktor-client/ktor-client-core/io.ktor.client/-http-client/index.html) class from the Ktor library, which helps perform network requests. The core of
its functionality is a single function, which takes all the information you may use in an HTTP request:

```kotlin
class HttpClient {
    fun request(method: String, url: String, headers: Map<String, String>): HttpResponse {
        // Network code
    }
}
```

In practice, most people just want to perform a GET or POST request, so it makes sense for the library to provide shorter
names for that common use case. However, those don't require writing new network code, only the specific request call.
In other words, they are perfect candidates to be defined as extension functions:

```kotlin
fun HttpClient.get(url: String): HttpResponse = request("GET", url, emptyMap())
fun HttpClient.post(url: String): HttpResponse = request("POST", url, emptyMap())
```

This extension-oriented approach is widely used in Kotlin's [standard library](https://kotlinlang.org/api/latest/jvm/stdlib/)
and other libraries. For example, the `String` class has many [extension functions](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-string/#extension-functions)
to help you work with strings.

For more information about extension functions, see [Extensions](extensions.md).

## Practice

### Exercise 1 {initial-collapse-state="collapsed" collapsible="true" id="extension-functions-exercise-1"}

Write an extension function called `isPositive` that takes an integer and checks whether it is positive.

|---|---|
```kotlin
fun // Write your code here

fun main() {
    println(1.isPositive())
    // true
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-extension-functions-exercise-1"}

|---|---|
```kotlin
fun Int.isPositive(): Boolean = this > 0

fun main() {
    println(1.isPositive())
    // true
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution" id="kotlin-tour-extension-functions-solution-1"}

### Exercise 2 {initial-collapse-state="collapsed" collapsible="true" id="extension-functions-exercise-2"}

Write an extension function called `toLowercaseString` that takes a string and returns a lowercase version.

<deflist collapsible="true">
    <def title="Hint">
        Use the <a href="https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/lowercase.html"> <code>.lowercase()</code>
        </a> function for the <code>String</code> type. 
    </def>
</deflist>

|---|---|
```kotlin
// Write your code here

fun main() {
    println(input.toLowercaseString("Hello World!"))
    // hello world!
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-extension-functions-exercise-2"}

|---|---|
```kotlin
fun String.toLowercaseString(): String = this.lowercase()

fun main() {
    println("Hello World!".toLowercaseString())
    // hello world!
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Example solution" id="kotlin-tour-extension-functions-solution-2"}

## Next step

[Intermediate: Scope functions](kotlin-tour-intermediate-functions-part-2.md)