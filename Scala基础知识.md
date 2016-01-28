# Scala基础知识
#一、常用类型
## 1.数组
- 变长数组: 数组缓冲
ArrayBuffer相当于Java的ArrayList
``` scala
val b = new ArrayBuffer[Int]() 

for(i <- 0 until b.length){}
for(item <- b){}
```
- 数组转换
``` scala
for(item <- b){
	yield ***
}
```
## 2.线程安全的集合与并行集合
线程安全: 混入特质即可，例如SynchronizedBuffer/Map?Queue
并行集合: coll.par.sum
##3. 单例类型 type
- 函数返回this.type,防止父类返回this时，不能调用子类的方法
```scala
class Document{
def setName() = { ...; this}
}
class Book extends Document{
def addChapter() = {...;this}
}
//返回类型为this时，  obj.setName().addChapter()错误
//返回类型为this.type--->OK
```
- 使用单例对象作为参数
```scala
def set(obj:Title.type)
```
##4.类型别名type
``` type Index= HashMap[String,Strin] ```
##5.存在类型forsome
Array[T] forsome {type T <: JComponent} == Array[_<:JComponent]
##6.类型投影
潜逃类专用： O#I
## 3.与Java的互操作
scala.collection.JavaConversions._
## 4.Some和Option
Some("ABC")的类型为Option[String],  optiond的值为Some(*) 或者None
## 5.类型参数
trait Reader{
type in 
type out 
def read(in:In):Out
}
#二、类&包
## 1.类构造器
1. 每一个辅助构造器都必须以一个对以前已定义的其他辅助构造器或者主构造器的调用开始
2. 没有显示定义主构造器则自动拥有一个无参的主构造器
3. 主构造器可以设置为private    ``` Class Person private(val id:Int) ```
4. 超类的构造和子类的主构造器交织     
``` Class Student(val age:Int,val name:String) extends Person(name) ```
## 2.类属性
1. 共有的var字段，自动生成getter和setter，函数名称为name和name_
2. 如果一个字段是私有的，那么 getter和setter也是私有的
3. 如果字段是val，则只有getter生成
4. 如果不需要任何getter和setter，将字段声明为private[this]
5. @BeanProperty用于生产Java风格的getter和setter
## 3.类字段
- 字段重写
可以使用def val进行重写
def可以重写另一个def
val只能重写另一个val或者不带参数的def
var 只能重写另一个抽象的var
- 抽象字段
没有初始值的字段，子类必须赋值
## 4.特质trait
应用程序App 特质，省去main方法的书写
## 5. 枚举
枚举的类型是Enumeration.Value
``` scala
object Color extends Enumeration{
type Color = Value
val RED, YELLOW, GREEN = Value
}
```
## 6.包
包对象可以对应工具方法和常量
``` scala
package object people{
// 不需要限定符
val defaultName = ""
}
```
## 7.类方法
- apply和update方法
apply常用伴生对象创建对象，省去new
update方法常用语更新
- 提取器unapply
apply的逆向操作
``` scala
Object Fraction{
def unapply(arg:Fraction) =  if(arg.den ==0) None else Some(arg.num, arg.den)
}
//num,den被提取出来
val Fraction(num,den) = arg
```
## 8.注解
- 继承Annotation特质
``` scala
class unchecked extends annotation.Annotation
```
- Java修饰符   不用关键字使用注解
@volatile
@transient
@strictfp
@native
- 标记接口
@cloneable 
@remote
- 受检异常
@throws 异常注解
- 变长参数
@varargs Java访问时更方便
- 内联函数
@inline/@noinline
- box
@specialized
def allDifferent[@specialized (Long,Double) T](x: T, y:T, z:T )
## 9. copy方法
copy方法可以复制一个与现有对象值相同的对象
也可以进行参数修改
``` scala
val price = amt.copy(value = 1.1d)
```

#三、函数
##1.curry化
相当于二元函数
``` scala
def mulOneAtATime(x:Int)(y:Int) = x*y
val fun = mulOneAtATime(3)相当于fun是函数: y-->3*y
```
## 2.化简折叠和扫描
list.reduceLeft(op)即将op相继应用到元素，即List(1,7,3,9).reduceLeft(_+_) = 1+7+3+9
list.reduceRight(op)即将op相继应用到元素，方向是反的，即List(1,7,3,9).reduceLeft(_ - _) = 1-(7-(3-9))
coll.foldLeft(initValue)(op)  带有初始值的curry化函数 ==> (initValue /: coll)(op)
coll.foldRight(initValue)(op)  带有初始值的curry化函数 ,方向相反==> (initValue :\ coll)(op)
scanLeft和scanRight则是包含中间结果的
# 四、流
Stream[_]

 #::用于构建一个流num #:: ** 
 使用force进行强行求值
# 五、模式匹配
##1.匹配嵌套结构
case Bundle(_ , _ , art @ Article(_ , _) ,rest @ _* )
即将值分别进行绑定
##2.密封类
seled abstract class Amount
case class A extends Amount
case class B extends Amount
编译器将会自动检查模式匹配的完整性
 
#六、其他
## 1. 文件读写
``` scala
import scala.io.Source
val source = Source.fromFile("path", "UTF-8")
val iterator = source.getLines

val iter = source.buffered
for(c <- iter)
```
#七、隐式转换
##1.隐式转化
以implicit关键字声明的带有单个参数的函数。
一般放置于伴生对象中，外部需要专门import之，将会自动转换
也可以专门排除之，  import ***.{fun=>_ , _}
##2.隐式参数
函数可以带有一个标记为implicit的参数列表，此时编译器将会查找缺省值
## 3.类型证明
T=:=U  相等
T<:<U 子类型
T<%<U 隐式转换