---
title: 001-java-保留两位小数以及浮点类型精度问题
categories:
  - java-base
abbrlink: 7aa0e1b
date: 2020-02-11 14:24:04
---

摘要：保存成两个小数
<!-- more -->

# 几种方式
```java
        int roundLong = 8710;
        System.out.println((roundLong * 0.01));
        System.out.println((roundLong /100.0));
        // 方式一、DecimalFormat
        DecimalFormat df = new DecimalFormat(".00");
        System.out.println(df.format(roundLong * 0.01));
        // 方式二、String.format
        System.out.println(String.format("%.2f", (roundLong * 0.01)));
        // 方式三、BigDecimal
        BigDecimal bg = new BigDecimal(roundLong * 0.01);
        double d3 = bg.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();
        System.out.println(d3);
        // 方式四：通过NumberFormat类实现
        NumberFormat nf = NumberFormat.getNumberInstance();
        nf.setMaximumFractionDigits(2);
        System.out.println(nf.format(roundLong * 0.01));
```
输出：
```
  87.10000000000001
  87.1
  87.10
  87.10
  87.1
  87.1
```

# 浮点类型精度问题
``` JAVA
        int t2 = 8710;
        System.out.println((t2 * 0.1));
        System.out.println((t2 * 0.01));
        System.out.println((t2 * 0.001));
        System.out.println((t2 * 0.0001));

        t2 = 870;
        System.out.println((t2 * 0.1));
        System.out.println((t2 * 0.01));
        System.out.println((t2 * 0.001));
        System.out.println((t2 * 0.0001));

        double number1 = 1;
        double number2 = 20.2;
        double number3 = 300.03;
        double result = number1 + number2 + number3;
        System.out.println(result);
```
输出
```
871.0
87.10000000000001
8.71
0.871
87.0
8.700000000000001
0.87
0.08700000000000001
321.22999999999996
```
此处涉及，计算机浮点设计问题，详细可查阅资料。
float和double只能用来做科学计算或者是工程计算，在商业计算中我们要用 java.math.BigDecimal。
主要是说明应用系统设计、数据库设计，商业计算标准金额时，请尽量避开浮点直接运算，推荐 long、BigDecimal等其他方式进行



