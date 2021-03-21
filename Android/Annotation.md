# Annotation

更多内容参考[本帖](https://blog.csdn.net/briblue/article/details/73824058)

------

##  Annotation 架构

![Annotation 架构图](https://images0.cnblogs.com/blog/497634/201309/28123151-d471f82eb2bc4812b46cc5ff3e9e6b82.jpg)

从中，我们可以看出：

- 1个Annotation 和 1个RetentionPolicy关联。
         可以理解为：每1个Annotation对象，都会有唯一的RetentionPolicy属性。

- 1个Annotation 和 1~n个ElementType关联。
         可以理解为：对于每1个Annotation对象，可以有若干个ElementType属性。

- Annotation 有许多实现类，包括：Deprecated, Documented, Inherited, Override等等。
  Annotation 的每一个实现类，都“和1个RetentionPolicy关联”并且“和1~n个ElementType关联”。



## Annotation 组成部分

### 1. Annotaion 组成部分

java annotation 的组成中，有3个非常重要的主干类。它们分别是：

1. Annotation.java

   ```java
   package java.lang.annotation;
   public interface Annotation {
   
       boolean equals(Object obj);
   
       int hashCode();
   
       String toString();
   
       Class<? extends Annotation> annotationType();
   }
   ```

   

2. ElementType.java

   ```java
   package java.lang.annotation;
   
   public enum ElementType {
       TYPE,               /* 类、接口（包括注释类型）或枚举声明  */
   
       FIELD,              /* 字段声明（包括枚举常量）  */
   
       METHOD,             /* 方法声明  */
   
       PARAMETER,          /* 参数声明  */
   
       CONSTRUCTOR,        /* 构造方法声明  */
   
       LOCAL_VARIABLE,     /* 局部变量声明  */
   
       ANNOTATION_TYPE,    /* 注释类型声明  */
   
       PACKAGE             /* 包声明  */
   }
   ```

3. RetentionPolicy.java

   ```java
   package java.lang.annotation;
   public enum RetentionPolicy {
       SOURCE,            /* Annotation信息仅存在于编译器处理期间，编译器处理完之后就没有该Annotation信息了  */
   
       CLASS,             /* 编译器将Annotation存储于类对应的.class文件中。默认行为  */
   
       RUNTIME            /* 编译器将Annotation存储于class文件中，并且可由JVM读入 */
   }
   ```



## Java Annotation

### 1. Annotation 定义

```java
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation1 {
}
```



### 2. 元注解

元注解是什么意思呢？

元注解是可以注解到注解上的注解，或者说元注解是一种基本注解，但是它能够应用到其它的注解上面。

如果难于理解的话，你可以这样理解。元注解也是一张标签，但是它是一张特殊的标签，它的作用和目的就是给其他普通的标签进行解释说明的。

元标签有 `@Retention`、`@Documented`、`@Target`、`@Inherited`、`@Repeatable` 5 种。



### 3. @Retention

Retention 的英文意为保留期的意思。当 @Retention 应用到一个注解上的时候，它解释说明了这个注解的的存活时间。

它的取值如下：

`RetentionPolicy.SOURCE` 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。
`RetentionPolicy.CLASS` 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。
`RetentionPolicy.RUNTIME` 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。



### 4. @Documented

顾名思义，这个元注解肯定是和文档有关。它的作用是能够将注解中的元素包含到 Javadoc 中去。



### 5. @Target

Target 是https://blog.csdn.net/briblue/article/details/73824058https://blog.csdn.net/briblue/article/details/73824058目标的意思，@Target 指定了注解运用的地方。

你可以这样理解，当一个注解被 @Target 注解时，这个注解就被限定了运用的场景。

类比到标签，原本标签是你想张贴到哪个地方就到哪个地方，但是因为 @Target 的存在，它张贴的地方就非常具体了，比如只能张贴到方法上、类上、方法参数上等等。@Target 有下面的取值

- `ElementType.ANNOTATION_TYPE` 可以给一个注解进行注解
- `ElementType.CONSTRUCTOR` 可以给构造方法进行注解
- `ElementType.FIELD` 可以给属性进行注解
- `ElementType.LOCAL_VARIABLE` 可以给局部变量进行注解
- `ElementType.METHOD` 可以给方法进行注解
- `ElementType.PACKAGE` 可以给一个包进行注解
- `ElementType.PARAMETER` 可以给一个方法内的参数进行注解
- `ElementType.TYPE` 可以给一个类型进行注解，比如类、接口、枚举



### 6. @Inherited

Inherited 是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。
说的比较抽象。代码来解释。

```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}


@Test
public class A {}


public class B extends A {}


```



### 7. @Repeatable

Repeatable 自然是可重复的意思。@Repeatable 是 Java 1.8 才加进来的，所以算是一个新的特性。

什么样的注解会多次应用呢？通常是注解的值可以同时取多个。

举个例子，一个人他既是程序员又是产品经理,同时他还是个画家。

```java

@interface Persons {
	Person[]  value();
}


@Repeatable(Persons.class)
@interface Person{
	String role default "";
}


@Person(role="artist")
@Person(role="coder")
@Person(role="PM")
public class SuperMan{
	
}
```

注意上面的代码，@Repeatable 注解了 Person。而 @Repeatable 后面括号中的类相当于一个容器注解。

什么是容器注解呢？就是用来存放其它注解的地方。它本身也是一个注解。

我们再看看代码中的相关容器注解。

```java
@interface Persons {
	Person[]  value();
}
```

按照规定，它里面必须要有一个 value 的属性，属性类型是一个被 @Repeatable 注解过的注解数组，注意它是数组。

如果不好理解的话，可以这样理解。Persons 是一张总的标签，上面贴满了 Person 这种同类型但内容不一样的标签。把 Persons 给一个 SuperMan 贴上，相当于同时给他贴了程序员、产品经理、画家的标签。

我们可能对于 @Person(role=“PM”) 括号里面的内容感兴趣，它其实就是给 Person 这个注解的 role 属性赋值为 PM 。



## 注解的属性

注解的属性也叫做成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
	
	int id();
	
	String msg();

}
```

上面代码定义了 TestAnnotation 这个注解中拥有 id 和 msg 两个属性。在使用的时候，我们应该给它们进行赋值。

赋值的方式是在注解的括号内以 value="" 形式，多个属性之前用 ，隔开。

```java
@TestAnnotation(id=3,msg="hello annotation")
public class Test {

}
```

需要注意的是，在注解中定义属性时它的类型必须是 8 种基本数据类型外加 类、接口、注解及它们的数组。

注解中属性可以有默认值，默认值需要用 default 关键值指定。比如：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
	
	public int id() default -1;
	
	public String msg() default "Hi";

}
```

TestAnnotation 中 id 属性默认值为 -1，msg 属性默认值为 Hi。
它可以这样应用。

```java
@TestAnnotation()
public class Test {}
```

因为有默认值，所以无需要再在 @TestAnnotation 后面的括号里面进行赋值了，这一步可以省略。

另外，还有一种情况。如果一个注解内仅仅只有一个名字为 value 的属性时，应用这个注解时可以直接接属性值填写到括号内。

```java
public @interface Check {
	String value();
}
```

上面代码中，Check 这个注解只有 value 这个属性。所以可以这样应用。

```java
@Check("hi")
int a;
```

这和下面的效果是一样的

```java
@Check(value="hi")
int a;
```

最后，还需要注意的一种情况是一个注解没有任何属性。比如

```java
public @interface Perform {}
```

那么在应用这个注解的时候，括号都可以省略。

```java
@Perform
public void testMethod(){}
```



## 注解的提取

要想正确检阅注解，离不开一个手段，那就是反射。

### 1. 注解与反射

