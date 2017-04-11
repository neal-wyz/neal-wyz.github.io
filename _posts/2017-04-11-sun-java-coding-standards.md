---
layout: post
title: "sun公司java编码准则"
date: 2017-04-11
tags: java

---

先把源文档放出来: [Sun-Java-Coding-Standards]({{site.pdf}}/sunJavaStandard.pdf)
表示个人翻译水平非常业余，请大神们多多指正

---

# 介绍

## 为什么需要有代码约定
代码约定对于程序来说是很重要的:

* 80%的软件的生命周期都耗费在维护上
* 很难有哪个软件的原作者一直在维护的
* 约定可以提高软件代码的可读性，也让工程师可以迅速读懂新代码
* 如果你开源了你的项目，你需要确保它跟你的其他项目有一样的约定

出于工作惯例，每个写代码的人都要遵守这份约定

## 感谢
> This document reflects the Java language coding standards presented in the Java Language
Specification, from Sun Microsystems, Inc. Major contributions are from Peter King, Patrick
Naughton, Mike DeMoney, Jonni Kanerva, Kathy Walrath, and Scott Hommel.
This document is maintained by Scott Hommel. Comments should be sent to
shommel@eng.sun.com

# 文件名称
这一节列举常用的文件后缀和名称

## 文件后缀
java软件中的文件使用下面的后缀：
* *Java source*: .java
* *Java bytecode*: .class

## 文件名
常用的文件名包含：
* *GNUmakefile*:首选的文件名，我们使用自动编译构建软件
* *README*: 首选的文件名，总结特定目录的内容的文件

# 文件组织结构
一个文件的各个章节应该被空行隔开，并且可以有注释来标识每一个章节

应该避免有超过2000行的文件

## Java Source Files
每个Java源文件包含一个简单的公有类或接口，当一个私有类或接口与一个公有类相关时，可以把他们都放在这个公有类中，这个公有类应该是这个文件的第一个类或接口

Java源文件应该有以下特点：
* 头注释
* package 和 import 语句
* 类和接口的声明

### 头注释
所有的源文件都应该已c-style的注释开头，列出类名，版本信息，日期和copyright标识
```
/*
 * Classname
 *
 * Version information
 *
 * Date
 *
 * Copyright notice
 */
```

### package和import语句
绝大多数java源文件的非注释第一行都是package语句，紧跟着是import语句
```
package java.awt;

import java.awt.peer.CanvasPeer;
```

### 类和接口声明
下面是类或接口声明的各部分描述（按声明的顺序列举）
1. 类/接口文档注释（/\*\*...\*/）
2. class或interface语句
3. 类/接口实现注释：这个注释应该包含其他不适合放在文档注释中的类或接口的相关信息
4. 类（static）变量：首先是public类变量，然后是protected，然后是package级别（没有编辑权限），然后是private
5. 实例变量：：首先是public类变量，然后是protected，然后是package级别（没有编辑权限），然后是private
6. 构造函数
7. 方法：这些方法应该通过功能分类，而不是作用范围或者权限。比如，一个私有类方法可以在两个公有实例方法中间，这样做的目的是让代码更容易被人看懂和理解

# 缩进
应该使用四个空格的缩进，现在还没有规定缩进应该使用空格还是Tabs。Tabs严格来说是8个空格

## 行长度
避免一行超过80个字符，因为很多终端和工具都不能显示超过这个长度

**注意**：如果是用在文档中的示例，应该更短，一般不超过70个字符

## 换行
如果一个表达式需要换行，请遵循以下规则：
* 在逗号之后换行
* 在运算符之前换行
* Prefer higher-level breaks to lower-level breaks（网上有人说是先拆分低优先级表达式的意思）
* Align the new line with the beginning of the expression at the same level on the previous line（对齐新的一行与上一行的相同级别的表达式）
* 如果上述规则导致混淆代码或改变了正确代码的边距,就用8个空格的缩进代替

下面是方法调用换行的示例：

```java
someMethod(longExpression1, longExpression2, longExpression3,
         longExpression4, longExpression5);
```

```java
var = someMethod1(longExpression1,
                someMethod2(longExpression2,
                          longExpression3));
```

下面是两个算数表达式换行的例子：

鼓励使用第一种方式，因为换行发生在括号表达式的外面，是更高的等级发生的换行

```java
longName1 = longName2 * (longName3 + longName4 - longName5)
            + 4 * longname6; // PREFER
```

```java
longName1 = longName2 * (longName3 + longName4
                         - longName5) + 4 * longname6; // AVOID
```

下面是两个方法定义的缩进例子：

第一个是按照约定的案例，第二个的第二行第三行如果使用约定则会对齐到很右边的地方，所以使用8个空格的缩进来代替
```java
//CONVENTIONAL INDENTATION
someMethod(int anArg, Object anotherArg, String yetAnotherArg,
           Object andStillAnother) {
    ...
}
```

```java
//INDENT 8 SPACES TO AVOID VERY DEEP INDENTS
private static synchronized horkingLongMethodName(int anArg,
        Object anotherArg, String yetAnotherArg,
        Object andStillAnother) {
    ...
}
```

if语句的换行一般使用8个空格的规则，因为约定的四个空格让代买很难看，举个例子：
```java
//DON’T USE THIS INDENTATION
if ((condition1 && condition2)
    || (condition3 && condition4)
    ||!(condition5 && condition6)) { //BAD WRAPS
    doSomethingAboutIt();            //MAKE THIS LINE EASY TO MISS
}
```

```java
//USE THIS INDENTATION INSTEAD
if ((condition1 && condition2)
        || (condition3 && condition4)
        ||!(condition5 && condition6)) {
    doSomethingAboutIt();
}
```

```java
//OR USE THIS
if ((condition1 && condition2) || (condition3 && condition4)
        ||!(condition5 && condition6)) {
    doSomethingAboutIt();
}
```

这里有三种三元表达式可以使用的换行方式：
```java
alpha = (aLongBooleanExpression) ? beta : gamma;

alpha = (aLongBooleanExpression) ? beta
                                 : gamma;

alpha = (aLongBooleanExpression)
        ? beta
        : gamma;
```

# 注释
Java程序有两种注释：实现注释和文档注释
* 实现注释起源于C++，使用 /\*...\*/ 和 // 来标识
* 文档注释是Java特有的，使用 /\*\*...\*/ 标识，可以使用Java工具转换成HTML格式的文件

实现注释是注释代码和描述特殊实现的方式，文档注释是代码的说明书，是给那些没有必要获取源码来了解具体实现方式的开发者们读的

注释应该被用来描述代码并且提供代码中得不到的额外的信息，注释应该只包含帮助阅读和理解程序的信息，比如，不要把相应的包的构建，或者它应该在哪个目录下面放在注释中

重要的讨论和设计的决定是合适的，但是应该避免在代码中出现重复的信息。注释是很容易就出现过期。一般来说，要在代码编写过程中去掉可能会过期的注释

**注意**：注释频率高只能体现代码质量差，当你感觉一定要写个注释的时候，考虑重写一份干净的代码

* 注释不应该包含在星号或者字符画的大框里
* 注释中一定不要使用像换页和退格这类的特殊字符

## 实现注释的格式
* block：块
* single-line：单行
* trailing：拖尾
* end-of-line：行尾

### block注释
块注释用来提供文件、方法、数据结构和算法的描述，经常用在文章的开头和每个方法之前，也可以用在其他地方，比如方法内部，方法内部的注释应该和它所描述的代码在同一级别

每个块注释之前应该有一行空白：
```java
/*
 * 这是一个块注释。
 */
```

块注释可以使用/*-做开头，被indent(1)当作是不应该被重新编排的块注释头：
```java
/*-
 * Here is a block comment with some very special
 * formatting that I want indent(1) to ignore.
 *
 * one
 * two
 * three
 */
```

**注意**：如果你不使用indent(1)，你不需要在你的代码中使用/*-，或者给其他可能在你的代码中运行indent(1)的人一点特权

### 单行注释
紧跟着它描述的代码并且缩进相同的的单行注释可以使用短注释，如果单行注释写不下，那就考虑块注释，单行注释之前应该有一行空行：
```java
if (condition) {
    
    /* Handle the condition. */
    ...
}
```

### 拖尾注释
非常短的注释可以出现在它们描述代码的同一行，但是应该让注释尽可能远离代码。如果有许多拖尾注释出现在同一个代码块中，它们应该使用相同的缩进：
```java
if (a == 2) {
    return TRUE;                        /* special case */
} else {
    return isPrime(a);                  /* works only for odd a */
}
```

### 行尾注释
\/\/注释分隔符能够注释一整行或者一部分，它不应该用来生成多行的文字注释。但是，它可以用在非代码块的多行注释中：
```java
if (foo > 1) {
    // Do a double-flip.
    ...
}
else{
    return false;                       // Explain why here.
}
```

```java
//if (bar > 1) {
//
//    // Do a triple-flip.
//    ...
//}
//else{
//    return false;
//}
```

## 文档注释
更多的有关文档注释的细节，请看：
> [http://java.sun.com/products/jdk/javadoc/]()

文档注释描述Java类，接口，构造函数，方法，属性。每个类，接口，或者成员的文档注释都是写在/\*\*...\*/里面：
```java
/**
 * The Example class provides ...
 */
public class Example { ...
```

注意顶级的类和接口虽然成员是有缩进的，但是自己是没有缩进的，类和接口的第一行(/\*\*)也是不缩进的，随后的每一行都会有一个空格的缩进（垂直对齐星号）。成员，内置构造函数，对于第一行文档注释来说有四个空格，之后是五个空格（Members, including
constructors, have 4 spaces for the first doc comment line and 5 spaces thereafter. ）

如果你需要提供类,接口,变量或者方法的信息，不要用文档，使用一个实现块注释(参见5.1.1节)或在声明之后直接使用单行注释(见部分5.1.2)。例如,实现类的细节应该在这个类语句的声明块注释上，不应该在文档注释里

文档注释不应该放在方法或者构造函数定义块里，因为Java会把注释与后面的第一个声明联系起来

# 声明定义
##　每行声明的变量数量
最好是每行一个声明，方便注释：
```java
int level; // indentation level
int size; // size of table
```
要比下面的方式好
```java
int level, size;
```

不要把不同的类型放在同一行：
```java
int foo, fooarray[]; //WRONG!
```

**注意**：上面的例子在类型和实体之间使用一个空格。另外一种方式是使用tabs：
```java
int         level;          // indentation level
int         size;           // size of table
Object      currentEntry;   // currently selected table entry
```

## 初始化
试着在声明本地变量的时候赋初值。唯一不赋初值的原因是初始值是由第一次计算发生的时候决定

## 位置
把声明只放在代码块开始的地方（一个块是被包裹在花括号中的任何代码）。不要在第一次使用变量的时候再去声明，这有可能让粗心的程序员感到困惑，并且影响代码的可移植性：
```java
void myMethod() {
    int int1 = 0;       // beginning of method block
    if (condition) {
        int int2 = 0;       // beginning of "if" block
        ...
    }
}
```

for循环语句是个例外，在Java中可以在for语句中定义：
```java
for (int i = 0; i < maxLoops; i++) { ... }
```

避免使用局部声明来隐藏更高层的声明。比如不要在块内部使用相同的变量名：
```java
int count;
...
myMethod() {
    if (condition) {
        int count; // AVOID!
        ...
    }
    ...
}
```

## 类和接口的声明
在编写Java的类和接口时，应该遵守下面的格式规则：
* 不要在调用方法的时候在方法名和小括号之间插入空格
* 开括号“{”应该出现在一行的最后
* 闭括号“{”应该自己占一行，并且匹配相应的开括号，如果没有语句，那么开括号和右括号应该“{}”这样写
```java
class Sample extends Object {
    int ivar1;
    int ivar2;
    
    Sample(int i, int j) {
        ivar1 = i;
        ivar2 = j;
    }
    
    int emptyMethod() {}
    
    ...
}
```
* 方法之间应该用空行分隔的

# 语句
## 简单语句
每一个行应该至少有一个语句：
```java
argv++;             // Correct
argc++;             // Correct

argv++; argc--;     // AVOID!
```

## 组合语句
组合语句有一组包含在花括号中的语句“{ statement }”
* 花括号中的语句应该比组合语句本身多缩进至少一级
* 开括号应该在组合语句的一行末尾；闭括号应该在一行的开始并且与对应的语句的缩进相同
* 花括号被用来控制接口的时候需要包含语句，哪怕是单行语句，像if-else语句或者for语句。这样可以更容易的添加语句，不会因为忘记添加花括号引发bug

## return语句
一个带值的return语句不应该使用括号除非用其他形式返回值：
```java
return;

return myDisk.size();

return (size ? size : defaultSize);
```

## if,if-else,if else-if else语句
if-else类的语句应该用下面的形式：
```java
if (condition) {
    statements;
}
if (condition) {
    statements;
} else {
    statements;
}
if (condition) {
    statements;
} else if (condition) {
    statements;
} else {
    statements;
}
```

if语句一直使用花括号，避免下面这种情况：
```java
if (condition)  //AVOID! 没有使用 {}!
    statement;
```

## for语句
for语句应该是：
```java
for (initialization; condition; update) {
    statements;
}
```

一个空的for语句（所有的过程都在初始化，条件，更新项目）：
```java
for (initialization; condition; update);
```

在for语句的update中使用逗号运算符时不要使用超过三个变量。在for循环之前（initialization）或者之后（update）使用多个语句。

## while语句
```java
while (condition) {
    statements;
}
```

空的while语句：
```java
while (condition);
```

## do-while语句
```java
do {
    statements;
} while (condition);
```

## switch语句
```java
switch (condition) {
case ABC:
    statements;
    /* falls through */
case DEF:
    statements;
    break;
    
case XYZ:
    statements;
    break;

default:
    statements;
    break;
}
```

如果有不包含break语句的case，在上面例子中的/\* falls through \*/写个注释

每个switch语句应该有个default case。在default case中的break是多余的，但是这预防了另一个case加进来时的fall-through错误

## try-catch语句
```java
try {
    statements;
} catch (ExceptionClass e) {
    statements;
}
```

try-catch语句应该一直跟着一个不管try代码块是否成功执行都会执行的finally语句块：
```java
try {
    statements;
} catch (ExceptionClass e) {
    statements;
} finally {
    statements;
}
```

# 空白
## 空行
空行通过设置逻辑上的代码章节提高了可读性

以下情况使用两个空行：
* 源文件的章节之间
* 类和接口定义之间

以下情况使用一个空行：
* 方法之间
* 方法中的本地变量和它的第一个语句
* 在块注释和单行注释之前
* 方法中的逻辑章节之间，为了提高可读性

## 空格
以下情况应该使用空格：
* 关键词后面如果有括号，则需要空格隔开：
```java
while (true) {
    ···
}
```
* 方法名和它的参数之间的括号不用隔开，这样有助于区分方法调用和关键词
* 在参数列表中逗号之后应该由个空格
* 所有的位运算符应该用空格和它们的运算对象分隔开。空格不要分隔一元运算符与它们的运算对象，像负号，自增（“++”），自减（“--”）：
```java
a += c + d;
a = (a + b) / (c * d);

while (d++ = s++) {
    n++;
}
prints("size is " + foo + "\n");
```
* for语句的表达式应该被空格分隔：
```java
for (expr1; expr2; expr3)
```
* 强制类型转换应该跟随一个空格：
```java
myMethod((byte) aNum, (Object) x);
myMethod((int) (cp + 5), ((int) (i + 3)) + 1);
```

# 命名约定

命名约定让程序更易读更容易理解。它们也可以表明标识的具体功能--比如常量，包或者类--来帮助理解代码

* Packages：包名的前缀通常是全小写的ASCII字母，并且应该是个顶级域名，现在一般是com，edu，gov，mil，net，org，或者在ISO标准3166，1981中声明的代表国家的两个英文字母。包名的后续与组织的内部命名约定相关。可以指定目录名称是由分公司、部门、项目,机,或登录名称组成
```java
com.sun.eng;
com.apple.quicktime.v2;
edu.cmu.cs.bovik.cheese;
```
* Classes：类名应该是名词，内部的每一个单词的首字母大写。你要保证你的类名简单易懂，使用整词--避免使用首字母缩写和缩写，除非缩写是通用的，比如URL和HTML
```java
class Raster;
class ImageSprite;
```
* Interfaces：接口名称应该和类名称使用一样的大小写规则
```java
interface RasterDelegate;
interface Storing;
```
* Methods：方法应该是动词，首字母应该小写，内部单词的首字母大写
```java
run();
runFast();
getBackground();
```
* Variables：除了变量，所有的实例，类和类常量，均采用首字母小写混合的情况。 内部单词以大写字母开头。 变量名不应以下划线_或者美元符号$字符开头，尽管两者都允许。变量名称应该很短而有意义。应该选择有助理解的变量名称 - 也就是说，要让刚刚看到的人明白这是用来做什么的。 应避免使用一个字符的变量名称，除了临时的“一次性”变量。临时变量的通用名称为i，j，k，m和n为整数; c，d和e表示字符
```java
int         i;
char        c;
float       myWidth;
```
* Constants：变量的名称声明为类常量，ANSI常数应为全部大写字母并以下划线（“_”）分隔。（为避免调试，应避免使用ANSI常数。）
```java
static final int MIN_WIDTH = 4;
static final int MAX_WIDTH = 999;
static final int GET_THE_CPU = 1;
```

# 编程惯例
## 提供实例和类变量
没有好理由的话最好不要使用public修饰实例变量或类变量。通常情况下，实例变量并不需要明确设值或取值，这通常发生在方法调用的时候。

适当的public实例变量的一个例子是类本质上是数据结构，没有任何行为的情况。换句话说，如果你使用一个struct而不是一个类（如果是Java支持的struct），那么用public修饰类的实例变量是合适的。

## 类变量和方法
避免用对象去调用类（静态）变量或者方法。使用类名：
```java
classMethod();              //OK
AClass.classMethod();       //OK
anObject.classMethod();     //AVOID!
```

## 常量
数字常量（数值）不应直接编码，除了-1，0和1，它们可以作为计数器值出现在for循环中

## 变量赋值
避免在同一语句中给多个变量赋同一个值，这样很难理解：
```java
fooBar.fChar = barFoo.lchar = 'c'; // AVOID!
```

不要再容易和等号混乱的地方使用赋值运算符：
```java
if (c++ = d++) { // AVOID! (Java disallows)
    ...
}
```
应该是：
```java
if ((c++ = d++) != 0) {
    ...
}
```

不要尝试使用内嵌赋值去提高运行时的效率，这是编译器考虑的事：
```java
d = (a = b + c) + r; // AVOID!
```
应该写成：
```java
a = b + c;
d = a + r;
```

## 其他惯例
### 括号
在包含混合运算符的表达式中使用括号来避免运算符优先级问题是一个好主意。即使你很清楚运算符的优先级，但别人不一定清楚，不要假设其他的程序员也像你一样知道优先级。
```java
if (a == b && c == d)       // AVOID!
if ((a == b) && (c == d))   // USE
```

### 返回值
让你的代码结构匹配缩进：
```java
if (booleanExpression) {
    return true;
} else {
    return false;
}
```
应该写成：
```java
return booleanExpression;
```

一样的：
```java
if (condition) {
    return x;
}
return y;
```
应该写成：
```java
return (condition ? x : y);
```

### 条件运算符中'?'前的表达式
如果在三元运算符?:的?前有含有二元运算符的表达式，这个表达式应该在括号内部：
```java
(x >= 0) ? x : -x;
```

### 特殊注释
在注释中使用XXX标识一些假的但是可以起作用的东西。使用FIXME标识假的并且没作用的东西。

# 代码示例
## java源码示例
```java
/*
 * @(#)Blah.java 1.82 99/03/18
 *
 * Copyright (c) 1994-1999 Sun Microsystems, Inc.
 * 901 San Antonio Road, Palo Alto, California, 94303, U.S.A.
 * All Rights Reserved.
 *
 * This software is the confidential and proprietary information of Sun
 * Microsystems, Inc. ("Confidential Information"). You shall not
 * disclose such Confidential Information and shall use it only in
 * accordance with the terms of the license agreement you entered into
 * with Sun.
 */
 
package java.blah;

import java.blah.blahdy.BlahBlah;

/**
 * Class description goes here.
 *
 * @version 1.82 18 Mar 1999
 * @author Firstname Lastname
 */
public class Blah extends SomeClass {
    /* A class implementation comment can go here. */

    /** classVar1 documentation comment */
    public static int classVar1;

    /**
     * classVar2 documentation comment that happens to be
     * more than one line long
     */
    private static Object classVar2;

    /** instanceVar1 documentation comment */
    public Object instanceVar1;

    /** instanceVar2 documentation comment */
    protected int instanceVar2;

    /** instanceVar3 documentation comment */
    private Object[] instanceVar3;

    /**
     * ...constructor Blah documentation comment...
     */
    public Blah() {
        // ...implementation goes here...
    }

    /**
     * ...method doSomething documentation comment...
     */
    public void doSomething() {
        // ...implementation goes here...
    }

    /**
     * ...method doSomethingElse documentation comment...
     * @param someParam description
     */
    public void doSomethingElse(Object someParam) {
        // ...implementation goes here...
    }
}
```