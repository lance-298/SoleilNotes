#### 1 什么是内联函数？

内联函数 使用 inline 修饰，空间换时间，比正常函数少了压栈和出栈的操作，当函数体少，以及被频繁调用的函数才适合被定义为内联函数。简单来说，就是把调用函数替换成函数里面的内容。

Kotlin 内联函数详解
* 一、内联函数定义‌

    * 内联函数通过 inline 关键字声明，‌编译时直接将函数体代码插入调用处‌，消除传统函数调用的栈帧开销，并优化 Lambda 表达式性能。其核心机制类似于 C 语言的宏替换，但具备 Kotlin 的类型安全特性。

* 二、核心优点‌

    * 消除高阶函数开销‌
      * 非内联高阶函数的 Lambda 参数会生成匿名类对象（如 Function0、Function1），内联后直接嵌入代码，避免对象创建的内存开销。
      * 示例：标准库的 let、map 等函数均依赖内联优化性能。

    * 支持非局部返回（Non-local Return）‌
      * 普通 Lambda 中 return 仅退出 Lambda 自身，内联后可直接从外层函数返回。
      * ```kotlin
        inline fun findUser(users: List<User>, block: (User) -> Boolean): User? {
            for (user in users) {
            if (block(user)) return user // 直接退出外层函数
            }
            return null
            }  
      * ```

    * 编译时优化‌
      * 内联后的代码支持常量折叠、循环展开等编译器优化，提升运行时效率。
      
* 三、主要缺点‌

    * 代码膨胀‌
      * 多次调用大型内联函数会导致字节码冗余，增大 APK 体积。
      * 适用原则：‌仅内联小型高频函数‌（如工具函数、集合操作）。

    * 编译时间增加‌
      * 内联展开会增加编译期工作量，影响大型项目的构建速度。
  
    * 使用限制‌
      * 不可递归‌：内联函数无法直接或间接调用自身。
      * 部分参数禁用内联‌：可通过 noinline 标记不需要内联的 Lambda 参数。
      
* 四、适用场景‌
场景‌	‌说明‌	‌示例‌
高频调用的高阶函数‌	接收 Lambda 参数的函数通过内联避免对象创建开销	repeat、let、apply 等扩展函数
性能敏感代码‌	需要极致优化的关键路径代码（如渲染循环、密集计算模块）	数学计算工具类、数据解析逻辑
工具类函数‌	小型且被多处调用的工具方法（内联后代码量可控）	类型转换、日志工具函数

* 五、应避免场景‌
  * 大函数内联‌
    * 函数体超过 3-5 行时，内联可能导致代码膨胀明显。
  * 递归逻辑‌
      * 内联函数无法处理递归调用场景。
  * 低频调用函数‌
      * 非高频调用时，内联的优化收益小于代码膨胀代价。
    
* 六、最佳实践‌
优先内联小型高阶函数‌（如接收单个 Lambda 的工具函数）
组合使用 inline 与 noinline‌
```kotlin
inline fun requestData(
crossinline onSuccess: (Data) -> Unit,
noinline onError: (Exception) -> Unit // 不内联的错误处理回调
) { /* ... */ }
```


监控字节码变化‌
通过 Android Studio 的 ‌Kotlin Bytecode‌ 工具分析内联后的实际影响。

通过合理使用内联函数，可在保证代码简洁性的同时显著提升性能，但需严格权衡代码体积与执行效率。


* 引申：标准库中的高阶函数
Kotlin 标准库中以下常用扩展函数和工具函数默认通过 inline 关键字实现内联优化：

‌作用域函数‌
let
run
with
apply
also
原理：这些函数接收 T.() -> R 或 (T) -> R 类型的 Lambda 参数，通过内联避免创建临时 Function 对象。

‌集合操作函数‌
map
filter
forEach
any/all
原理：避免处理集合时产生中间对象，提升性能。

‌工具函数‌
repeat（循环执行操作）
synchronized（同步代码块）
原理：通过代码内联减少函数调用开销，实现高效执行18。

#### 2 apply、run、let、also、with 之间的区别？

with、T.run、T.apply 接收者是 this，T.let、T.also 接收者是 it。

with、T.run、T.let 返回值是作用域的最后一个对象（this），T.apply、T.also 返回值是调用者本身(itself)。


* 作用域函数
kotlin标准库包含几个函数，唯一的目的是在对象的上下文中执行代码块。
当对一个对象调用这样的函数并提供一个lambda表达式时，它会形成一个临时作用域。
在此作用域中，您可以调用该对象的方法而无需任何额外的限定符。

共有以下五种：let run with apply also

以下是根据预期目的选择作用域函数的简短指南：
对一个非空（non-null）对象执行 lambda 表达式：let
将表达式作为变量引入为局部作用域中：let
对象配置：apply
对象配置并且计算结果：run
在需要表达式的地方运行语句：非扩展的 run
附加效果：also
一个对象的一组函数调用：with

函数	对象引用	返回值	是否是扩展函数
let	it	Lambda 表达式结果	是
run	this	Lambda 表达式结果	是
run	-	Lambda 表达式结果	不是：调用无需上下文对象
with	this	Lambda 表达式结果	不是：把上下文对象当做参数
apply	this	上下文对象	是
also	it	上下文对象	是

#### 3 数据类的使用，data class 会继承什么？

默认生成下面：

* equals() / hashCode()
* toString() 
* componentN()
* copy()

#### 4  "==" 和 "===" 区别？

== 比较的是数值是否相等, 而 === 比较的是两个对象的地址是否相等。



#### 5 kotlin 中 var、val、const val 区别？

var 定义变量 private，带有 public 的 set 和 get 属性。

val 定义常量 private，带有 public 的 get 方法，可见性为 private final static，并且 val 会生成方法getNormalObject()，通过方法调用访问。

const val 定义的常量，可见性为 public final static，可以直接访问。

#### 6 介绍一下伴生对象和静态成员？

```kotlin
class NewFragment {
    companion object {
       val instance = NewFragment()
    }
}
//类似于 Java 中使用类访问静态成员的语法
```


* 一、伴生对象核心特性‌

定义方式‌:
使用 companion object 在类内部声明，一个类最多只能有一个伴生对象。
```kotlin
class MyClass {
companion object {
const val PI = 3.14
fun printMsg() = println("Hello")
}
}
```

功能定位‌:
替代 Java 的静态成员（属性和方法）
可访问外部类的私有成员（如私有构造函数）
支持扩展函数和继承

调用语法‌
通过类名直接访问伴生对象成员，无需实例化：
```kotlin
MyClass.PI         // 访问属性
MyClass.printMsg() // 调用方法
```

* 二、与Java静态成员的对比‌
特性‌	‌Kotlin 伴生对象‌	‌Java 静态成员‌
声明方式‌	companion object 内部定义	static 关键字修饰
访问控制‌	可访问外部类私有成员	仅能访问类公开成员
单例性‌	默认单例（整个类共享）	与类绑定，全局唯一
扩展支持‌	支持扩展函数	不支持


* 三、典型使用场景‌
工厂模式‌
通过伴生对象封装对象创建逻辑：
```koltin
class User private constructor(val name: String) {
companion object {
fun create(name: String) = User(name.trim())
}
}
```

常量定义‌
使用 const val 声明编译时常量：
```koltin
companion object {
const val MAX_COUNT = 100
}
```

* 工具方法‌
提供类相关的静态工具方法：
```koltin
companion object {
fun formatDate(time: Long): String { ... }
}
```


* 四、进阶特性‌
命名伴生对象‌
可为伴生对象指定名称（默认名称为 Companion）：
```koltin
companion object Parser {
fun parse(json: String) = { }
}
// 调用：MyClass.Parser.parse(...)
```



JVM 注解支持‌
@JvmStatic：将伴生对象方法暴露为真正的 Java 静态方法
@JvmField：将属性暴露为 Java 静态字段


* 五、注意事项‌
性能差异‌：伴生对象成员在字节码中实际是通过内部类实现的，与 Java 静态成员底层机制不同
线程安全‌：伴生对象初始化是线程安全的（首次访问时初始化）
避免滥用‌：大型伴生对象可能导致类职责不清晰，建议拆分逻辑

通过伴生对象，Kotlin 在保留面向对象特性的同时，提供了比 Java 静态成员更灵活的设计模式支持。



#### 7 @JvmField 和 @JvmStatic 的使用

```kotlin
class NumberTest {
    companion object {
        @JvmField //修饰属性
        var flag = false

        @JvmStatic //修饰方法
        fun plus(num1: Int, num2: Int): Int {
            return num1 + num2
        }
    }
}

 public static void main(String[] args) {
      System.out.println(NumberTest.plus(2, 3));
      NumberTest.flag = true;
      System.out.println(NumberTest.flag);
}
```

#### 8  @JvmOverloads 的作用？

为了暴露多个重载方法。

```java
fun f(a: String, b: Int = 0, c: String="abc"){}
//相当于 java 中
void f(String a, int b, String c){}

@JvmOverloads 
fun f(a: String, b: Int=0, c:String="abc"){}
//相当于 java 中
void f(String a)
void f(String a, int b)
void f(String a, int b, String c)
```

#### 9 List 与 MutableList 区别？

List ：只能读，不能更改元素；

MutableList ：可读写，返回的是一个 ArrayList；

#### 10 Kotlin 中的数据类型有隐式转换吗？

没有，需要显式的转换。

#### 11 Kotlin中 Unit 类型的作用以及与Java中 Void 的区别？

* Java中必须指定返回类型，void 不能省略，但是在 kotlin 中，如果返回为 unit，可以省略。
* Java中 void 为一个关键字，但是在 kotlin 中 Void 是一个类。