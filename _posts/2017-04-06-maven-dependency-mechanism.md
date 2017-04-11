---
layout: post
title: "maven依赖机制"
date: 2017-04-11
tags: maven

---

# 依赖机制介绍
依赖管理是Maven最为人知的功能之一，也是Maven擅长的领域之一。管理单个项目的依赖关系没有太多困难，但是当你开始处理由多个或几百个模块组成的多模块项目和应用程序时，Maven可以帮你保持高度的可控性和稳定性

## 依赖传递（Transitive Dependencies）
依赖传递是2.0的新特性，这样可以避免你 需要寻找和声明自己使用的依赖所需的库，并自动把它们包含到项目中。

这个特性是因为从声明的远程仓库读取你使用的依赖的项目文件，一般来说，那些项目的所有依赖在你的项目中都有用到，包括这些项目从它们的父辈继承下来的，或者是它们的依赖。（这一段看了很久才看懂，翻译也很生硬，具体意思就是，你使用的依赖在远程仓库也是一个项目，这个项目的依赖或者是从父项目继承来的依赖也会传递到你的项目中）

对于依赖传递的层数没有具体的限制，但是如果发现了循环依赖则会出现问题

因为这个特性，项目中包含的库会迅速增加，因为这个原因，有一些其他特性用来限制哪个依赖会被包含：
* **依赖调解**（Dependency mediation）：在遇到多个版本的情况下使用，对于 Maven 2.0 来说，会使用“最近定义”，并且可以随时通过在 POM 中声明版本来指定要使用的版本，如果两个依赖有相同深度，从 2.0.9 开始，声明顺序中的第一个声明将被使用。（“最近定义”表示所使用的版本将是与依赖关系树中最接近的项目，例如。如果将A，B和C的依赖关系定义为 **A→B→C→D 2.0** 和 **A→E→D 1.0**，那么建立A时将使用D 1.0，因为从A到D的路径E较短。您可以在A中明确添加一个依赖关系到D 2.0以强制使用D 2.0）
* **依赖管理**（Dependency management）：这允许项目作者直接在依赖传递中或没有指定版本时声明要使用的版本。在前面的例子中，A与D有依赖继承关系，但是A并没有直接使用D，但是A可以指定D被引用时的版本，或者在A的依赖管理里添加D作为A的引用
* **依赖范围**（Dependency scope）：允许你只引入当前生命周期阶段需要的依赖
* **依赖排除**（Excluded dependencies）：如果X依赖于Y，Y取决于Z，你可以使用 **exclude** 元素将Z显式排除
* **可选依赖**（Optional dependencies）：如果Y依赖于Z，你可以使用 **optional** 元素将Z标记为可选依赖关系。当X依赖于Y时，X将仅依赖于Y，而不依赖于Y的可选依赖项Z。你可以根据自己的选择明确地添加对Z的依赖。（可以将可选依赖视为默认排除）

## 依赖范围（Dependency scope）
用来限制依赖传递，也会影响各种构建任务的类路径（classpath）

有6中范围可以选择：
* **compile**：默认范围，在项目的所有类路径下都能获得，所以，这些依赖在会被传递到依赖此项目的项目中
* **provided**：和 **compile** 比较像，但是声明你想要JDK或者其他容器在运行时提供这些依赖， 举个例子，当你使用JavaEE构建一个web项目的时候，设置Servlet API和相关的JavaEE API的 **scope** 为 provided，因为web容器会提供这些类。**provided** 只能用在编译和测试类路径，不会被传递
* **runtime**：这声明了依赖不在编译时需要，而在执行时需要，它在运行时和测试的类路径中，不在编译的类路径中
* **test**：此范围表示正常使用应用程序不需要依赖关系，仅用于测试编译和执行阶段。这个范围是不可传递的
* **system**：与 provided 类似，你必须明确提供相关的jar包，这个依赖始终可用，而且不会去仓库里找这个依赖
* **import**（Maven 2.0.9 +）：这个范围只支持在 <dependencyManagement> 节点中声明的类型为pom的依赖使用，它表示这个依赖会被 <dependencyManagement> 节点中声明的有效的依赖列表所替代，当它们被替换掉，import 范围的依赖实际上没有参与限制依赖传递。

每个范围（除了import）都会影响依赖传递关系，接下来按生命周期来列举一下当依赖的scope明确声明后在项目中的结果：

**compile**：
* compile：compile（这里会被 runtime 所替代，所以所有的 compile 的依赖都要明确的给出。这也是你依赖的库扩展了另一个库的类，强制在 compile 时期可用的原因）
* provided：provided
* runtime：runtime
* test：test

**provided**：
* 四种声明的依赖均被忽略

**runtime**：
* compile：runtime
* provided：provided
* runtime：runtime
* test：test

**test**：
* 四种声明的依赖均被忽略

## 依赖管理（Dependency Management）
依赖管理是集中整理依赖信息的机制，当你有一组项目都继承共同的父项目的时候，你可以把所有的信息都放在共有的POM中，并且在子POM中只是简单的引用一下就可以了，举几个例子：

项目A：
```xml
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>group-a</groupId>
      <artifactId>artifact-a</artifactId>
      <version>1.0</version>
      <exclusions>
        <exclusion>
          <groupId>group-c</groupId>
          <artifactId>excluded-artifact</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    <dependency>
      <groupId>group-a</groupId>
      <artifactId>artifact-b</artifactId>
      <version>1.0</version>
      <type>bar</type>
      <scope>runtime</scope>
    </dependency>
  </dependencies>
</project>
```

项目B：
```xml
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>group-c</groupId>
      <artifactId>artifact-b</artifactId>
      <version>1.0</version>
      <type>war</type>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>group-a</groupId>
      <artifactId>artifact-b</artifactId>
      <version>1.0</version>
      <type>bar</type>
      <scope>runtime</scope>
    </dependency>
  </dependencies>
</project>
```

可以看到这两个项目有共同的依赖并且每一个都有一个相对重要的依赖
这些信息可以放在一个父级POM中：
```xml
<project>
  ...
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>group-a</groupId>
        <artifactId>artifact-a</artifactId>
        <version>1.0</version>
 
        <exclusions>
          <exclusion>
            <groupId>group-c</groupId>
            <artifactId>excluded-artifact</artifactId>
          </exclusion>
        </exclusions>
 
      </dependency>
 
      <dependency>
        <groupId>group-c</groupId>
        <artifactId>artifact-b</artifactId>
        <version>1.0</version>
        <type>war</type>
        <scope>runtime</scope>
      </dependency>
 
      <dependency>
        <groupId>group-a</groupId>
        <artifactId>artifact-b</artifactId>
        <version>1.0</version>
        <type>bar</type>
        <scope>runtime</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
</project>
```
上面声明的A，B的POM就可以写成：
```xml
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>group-a</groupId>
      <artifactId>artifact-a</artifactId>
    </dependency>
 
    <dependency>
      <groupId>group-a</groupId>
      <artifactId>artifact-b</artifactId>
      <!-- 这不是jar类型的依赖，所以要明确声明type -->
      <type>bar</type>
    </dependency>
  </dependencies>
</project>
```
```xml
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>group-c</groupId>
      <artifactId>artifact-b</artifactId>
      <!-- 这不是jar类型的依赖，所以要明确声明type -->
      <type>war</type>
    </dependency>
 
    <dependency>
      <groupId>group-a</groupId>
      <artifactId>artifact-b</artifactId>
      <!-- 这不是jar类型的依赖，所以要明确声明type -->
      <type>bar</type>
    </dependency>
  </dependencies>
</project>
```

**注意**：最简单的匹配依赖的属性集合应该是{groupId, artifactId, type, classifier}，大多数情况下，依赖都是jar类型并且classifier为null，所以允许我们只使用{groupId, artifactId}，默认的type为jar并且classifier为null

另一点依赖管理的功能，也是非常重要的一点是控制依赖传递中使用的版本：

项目A：
```xml
<project>
 <modelVersion>4.0.0</modelVersion>
 <groupId>maven</groupId>
 <artifactId>A</artifactId>
 <packaging>pom</packaging>
 <name>A</name>
 <version>1.0</version>
 <dependencyManagement>
   <dependencies>
     <dependency>
       <groupId>test</groupId>
       <artifactId>a</artifactId>
       <version>1.2</version>
     </dependency>
     <dependency>
       <groupId>test</groupId>
       <artifactId>b</artifactId>
       <version>1.0</version>
       <scope>compile</scope>
     </dependency>
     <dependency>
       <groupId>test</groupId>
       <artifactId>c</artifactId>
       <version>1.0</version>
       <scope>compile</scope>
     </dependency>
     <dependency>
       <groupId>test</groupId>
       <artifactId>d</artifactId>
       <version>1.2</version>
     </dependency>
   </dependencies>
 </dependencyManagement>
</project>
```

项目B：
```xml
<project>
  <parent>
    <artifactId>A</artifactId>
    <groupId>maven</groupId>
    <version>1.0</version>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <groupId>maven</groupId>
  <artifactId>B</artifactId>
  <packaging>pom</packaging>
  <name>B</name>
  <version>1.0</version>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>test</groupId>
        <artifactId>d</artifactId>
        <version>1.0</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>test</groupId>
      <artifactId>a</artifactId>
      <version>1.0</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>test</groupId>
      <artifactId>c</artifactId>
      <scope>runtime</scope>
    </dependency>
  </dependencies>
</project>
```
当maven在项目B中执行时，abcd都会使用他们在pom中声明的1.0的版本
* a和c都是明确声明了版本，所以由依赖调解（Dependency Mediation）决定使用1.0的版本，并且都因为明确声明而有runtime的范围
* b在B的父项的依赖管理中定义，并且由于依赖管理优先于依赖调解，所以会选择1.0的版本。b也将具有 compile 的范围
* 因为d在B的依赖管理中声明，所以会选择1.0的版本，因为依赖管理的优先级比依赖调解要高，而且当前POM中的声明比父级POM中的声明优先级要高。

## 依赖导入（Importing Dependencies）
使用这个功能前请慎重考虑，只在2.0.9+版本的依赖中有效，推荐你用插件限制强制使用2.0.9+的版本

前面的几个例子演示了通过继承表示依赖管理。但是大部分项目都是通过一对一的继承来完成依赖管理，所以，我们可以通过定义import范围声明一个POM实体

Project B：
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>maven</groupId>
  <artifactId>B</artifactId>
  <packaging>pom</packaging>
  <name>B</name>
  <version>1.0</version>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>maven</groupId>
        <artifactId>A</artifactId>
        <version>1.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <dependency>
        <groupId>test</groupId>
        <artifactId>d</artifactId>
        <version>1.0</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>test</groupId>
      <artifactId>a</artifactId>
      <version>1.0</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>test</groupId>
      <artifactId>c</artifactId>
      <scope>runtime</scope>
    </dependency>
  </dependencies>
</project>
```

假设A是定义好的POM，所有A管理的依赖都会被并入项目B中，除了定义在B中的D。

Project X：
```xml
<project>
 <modelVersion>4.0.0</modelVersion>
 <groupId>maven</groupId>
 <artifactId>X</artifactId>
 <packaging>pom</packaging>
 <name>X</name>
 <version>1.0</version>
 <dependencyManagement>
   <dependencies>
     <dependency>
       <groupId>test</groupId>
       <artifactId>a</artifactId>
       <version>1.1</version>
     </dependency>
     <dependency>
       <groupId>test</groupId>
       <artifactId>b</artifactId>
       <version>1.0</version>
       <scope>compile</scope>
     </dependency>
   </dependencies>
 </dependencyManagement>
</project>
```

Project Y：
```xml
<project>
 <modelVersion>4.0.0</modelVersion>
 <groupId>maven</groupId>
 <artifactId>Y</artifactId>
 <packaging>pom</packaging>
 <name>Y</name>
 <version>1.0</version>
 <dependencyManagement>
   <dependencies>
     <dependency>
       <groupId>test</groupId>
       <artifactId>a</artifactId>
       <version>1.2</version>
     </dependency>
     <dependency>
       <groupId>test</groupId>
       <artifactId>c</artifactId>
       <version>1.0</version>
       <scope>compile</scope>
     </dependency>
   </dependencies>
 </dependencyManagement>
</project>
```

Project Z：
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>maven</groupId>
  <artifactId>Z</artifactId>
  <packaging>pom</packaging>
  <name>Z</name>
  <version>1.0</version>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>maven</groupId>
        <artifactId>X</artifactId>
        <version>1.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <dependency>
        <groupId>maven</groupId>
        <artifactId>Y</artifactId>
        <version>1.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
</project>
```

上面的例子中，Z从X和Y中导入了依赖管理。因为XY都有a的依赖，所以会选择在X中先定义并且没有在Z中定义的1.1版本使用。

这个过程是递归的，如果X引入了项目Q，当Z在导入时就会发现Q中所有的依赖都会在X中出现

当import被用来定义多项目构建部分的相关库时效率更高，项目可以很方便的从库中使用实体，但是会有项目和库中定义的依赖版本不一样的问题，下面的例子说明了“bill of materials”（BOM）是如何使用的

根项目就是BOM的pom，定义了在库中创建的实体的版本，其他想要使用这个库的应该先在自己pom的dependencyManagement节点中导入这个pom

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.test</groupId>
  <artifactId>bom</artifactId>
  <version>1.0.0</version>
  <packaging>pom</packaging>
  <properties>
    <project1Version>1.0.0</project1Version>
    <project2Version>1.0.0</project2Version>
  </properties>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>com.test</groupId>
        <artifactId>project1</artifactId>
        <version>${project1Version}</version>
      </dependency>
      <dependency>
        <groupId>com.test</groupId>
        <artifactId>project2</artifactId>
        <version>${project1Version}</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <modules>
    <module>parent</module>
  </modules>
</project>
```

把BOM的pom当父级的parent子项目，是一个正常的多项目pom：
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.test</groupId>
    <version>1.0.0</version>
    <artifactId>bom</artifactId>
  </parent>
 
  <groupId>com.test</groupId>
  <artifactId>parent</artifactId>
  <version>1.0.0</version>
  <packaging>pom</packaging>
 
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.12</version>
      </dependency>
      <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.1.1</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <modules>
    <module>project1</module>
    <module>project2</module>
  </modules>
</project>
```

实际的项目pom：
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.test</groupId>
    <version>1.0.0</version>
    <artifactId>parent</artifactId>
  </parent>
  <groupId>com.test</groupId>
  <artifactId>project1</artifactId>
  <version>${project1Version}</version>
  <packaging>jar</packaging>
 
  <dependencies>
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
    </dependency>
  </dependencies>
</project>
```

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.test</groupId>
    <version>1.0.0</version>
    <artifactId>parent</artifactId>
  </parent>
  <groupId>com.test</groupId>
  <artifactId>project2</artifactId>
  <version>${project2Version}</version>
  <packaging>jar</packaging>
 
  <dependencies>
    <dependency>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
    </dependency>
  </dependencies>
</project>
```

下面的项目展示了如何在项目中使用这个库并且不用声明依赖的项目的版本：
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.test</groupId>
  <artifactId>use</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>
 
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>com.test</groupId>
        <artifactId>bom</artifactId>
        <version>1.0.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.test</groupId>
      <artifactId>project1</artifactId>
    </dependency>
    <dependency>
      <groupId>com.test</groupId>
      <artifactId>project2</artifactId>
    </dependency>
  </dependencies>
</project>
```

创建导入依赖的项目时注意下面的问题：
* 不要试着导入定义在当前pom的子模块的pom，这回导致构建失败因为无法定义pom的位置
* 不要循环引用pom，即A，B互相import
* 项目中涉及到有依赖传递的实体的时候，需要通过依赖管理声明实体的版本，不然会出现构建错误，有个最好的解决办法就是让实体在构建过程中一直保持着版本（不太懂这个点）

## 系统依赖
**需要注意，这种方法不赞成使用**

系统范围的依赖一直可用并且不会在仓库中查找，它们通常定义的是由JDK或者JVM提供的依赖，常用的例子就是JDBC标准扩展和 Java Authentication and Authorization Service (JAAS)

简单的例子：
```xml
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>javax.sql</groupId>
      <artifactId>jdbc-stdext</artifactId>
      <version>2.0</version>
      <scope>system</scope>
      <systemPath>${java.home}/lib/rt.jar</systemPath>
    </dependency>
  </dependencies>
  ...
</project>
```

如果你用的实体定义在JDK的tools.jar包里：
```xml
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>sun.jdk</groupId>
      <artifactId>tools</artifactId>
      <version>1.5.0</version>
      <scope>system</scope>
      <systemPath>${java.home}/../lib/tools.jar</systemPath>
    </dependency>
  </dependencies>
  ...
</project>
```

---

原文地址：[http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html]()

非全文翻译，水平有限，仅供参考，欢迎交流~~