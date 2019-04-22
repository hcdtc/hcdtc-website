+++
title = "RandomUtils工具类"
description = ""
weight = 10
home = true
+++

# RandomUtils工具使用

> 工具类：`org.apache.commons.lang3.ArrayUtils`     
> 版本：`commons-lang3:3.7`    
> 地址：  <a href="https://commons.apache.org/proper/commons-lang/javadocs/api-release/index.html" target="_blank">RandomUtils API</a>  
> 说明：  `RandomUtils`是专门用来生成随机数的工具类，支持5种数字类型(byte、int、long、float、double)和布尔型(boolean)。

## 概览
![](/img/docs/30-development-manual/2-back-end/99-dev-utils/2-common-lang3/RandomUtils-1.png)  

## 随机布尔值
```java  
boolean flag = RandomUtils.nextBoolean(); // true
```  
## 随机字节数组  
> 参数：返回数组的大小  

```java  
byte[] number = RandomUtils.nextBytes(6); // B@5679c6c6
```  
## 随机int型整数
> 1.返回一个0 - Integer.MAX_VALUE之间的随机整数  

```java  
int number = RandomUtils.nextInt(); // 1599277713
```  
> 2.返回一个在指定区间内的int型随机整数  
参数：startInclusive     最小值（包含，非负）  
参数：endExclusive      最大值（不包含）

```java  
int number = RandomUtils.nextInt(20, 60);  // 42
```  
## 随机long型整数
> 1.返回一个0 - Long.MAX_VALUE之间的随机整数  

```java  
long number = RandomUtils.nextLong(); // 2057158075517831168
```  
> 2.返回一个在指定区间内的long型随机整数  
参数：startInclusive     最小值（包含，非负）  
参数：endExclusive      最大值（不包含）  

```java  
long number = RandomUtils.nextLong(34, 68); // 36
```    
## 随机double型浮点数  
> 1.返回一个0 - Double.MAX_VALUE之间的随机浮点数  

```java  
double number = RandomUtils.nextDouble();  // 5.944275232515714E307
```  
> 2.返回一个在指定区间内的double型随机浮点数  
参数：startInclusive     最小值（包含，非负）  
参数：endExclusive      最大值（不包含）  

```java  
double number = RandomUtils.nextDouble(23.0, 34);  // 31.159880330798867
```  
## 随机float型浮点数  
> 1.返回一个0 - Float.MAX_VALUE之间的随机浮点数  

```java  
float number = RandomUtils.nextFloat();  // 1.5128506E37
```  
> 2.返回一个在指定区间内的float型随机浮点数  
参数：startInclusive     最小值（包含，非负）  
参数：endExclusive      最大值（不包含）  

```java  
float number  = RandomUtils.nextFloat(23, 56);  //29.496506
```   


