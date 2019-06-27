---
layout:     post
title:      SpringMVC·form表单Date类型问题导致的400问题
subtitle:   在实际工作中会经常遇到各种传值类型导致的问题，在此记录一个date类型问题。
date:       2019-06-27
author:     lwj108
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - java
    - code
---
## 问题描述
前端传yyyy-MM-dd hh:mm:ss格式的时间其实是String类型导致JavaBean中的Date类型Setter报错，从而导致api请求400.

## 问题解决
### 我的解决方式：
在对应的实体类的对应的非字符串类型的变量的setter方法中传入string类型的，然后在里边用SimpleDateFormat或者Integer进行转化
```java
public void setReleaseEndTime(String str) {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    Date releaseEndTime;
    try {
        releaseEndTime = sdf.parse(str);
        this.releaseEndTime = releaseEndTime;
    } catch (ParseException e) {
        e.printStackTrace();
    }
}
```

### 网上有：

* 1、把实体类的javabean里边的类型都改成string类型了，在配置SQL语句时用数据库函数to_date或者to_number转化的，如果再java中用到这个字符串类型的日期的话，有必要的话，就用For format=new SimpleDateFormat("yyyy-MM-dd"),format.parse()来转换。

* 2、还可以在实体类中定义Date和int类型对应的字符串类型成员变量,这样前台的表单中field或者name与之对应上即可，这样也成功转成实体类了，不过转成之后，得在java中把它字符串类型的转成对应的Date或者int类型赋给相应的成员变量即可。

* 3、最后还有一种方法，就是实体类的日期属性上加@DateTimeFormat(pattern="yyyy-MM-dd")注解，大部分是可以成功使用的。如果这种方法不可用的话，你看继续尝试如下方法：不过这个前提是前台穿过的日期为json形式而非字符串形式，如前台类似$("#id").val()来获取日期直接传给后台的话是不行的，你需要在前台引入JSON官网的json.js库或者引入jQuery的jquery.json-2.4.js库，然后如果是前者的话就new Date(stringDate).parseJSON()来转化再传给后台，如果是后者的话，$.toJSON(new Date(stringdate))来传给后台，这种方式比较麻烦，有网友留言特意讨论了一下这个问题，所以建议采用第一种方式。
 
**参考文档**：
[SpringMVC中出现" 400 Bad Request "错误（用@ResponseBody处理ajax传过来的json数据转成bean）的解决方法](https://blog.csdn.net/chenleixing/article/details/43740759  "参考文章")