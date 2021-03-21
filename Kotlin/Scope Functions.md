The Kotlin standard library contains several functions whose sole purpose is to execute a block of code within the context of an object. When you call such a function on an object with a [lambda expression](https://kotlinlang.org/docs/reference/lambdas.html) provided, it forms a temporary scope. In this scope, you can access the object without its name. Such functions are called *scope functions*. There are five of them: `let`, `run`, `with`, `apply`, and `also`.

Basically, these functions do the same: execute a block of code on an object. What's different is how this object becomes available inside the block and what is the result of the whole expression.

Due to the similar nature of scope functions, choosing the right one for your case can be a bit tricky. The choice mainly depends on your intent and the consistency of use in your project. Below we'll provide detailed descriptions of the distinctions between scope functions and the conventions on their usage.（5个函数的作用差不多，采用什么主要凭借自己的意愿以及代码中的连贯性）

它们的区别主要在两点：

1. context：分为 this 和 it；
2. 返回值。

context:

​	context 为 this 的函数为：`run`, `with`, `apply`

​	context 为 it 的函数为： `let`, `also`

返回值：

​	`apply`, `also` 返回 context object

​	`let`, `run`, `with` 返回表达式结果