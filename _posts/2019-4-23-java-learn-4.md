---
layout: post
title: java 类文件结构
tags: java note
---

# Java 类文件结构

## 无关性的基石

Java虚拟机有两个无关性，即平台无关性和语言无关性。

平台无关性指Java程序可以在各种平台执行，语言无关性指Java虚拟机可以支持其他语言运行。

语言无关性的基础是虚拟机和字节码存储格式。

## Class类文件的结构

Class文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑排列在Class文件中。

根据Java虚拟机规范的规定，Class文件格式采用类似C语言结构体的为机构来存储，这种伪结构中只有两种数据类型：无符号数和表。

无符号数属于基本的数据类型，以u1、u2、u4、u8表示1、2、4、8个字节的无符号数。

表是有多个无符号数或其他表作为数据项构成的复合数据类型，所有表都习惯性地以`_info`结尾。表用于描述有层次关系的复合结构的数据，整个Class文件本质上就是一张表，由下图所示数据项构成。

>     ClassFile {
>         u4             magic;  
>         u2             minor_version;  
>         u2             major_version;  
>         u2             constant_pool_count;    
>         cp_info        constant_pool>  [ constant_pool_count-1];
>         u2             access_flags;
>         u2             this_class;
>         u2             super_class;
>         u2             interfaces_count;
>         u2             interfaces[ interfaces_count]>;
>         u2             fields_count;
>         field_info     fields[fields_count];
>         u2             methods_count;
>         method_info    methods[methods_count];
>         u2             attributes_count;
>         attribute_info attributes[ attributes_count]>;
>     }

当描述同一类型但数量不定的多个数据是，通常会使用一个前置的容量计数器加若干个连续数据项的形式，这一系列连续的某一类型数据为某一类型的集合。

### 魔数与Class文件版本

每个Class文件前四个字节称为魔数（magic），由于确定是否为能被接受的Class文件。紧接着的四个字节存储的Class文件的版本号，第5、6个字节是次版本号（minor_version），7、8个字节是主版本号（major_version）。Java版本号从45开始，每个大版本发福主版本号加一，高版本JDK向下兼容Class文件，但不向上兼容未来办恶补呢Class文件。

### 常量池

紧接着版本号之后的是常量池入口(constant_pool)。

由于常量池中常量数目不固定，所以在常量池入口需要放置一个u2类型的数据，代表常量池容量计数值（constant_pool_count）。这个容量技术从1开始（即1代表0个常量）。这是为了使得后面指向常量池索引值的数据在特定情况下可以用索引值0表示不引用常量池项目。class文件只有常量池容量计数从1开始。

常量池主要存放两类常量：**字面值**和**符号引用**。

字面值类似Java语言层面的常量，如文本字符串、被声明为final的常量值等。

符号引用属于编译的概念，包括：
- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

常量池的每一项常量都是一个表，共有11中结构不同的表结构，他们的第一位都是一个u1类型的标志位，代表这个常量属于哪种常量类型，其含义如下。


>     Constant Type	Value
>     CONSTANT_Class	7
>     CONSTANT_Fieldref	9
>     CONSTANT_Methodref	10
>     CONSTANT_InterfaceMethodref	11
>     CONSTANT_String	8
>     CONSTANT_Integer	3
>     CONSTANT_Float	4
>     CONSTANT_Long	5
>     CONSTANT_Double	6
>     CONSTANT_NameAndType	12
>     CONSTANT_Utf8	1
>     CONSTANT_MethodHandle	15
>     CONSTANT_MethodType	16
>     CONSTANT_InvokeDynamic	18

由于Class文件中方法、字段等都需要引用`CONSTANT_Utf8_info`型常量描述名称，所以当名称超过64KB时会无法编译。

其具体类型和描述此处略去。详见[JVM规范-Class文件格式](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.4)。

### 访问标志

常量池之后的两个字节表示访问标志（access_flag），用来识别类或接口层次的访问信息，类或接口的属性以及访问权限等。

### 类索引、父索引、接口索引集合

类索引（this_class）和父类索引（super_class）是u2的数据，接口索引集合（interfaces）是一组（u2）类型数据集合。

这三个数据用来确定这个类的继承关系。从这里也可以看出，Java不允许多重继承。

类索引和父类索引分别指向一个CONSTANT_Class_info常量，通过该常量查找全限定名字符串。

### 字段表集合

字段表（field_info）用于描述接口或类中声明的变量。

其结构如下：
>     field_info {
>         u2             access_flags;
>         u2             name_index;
>         u2             descriptor_index;
>         u2             attributes_count;
>         attribute_info attributes[attributes_count];
>     }

字段（field）包括了类级变量或实例级变量，但不包括方法内部声明的变量。

字段一般包含以下信息：
- 作用域（public等修饰符）。
- 类级还是实例级（static修饰符）。
- 可变性（final）。
- 并发可见（volatile）。
- 可否序列化（transient）。
- 字段数据类型。
- 字段名称等。

上面所写的前五项包括是否由编译器生成，是否是枚举类都包含在access_flags里，这些都是字段的修饰符。

在access_flags之后的是name_indedx和descriptor_index，这两个都是对常量池的引用，分别代表字段的简单名称和字段及方法的描述符。

----
这里简单介绍一下全限定名、简单名称、描述符的概念。

全限定名：
类似“com/java/classz/TestClass”这样的名称，只是把类全名中的“.”替换成了“/”而已。一般会在最后加入一个“;”表示结束。    
简单名称：
没有类型和参数修饰的方法或字段名称，如String.equal(String s)的简单名称是equal。     
描述符：
它的作用是描述字段的数据类型和方法的参数列表以及返回值。其格式是先参数列表，后返回值，参数列表放在()之内，返回值紧跟其后。   
这里附上官方文档的[描述符语法](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.3.2)，这里使用的是ll语法，应该是为了实现简单。

----

descriptor之后是attributes，用于储存一些额外的信息，这部分内容留在之后介绍。

字段表集合中不会列出结成的字段，但可能会增加一些字段，例如内部类为了访问外部类，可能会有外部类的指针。在Java中，字段也是无法重载的，不过比较tricky的是，尽管在Java语言中字段的名称必须不同，但是对字节码来说，描述符不一致的情况，字段重名是合法的。

### 方法表集合

这里结构几乎和字段表一致，其结构如下。
>     method_info {
>         u2             access_flags;
>         u2             name_index;
>         u2             descriptor_index;
>         u2             attributes_count;
>         attribute_info attributes[attributes_count];
>     }


这里的access_flags也包含了对于方法可用的修饰符，与字段不同，这里没有volatile和transient，而多了注入synchronized和strictfp等修饰符。

方法的代码放置在方法属性表集合(attributes)中的code属性里，这个表的具体结构也放在之后讲解。

若子类不重写父类方法，则方法表集合不出现父类方法信息，这里也会出现编译器自动添加的方法。

重载方法需要具有不同的特征签名。在Java语言中： 特征签名 = 方法名 + 参数类型 + 参数顺序；在Class文件：特征签名 = 方法名 + 参数类型 + 参数顺序 + 返回值类型 = 方法名 + 描述符。也就是说，只要返回值不同，那么两个方法就可以共存。

### 属性表集合

属性表(attribute_info)在之前出现过多次，Class文件中、字段表集合中、方法表集合中。他的结构如下：

>     attribute_info {
>         u2 attribute_name_index;
>         u4 attribute_length;
>         u1 info[attribute_length];
>     }

他的限制更小一些，任何人都可以向其中写入自己定义的属性信息，JVM在运行时会忽略掉他不认识的属性。

为了正确解析，JVM规范预先定义了一些[属性](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7)。

具体的讲解查询JVM规范，这里就不过多介绍了。
