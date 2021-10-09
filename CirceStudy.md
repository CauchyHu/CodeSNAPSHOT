# Circe
--- 
## Encoding和Decoding
***Encoder[A]是将A类型转化成JSON的函数，Decoder[A]是将Json转化成一个A对象或者是exception的函数。***
circe提供了scala标准库中类型的的implicit函数，可以方便的对String，Int等基本类型进行处理，同时也提供了List[A]、Option[A]和其他泛型类型的处理，只要是A有对应的Encoder。

可以使用 .asJson 将一个数据对象转化成Json对象：
```
import io.circe.syntax._
// import io.circe.syntax._
 
val intsJson = List(1, 2, 3).asJson
// intsJson: io.circe.Json =
// [
//   1,
//   2,
//   3
// ]
```
使用 .as 将一个Json对象转化成数据对象：
```
intsJson.as[List[Int]]
// res0: io.circe.Decoder.Result[List[Int]] = Right(List(1, 2, 3))
```

parser模块中的decode函数可以直接把json string转化成对象：
```
import io.circe.parser.decode
// import io.circe.parser.decode
 
decode[List[Int]]("[1, 2, 3]")
// res1: Either[io.circe.Error,List[Int]] = Right(List(1, 2, 3))
```
返回的结果是Either类型
