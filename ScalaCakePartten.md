# Scala 自类型与蛋糕模式
## 应用场景
--- 
现在有特质A和B，特质A上面实现了A1、A2,特质B上面实现了B1、B2,具体情景如下：
```
trait A {
  val text = "hello"
  def func1(): Unit = println(text)
}

trait B {
  val adress = "home"
  def func2(text: String): String = adress
}

class A1 extends A {
  override val text = "Hello~ My Lady~"
}

class A2 extends A {
  ???  //省略不写，只做演示
}

class B1 extends B {
  override val adress = "office"
  override def func2(text: String): String = s"${text} welcome to ${adress}"
}

class B2 extends B {
  ???  //省略不写，仅做演示
}
```
现在有一场景，我们需要在特质A的实现类A1和A2上面分别使用B1和B2的功能，这时候我们如何优雅的去实现这个功能并且方便后续扩展（尽量不使用判断语句）
这时候就可以使用蛋糕模式，把B包装为一个trait，里面包含一个类型B的成员变量，并将trait的方法进行重载。代码修改如下：
```
trait A {
  self: B => 
  val text = "hello"
  def func1(str: String): Unit = println(s"$text $str")
}

trait B {
  val adress = "home"
  def func2(): String = adress
}

class B1 extends B {
  override val adress = "office"
  override def func2(): String = s"welcome to ${adress}"
}

trait C extends B {
  val b1 = new B1
  override val adress = b1.adress
  override def func2(): String = b1.func2
}

class A1 extends A with C {
  override val text = "My Lady~"
}
```
这时候执行如下语句：
```
val a = new A1
a.func1(a.cunc2())
```
结果为：

![image](https://user-images.githubusercontent.com/25081842/132299880-8ba75566-5e35-41a6-b676-241bada87460.png)

这样做比继承或组合好的地方是：我们不会对原有代码改动太多，我们只是在特质A上面添加了一个自类型以及在相应类上面扩展特质，并没有对原有代码做特别大改动
而且这样做另一个好处是，在最上层的抽象（也就是特质这一层），在多态使用的场景中会避免不必要的判断，造成代码看起来凌乱不优雅，体现不出代码设计的优美。

如果上面这个例子没有体现出来，请看下面这个例子：
若现在有一个Engine特质，现在分别有实现类：EngineA、EngineB和EngineC
```
trait Engine{
  def f = println("Engine") 
}

class EngineA extends Engine{
  override def f = println("EngineA")
}

class EngineB extends Engine{
  override def f = println("EngineB")
}

class EngineC extends Engine{
  override def f = println("EngineC")
}
```
每个Engine有一个对应的Parser类中实现的m功能：ParserA、ParserB和ParserC
```
trait Parser{
  def m = println("Parser m function")
}

class ParserA extends Parser{
  override def m = println("ParserA m function")
}

class ParserB extends Parser{
  override def m = println("ParserB m function")
}

class ParserC extends Parser{
  override def m = println("ParserC m function")
}
```
在最上层的抽象当中，有这么一个需求：
我们需要在一个方法当中实现不同Engine类中使用m功能解决某个问题。
这时候就可以通过自类型进行设计，具体如下：
首先，将Parser各类包装为trait：
```
trait ParserAHelper extends Parser{
  val parser = new ParserA
  override def m = parser.m
}

trait ParserBHelper extends Parser{
  val parser = new ParserB
  override def m = parser.m
}

trait ParserCHelper extends Parser{
  val parser = new ParserC
  override def m = parser.m
}
```
然后，对Engine特质以及实现类进行修改，如下：
```
trait Engine {
  self: Parser =>
  def f = println("Engine") 
  
  //需要添加的函数，实现对应的Parser调用
  def mFunc = m
}

class EngineA extends Engine with ParserAHelper{
  override def f = println("EngineA")
}

class EngineB extends Engine with ParserBHelper{
  override def f = println("EngineB")
}

class EngineC extends Engine with ParserCHelper{
  override def f = println("EngineC")
}
```
然后，将Parser各类包装为trait：

之后，实现一个函数：
```
def mFunc(e: Engine): Unit = {
  e.mFunc
}
mFunc(new EngineA)
mFunc(new EngineB)
mFunc(new EngineC)
```

![image](https://user-images.githubusercontent.com/25081842/132318657-ff848475-21d9-4430-b2fa-ae23f24c53d7.png)

