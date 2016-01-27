#Scala的反射机制
1. Manifest & ClassManifest
Manifest是在编译时捕捉的，编码了“捕捉时”所致的类型信息。然后就可以在运行时检查和使用类型信息，但是manifest只能捕捉当Manifest被查找时在隐式作用域里的类型。
``` scala 
def first[A : ClassManifest](x:Array[A]) = Arraay(x(0))
```
2. ClassTag & TypeTag
scala在2.10里却用TypeTag替代了Manifest，用ClassTag替代了ClassManifest，原因是在路径依赖类型中，Manifest存在问题：
``` scala 
scala> class Foo{class Bar}

scala> def m(f: Foo)(b: f.Bar)(implicit ev: Manifest[f.Bar]) = ev

scala> val f1 = new Foo;val b1 = new f1.Bar
scala> val f2 = new Foo;val b2 = new f2.Bar

scala> val ev1 = m(f1)(b1)
ev1: Manifest[f1.Bar] = Foo@681e731c.type#Foo$Bar

scala> val ev2 = m(f2)(b2)
ev2: Manifest[f2.Bar] = Foo@3e50039c.type#Foo$Bar

scala> ev1 == ev2 // they should be different, thus the result is wrong
res28: Boolean = true
```
ev1 不应该等于 ev2 的，因为其依赖路径（外部实例）是不一样的。
还有其他因素，所以在2.10版本里，使用 TypeTag 替代了 Manifest
TypeTag:由编辑器生成,只能通过隐式参数或者上下文绑定获取
可以有两种方式获取:
``` scala
scala> import scala.reflect.runtime.universe._
import scala.reflect.runtime.universe._

//使用typeTag
scala> def getTypeTag[T:TypeTag](a:T) = typeTag[T]
getTypeTag: [T](a: T)(implicit evidence$1: reflect.runtime.universe.TypeTag[T])reflect.runtime.universe.TypeTag[T]

//使用implicitly 等价的 
//scala>def getTypeTag[T:TypeTag](a:T) = implicitly[TypeTag[T]]

scala> getTypeTag(List(1,2,3))
res0: reflect.runtime.universe.TypeTag[List[Int]] = TypeTag[List[Int]]

```
通过TypeTag的tpe方法获得需要的Type(如果不是从对象换取Type 而是从class中获得 可以直接用 typeOf[类名])

3. 反射获取TypeTag和ClassTag
String---------->Calss-------------->Manifest-------->TypeTag
Class.forName    ManifestFactory.classType scala.reflect.runtime.universe.manifestToTypeTag
``` scala
import scala.reflect.runtime.universe
import scala.reflect.ManifestFactory

val className = "java.lang.String"
val mirror = universe.runtimeMirror(getClass.getClassLoader)
val cls = Class.forName(className)
val t = universe.manifestToTypeTag(mirror, ManifestFactory.classType(cls))
```