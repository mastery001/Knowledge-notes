

[TOC]

# HelloWorld
(1)最简单的HelloWorld输出方式：
```grrovy
// println为Groovy中的标准输出函数
println 'HelloWorld!'
```
(2)基于类的HelloWorld输出方式：
```Groovy
/**
* 1.Groovy的修饰符与Java类似有public,private,protected;作用域可以参考Java
* 2.由下面的代码可以看出，Groovy的默认修饰符为public
*/
class HelloWorld {
    static void main(def args) {
        println "HelloWorld!"
    }
}
```
(3)基于继承Script类的HelloWorld
> 脚本（script）总是能编译成类，groovy编译器会将脚本编译成类，并且将执行该类中的run方法

```groovy
import org.codehaus.groovy.runtime.InvokerHelper;

class HelloWorld extends Script{

    /**
    * 继承Script类必须实现run方法
    */
    def run() {
        println "HelloWorld!"
    }
    
    static void main(def args) {
        /*
        *使用InvokerHelper调用runScript方法，因为main函数与HelloWorld类在同一包下
        *所有可直接输入HelloWorld调用，否则需要带上包名访问
        */
        InvokerHelper.runScript(HelloWorld , args)
    }
}
```

## Comments(注释)
-  Single line comment（单行注释）:使用'//'
-  Multiline comment(多行注释):使用'/* */'
-  GroovyDoc comment(grrovy文档注释):使用'/** */',该注释通常都作用于：
 -  [x]类型定义（类，接口，枚举，注解）
 -  [x]字段和属性的定义
 -  [x]方法定义
   一般来说，注释都位于定义上方。
-  Shebang line：类似UNIX系统的第一行注释，用'#!',作用是指定运行的环境.例如
```java
#!/usr/bin/env groovy
```

## Type And Variable(类型与变量)

### Type
> Groovy中的类型分为primitive type和Java type.

（1）下面为Groovy的基本类型，与Java相同

|Type     | comments
|-------  | --------
| int     | 对应Integer
| short   | 对应Short
| byte    | 对应Byte
| char    | 对应Character
| long    | 对应Long
| float   | 对应Float
| double  | 对应Double
| boolean | 对应Boolean,在Groovy中，null的值为false

（2）下面为Groovy中的几个重要的类型

|Type     | comments
|-------  | --------
| String  | 对应java.lang.String类
| GString | Groovy中特有的字符类，对应groovy.lang.GString
| List    | 列表，使用的是Java中的java.util.List
| Array   | 数组
| Map     | 映射，Grrovy的map默认使用的是java.util.LinkedHashMap

在之后的章节会具体这几个重要的类型

### Variable
Groovy中变量定义存在两种方式：
-  与Java相同，通过基本类型或者类名声明，例如： int a ;  Object obj ;
-  在Groovy中，万物皆对象，统一用def来定义，例如： def a

## Type Coercion（类型转换）
与任何语言相同，Groovy也存在类型转换这一特性，有两种转换形式：
```comments
与Java相同，在变量之前转换成用括号括起来的类型。如a=(char)1
使用as：Groovy具有特殊的类型转换的方式，即使用as，例如1 as Character , 即可将1转换成char类型。
```
## String Type(String类型)
> Groovy支持java.lang.String类型的字符串和本身自带的groovy.lang.GString类型的字符串（在其他语言中称作插值字符串）

### java.lang.String
```comments
1.单引号字符串：'a single quoted string'
2.字符连接，通过'+'： 'a + b' == 'ab'
3.三引号字符串：'''a triple single quoted string''' ;支持多行字符，
```


### groovy.lang.GString

## Imports（导入）
> Groovy与Java相同，都有包的概念，所有导入的方式与java相同，下面将介绍Groovy中独特的部分

### Default Imports(默认导入)
在Groovy中，存在一些默认导入的包，如下所示：
```groovy
java.lang.*
java.util.*
java.io.*
java.net.*
groovy.lang.*
groovy.util.*
java.math.BigInteger
java.math.BigDecimal
```

### import aliasing(导入别名)
在导入过程中，可将导入的类或静态导入的方法赋予别名

1.静态导入方法
```groovy
//由于groovy默认导入了java.util.*,所以可直接引用Calendar
import static Calendar.getInstance as now

assert now().class == Calendar.getInstance().class
```
2.导入类
```groovy
import java.sql.Date as SQLDate

SQLDate sqlDate = new SQLDate(1000L)

assert sqlDate instanceof java.sql.Date
```

## Meta-annotations(元注解)
>groovy中的注解基本与java相同，但是也在java的基础上添加了元注解：
>1.支持将多个注解整合成一个注解

常用的注解方式
```groovy
@Service
@Transactional
class MyTransactionalService {}
```
元注解方式
```groovy
import groovy.transform.AnnotationCollector

@Service
@Transactional
@AnnotationCollector
@interface TranscationalService {
    //TranscationalService即元注解
}

@TranscationalService
class MyTransactionalService{

}
```

## Traits
> Traits类似于java和groovy中的抽象类，但使用时必须用implements

trait 类名{}
例如：
```groovy
trait FlyingAbility {
    String fly() { "I'm flying!" }
}

class Bird implements FlyingAbility {
}
def b = new Bird()
assert b.fly() == "I'm flying!"
```

支持多重继承
```groovy
trait Named{
    String name
}

trait Id {
    Long id
}
/**
* 继承使用extends
*/
trait Polite extends Named {
    String introduce() { "Hello, I am $name"}
}

/**
* 多重继承,使用implements
*/
trait Person implements Named , Id {

}
```
方法冲突
```
trait A {
    String exec() { 'A' }
}
trait B {
    String exec() { 'B' }
}
class C implements A,B {}

def c = new C()
// 默认执行的是最后一个实现，即B
assert c.exec() == 'B'

/**
* 通过覆盖方法来指定实现
*/
class C1 implements A,B {
    String exec() { A.super.exec() }
}
def c1 = new C1()
assert c.exec() == 'A'
```


