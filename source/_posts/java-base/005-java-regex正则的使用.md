---
title: 005-java-regex正则的使用
categories:
  - java-base
abbrlink: 1ad49383
date: 2021-01-15 13:09:10
---

摘要：主要作用：对数据进行匹配，可以执行更加复杂的字符串验证、拆分、替换功能。
<!-- more -->

# jdk核心类
- 核心包：java.util.regex
- 核心类：
    - Pattern:表示一个规则的意思，即：正则表达式的规则需要在Pattern类中使用。
    - Matcher:表示使用Pattern指定好的验证规则

# Pattern

## 常用方法
```
public static Pattern compile(String regex)      指定正则表达式规则,以及实例获取
public Matcher matcher(CharSequence input)  返回Matcher类的实例
public String[] split(CharSequence input)      字符串拆分
```

# Matcher
如果要验证一个字符串是否符合规范，则可以使用Matcher类。

## 常用方法
```
public Matcher matcher(CharSequence input)  可以为matcher类实例化
public boolean matches()                执行验证，进行字符串的验证
public String replaceAll(String replacement) 字符串替换，与String中的相同。
public String[] split(CharSequence input)   拆分，在String中也存在拆分操作。
```

# 实例

## matcher匹配实例
```java
    @Test
    public void testDate() {
        String str = "1993-07-27";        // 指定好一个日期格式的字符串
        String pat = "\\d{4}-\\d{2}-\\d{2}";    // 指定好正则表达式
        Pattern p = Pattern.compile(pat);    // 实例化Pattern类
        Matcher m = p.matcher(str);    // 实例化Matcher类
        if (m.matches()) {        // 进行验证的匹配，使用正则
            System.out.println("日期格式合法！");
        } else {
            System.out.println("日期格式不合法！");
        }
        //日期格式合法！
    }
```

## split拆分
```java
    @Test
    public void testSplit() {
        // 要求将里面的字符取出，也就是说按照数字拆分
        String str = "A1B22C333D4444E55555F" ;	// 指定好一个字符串
        String pat = "\\d+" ;	// 指定好正则表达式
        Pattern p = Pattern.compile(pat) ;	// 实例化Pattern类
        String s[] = p.split(str) ;	// 执行拆分操作
        for(int x=0;x<s.length;x++){
            System.out.print(s[x] + "\t") ;
        }
        //A B	C	D	E	F	
    }    
```

## replaceAll替换
```java
    @Test
    public void testReplace() {
        // 要求将里面的字符取出，也就是说按照数字拆分
        String str = "A1B22C333D4444E55555F" ;	// 指定好一个字符串
        String pat = "\\d+" ;	// 指定好正则表达式
        Pattern p = Pattern.compile(pat) ;	// 实例化Pattern类
        Matcher m = p.matcher(str) ;	// 实例化Matcher类的对象
        String newString = m.replaceAll("_") ;
        System.out.println(newString) ;
        //A_B_C_D_E_F
    }
```

# String对正则表达式的支持

## 核心方法

matches、replaceAll、split 内部实现还是使用的Pattern、Matcher类操作

```java
    @Test
    public void testString() {
        boolean temp = "1983-07-27".matches("\\d{4}-\\d{2}-\\d{2}") ;
        System.out.println("字符串验证：" + temp) ;

        String str1 = "A1B22C333D4444E55555F".replaceAll("\\d+","_") ;
        System.out.println("字符串替换操作：" + str1) ;

        String s[] = "A1B22C333D4444E55555F".split("\\d+") ;
        System.out.print("字符串的拆分：") ;
        for(int x=0;x<s.length;x++){
            System.out.print(s[x] + "\t") ;
        }
    }
```
matches 
```java
public boolean matches(String regex) {
        return Pattern.matches(regex, this);
    }
```

# 常规通用正则
## 字符类
```
1    字面值转义    \x
2    分组        [...]
3    范围         a-z
4    并集         [a-e][i-u]
5    交集         [a-z&&[aeiou]]
```

```
构造      匹配

字符
x       字符 x
\\      反斜线字符
\0n     带有八进制值 0 的字符 n (0 <= n <= 7)
\0nn    带有八进制值 0 的字符 nn (0 <= n <= 7)
\0mnn   带有八进制值 0 的字符 mnn（0 <= m <= 3、0 <= n <= 7）
\xhh    带有十六进制值 0x 的字符 hh
\uhhhh  带有十六进制值 0x 的字符 hhhh
\t      制表符 ('\u0009')
\n      新行（换行）符 ('\u000A')
\r      回车符 ('\u000D')
\f      换页符 ('\u000C')
\a      报警 (bell) 符 ('\u0007')
\e      转义符 ('\u001B')
\cx     对应于 x 的控制符

字符类
[abc]       a、b 或 c（简单类）
[^abc]      任何字符，除了 a、b 或 c（否定）
[a-zA-Z]a   到 z 或 A 到 Z，两头的字母包括在内（范围）
[a-d[m-p]]  a 到 d 或 m 到 p：[a-dm-p]（并集）
[a-z&&[def]]d、e 或 f（交集）
[a-z&&[^bc]]a 到 z，除了 b 和 c：[ad-z]（减去）
[a-z&&[^m-p]]a 到 z，而非 m 到 p：[a-lq-z]（减去）

预定义字符类
.       任何字符（与行结束符可能匹配也可能不匹配）
\d      数字：[0-9]
\D      非数字： [^0-9]
\s      空白字符：[ \t\n\x0B\f\r]
\S      非空白字符：[^\s]
\w      单词字符：[a-zA-Z_0-9]
\W      非单词字符：[^\w]

POSIX 字符类（仅 US-ASCII）
\p{Lower}   小写字母字符：[a-z]
\p{Upper}   大写字母字符：[A-Z]
\p{ASCII}   所有 ASCII：[\x00-\x7F]
\p{Alpha}   字母字符：[\p{Lower}\p{Upper}]
\p{Digit}   十进制数字：[0-9]
\p{Alnum}   字母数字字符：[\p{Alpha}\p{Digit}]
\p{Punct}   标点符号：!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
\p{Graph}   可见字符：[\p{Alnum}\p{Punct}]
\p{Print}   可打印字符：[\p{Graph}\x20]
\p{Blank}   空格或制表符：[ \t]
\p{Cntrl}   控制字符：[\x00-\x1F\x7F]
\p{XDigit}  十六进制数字：[0-9a-fA-F]
\p{Space}   空白字符：[ \t\n\x0B\f\r]

java.lang.Character 类（简单的 java 字符类型）
\p{javaLowerCase}   等效于 java.lang.Character.isLowerCase()
\p{javaUpperCase}   等效于 java.lang.Character.isUpperCase()
\p{javaWhitespace}  等效于 java.lang.Character.isWhitespace()
\p{javaMirrored}    等效于 java.lang.Character.isMirrored()

Unicode 块和类别的类
\p{InGreek}         Greek 块（简单块）中的字符
\p{Lu}              大写字母（简单类别）
\p{Sc}              货币符号
\P{InGreek}         所有字符，Greek 块中的除外（否定）
[\p{L}&&[^\p{Lu}]]  所有字母，大写字母除外（减去）
\pP                 标点字符。
\pM                 标记符号（一般不会单独出现）；
\pZ                 分隔符（比如空格、换行等）；
\pS                 符号（比如数学符号、货币符号等）；
\pN                 数字（比如阿拉伯数字、罗马数字等）；
\pC                 其他字符

边界匹配器
^       行的开头
$       行的结尾
\b      单词边界
\B      非单词边界
\A      输入的开头
\G      上一个匹配的结尾
\Z      输入的结尾，仅用于最后的结束符（如果有的话）
\z      输入的结尾

Greedy 数量词
X?      X，一次或一次也没有
X*      X，零次或多次
X+      X，一次或多次
X{n}    X，恰好 n 次
X{n,}   X，至少 n 次
X{n,m}  X，至少 n 次，但是不超过 m 次

Reluctant 数量词
X??     X，一次或一次也没有
X*?     X，零次或多次
X+?     X，一次或多次
X{n}?   X，恰好 n 次
X{n,}?  X，至少 n 次
X{n,m}? X，至少 n 次，但是不超过 m 次

Possessive 数量词
X?+     X，一次或一次也没有
X*+     X，零次或多次
X++     X，一次或多次
X{n}+   X，恰好 n 次
X{n,}+  X，至少 n 次
X{n,m}+ X，至少 n 次，但是不超过 m 次

Logical 运算符
XY      X 后跟 Y
X|Y     X 或 Y
(X)     X，作为捕获组

Back 引用
\n      任何匹配的 nth 捕获组

引用
\       Nothing，但是引用以下字符
\Q      Nothing，但是引用所有字符，直到 \E
\E      Nothing，但是结束从 \Q 开始的引用

特殊构造（非捕获）
(?:X)               X，作为非捕获组
(?idmsux-idmsux)    Nothing，但是将匹配标志i d ms u x on - off
(?idmsux-idmsux:X)  X，作为带有给定标志 i d m s u x on - off的非捕获组
(?=X)               X，通过零宽度的正 lookahead
(?!X)               X，通过零宽度的负 lookahead
(?<=X)              X，通过零宽度的正 lookbehind
(?<!X)              X，通过零宽度的负 lookbehind
(?>X)               X，作为独立的非捕获组
```

# Unicode 正则表达式标准
[http://www.unicode.org/reports/tr18/](http://www.unicode.org/reports/tr18/)
[http://www.unicode.org/Public/UNIDATA/UnicodeData.txt](http://www.unicode.org/Public/UNIDATA/UnicodeData.txt)
文本文档一行是一个字符，第一列是 Unicode 编码，第二列是字符名，第三列是 Unicode 属性

## java中Unicode作用与使用
Java 中用于 Unicode 的正则表达式数据都是由 Unicode 组织提供的。
