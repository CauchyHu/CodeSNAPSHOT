# Scala 自类型与蛋糕模式
## 应用场景
--- 

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
首先，将Parser各类再包装为trait：
```
trait ParserHelper extends {
  val parser: Parser
  def m = parser.m
}

trait ParserAHelper extends ParserHelper{
  override val parser = new ParserA
}

trait ParserBHelper extends ParserHelper{
  override val parser = new ParserB
}

trait ParserCHelper extends ParserHelper{
  override val parser = new ParserC
}
```
然后，对Engine特质以及实现类进行修改，如下：
```
trait Engine {
  self: ParserHelper =>
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

这样做好的地方是：我们不会对原有代码改动太多，我们只是在特质上面添加了一个自类型以及在相应类上面扩展特质，并没有对原有代码做特别大改动
而且这样做另一个好处是，在最上层的抽象（也就是特质这一层），在多态使用的场景中会避免不必要的判断，造成代码看起来凌乱不优雅，体现不出代码设计的优美。

--- 

## 自由混入特质

    上述的应用场景是每一个实现类都要扩展相应的某个功能，但是若每个实现类中个别实例化对象需要m功能，有些实例化对象又不需要m功能，这时候，这种实现方式又不是很优雅了。面对这种情况则可以通过对实现类的实例化对象进行特质的自由混入。
    首先，我们的Engine、EnginA、EnineB和EngineC还是保持原有，即：
    
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
    之后，我们的特质，以及对应的实现m功能，与前面蛋糕模式添加一致，即：
```
trait ParserHelper extends {
  val parser: Parser
  def m = parser.m
}

trait ParserAHelper extends ParserHelper{
  override val parser = new ParserA
}

trait ParserBHelper extends ParserHelper{
  override val parser = new ParserB
}

trait ParserCHelper extends ParserHelper{
  override val parser = new ParserC
}
```
最后，我们在需要EngineA、EngineB或EngineC的实例化对象上面自由混入相应对象即可。
```
val engineA = new EngineA with ParserAHelper
engineA.m

val engineB = new EngineB with ParserBHelper
engineB.m

val engineC = new EngineC with ParserCHelper
engineC.m
```

结果如下：

![image](https://user-images.githubusercontent.com/25081842/132462805-bec1b810-d1ef-4361-aa0a-fc36b7dd81b7.png)
