# scala 程序结构

```scala
// object ： 一个伴生对象，就可以理解成为一个对象
// HelloScala 是一个对象的名字，底层真正对应的类名是HelloScala$
// 对象是 HelloScala$ 类型的一个静态对象 MODULE$
// 当编写一个 object HelloScala的时候，底层会生成两个.class文件，分别是HelloScala, HelloScala$， 即：会生成两个class 文件。
// object HelloScala 对应的是 HelloScala$ 的一个静态对象（所以就是单例咯）
// 伴生对象里面放的是静态方法 和 静态属性！
object HelloScala{
    // def 表示一个方法，是一个关键字
    // main 表示方法的名字，表示程序的入口
    // args: Array[String] 参数名字在前，类型在后
    // Unit =  表示该函数的返回值为空
    def main(args: Array[String]):Unit = {
        
    }
}
```

## 字符串输出的三种方式

```scala
println("a" + b + "c")
printf("%s, %d", "name", 15)
// 类似 shell的 string语法
println(s"$name, $age")
// 大括号表示， 里面是一个表达式
println(s"${age + 10}")
```



# 变量 & 类型

* scala 中：小数默认为 double，整数默认为 int
* `val` ： 数值不可变，不可修改。只是表示的引用不可变，但是引用的对象内部是可以变的
* `var`:  变量可以修改
* scala中数据都是对象，没有原生类型
* scala中数据类型分为两大类，AnyVal（值类型）,AnyRef(引用类型)，这两种也都是对象
* AnyVal: 
  * Byte, Short, Int, Long, FLoat, Double, Boolean, Char, StringOps, **Uint**(等价于空值)
* AnyRef:
  * Scala Collections, All Java Classes, Other Scala Classes，String
* 底层类
  * Null：所有AnyRef的子类，只有一个值 null，这个对象可以赋值给任意其他引用类的对象  （Unit：空值，Null：空引用）
  * Nothing: 所有类的子类：在开发中，Noting的值可以赋值给任意类型的对象，如果函数没法正常返回（抛出异常时候），会返回该类型的对象
  * 这个需要和object理解：object是所有类型的父类
    * java中的object对应scala中的 Any
* 隐式转换：
  * **低精度** 向 **高精度** 自动转换，高到低是需要手动转换的哦。
* 1.2 (Double), 1.2f(Float), 1(Int), 1l(Long)
* 科学记数法: 1e-2, 1E-2

```scala
var a : Int = 100
val b : Int = 100

// 类型推导
var num = 10 //推导为int类型，类型确定之后就不能修改了
num.isInstanceOf[INt] //判断类型
```

```scala
object HelloWorld {

  def main(args: Array[String]): Unit = {
    val dog = new Dog
    dog.age = 18
    dog.name = "hello"
    // 该句报错，因为 dog 是val，相当于是个C++ 的底层 const
    dog = new Dog
  }
}

class Dog {
  // _ 在这里表示 默认值
  var age: Int = _
  var name : String = _
}
```
* Char: 2字节（16位） unicode 码
* Boolean: 只有 `true` 和 `false`, 不能用 1 或 0
* Unit: 只有一个实例对象 ()
* Null: 只有一个实例对象 null, 任何 AnyRef 的子类
* Nothing: 主要只用来抛异常的, 如果一个方法抛出了异常，调用该方法的返回值就变成了 `Nothing` 的对象，这里需要和scala的异常处理一起理解
* Any: 相当于 java 的 object
* Char 与 Byte, Short 不能互相隐式转换, 三者可以混合计算, 计算的时候会转成 int 然后计算. Byte 和 Short 可以互相转换
* 类型判断 `a.isInstanceOf[Int]`
* 类型转换? `a.asInstanceOf[Int]`



```scala
object HelloWorld {
    
    def main(args: Array[String]): Unit = {
        val v1 = m1();
        println(v1) // 打印出来的就是 ()
        
    }
    
    def m1(): Unit = {
        println("hello")
    }
    
    def m2(): Nothing = {
        throw new NullPointerException
    }
    
    def m3(a: Int): Int = {
        if (a == 0) {
            throw new NullPointerException // 返回Nothing
        } esle {
            return a //返回 Int, 因为 方法的返回值可以放返回值的 父类。所以使用 Int 就可以了
        }
    }
    
    
}
```



## 自动类型转换





## 隐式类型转换

**当编译器一次编译失败的时候，会在当前的环境中查找让代码通过编译的方法，用于将类型进行转换，实现二次编译**

* 隐式函数
  * 隐式转换可以在不修改任何代码的情况下，扩展某个类的功能
* 隐式参数
  * 函数的形参可以通过`implicit` 关键字声明为隐式参数，调用的时候如果 调用者不传入该形参，编译器就会在相应的作用域中寻找符合条件的隐式值
    * 同一个作用域中，相同类型的隐式值只能有一个  （**同一个作用域如何理解**？？）
    * 编译器按照隐式值的类型找值，于隐式值的名字无关
    * 隐式参数优先于默认参数
* 隐式类：扩展类的功能 （2.10+）
  * 其所带的构造参数有且只能有一个
  * 隐式类必须被定义在 "类/半生对象/包对象" 中，即隐式类不能是顶级的
* 隐式解析机制
  * 首先会在代码的作用域下查找隐式转换操作，如果找不到
  * 在隐式参数的类型作用域里查找。类型的作用域是指与类型相关的 伴生对象，和类型所在包的包对象。

```scala
class MyRichInt(val self: Int) {
    def myMax(i: Int): Int = if(self < i) i else self
}

object TestImplicitFunction {
    implicit def convert(arg: Int): MyRichInt = {
        new MyRichInt(arg)
    }
    
    def main(args: Array[String]): Unit = {
        // 如果第一次编译错误，那么编译器就会在当前作用域范围内查找对应调用功能的转化规则，这个调用是由编译器完成的，所以称之为隐式转换
        2.myMax(6)
    }
    
}

```



```scala
object TestImplicitVar{
    implicit val str: String = "hello world"
    
    def hello(implicit arg: String = "good bye") {
        
    }
    
    def main(args: Array[String]) {
        hello
        
        implicit[String] // 获取String类型的隐式变量
    }
}
```



```scala
object TestImplicitClass {
	implicit class MyRichInt(arg: Int) {
        def myMax(i: Int): Int = {
            if (arg < i) i else arg
        }
    }
    
    def main(args: Array[String]) {
        1.myMax(3)
    }
    
}
```




## 强制类型转换
* a.toInt, a.toInt(), 可能会导致精度降低或者溢出
* string 类型转换: 
```scala
val a: Int = 10
val aStr: String = 10 + ""; // +个空串 就 ok 了
aStr.to??
2.7.toInt //秀啊
```



# 标识符 (命名规范)

```scala
//  两个操作符 可以作为 命名的开头, 必须是两个.... 为啥要这么搞? 有啥必要吗?
val ++ = "hello world"

// 返引号 可以使得 关键字变成变量使用
val `true` = "hello world"
println(`true`)
```

* `_` 的 N 中功能
```scala
import scala.io._ //引入该包下的所有东西
```

# 运算符
* `+`: 可以用来字符串相加
* `*`: 可以用来字符串的 倍数拷贝
* `/`: 
* `==, !=, <, >, <=, >=`
  * `==`: 判断值是否相等（会调用equals的方法重载），使用 `s1.eq(s2)` 用来判断地址是否相等 （这里和 java 不一样）
* `=, +=, -=, *=, /=, %=, <<=, >>=, &=, ^=, |=` 
* `&, |, ^, ~, <<, >>, >>>`
* scala 支持代码块返回值

```scala
val res = {
    90
}
```
* 三元运算符 `val num if(5>4) 5 else 4`
* 命令行输入`val val = StdIn.readLine()`
* 方法调用

```scala

val n1 = 10
val n2 = 11

n1.+(n2) // 原始方法调用写法
n1 +(n2) // .可以省略
n1 + n2 // 如果方法的参数只有一个，那么小括号也可以省略

"1".toInt()
"1".toInt // 如果没有参数，小括号可以省略
"1" toInt

```

# 分支控制
* 没有 switch
* for 循环
```scala
// if else 是有返回值的，定义为代码块的最后一行！！！！

if (expression) {

} else if (expression2) {

}


val n1 = if(true) {
    // doSomething
    10
} else {
    // doSomething
    11
}


// for 表达式, for 推导式
// 1 to 3 :  两边都是 闭合
// 1 until 3: 前闭 后 开
// 也可以这样直接对集合进行遍历
// to 实际也是一个方法调用！！！
for(i <- 1 to 3){

}

// 循环守卫, 如果为 false 就直接跳过, 相当于 continue, scala 中没有 continue 和 break
for (i <- 1 to 3 if i!=2) {

}

// 循环步长, 步长为2
for (i <- 1 to 3 by 2) {
    
}

// 反转遍历

for (i <- 1 to 3 reverse) {
    
}

// for 引入变量
for (i <- 1 to 3; j = 4-i) {

}
// 等价于
for (i <- 1 to 3) {
    j = 4-i
    ...
}
// 引入变量可以写多行
for {
    i <- 1 to 3
    j = 4-i
    z = 3 + j
    }{
    
	...    
}


// 嵌套循环, 等价于嵌套循环的
for (i <- 1 to 3; j <- 1 to 3){
}
// 等价于
for (i <- 1 to 3) {
    for (j <- 1 to 3) {
        
    } 
}



// 循环返回值, 将遍历过程中产生的值放到一个 vector 中去， 使用yield关键字. 如果不用 yield，编译器会认为返回的 Unit
val res = for(i<-1 to 10) yield i

// yield + 代码块返回值
val res2 = for(i<-1 to 10) yield {
    if (i%2 == 0)
    	i
    else
    	"not even"
}
```
* while 循环（不建议使用）

```scala
while (expression) {
    
}
```

* 循环中断 (`使用抛出异常的方式来退出循环`)

```scala
// breakable 高阶函数，接收函数的函数
// breakable(op: =>Unit) 接收一个没有形参，没有输出的函数，
// breakable 对 op 抛出的异常 进行捕获处理
// 当传入的是代码块的时候， 一般会将 小括号 转成 大括号
Breaks.breakable( // 用来 catch 异常
    while (i < 20) {
        n += 1
        if (n == 18) {
            Breaks.break() //没有 break 关键字，只有个函数，该函数只是扔出一个异常
        }
    }
)
// 当传入的是代码块的时候， 一般会将 小括号 转成 大括号
Breaks.breakable{
    while (i < 20) {
        n += 1
        if (n == 18) {
            Breaks.break() //没有 break 关键字，只有个函数，该函数只是扔出一个异常
        }
    }
}
```

* 如何实现 continue
  * 使用 `if else`
  * 也可以使用 `循环守卫`

* 模式匹配: match-case



# 函数式编程

> 函数式编程 vs 命令式编程（面向对象，面向过程）
>
> 命令式编程：对计算机友好
>
> 函数式编程：对程序员友好 （最好所有的 值都是常量，剩下的交给编译期优化），没有副作用，幂等性

* 推荐递归解决问题
* 方法和函数几乎可以等同：定义，使用，运行机制一样
  * 函数：为完成某一功能的语句的集合
  * 方法：类中的函数称之为方法。（注意：伴生对象中的函数是函数而不是方法）
  * `scala` 中的函数可以随处定义（`除了 类 和 伴生对象 外不能定义`）
  * 函数的使用方式更加的灵活多样，（方法转函数）
  * scala 中，函数是一等公民，函数也是对象，函数的创建不依赖类或者对象。java中的函数创建要依赖类，抽象类，或者接口。

```scala
object HelloWorld{
    def main(args: Array[String]): Uint = {
        val dog = new Dog
        dog.sum(1, 2)
        val f1 = dog.sum _ // 方法转函数
        f1(1, 2)
        
        // 这个是 lambda 表达式吧
        val f2 = (n1: Int, n2: Int) => {
            n1 + n2
        }
        f2(1, 2)
      	// 直接定义函数
      	def sayHi(): Unit = {
          	println("hi")
        }
      sayHi()
      
    }
}
class Dog {
    // 此处是方法， 这里还有默认值的 demo
    def sum(n1: Int, n2: Int = 10): Int = {
        n1 + n2
    }
}
```

* 函数的定义
  * `def func_name(paraname: type):ReturnType = {func_body}`
  * `def func_name(paraname: type) = {func_body}`: 表示自动推断返回值类型
  * `def func_name(paraname: type){func_body}`: 表示没有返回值
  * 如果没有return，`默认认为执行的最后一句的值作为返回值`
  * 如果没有返回值，return不生效
* 函数的调用机制：依旧是压栈出栈来完的
* 细节总结：
  * 函数如果没有形参，调用的时候可以不加括号
  * 省略 `: Type` 会进行返回参数类型推断
    * `使用返回值类型推断的时候` 不要使用 `return` 进行返回
  * 如果省略了 `: Type =` ： 表示该函数没有返回值，这时候即使写了 `return` 也是无效的。
    * 如果使用`: Unit = ` ： 也是表示没有返回值，这时候即使写了 `return` 也是无效的。
  * 如果不确定返回值类型，最好使用自动类型推断
  * 可以方法内定义方法，函数内定义函数？
    * 方法内定义的方法 的 **地位** 其实和普通的类方法一个级别的。
  * scala 函数 形参默认是 **`val`** 的
  * 递归程序不能使用类型推断！！！必须指定返回的数据类型
  * 支持可变参数： `def sum(args: Int*)` 
  * **过程**： 没有返回值的函数（方法）
* 函数至简原则
  * return 可以省略
  * 如果函数体只有一行代码，花括号可以省略
  * 返回值的类型如果能推断出来，就可以省略 `:Type` 一起省略
  * 如果有`return`，必须要指定返回值类型
  * 如果函数指定了返回类型 `Unit` ，那么在函数体内写`return` 也是没用的
  * 如果期望是无返回值类型，可以直接省略 `=`, `def func() {}`
  * 如果函数没有形参
    * 如果函数声明时有小括号，调用时小括号可加可不加
    * 如果函数声明时没有小括号，调用是不能加小括号
  * 如果不关心名称，只关心处理逻辑，那么`函数名 & def` 可以省略

```scala
// lambda 表达式，匿名函数
(name: String) => {println(name)}

val fun = (name: String) => {println(name)}
fun("hello")

//函数类型 String => Unit. 前面是参数，后面是返回值
```



* 匿名函数的精简原则
  * 形参的省略
    * 形参的类型可以省略，这个可以自行推倒
    * 如果形参只有`一个参数`，括号可以省略, 没有参数和多个参数，不能省略
  * 函数体的省略
    * 如果函数体只有一行，花括号可以省略
  * 参数名称的省略
    * 如果形参在函数体中只出现一次，那么形参可以省略，且后面的参数可以用 `_` 代替 （=> 也可以省略）
      * 如果形参有多个，也可以这么使用，但是首先前提是，形参在函数体中只出现一次，且出现的顺序和形参顺序一致！！
    * 如果可以推断出当前传入的  是匿名函数的函数体，而不是调用语句，那么 `_` 都可以省略

```scala
def f(func: String -> Unit) {
  func("hello")
}

f((name: String) => {println(name)})
f(name => {println(name)})
f(name => println(name))
f(println(_))
f(println)

```



```scala
// 函数作为值传递

def f(i: Int){println(i)}

val f1 = f _ //(注意，当函数作为值传递的时候，不能直接用 函数名，因为 scala 中，直接用函数名表示的是函数调用，所以 scala 使用 f _ 方式来表示函数整体)
val f2: Int=>Unit = f // 这种是可以直接使用 函数名传递的， println(f2) 这里打印出来的就是地址了


// 函数作为参数



// 函数作为函数返回值

def fff(): Int=>Unit = {
 	def inner(i: Int) {
    println(i)
  } 
  inner
}
```

## 闭包

> 内层函数，访问了外层函数的变量，然后这个内层函数被返回出来了。



```scala
def addBy(a: Int): Int => Int = {
  def addB(b: Int): Int  = {
    a + b
  }
  addB
}

val addBy4 = addBy(4) _ 
val bPlus4 = addBy4(10)


// 精简
def addBy(a: Int): Int => Int = {
  (b: Int) => {
    a + b
  }
}

def addBy(a: Int): Int => Int = {
  b => {
    a + b
  }
}

def addBy(a: Int): Int => Int = {
    a + _
}

def addBy(a: Int): Int => Int = a + _
```



## 函数科里化

* 把一个参数列表的多个参数，变成多个参数列表。有啥用的呢？
* 函数编程中，接收多个 `参数的函数` 都可以转化为接收 单个参数的函数，这个转化过程就是科里化。
* 科里化只是证明了参数只需要一个参数而已。

```scala
def mul(x: Int)(y: Int) = x * y
mul(10)(20)
```

```scala
// 一个函数只是处理一件事情的思想

```

```scala
def add(a: Int, b: Int): Int = {
  
}

// 这个的底层实现就是闭包
def add2(a: Int)(b: Int): Int = {
  
}
```



## 递归

> Scala 中的递归必须声明函数的返回值类型

```scala
def fact(n: Int): Int = {
	if (n == 0) return 1
  fact(n-1) * n
}
```

* 尾递归优化 (编译期可以对栈帧进行优化)
  * 最后一句只返回自身的调用，不会引用其它地方的值。

```scala
def tailFact(n: Int):Int = {
  @tailrec // 这个注解会让 scala 来检查这个是不是正确的 尾递归实现
  def loop(n: Int, currRes: Int): Int = {
    if (n == 0) return currRes
    loop(n-1, currRes * n)
  }
  loop(n, 1)
}
```



## 控制抽象

* 值调用：调用方法的时候传入值
* 名调用：调用方法的时候将代码传过去

```scala
// 值作为形参
def f0(a: Int) {
  println(a)
}

def f1():Int = {
  println("1")
  12
}

f0(23) // 值调用
f0(f1()) // 值调用

// 代码块作为形参, =>Int 表示代码块的返回值为 Int
def f2(a: =>Int):Unit = {
  println(a) // a 代码块会执行!!!，代码块会有返回值
}

f2(23) // 23 也是代码块
f2(f1()) // f1() 也是代码块。因为一行，所以 {} 省略掉了
f2({
  println("hello")
  29
})

// 实参是代码块的话，小括号是可以省略的
f2{
  println("hello")
  29
}

```





## 惰性函数

* 尽可能延迟表达式求值。惰性集合在需要时提供其元素，无需预先计算他们。
* 优点：可以将耗时的计算推迟到绝对需要的时候。
* scala对惰性计算提供了原生的支持。
* 当函数的返回值声明为 lazy 时，就成了惰性函数，在真正取值的时候才会被调用
* 细节：
  * lazy 只能修饰 val
  * 

```scala
lazy val i = 100 // 这个变量值也是在使用的时候才会真正的分配
lazy val res = func() //这时候并没有实际调用
println(res) // 这时候才会真正调用！但是只会调用一次。
println(res) // 这里不会再调用 func 了。
```



## 偏函数 partial function

* 应用场景: 
  * 对于符合某个条件进行操作, 对不符合条件的不进行操作, (包在 大括号里的一堆 case)
  * 多个偏函数组合在一起可以构成一个完整的操作
```scala
val list = List(1, 2, 3, "hello")
// Any 表示输入是 Any 类型, 返回是 Int 类型。两个泛型参数，第一个输入类型，第二个是返回值类型
val parFunc = new PartialFunction[Any, Int] {
    // 如果返回 true, 交给 apply 处理, 如果返回 false, 则不处理
    override def isDefinedAt(x: Any) = x.isInstanceOf[Int]
    override def apply(v1: Any) = {
        v1.asInstanceOf[Int] + 1
    }
}

val parFunc2: PartialFunction[List[Int], Int] = {
    case x :: y :: _ => y
}

// 这里不要使用 map, 使用 collect 哦
list.collect(parFunc)

// PartialFunction 的简写形式
def f2:PartialFunction[Any, Int] = {
    case i: Int => i + 1
}

// 更简写形式........
list.collect{case i: Int => i + 1}




val list = List(("a", 2), ("b", 4))
list.map(tuple => (tuple_1, tuple_2 * 2))

list.map(tuple => {
    typle match {
        case (word, count) => (word, count * 2)
    }
})


// 省略 lambda 表达式的写法, 得到一个偏函数的写法

list.map{
    case (word, count) => (word, count * 2)
}




// 偏函数 demo，求绝对值

val posAbs: PartialFunction[Int, Int] = {
    case x if x > 0 => x
}

val negAbs: PartialFunction[Int, Int] = {
    case x if x < 0 => -1
}

val zeroAbs: PartialFunction[Int, Int] = {
    case 0 => 0
}

// 多个偏函数构建称一个整体的函数
def abs(x: Int): Int = (posAbs orElse negAbs orElse zeroAbs) (x)

```

## 作为参数的函数

```scala
// _ 表示, 集合遍历出来的 每一个元素
list.map(func(_))
```

## 匿名函数

```scala
// 函数表达式, (x: Double) => 3*x  , 说明是匿名函数, => 指示了匿名函数, = 是 函数使用的. 不需要返回类型!!
// 函数 赋值个 一个变量
val triple = (x: Double) => 3*x

```
* 匿名函数参数类型推断
    * 参数类型可以推断时, 可以省略参数类型
    * 当传入的函数, 只有单个参数时, 可以省去括号
    * 如果变量只在 => 右边出现一次, 可以用 _ 来代替

```
list.map((x: Int)=> x+1)
list.map(x => x+1)
list.map(_ + 1)

list.reduce((n1: Int, n2: Int) => n1 + n2)
list.reduce((n1, n2)=> n1 + n2)
list.reduce(_ + _)
```

## 高阶函数
* 接收函数,  也可以返回一个函数

```scala

def test(f: Double => Double) = {

}

// 返回匿名函数, 这里还有一个闭包
def test2(x: Int) = {
    (y : Int) => x - y
}

```





# 异常处理

* `throw new Exception`
* `throw` 抛出异常，抛出的是一个 `Nothing` 类型, 可以看做 return 语句了。

```scala
try{
    
}catch{
    // => 关键字： 表示后面是 异常处理代码。
    case ex: ValueError => {}
    case ex: Exception => {}
}finally{
    // 不管有没有捕获异常，都会执行 finally
} 

def test(): Nothing = {
    throw new Exception("exception occured")
}

```

* throw comment

```scala
@throws(classOf[NumberFormatException])
def f11(a: String){
    a.toInt
}
```



# 面向对象基础

基本语法：

* 包：package 包名：1）区分相同的名字，2）用来管理类，3）控制访问范围
  * 命名规范：com.公司名.项目名.业务模块名
* 两种包管理方式. 包是一个逻辑上的管理结构。和物理文件目录没啥太大关系
  * 一种是 java 方式
  * 另一种 scala 方式: 源文件中直接指明包结构
    * 一个文件可以写多个包
    * 子包里的类可以 **直接** 引用父包声明的东西。外层**不能直接**访问内层，需要显式导入包

```scala
package com {
  package company {
    package demo {
      
    }
  }
}
```

* 包对象：给一个包定义一个同名的对象，作为该包下的所有 class 和 object 的共享变量

```scala
// 包对象的声明需要和包放在同一级下
package object packageName {
  
  val name = "hello"
  def func() {
    println("what")
  }
}

// 如何用，在相同包的 源代码中，可以直接访问！
```



* 导入包说明
  * 源文件开头导入
  * 任意位置导入（函数体中，类中。。。）
  * `import java.util._` 通配符导入
  * `import java.util.{ArrayList => AL}` : 起别名
  * `import java.util.{HashSet, ArrayList}` : 引入多个
  * `import java.util.{ArrayList =>_, _}` 将 ArrayList 屏蔽



```scala
object Demo{
    def main(args: Array[String]){
        val cat = new Cat
        // 这里实际调用的是 name_$eq 方法来设置值的。
        cat.name = "hello"
        println(cat.name) //这里实际上调用的是 cat.name()
    }
}

// 类默认是 public
class Cat {
    // 默认是 private
    // 同时会生成两个 public 方法 name()负责getter, name_$eq()，负责 setter
    var name: String = "name" //一定是要给初始值的
    
  	// scala 会默认生成 getter setter 方法。这些不符合 java bean 的规范
  	var age: Int = _ // _ 表示默认值
  
  	// scala 不会生成 getter setter 方法，需要我们手动实现
  	private var career = _
  	
  	@BeanProperty // 生成符合 javaBean 规范的 getter setter
  	var height: Double = _
}
```

* 基本语法 `[修饰符] class 类名`，修饰符默认 `public`, 一个源文件只能有一个 public 类，且这个名字需要和文件名字一致
* 属性定义：`[修饰符] val name: Type = DefaultValue`
  * 必须显示给初始值
* scala 一个文件可以包含多个类， 默认都是 public 的

## 访问权限

* 类默认 `public`。属性：默认为 `public`, 是通过 scala 生成 getter 和 setter 方法来实现的
* private: 只有在类中和伴生对象中才可以使用
* Protected : 只有子类可以访问
* private[packageName]: (属性的访问权限) 包名下的其他类可以使用。其余包不能使用



```scala
class Person{
  private var id: Int = _
  protected var name: String = _
  var sex: String = _
  private[packageName] var age: Int = _
  
  
}
```







## 方法

* 方法和函数一致，在 class 里面是方法，在 object 里面是函数？

## 构造器

* scala 构造器包括 `主构造器` 和 `辅助构造器`

```scala
class Name(形参列表) { // 主构造器
    
  	def this(){      // 辅助构造器,必须只能是 this。辅助构造器只能直接或者间接的调用 主构造器
        
    }
    def this(){      // 辅助构造器
        
    }
}


// 这样 主构造器就私有化了, inName 是局部变量. 如果用 val 修饰一下, otherName 就是个只读的私有属性了. 如果用 var 修饰, 就变成了可读写属性!
class Person private (inName: String, inAge: Int, val otherName: String, var ootherName: String) {
    
  	// 这部分底层实际上是包装到 一个构造函数里的（主构造器）
    var name: String = inName
    var age: Int = inAge
    
    age += 10
    // 这样辅助构造器就私有化了
    private def this(name: String){
        this(name, 10)// 第一行一定要调用主构造器！！！！
        
    }
    
    override def toString: String = {
        
    }
    // 这个地方同样被搞进了 一个构造函数里
    name += "aa"
}
```

* 细节：
  * 主构造器：**实际上是将 除 函数的语句 都包装到一个 构造器里。**
  * 辅助构造器：第一行一定要调用主构造器（直接或者间接）
  * BeanProperty: 对于属性 @BeanProperty 就自动生成了其 setXXX 和 getXXX 方法. 原来自动生成的方法也可以使用!

* 主构造器的形参：三种类型
  * 无任何修饰符，这个参数就是一个局部变量
  * var 修饰，作为类的成员属性使用， 可读可写
  * val 修饰，作为类的成员属性使用，可读不可写



## 继承 & 多态

```scala
// 先父类构造器，再子类构造器
class Father(var name: String, var age: String) {
  
}

// extends 后面指明 父类的哪个构造器被调用。
class SubClass(name: String, age: String) extends FatherClass(name, age) {
  
}
```



## 抽象类

```scala
// 不能基于抽象类 构建对象
abstract class Person {
  val name: String // 可以不指定初始值。也就只有 抽象类里面可以这么写了
  def hello(): Unit // 没有函数体
}
```

* 重写非抽象方法需要用 `override`， 重写抽象方法无需 `override`
* 子类调用父类的方法，使用 `super` 关键字。 `super.??`
* 父类属性如果是 `val` 子类可以 `override` 之。父类属性如果是`var`，子类直接重新赋值就可以了。



## 匿名子类

```scala
abstract class Person {
	var name: String
  def hello(): Unit
}


val person = new Person{
  override var name: String = _
  override def hello(){
  	println(name)
  }
}
```



## 单例对象 & 伴生对象

> 里面放的是 静态成员
>
> 伴生类，伴生对象 之间的属性方法 是可以互相调用的。
>
> 伴生类，伴生对象，需要在同一个文件里。名字要完全一致

伴生对象一个特殊的方法: apply

```scala
object Student {
  
  def apply(){} // 通过 Student() 调用的就是这个 apply 方法
}
```



## Trait （类似 java接口interface）

> 抽象方法，抽象属性，具体方法，具体属性。都可以有。。。。 和抽象类不就重复了？？？？？

* 没有父类：`class ClassName extends Trait1 with Trait2 with Trait3... {}`
* 有父类: `class ClassName extends FatherClass with Trait1 with Trait2 ...{}`

```scala

trait Talent {
  
}

class Student{
  
}

// 创建对象的时候才 搞出来一个 特质， trait 的动态汇入 mixup. 
// 如果继承的时候发生了冲突，子类需要重写。重写的时候可能用 super, 方法叠加是从后向前
// super. ..... super[Talent].  ......
val studentWithTalent = new Student with Talent {
  
}

```



## 特征的自身类型（self type）

> 等价于 类的 继承。 trait 的继承叫 trait self type

```scala
class User(val name: String, val pwd: String)
trait UserDao {
  
  _: User =>  // 写了这句之后，UserDao 就能假设自己有一个 User 对象？
  
  def insert(){
    println(this.name) // 可以使用 this 调用 User 的属性了
  }
}


class RegisterUser(name: String, pwd: String) extends User(name, pwd) with UserDao

val ru = new RegisterUser("a", "b")
ru.insert

```



## 类型检查与转换

```scala
a.isInstanceOf[T]
b.asInstanceOf[T] // 
classOf(obj) // 获取对象的类名
```



## 枚举类与应用类

* 枚举类：需要继承 `Enumeration`
* 应用类：需要继承 `App`

```scala
object Color extends Enumeration {
    val RED = Value(1, "red")
    val YELLOW = Value(2, "yellow")
}

object Test {
    
    def main(args: Array[String]): Unit = {
        
        println(Color.YELLOW)
        
    }
    
}

```



```scala
// 只能用 object 来继承
object TestApp extends APP {
    println("这里就可以直接执行了，不用定义main，因为 APP 已经帮我们处理好了")
}
```






## 打包 & import
* 子包可以直接使用父包的东西
* 父包使用子包的, 必须要引入
* anywhere can import

```scala
package aa.bb.cc

// 与上面的等价
package aa.bb
package cc

// 一个文件中可以创建多个包, 每个包里面都可以写 class, trait, object
package aa{
    package bb {
        package cc {
        
        }
    }
}
```
* 解决java 中不能在 类外 定义对象的限制: 使用包对象来解决
```scala

// 每一个包都可以有一个包对象
// 包对象的名字需要和子包的名字一致
// 在包对象中可以定义变量和方法
// 包对象中定义的 东西, 可以直接在包中使用了
// 包对象只能在 父包中定义, 和包是同级的!!!
// package object scala{}  创建包对象,里买可以搞事情了.
package object scala {
    
}

// 打包
pakcage scala {
    
}

import scala.collection.mutable._
// 重命名 与 隐藏掉
import scala.collecton.mutable.{HashTable=>JavaHashTable, List=>_, Other}

```

## 隐式类

* 动态给一个 类添加方法

```scala
// 动态给 string 添加方法
implicit class People(s: String) {
    def checkEq(ss: String)(f: (String, String)=> Boolean): Boolean = {
        
    }
}
```



## sealed 密封类

**当前类** 的 **子类** 是不能定义在当前 源文件 之外的！！！！



# 伴生类与伴生对象

```scala
// for non-static
// default private
// no public key-word
class Clerk {
    // default private
    var name: String = _
}

// can access private property of the Cleak class
// for static 
object Clerk{

}
```


# scala 集合包

* 同时支持 **可变集合** 和 **不可变集合** ， **不可变集合可以安全的并发访问**
  * 不可变集合：集合本身不可变， 元素个数不可变，但是 **存储的值可以变** 的。
  * 可变集合：可动态增长
* 两个主要的包
  * `scala.collection.immutable`
  * `scala.collection.mutable`
* scala 默认采用不可变集合（几乎所有）。
* 集合有三大类型： `Seq, Set, Map`

## Array

```scala
// 定长数组, scala中小括号进行索引， [泛型]。对应的可变数组叫 ArrayBuffer
val arr = new Array[Int](10)
for (item <- arr) {
    println(item)
}
// 只能改查
// 修改, arr(3) ： 实际调用的是 类中定义的 apply 方法
// arr(3) = 10 实际调用的类中定义的 update 方法
arr(3) = 10

// 查
for (element <- arr) {
    println(element)
}

val iterator = arr.iterator
while (iterator.hasNext) {
    println(iterator.next)
}


arr.foreach(println)


// 第二种方式 定义数组，直接初始化数组，数组的类型和初始化的类型的是有关系的
// 这里其实使用的 object Array。使用伴生对象创建 Array
var arr02 = Array("1", 1)
for (index <- 0 until arr02.length) {
    arr02(index)
}

// 创建一个新的数组，将当前的ele copy 过去，然后将新数组返回。想添加元素，必须要创建要给新的数组
val arr2 = arr.:+(10) // 10加到数组的后面去
val arr3 = arr.+:(30) // 30加到数组的前面去

arr :+ 10    // 带冒号的运算符，冒号距离 操作的对象比较近
10 +: arr
```



```scala
// 变长数组
val arr = ArrayBuffer[Int]()
arr.append(7) //:+ 主要是给不可变数组使用的。
arr += 19 // 屁股后面添加元素， 对应 append
19 +=: arr // 前面加。  重点这个冒号.  对应 insert


arr.append(8)
// 可变参数
arr.append(7,9,0) 
arr(0) = 10
arr.remove(idx)

// 到 定长 和 变长 之间的转换
arr.toArray.toBuffer 


// 删除元素
remove
arr -= 13 // 找到数值为 13 的值，将其删除
```





## 多维数组

```scala
// 创建
val arr = Array.ofDim[Double](3,4)
arr[1][1] = 10
for (item <- arr) {
    for (item2 <- item) {
        
    }
}

for (i <- 0 until arr.length; j <- 0 until arr(i).length) {
    
    println(arr(i)(j))
    
}


for (i <- arr.indices; j <- arr(i).indices) {
    println(arr(i)(j))
}

```



## 元组

```scala
// 元祖最多能放 22 个元素
// 类型 是 Tuple4
val tuple1 = (1, 2, 3, 4, "HELLO")

// 访问 元祖的第一个元素, 有两种方式
tuple1._1 
tuple1.productElement(0)

// 遍历, 需要使用到迭代器
for (item <- tuple1.productIterator) {
    
}

// 二元组的特殊写法
“a” -> 2
```



## List

* `List` 只有 不可变的, 可变的是 `ListBuffer`
* `Nil` 一个空集合

```scala
val list = List(1, 2, 3)
val nullList = Nil // 空集合

// 访问
list(1) // list(1) = 10, 这个操作是不行的。因为没有 update 方法

// 遍历
for (item <- list) {
    
}

// 追加元素， 原 list 没有变， 只是返回一个新的 list
// 注意 + 的位置， 靠近 加的值
val list2 = 4 +: list
val list3 = list2 :+ 5

// Nil 一定要放到最右边， 最右边的一定是一个集合。 冒号结尾的运算符，调用顺序是 从右到左
// 得到的结果是 从左到又 依次放到 Nil 集合中去.   :: 实际是一个 List 的子类
val list5 = 1 :: 2 :: 3 :: list3 :: Nil
// ::: list 内容展开然后放到 集合中去 ！！！   冒号结尾的运算符，调用顺序是从右向左
val list6 = 1 :: 2 :: 3 :: list3 ::: Nil

// ++ 也是两个list合并到一起！！
list1 ++ list2
```

* ListBuffer: 可变的 list 集合

```scala
var lb = ListBuffer[Int](1,2,3)
// 访问
lb(2)
lb(2) = 100

// 遍历
for (item <- lb) {
    
}

// 添加
lb += 10
lb.append(11)
lb ++= lb2 // 一个集合加到另外一个集合中去， 元素级别相加。lb2添加到 lb中
lb ++=: lb2 //lb 添加到 lb2中

val lst2 = lst1 ++ lst3 // 同上
val lst5 = lst :+ 5 // lst 不变
val nullLb = new ListBuffer[Int]

// 删除
list.remove(10)
```



## 队列 Queue

* 先入先出

```scala
val q = new mutable.Queue[Int]

// 增加， 默认是增加到 队列的屁股后面的。
q += 1

q ++= list // 将 list 中的元素批量添加
q += list // 将 list 整体加到 queue 中去

// 入队列 与 出队列
q.dequeue()
q.enqueue(1,2,3)

// 队列的 头 和 尾
q.head // 第一个元素
q.last // 最后一个元素
q.tail // 返回除了第一个元素以外的所有元素 (返回的是一个队列)
t1.tail.tail // 



// Immutable Queue
val queue2 = Queue("a", "b", "c") // 入队，出队操作会产生新的 Queue 对象
```



## Map

* 存的是 key-value
* 常用的是可变的版本
* 不可变的版本 是有序的。
* map 的底层是  Tuple2 类型

```scala
val iMap = immutable.Map("a" -> 1, "b" -> 2)
iMap.foreach(println)

iMap.foreach((kv:(String, Int)) => println(kv))

for (key <- iMap.keys) {
    
}

for (val <- iMap.values) {
    
}


val map = mutable.Map("a"->1, "b"->2)
val map2 = new mutable.Map[String, Int]
val map3 = mutable.Map(("A", 1), ("B", 2))
// 访问, 
val val1 = map2("a") // 如果不在里面， 会抛异常
map2.contains("a") // 判断key 是否存在
map2.get("a").get // 如果key存在 map2.get("a") 返回的是一个 Some， 这时候 再 get 一次即可， 如果key不存在，map2.get("a") 返回的是 None
map2.getOrElse("a", default_value) // 有则返回， 否则 返回默认值
// 遍历
for ((k, v) <- map2) {
    
}

for (k <- map2.keys) {
    
}

for (v <- map2.values) {
    
}


for (kv <- map2) {
	// kv 是 Tuple2    
}

// 增加, 存在就更新， 不存在就添加， 如果是 immutable,Map， 值都不让改！！！
map2("AA") = 20
map2 += ("D"->4)
map2 += ("D"->4, "E"->5)
// 删除, 直接写 key 就可以了， key不存在 也不会报错
map2 -= ("D", "E")

map1 ++= map2
map1 ++ map2

```



## Set

```scala
val set = Set(1, 2, 3)

// set 因为不用考虑 末尾加，还是开头加 所以没有 : 引入
set + 4 // 加一个元素
set1 ++ set2 // 合并两个集合
set1 - 13 //删除一个元素
set1 -- set2 //从一个集合删除另一个集合

val mutableSet = mutable.Set(1, 2, 3)

// 添加
mutableSet += 4
mutableSet += (4)

mutableSet ++= mutableSet2

// 删除
mutableSet.remove(2)
mutableSet -= 4

// 遍历
```



## 并行集合

```scala
// Thread.currentThread.getName, getId

// par表示是并行计算
(1 to 100).par.map()
```





# 集合操作

* 长度或者大小
  * `size, length`
* 迭代
  * `for (ele <- list),   for(ele <-list.iterator)`
* 转成 string
  * `mkString`
* 包含
  * `contains`
* `.head, .tail， .last, init`: 不是头，剩下的就是尾！！ 
* 反转 ： `reverse`
* `take(3), takeRight(3)`
* `drop`
* `union(并集), intersect（交集）, diff（差集）`
* `zip!` 
* 滑窗 `list.sliding(3), list.window`
* 计算
  * `list.sum, list.product, list.max, list.min`
  * `list.sorted, list.sorted.reverse, list.sorted(Ordering[Int].reverse)`
* 高级函数
  * `filter`: true 的会保留
  * `map`
  * `flatten` : 如果 list 的单元素是个 list，就会将内部的 list 进行展平
  * `flatMap`： 实际上是先 map操作，我们实现的只需要是 map 操作，map 的结果进行 flat
  * `group` : 之后得到的是一个 Map。相同组放在相同的key
  * `reduce, reduceRight, reduceLeft`:   （不需要传初始聚合状态）
  * `fold`: 需要传一个初始聚合状态
* map 操作

```scala
// 高阶函数
def test(f: Double => Double, ni: Double) {
    f(n1)
}

// 无参的高阶函数
def teset2(f: ()=>Unit) {
    
}

def myPrint() {
    println("hello world")
}

// 为什么要有 _ 这个神奇的操作， 因为 scala 中对于 无参的函数，可以不加 () 直接调用，所以如果想将一个 函数赋值一个变量的话，那就需要 后面显式的加上一个 _ 告诉编译器，不要计算函数的值。
val f1 = myPrint _
```

* `flatMap`: 如果遍历的元素是个集合， 那就继续展开

* `reudceLeft, reduceRight` 从左开始算， 从右开始计算
*  `foldLeft, foldRight`: 

```scala
val list3 = List(1, 2, 3, 4)
// 等价于 list 3 左边 加上一个元素 5， 然后执行 reduce
val l4 = list3.foldLeft(5)(_ - _)
// 等价于 list3 右边 加上衣蛾元素 5， 然后执行 reduce
val l5 = list3.foldRight(5)(_ - _)

val l7 = (1 /: list3)(_ - _) // 等价于 list3.foldLeft(1)(_ - _)
val l8 = (list3 :\ 1)(_ - _) // 等价于 list3.foldRight(1)(_ - _)

```

* `scanLeft, scanRight` : 保存中间结果 的 `fold...`

```scala

```

* `zip`
  * 两个list个数不一致，则会导致数据丢失

```scala
val list1 = List(1, 2, 3)
val list2 = List(2, 3, 4)
val list3 = list1.zip(list2) // [(1, 2), (2,3)] 出来的是 Tuple2
```

* 迭代器, `list.iterator()`
  * `hasNext(), next()` 
  * 迭代器可以直接放到 for loop 中去

* `Stream`
* `.view` 方法 : 懒加载机制！！

```scala
// 这时候时候并没有执行 filter
val viewdemo = (1 until 10).view.filter(a => a%2 == 0)
println(viewdemo) // 这时候也没有调用
for (item <- videwdemo) {
    // 遍历的时候才会真正的调用。
}
```



```scala
// map 合并
map1.foldLeft(map2) (
(mergedMap, kv) => {
    val k = kv._1
    val v = kv._2
    mergedMap(key) = mergedMap.getOrElse(0) + value
    mergedMap
})
```





# 操作符重载

```scala
class Dog {
    def -(i: Int) {
    
    }
    // dog++, 后置操作符重载
    def ++() {
        
    }
    // !dog   unary 一定要加上, 表示是 前置运算符
    def unary_!() {
    
    }
}

// 中置操作符: a op b  等价于 a.op(b)
```



# 模式匹配

* match case 模式匹配, 两个都是关键字
* 如果都没有匹配上, 就会匹配 `case _`
* 如果没有匹配上, 且没有 `case _` 那就回抛出异常 `MatchError` !!!
* 即使多行代码，花括号也是可用，可不用的

```scala

variable match {
    case '+' => res = n1 + n2 // 默认是 break 的
    case '-' => {
        res = n1 - n2
    }
    case _ => res = n1
}

```
* match 中的守卫
```scala
// _ 只是用来接收当前的值
variable match {
    case i if (i < 10) => do something
    case _ => do something
    
}

varable match {
    // 意味着 mychar varable, 这个一直是会匹配上的. 这个玩意等价于 case _ => do something
    case mychar => do something
}

// 可以有返回值哦
val res = vari match {

}

```
* 类型匹配

```scala
obj match {
    // obj 给 a, 然后看 a 是不是 Int ? 
    case a: Int => a
    case b: Map[String, Int] => b
    // 如果 _ 出现的不是最后一个, 表示的隐藏变量名, 而不是默认匹配
    case _: Double => "double"
    case _ => "nothing"
}
```

* 数组匹配

```scala

arr match {
    // 匹配只有一个元素, 且第一个元素为 0 的数组
    case Array(0) => '0'
    // 匹配有两个元素的数组, 并将两个元素 赋值给 x, y
    case Array(x,y) => x + "=" + y
    // 匹配 以 0 开头的数组
    case Array(0, _*) => do something
    case Array(1, x, y) => do something
    
    case _ => ""
}

```

* 列表匹配

```scala
list match {
    // 只有 0 的
    case 0 :: Nil => '0'
    
    case List(0) => "只有一个0的"
    case List(1, x) => "两个元素，第一个是 1"
    case List(0, _*) => "N个元素，第一个是0"
    
    case x::y::Nil => x + " " + y
    // 0 打头的
    case 0::tail => "0..."
    case _ => "nothing"

}
```

* 匹配元祖
```scala
tuple match {
    // 第一个元素是 0 的二元组
    case (0, _) => 
    // 第二个元素是 0 的二元组
    case (y, 0) =>
    
}

val (x, y) = (10, "hello")  // 元组的匹配。用在变量声明上
val List(first, second, rest) = List(1, 2, 3)
val List(first, second, _*) = List(1, 2, 3, 4, 5, 6)
val fir :: sec :: rest = List(1, 2, 3, 4, 5)



val tupleList = List((1, 2), (2, 3))
for ((a, b) <- tupleList) {
    
}

// 有点像循环守卫
for ((1, _) <- tupleList) {
    
}
```

* 
* 对象匹配
> 如果 对象的 `unapply` 返回 Some集合 则认为匹配成功
>
> apply: 构建对象
>
> unapply: 解构对象
```scala
object Square {
    // 对象提取器: 
    def unapply(z: Double): Option[Double] = Some(math.sqrt(z))
    // 构造器
    def apply(z: double): Double = z*z
}

val number: Double = 36.0
number match {
    // 会调用 Square 的 unapply 方法. 传入 unapply 的值是 number, 如果返回了 Some 就认为是匹配成功,  Some里面的值就赋值给了 n
    case Square(n) => print(n)
}

```
* for 循环中的模式匹配

```scala
for ( (k, 0) <- map) {
    // 第二个元素是 0 的才会进去
}
```



# 样例类

```scala
abstract class Amount

// 会自动生成很多方法, 这玩意看起来就像是一个 脚手架, 构造器中的参数 默认是 val
// 样例类对应的 伴生对象 也提供了 apply, unapply 方法. 会自动生成一堆方法.
case class Dollar(value: Double) extends Amount
```
* 样例类：一个类被声明成了样例类
  * 编译器会自动生成其伴生对象（自动生成伴生对象的 apply, unapply 方法自动生成）
  * 样例类声明时的 构造器的参数，默认为 val
* 



# 泛型

* 又称 类型构造器！
* 逆变，协变，不变，是需要 类型构造器的编写者 指定的！！！

```scala
class ClassName[+T]; // 协变  如果 Apple 是 Fruit 的子类，那么 ClassName[Apple] 是 ClassName[Fruit] 的子类
class ClassName[-T]; // 逆变  如果 Apple 是 Fruit 的子类，那么 ClassName[Apple] 是 ClassName[Fruit] 的父类
class ClassName[T];  //不变   如果 Apple 是 Fruit 的子类，那么 ClassName[Apple] 是 ClassName[Fruit] 没有关系
```



* 泛型的上下限

```scala
class ClassBuilder[T<:Person] // 泛型上限, 
class ClassBuilder[T>:Person] // 泛型下限
```

