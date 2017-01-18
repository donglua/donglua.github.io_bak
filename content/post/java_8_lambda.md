+++
draft = false
date = "2017-01-18T14:40:44+08:00"
title = "Java 8 深入剖析 | Lambda 表示式"

+++

# 1. Lambda 表达式介绍

Lambda 表达式为 Java 添加了缺失的 函数式编程特性，使我们能将函数当作一等公民对待。

在将函数作为一等公民的语言中，Lambda 表达式的类型是函数。而在 Java 中，__Lambda 表达式是对象__，他们必须依附于一类特别的对象类型 —— __函数式接口（FunctionalInterface）__。

> 函数式接口
* 如果一个接口只有一个抽象方法，那么该接口是一个函数式接口
* 如果我们在某个接口上声明了 FunctionalInterface 注解，那么编译器就会按照函数式接口的定义来要求该接口
* 如果一个接口只有一个抽象方法，但没有给该接口声明 FunctionalInterface 注解，那么编译器依旧会将该接口看成函数式接口

Lamdba 表达式是一种匿名函数；它是没有声明的方法，即没有访问修饰符、返回值声明和名字。

Lambda 表达式的作用是传递行为，而不仅仅是值。提升了抽象层次，API 重用性更好，更加灵活。

# 2. Lambda 表示式基本语法

```Java
(arg1, arg2...) -> { body }

(type1 arg1, type2 arg2...) -> { body }
```
示例：

* ``` (int a, int b) -> { return a + b; }```
* ```(String s) -> System.out.println(s);```
* ```a -> return a * a;```
* ```() -> 43;```
* ```() -> { return 3.14; } ```
> 1. 一个 Lambda 表达式可以有零个或多个参数
> 2. 参数的类型可以明确声明，也可以根据上下文来推断。比如```(String s)```可以省略为```(s)```
> 3. 所有参数须包含在圆括号内，参数之间用逗号相隔。当只有一个参数且其类型可推导时，圆括号可省略，比如 ```a -> return a * a;```
> 4. 空圆括号代表参数集为空。比如 ```() -> 43;```
> 5. Lambda 表达式的主体可包含零条或多条语句。如果 Lambda 表达式的主体只有一条语句，花括号```{}```可省略。匿名函数的返回值类型与该主体表达式一致。
> 6. 如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号```{}```中（形成代码块）。匿名函数的返回值类型与代码块的返回类型一致，若没有则为空。

# 3. 方法引用 
> 方法引用（Method References）实际上是 Lambda 表达式的一种语法糖。我可以将方法引用看作是一个“函数指针”（指向一个函数的指针）。

例如：
```Java
list.forEach(item -> System.out.println(item))
```
可以改为使用方法引用：
```Java
list.forEach(System.out::println)
```

## 方法引用的四种类型：

1. 类名::静态方法名
```Java
public class Person {
    String name;
    LocalDate birthday;
    public static int compareByAge(Person a,Person b){
        return a.birthday.compareTo(b.birthday);
    }
}
```
```Java
Arrays.sort(people, Person::compareByAge);
```
2. 对象::实例方法名
```Java
list.forEach(System.out::println)
```
3. 类名::实例方法名
```Java
String[] cities = { "guangdong", "hunan", "hebei", "beijing" };
Arrays.sort(cities, String::compareToIgnoreCase);
```
4. 构造方法引用
```Java
String::new
```
