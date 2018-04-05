title: List 进行 remove 操作时抛出 java.lang.UnsupportedOperationException 异常分析
date: 2016-5-13  
tags:
    - original
categories:
    - Java
---

今天将一个数组转换成 List 然后进行 remove 操作时却抛出 java.lang.UnsupportedOperationException 异常。

```java
String pattern = " ^, v, m, n-music-name, $ ";
String[] patternSplit = Utils.getStringTrimSplit(pattern, ",");

// 去除模式中的^和$标识
List<String> natureList = Arrays.asList(patternSplit);
if ("^".equals(natureList.get(0))) {
    natureList.remove(0); // throw java.lang.UnsupportedOperationException
}
if ("$".equals(natureList.get(natureList.size() - 1))) {
    natureList.remove(natureList.size() - 1); // throw java.lang.UnsupportedOperationException
}

String[] natureArray = natureList.toArray(new String[natureList.size()]);
System.out.println(natureArray.length);
```

<!-- more -->

看了下源码才发现使用 Arrays.asList(arr) 转换的 List 并不能进行 add 和 remove 操作。  
Arrays.asList(arr) 返回的类型是 Aarrays$ArrayList 并不是 ArrayList，  
Aarrays$ArrayList 和 ArrayList 都继承 AbstractList，但是 AbstractList 中的 add 方法和 remove 方法都是直接抛出 UnsupportedOperationException，并没有直接实现。  
ArrayList 重写了 add 方法和 remove 方法,能够进行对应的添加和删除操作，Aarrays$ArrayList 却没有去重写，所以此时调用 add 方法和 remove   方法会抛出 UnsupportedOperationException。 

## 解决办法
```java
// old
List<String> natureList = Arrays.asList(patternSplit);
// new 
List<String> natureList = new ArrayList<>(Arrays.asList(patternSplit));
```

## 源码  
Arrays.asList(arr) 返回 Aarrays$ArrayList ， Aarrays$ArrayList 继承 AbstractList  
![](http://7xia33.com1.z0.glb.clouddn.com/UnsupportedOperationException1.jpg)  
ArrayList 继承 AbstractList  
![](http://7xia33.com1.z0.glb.clouddn.com/UnsupportedOperationException2.jpg)  
AbstractList 中的 add 方法和 remove 方法都直接抛出 UnsupportedOperationException  
![](http://7xia33.com1.z0.glb.clouddn.com/UnsupportedOperationException3.jpg)  
ArrayList 重写 add 方法和 remove 方法  
![](http://7xia33.com1.z0.glb.clouddn.com/UnsupportedOperationException4.jpg)   


<br>