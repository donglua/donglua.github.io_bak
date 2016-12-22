+++
title = "Dex 文件结构"
draft = false
date = "2016-11-16T23:13:34+08:00"
+++

<!--more-->
## 1. Dex 文件

* Dex（Dalivk Executable）文件是 Android 系统可执行文件，包含应用程序的全部操作指令以及运行时数据。
* 虚拟机在执行一个应用程序时，会实时地根据程序需要从 Dex 文件中读取相应的数据以保证程序的正确运行。
* 为了运行在资源有限的 Android 系统中，标准的 Java 文件在经过编译后，还需要通过 dx 工具将编译生成的 class 文件整合成一个 Dex 文件。这样使得文件结构十分紧凑，减少了冗余，也提高了类的查找速度。
![java to dex](http://ogkb67oc8.bkt.clouddn.com/Groupjava_to_dex.png)

如何将```Hello.java```编译成一个 .dex 文件：
```java
pubic class Hello {
  public static void main(String[] args) {
    System.out.println("Hello, world");      
  }
}
```

执行 ```javac Hello.java``` 生成 Hello.class 文件

执行 ```dx --dex --output=Hello.dex Hello.class``` 把 Hello.class 编译成 Hello.dex 文件


## 2.Dex 文件的结构和特点

![dex file](http://ogkb67oc8.bkt.clouddn.com/dex_file.png)

Dex 文件的结构，实际上就是由多个不同的结构体数据以首尾相接的方式拼凑而成。Dex 文件的定义在源码中的```AOSP/dalvik/libdex/DexFile.h```文件中，主要结构如下：

```C++
struct DexFile {
  const DexHeader*    pHeader;
  const DexStringId*  pStringIds;
  const DexTypeId*    pTypeIds;
  const DexFieldId*   pFieldIds;
  const DexMethodId*  pMethodIds;
  const DexProtoId*   pProtoIds;
  const DexClassDef*  pClassDefs;
  const DexLink*      pLinkData;

  ...
};
```

首先介绍一下各部分数据的排列方式和主要功能：

| 数据名称 	| 含义   
|:--------|:------|
| header      	| 文件的头部，记录 Dex 文件的相关属性|  
| strnig\_ids 	| 字符数据索引，记录了各个字符在数据区的地址偏移量| 　
| type\_ids 	| 类型数据索引，记录了各个类型的字符串索引| 　
| proto\_ids   	| 原型数据索引，记录方法声明的字符串、返回类型字符串、参数列表| 　
| field\_ids  	| 字段数据索引，记录了所属类，声明类型以及方法名等信息| 　
| method\_ids 	| 类方法声明，记录方法所属类名、方法声明、以及该方法名等信息| 　
| class\_defs 	| 类定义数据，记录了指定类各类信息，包括接口、超类、类数据偏移量等 | 　
| link\_data	 	| 连接数据区 |

## 3. 各部分结构简介
### 3.1 header

* header 是 Dex 文件的文件头，记录了 Dex 文件的基本信息及大致的数据分布。
* 每一项数据所占用的内存空间是相应固定的，总长度固定为 0x70。
* 虚拟机在处理 Dex 文件时，只需根据固定的规则读取文件头，即可获取目标 Dex 文件的大致信息。

### 3.2 string\_ids
* 这一区域存储的是 Dex 文件字符串资源的索引信息，也就是在 Dex 文件数据区所在的真实物理偏移量。
* 在 Dex 文件中, 每个字符串都对应一个 DexStringId 数据结构, 该数据结构也是在 ```dalvik/libdex/DexFile.h```文件中定义，只有一个成员变量。

```cpp
struct DexStringId {
  u4 stringDataOff; /* 在Dex文件中的实际偏移量 */
};
```
* 读取数据的时候，将 Dex 文件在内存的起始地址加上这个偏移量就得到该字符串在内存中的物理地址。

### 3.3 type\_ids
* 这一区域存储的是类型资源的索引信息。
* ```dalvik/libdex/DexFile.h``` 也定义了一个 DexTypeId 来存储这部分信息。

```cpp
struct DexTypeId {
  u4  descriptorIdx; /* 指向字符串索引表 */
};
```
* 在 Dex 文件中, 类型是以字符串的形式保存在数据区中的, 因此, ```descriptorIdx``` 所保存的是目标类型在字符串索引表中的序列号。

### 3.4 proto\_ids
* 存储的是方法原型（method prototype）资源的索引信息。
* 对应源码中的```DexProtoId```数据结构，前两个变量代表的真实资源都是一个字符串，分别表示方法的声明和返回类型。parametersOff 的真实资源是一个 DexTypeList 数据结构。


```cpp
struct DexProtoId {
    u4  shortyIdx; /* 方法声明字符串，指向字符串索引表s */
    u4  returnTypeIdx; /* 方法返回类型，指向字符串索引表 */
    u4  parametersOff; /* 参数列表，指向DexTypeList数据结构的偏移地址 */
};

struct DexTypeItem {
    u2  typeIdx; /* 参数的类型，指向 DexTypeId 索引表 */
};

struct DexTypeList {
    u4  size;  /* DexTypeItem 的个数 */
    DexTypeItem list[1]; /* DexTypeItem 数据结构列表 */
};
```

* ```DexProtoId```的成员变量都是表示数据的偏移地址，所以即使```DexTypeItem```数量不确定，```DexProtoId```是一个固定值12B。

### 3.5 field\_ids
* 存储的是字段资源的索引信息。
* 对应源码中的```DexFieldId```数据结构

```cpp
struct DexFieldId {
    u2  classIdx; /* 所属类，指向一个DexTypeId */
    u2  typeIdx; /* 该字段类型，指向一个DexTypeId */
    u4  nameIdx; /* 字段名，指向一个DexStringId */
};
```
### 3.6 method\_ids
* 存储的是类方法数据的索引信息。
* 对应源码中的```DexMethodId```数据结构，

```cpp
struct DexMethodId {
    u2  classIdx; /* 所属类 */
    u2  protoIdx; /* 方法原型类型，指向一个DexProtoId */
    u4  nameIdx; /* 方法名，指向一个DexStringId */
};
```

### 3.7 class\_defs
* 存储了一个类所需的资源。
* 对应源码的```DexClassDef```数据结构

```cpp
struct DexClassDef {
  u4  classIdx;  /* 所属类，指向一个DexTypeId */
  u4  accessFlags; /* 访问标示符 */
  u4  superclassIdx; /* 超类，指向一个DexTypeId */
  u4  interfacesOff; /* 实现的接口，指向一个DexTypeList */
  u4  sourceFileIdx; /* 源文件名，指向一个DexStringId */
  u4  annotationsOff; /* file offset to annotations_directory_item */
  u4  classDataOff; /* 类数据，指向一个DexClassData */
  u4  staticValuesOff; /* file offset to DexEncodedArray */
};
```

## 参考资料

[http://www.blogfshare.com/dex-format.html](http://www.blogfshare.com/dex-format.html)

[https://www.zybuluo.com/dodola/note/554061](https://www.zybuluo.com/dodola/note/554061)

[https://source.android.com/devices/tech/dalvik/dex-format.html](https://source.android.com/devices/tech/dalvik/dex-format.html)
