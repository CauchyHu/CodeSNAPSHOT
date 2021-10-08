# SQL语句切分算法学习
## 问题背景
> 一个字符串内包含了多行SQL语句，需要解析字符串中有几个SQL执行语句，因此我们需要按照“；”符号进行一个有效分割。
问题的本质就是一个字符串内的“；”符是否为有效分隔符。

算法思路：
设置一个当前索引，若当前索引指向一个分号（;）则需要判断该符号是否为一个有效分号，有效则做分割，无效则跳过。 
无效case：
- 在单引号内：'......;......'
- 在双引号内:  "......;......"
- 在单行注释内：--.....;.....
- 在注释块内：/* .....;.....*/
```
object SectionType extends Enumeration{
  type SectionType = Value
  val SINGLE_QUOTED, DOUBLE_QUOTED, LINE_COMMENT, BLOCK_COMMENT = Value   //分别对应‘，“，--，/**/
}
```

有效分隔符所处场景可为：null
在进行遍历过程中，还需要设置一个flag用于标记当前处于哪个case。
- 单引号case下，只有再遇到单引号才能退出该场景。
- 双引号case下，只有再遇到双引号才能退出该场景。
- 单行注释case下，遇到回车符才能退出该场景
- 注释块case下，遇到*/符号才能退出该场景
设置一个标记前一次切分位置的索引，用于获取子字符串。
参照别人已实现代码：
```
import scala.collection.mutable.ListBuffer

object SQLSplitUtils {

  object SectionType extends Enumeration{
    type SectionType = Value
    val SINGLE_QUOTED, DOUBLE_QUOTED, LINE_COMMENT, BLOCK_COMMENT = Value   //分别对应‘，“，--，/**/
  }

  def decompse(statements: String): Seq[String] = {
    splitSQL(s"$statements;").collect{
      case str if str.trim.nonEmpty => str.trim
    }
  }

  def splitSQL(statements: String): Seq[String] = {
    val list = new ListBuffer[String]
    var currentIndex = 0
    var preSplitIndex = 0
    var isPreEscaped = false
    var sectionType: SectionType.SectionType = null
    var command = new StringBuilder
    while(currentIndex < statements.length)
      if(!isPreEscaped && sectionType == null && statements.startsWith("'", currentIndex)){
        sectionType = SectionType.SINGLE_QUOTED
        currentIndex += 1
      }else if(!isPreEscaped && (sectionType eq SectionType.SINGLE_QUOTED) && statements.startsWith("'", currentIndex)){
        sectionType = null
        currentIndex += 1
      }else if(!isPreEscaped && sectionType == null && statements.startsWith("\"", currentIndex)){
        sectionType = SectionType.DOUBLE_QUOTED
        currentIndex +=1
      }else if(!isPreEscaped && sectionType.eq(SectionType.DOUBLE_QUOTED) && statements.startsWith("\"", currentIndex)){
        sectionType = null
        currentIndex += 1
      }else if(sectionType == null && statements.startsWith("--", currentIndex)){
        sectionType = SectionType.LINE_COMMENT
        isPreEscaped = false
        currentIndex += 2
      }else if(sectionType.eq(SectionType.LINE_COMMENT) && statements.startsWith("\n", currentIndex)){
        sectionType = null
        isPreEscaped = false
        currentIndex += 1
      }else if(sectionType == null && statements.startsWith("/*", currentIndex)){
        sectionType = SectionType.BLOCK_COMMENT
        currentIndex += 2
      }else if(sectionType.eq(SectionType.BLOCK_COMMENT) && statements.startsWith("*/", currentIndex)){
        sectionType = null
        isPreEscaped = false
        currentIndex += 2
      }else if(statements.startsWith("//", currentIndex)){
        isPreEscaped = !isPreEscaped
        currentIndex += 1
      }else if(!isPreEscaped && sectionType == null && statements.startsWith(";", currentIndex)){
        getSqlStatement(list, statements.substring(preSplitIndex,currentIndex),command)
        currentIndex += 1
        preSplitIndex = currentIndex
        isPreEscaped =false
      }
      else{
        isPreEscaped = false
        currentIndex += 1
      }
    list.toList
  }

  def getSqlStatement(list: ListBuffer[String], str: String, command: StringBuilder) = {
    if(str.endsWith("//")){
      command.append(str.substring(0, str.length-1)).append(";")
    }else{
      command.append(str)
      list += command.toString()
      command.setLength(0)
    }
  }
}
```

不理解的点：
在进行切分时需要判断子字符串末尾为//符号时，为什么需要去掉一个/且还要加一个分号，且不做分割！这是一个不理解的点，猜测可能是有某些特殊case导致，例如/前面的SQL句子和后面句子有关联？待考证！

后续利用一个case：
```
val sql0 = "select * from a //"
val res = SQLSplitUtils.decompse(sql0)
res.foreach(a => println("\n" + a)) 
```

发现结果为空，所以又在92行代码处加了一行代码，不知道正不正确：
if(command.length != 0) list += command.toString
![image](https://user-images.githubusercontent.com/25081842/136535399-32c02daa-e59a-4560-ad43-f58ad391596f.png)


