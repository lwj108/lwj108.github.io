---
layout:     post
title:      java8 String与list互转方法
subtitle:   java8 stream()与String.join()可以极大简洁的编码🤓
date:       2019-06-26
author:     lwj108
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - java
    - code
---
```java
public void test() {
    //字符串转list<String>
    String str = "测试1,测试2，测试3，测试4";
    //此处为了将字符串中的空格去除做了一下操作
    List<String> list= Arrays.asList(str .split(",")).stream().map(s -> (s.trim())).collect(Collectors.toList());
    //list<String>转字符串（以逗号隔开）
    System.out.println(String.join(",", list));
}
```