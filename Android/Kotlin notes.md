# Kotlin notes

## Keywords or Built-in function

1. also

   `also` is like `apply`: it takes the receiver, does some action on it, and returns that receiver. The difference is that in the block inside `apply` the receiver is available as `this`, while in the block inside `also` it's available as `it` (and you can give it another name if you want). This comes handy when you do not want to shadow `this` from the outer scope:

   ```kotlin
   class Block {
       lateinit var content: String
   }
   
   fun Block.copy() = Block().also {
       it.content = this.content
   }
   
   // using 'apply' instead
   fun Block.copy1() = Block().apply {
       this.content = this@copy1.content
   }
   
   fun main(args: Array<String>) {
       val block = Block().apply { content = "content" }
       val copy = block.copy()
       println("Testing the content was copied:")
       println(block.content == copy.content)
   }
   ```

2. takeIf

   `takeIf` is like `filter` for a single value. It checks whether the receiver meets the predicate, and returns the receiver, if it does or `null` if it doesn't. Combined with an elvis-operator and early returns it allows to write constructs like:

   ```kotlin
   val outDirFile = File(outputDir.path).takeIf { it.exists() } ?: return false
   // do something with existing outDirFile
   ```

   ```kotlin
   fun main(args: Array<String>) {
       val input = "Kotlin"
       val keyword = "in"
   
       val index = input.indexOf(keyword).takeIf { it >= 0 } ?: error("keyword not found")
       // do something with index of keyword in input string, given that it's found
       
       println("'$keyword' was found in '$input'")
       println(input)
       println(" ".repeat(index) + "^")
   }
   ```

3. takeUnless

   `takeUnles` when replaces the switch operator of C-like languages. In the simplest form it looks like this

   ```kotlin
   when (x) {
       1 -> print("x == 1")
       2 -> print("x == 2")
    else -> { // Note the block
           print("x is neither 1 nor 2")
    }
   }
   ```
   
   when matches its argument against all branches sequentially until some branch condition is satisfied. when can be used either as an expression or as a statement. If it is used as an expression, the value of the satisfied branch becomes the value of the overall expression. If it is used as a statement, the values of individual branches are ignored. (Just like with if, each branch can be a block, and its value is the value of the last expression in the block.)
   
   The else branch is evaluated if none of the other branch conditions are satisfied. If when is used as an expression, the else branch is mandatory, unless the compiler can prove that all possible cases are covered with branch conditions (as, for example, with enum class entries and sealed class subtypes).
   
   If many cases should be handled in the same way, the branch conditions may be combined with a comma:
   
   ```kotlin
   when (x) {
       0, 1 -> print("x == 0 or x == 1")
       else -> print("otherwise")
   }
   ```
   
   We can use arbitrary expressions (not only constants) as branch conditions
   
   ```kotlin
   when (x) {
       parseInt(s) -> print("s encodes x")
       else -> print("s does not encode x")
   }
   ```
   
   We can also check a value for being in or !in a range or a collection:
   
   ```kotlin
   when (x) {
       in 1..10 -> print("x is in the range")
       in validNumbers -> print("x is valid")
       !in 10..20 -> print("x is outside the range")
       else -> print("none of the above")
   }
   ```
   
   Another possibility is to check that a value is or !is of a particular type. Note that, due to smart casts, you can access the methods and properties of the type without any extra checks.
   
   ```kotlin
   fun hasPrefix(x: Any) = when(x) {
       is String -> x.startsWith("prefix")
       else -> false
   }
   ```
   
   when can also be used as a replacement for an if-else if chain. If no argument is supplied, the branch conditions are simply boolean expressions, and a branch is executed when its condition is true:
   
   ```kotlin
   when {
       x.isOdd() -> print("x is odd")
       x.isEven() -> print("x is even")
       else -> print("x is funny")
   }
   ```
   
   Since Kotlin 1.3, it is possible to capture when subject in a variable using following syntax:
   
   ```kotlin
   fun Request.getBody() =
           when (val response = executeRequest()) {
               is Success -> response.body
               is HttpError -> throw HttpException(response.status)
           }
   ```
   
   Scope of variable, introduced in when subject, is restricted to when body.
   
   See the grammar for when.s` is the same as `takeIf`, but it takes the inverted predicate. It returns the receiver when it *doesn't* meet the predicate and `null` otherwise. So one of the examples above could be rewritten with `takeUnless` as following:
   
   ```kotlin
   val index = input.indexOf(keyword).takeUnless { it < 0 } ?: error("keyword not found")
   ```
   
   It is also convenient to use when you have a callable reference instead of the lambda:
   
   ```kotlin
   private fun testTakeUnless(string: String) {
       val result = string.takeUnless(String::isEmpty)
   
       println("string = \"$string\"; result = \"$result\"")
   }
   
   fun main(args: Array<String>) {
       testTakeUnless("")
       testTakeUnless("abc")
   }
   ```
   
4. any

   [any()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/any.html)





## Operator

1. Safe Calls ( ?. )

   ```kotlin
   val a = "Kotlin"
   val b: String? = null
   println(b?.length)
   println(a?.length) // Unnecessary safe call
   ```

   This returns `b.length` if `b` is not null, and *null* otherwise. The type of this expression is `Int?`.

   Safe calls are useful in chains. For example, if Bob, an Employee, may be assigned to a Department (or not), that in turn may have another Employee as a department head, then to obtain the name of Bob's department head (if any), we write the following:

   ```kotlin
   bob?.department?.head?.name
   ```

   Such a chain returns *null* if any of the properties in it is null.

   To perform a certain operation only for non-null values, you can use the safe call operator together with [`let`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/let.html):

   ```kotlin
   val listWithNulls: List<String?> = listOf("Kotlin", null)
   for (item in listWithNulls) {
       item?.let { println(it) } // prints Kotlin and ignores null
   }
   ```

   A safe call can also be placed on the left side of an assignment. Then, if one of the receivers in the safe calls chain is null, the assignment is skipped, and the expression on the right is not evaluated at all:

   ```kotlin
   // If either `person` or `person.department` is null, the function is not called:
   person?.department?.head = managersPool.getManager()
   ```

2. Elvis Operator ( ?: )

   When we have a nullable reference `r`, we can say "if `r` is not null, use it, otherwise use some non-null value `x`":

   ```kotlin
   val l: Int = if (b != null) b.length else -1
   ```

   Along with the complete *if*-expression, this can be expressed with the Elvis operator, written `?:`:

   ```kotlin
   val l = b?.length ?: -1
   ```

   If the expression to the left of `?:` is not null, the elvis operator returns it, otherwise it returns the expression to the right. Note that the right-hand side expression is evaluated only if the left-hand side is null.

   Note that, since *throw* and *return* are expressions in Kotlin, they can also be used on the right hand side of the elvis operator. This can be very handy, for example, for checking function arguments:

   ```kotlin
   fun foo(node: Node): String? {
       val parent = node.getParent() ?: return null
       val name = node.getName() ?: throw IllegalArgumentException("name expected")
       // ...
   }
   ```

3. NPE-lover ( !! ) (NPE: Null Pointer Exception)

   The third option is for NPE-lovers: the not-null assertion operator (`!!`) converts any value to a non-null type and throws an exception if the value is null. We can write `b!!`, and this will return a non-null value of `b`(e.g., a `String` in our example) or throw an NPE if `b` is null:

   ```kotlin
   val l = b!!.length
   ```

   Thus, if you want an NPE, you can have it, but you have to ask for it explicitly, and it does not appear out of the blue.

4. Safe Casts

   Regular casts may result into a `ClassCastException` if the object is not of the target type. Another option is to use safe casts that return *null* if the attempt was not successful:

   ```kotlin
   val aInt: Int? = a as? Int
   ```

5. Collections of Nullable Type

   If you have a collection of elements of a nullable type and want to filter non-null elements, you can do so by using `filterNotNull`:

   ```kotlin
   val nullableList: List<Int?> = listOf(1, 2, null, 4)
   val intList: List<Int> = nullableList.filterNotNull()
   ```



## Expression

1. When

   *when* replaces the switch operator of C-like languages. In the simplest form it looks like this

   ```kotlin
   when (x) {
       1 -> print("x == 1")
       2 -> print("x == 2")
       else -> { // Note the block
           print("x is neither 1 nor 2")
       }
   }
   ```

   *when* matches its argument against all branches sequentially until some branch condition is satisfied. *when* can be used either as an expression or as a statement. If it is used as an expression, the value of the satisfied branch becomes the value of the overall expression. If it is used as a statement, the values of individual branches are ignored. (Just like with *if*, each branch can be a block, and its value is the value of the last expression in the block.)

   The *else* branch is evaluated if none of the other branch conditions are satisfied. If *when* is used as an expression, the *else* branch is mandatory, unless the compiler can prove that all possible cases are covered with branch conditions (as, for example, with [*enum* class](https://kotlinlang.org/docs/reference/enum-classes.html) entries and [*sealed* class](https://kotlinlang.org/docs/reference/sealed-classes.html) subtypes).

   If many cases should be handled in the same way, the branch conditions may be combined with a comma:

   ```kotlin
   when (x) {
       0, 1 -> print("x == 0 or x == 1")
       else -> print("otherwise")
   }
   ```

   We can use arbitrary expressions (not only constants) as branch conditions

   ```kotlin
   when (x) {
       parseInt(s) -> print("s encodes x")
       else -> print("s does not encode x")
   }
   ```

   We can also check a value for being *in* or *!in* a [range](https://kotlinlang.org/docs/reference/ranges.html) or a collection:

   ```kotlin
   when (x) {
       in 1..10 -> print("x is in the range")
       in validNumbers -> print("x is valid")
       !in 10..20 -> print("x is outside the range")
       else -> print("none of the above")
   }
   ```

   Another possibility is to check that a value *is* or *!is* of a particular type. Note that, due to [smart casts](https://kotlinlang.org/docs/reference/typecasts.html#smart-casts), you can access the methods and properties of the type without any extra checks.

   ```kotlin
   fun hasPrefix(x: Any) = when(x) {
       is String -> x.startsWith("prefix")
       else -> false
   }
   ```

   *when* can also be used as a replacement for an *if*-*else* *if* chain. If no argument is supplied, the branch conditions are simply boolean expressions, and a branch is executed when its condition is true:

   ```kotlin
   when {
       x.isOdd() -> print("x is odd")
       x.isEven() -> print("x is even")
       else -> print("x is funny")
   }
   ```

   Since Kotlin 1.3, it is possible to capture *when* subject in a variable using following syntax:

   ```kotlin
   fun Request.getBody() =
           when (val response = executeRequest()) {
               is Success -> response.body
               is HttpError -> throw HttpException(response.status)
           }
   ```

   Scope of variable, introduced in *when* subject, is restricted to *when* body.

   See the [grammar for *when*](https://kotlinlang.org/docs/reference/grammar.html#whenExpression).



## Grammar

1. SAM ( Single Abstract Method)

   Just like Java 8, Kotlin supports SAM conversions. This means that Kotlin function literals can be automatically converted into implementations of Java interfaces with a single non-default method, as long as the parameter types of the interface method match the parameter types of the Kotlin function.

   You can use this for creating instances of SAM interfaces:

   ```kotlin
   val runnable = Runnable { println("This runs in a runnable") }
   ```

   …and in method calls:

   ```kotlin
   val executor = ThreadPoolExecutor()
   // Java signature: void execute(Runnable command)
   executor.execute { println("This runs in a thread pool") }
   ```

   **If the Java class has multiple methods taking functional interfaces, you can choose the one you need to call by using an adapter function that converts a lambda to a specific SAM type**. Those adapter functions are also generated by the compiler when needed:

   ```kotlin
   executor.execute(Runnable { println("This runs in a thread pool") })
   ```

   Note that SAM conversions only work for interfaces, not for abstract classes, even if those also have just a single abstract method.

   Also note that this feature works only for Java interop; since Kotlin has proper function types, automatic conversion of functions into implementations of Kotlin interfaces is unnecessary and therefore unsupported.

2. [Operator overloading](https://kotlinlang.org/docs/reference/operator-overloading.html?_ga=2.36897571.2001104145.1557971455-1529641365.1557742512)

3. [Destructuring Declarations](<https://kotlinlang.org/docs/reference/multi-declarations.html?_ga=2.167493438.1815675817.1559011923-1529641365.1557742512>) & componentN()[^ 1]

   Sometimes it is convenient to *destructure* an object into a number of variables, for example:

   ```kotlin
   val (name, age) = person
   ```

   This syntax is called a *destructuring declaration*. A destructuring declaration creates multiple variables at once. We have declared two new variables: `name` and `age`, and can use them independently:

   ```kotlin
   println(name)
   println(age)
   ```

   A destructuring declaration is compiled down to the following code:

   ```kotlin
   val name = person.component1()
   val age = person.component2()
   ```

   The `component1()` and `component2()` functions are another example of the *principle of conventions* widely used in Kotlin (see operators like `+` and `*`, *for*-loops etc.). Anything can be on the right-hand side of a destructuring declaration, as long as the required number of component functions can be called on it. And, of course, there can be `component3()` and `component4()` and so on.

   Note that the `componentN()` functions need to be marked with the `operator` keyword to allow using them in a destructuring declaration.

   Destructuring declarations also work in *for*-loops: when you say:

   ```kotlin
   for ((a, b) in collection) { ... }
   ```

   Variables `a` and `b` get the values returned by `component1()` and `component2()` called on elements of the collection.





[^ 1]: function componentN() only used in `data class`