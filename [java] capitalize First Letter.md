---
title: "【Java】将字符串中的首字母转为大写或小写"
date: 2023-06-08T11:19:11+09:00
draft: false
categories:
- IT
tags:
- java
---



定义一个 `StringUtils` 类，并在其中定义两个静态方法 `capitalizeFirstLetter()` 和 `uncapitalizeFirstLetter()`。这两个方法分别接受一个字符串参数，并返回首字母大写和首字母小写的结果。

```java
public class StringUtils {
    public static String capitalizeFirstLetter(String str) {
        if (str == null || str.isEmpty()) {
            return str;
        }
        
        return str.substring(0, 1).toUpperCase() + str.substring(1);
    }
    
    public static String uncapitalizeFirstLetter(String str) {
        if (str == null || str.isEmpty()) {
            return str;
        }
        
        return str.substring(0, 1).toLowerCase() + str.substring(1);
    }
    
    public static void main(String[] args) {
        String str = "hello world";
        
        String capitalized = capitalizeFirstLetter(str);
        System.out.println("Capitalized: " + capitalized);
        
        String str2 = "Hello World";
        String uncapitalized = uncapitalizeFirstLetter(str2);
        System.out.println("Uncapitalized: " + uncapitalized);
    }
}
```

运行代码后，将输出以下结果：

```tex
Capitalized: Hello world
Uncapitalized: hello world
```

