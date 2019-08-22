# StringUtils 工具类使用 

> 工具类：`org.apache.commons.lang3.StringUtils`   
> 版本：`commons-lang3:3.7`  
> 地址：<a href="https://commons.apache.org/proper/commons-lang/javadocs/api-release/index.html" target="_blank">StringUtils API</a>  
> 说明：StringUtils 是工作中使用最频繁的一个工具类，提供了丰富的字符串操作方法，如替换、判等、截取、计数等，每个类型都重载了很多方法，且不会抛出空指针异常。

## 概览

大部分方法都提供了很多重载方法，比如不区分大小写`xxIgnoreCase`、匹配任何`xxAny`、还有参数类型不同、参数个数不同的重载等，能适应绝大多数业务场景的需求。

![](./img/StringUtils.jpg)

## 静态常量

StringUtils 提供了最常用的常量如下，使用定义的常量能让我们的代码更清晰。  

![](./img/StringUtils-Constants.jpg)

## 花式截取字符串

```java
@Test
public void testSubstring() {
    // 最基本的截取
    System.out.println(  StringUtils.substring("abcde", 1, 2)  ); // b
    // 截取前半部分
    System.out.println(  StringUtils.substringBefore("role/site/admin-(1.site.1)", "-(")  ); // role/site/admin
    // 截取后半部分
    System.out.println(  StringUtils.substringAfter("role/site/admin-(1.site.1)", "-")  ); // (1.site.1)
    // 截取之间的
    System.out.println(  StringUtils.substringBetween("role/site/admin-(1.site.1)", "(", ")")  ); // 1.site.1
    // 截取左边的部分
    System.out.println(  StringUtils.left("aabbcc", 4)  ); // aabb
    // 截取右边的部分
    System.out.println(  StringUtils.right("aabbcc", 4)  ); // bbcc
    // 截取中间部分
    System.out.println(  StringUtils.mid("aabbcc", 1, 2)  ); // ab
}
```

## 各种判断空与非空

```java
@Test
public void testIsEmpty() {
    // 判断是否为空 只有为 null 或 "" 的时候才会true
    System.out.println(  StringUtils.isEmpty("  ")  );  // false
    // 判断不为空
    System.out.println(  StringUtils.isNotEmpty("  ")  ); // true
    // 判断是否为空 null、""、全空格 都为true
    System.out.println(  StringUtils.isBlank("  ")  ); // true
    // 判读不为空
    System.out.println(  StringUtils.isNotBlank("  ")  ); // false
    // 任何一个为空
    System.out.println(  StringUtils.isAnyEmpty("ab", " ")  ); // false
    // 任何一个为空
    System.out.println(  StringUtils.isAnyBlank("ab", "  ")  ); // true
    // 没有为空的
    System.out.println(  StringUtils.isNoneEmpty("ab", "")  ); // false
    // 所有的都为空
    System.out.println(  StringUtils.isAllBlank("ab", "  ")  ); // false
}
```

## 判断是否相等

```java
@Test
public void testEqual() {
    // 判断是否相等 区分大小写
    System.out.println(  StringUtils.equals("ab", "Ab")  ); // false
    // 判断是否相等 不区分大小写
    System.out.println(  StringUtils.equalsIgnoreCase("ab", "Ab")  ); // true
    // 判断其中一个相等
    System.out.println(  StringUtils.equalsAny("ab", "Ab", "ab", "abc")  ); // true
    // 判断其中一个相等
    System.out.println(  StringUtils.equalsAnyIgnoreCase("ab", "Ab", "abc")  ); // true
}
```

## 补齐

```java
@Test
public void testPad() {
    // 左边补齐
    System.out.println(  StringUtils.leftPad("abc", 5, "x")  ); // xxabc
    // 右边补齐
    System.out.println(  StringUtils.rightPad("abc", 5, "x")  ); // abcxx
    // 两边补齐
    System.out.println(  StringUtils.center("abc", 5, "x")  ); // xabcx
}
```

## 缩短省略
```java
@Test
public void testAbb() {
    // 共计10位数 超出省略 默认 ...
    System.out.println(  StringUtils.abbreviate("abcdefghijklmn", 10)  ); // abcdefg...
    // 指定省略符
    System.out.println(  StringUtils.abbreviate("abcdefghijklmn", "***", 10)  ); // abcdefg***
    // 两边省略
    System.out.println(  StringUtils.abbreviate("abcdefghijklmn", 5, 10)  ); // ...fghi...
}

```

## 去掉控制字符和指定字符
```java
@Test
public void testTrim() {
    // trim 主要用于去掉控制字符 ,即ASC码表小于等于32的字符，strip主要用于去掉指定字符，默认空格

    // 去掉两边空白
    System.out.println("[" + StringUtils.trim(" ab cd ") + "]"); // [ab cd]
    // trim 去掉的是ASC码表小于等于32的字符(特殊字符)
    System.out.println("[" + StringUtils.trim(" \n ab cd ") + "]"); // [ab cd]
    // trim 去掉的是ASC码表小于等于32的字符(特殊字符)
    System.out.println("[" + StringUtils.trim(" ab \n cd ") + "]"); // [ab \n cd]
    // 如果为空则返回null
    System.out.println("[" + StringUtils.trimToNull("    ") + "]"); // [null]
    // 如果为空返回空
    System.out.println("[" + StringUtils.trimToEmpty(null) + "]"); // []

    // strip 行为和trim基本一样，可以指定去掉的字符串
    char ch = 30;
    System.out.println("[" + StringUtils.strip(ch+"  ab cd ") + "]"); // [ab cd]
    // 去掉开头的字符，null默认为空格
    System.out.println("[" + StringUtils.stripStart("  ab cd  ", null) + "]"); // [ab cd  ]
    // 去掉开头的字符，null默认为空格
    System.out.println("[" + StringUtils.stripStart("ab cd  ", "a") + "]"); // [b cd  ]
    // 去掉末尾的空格
    System.out.println("[" + StringUtils.stripEnd("  ab cd  ", null) + "]"); // [  ab cd]
}
```



