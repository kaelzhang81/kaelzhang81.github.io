---
layout:     post
title:      "IdentifierNamingPostForWorldWideWebBlog"
subtitle:   "Google代码健康"
date:       2020-07-09 8:00:00
author:     "Kaelzhang"
header-img: "img/post-bg-js-module.jpg"
catalog:    true
tags:
    - DevOps
    - 代码质量
---

# Google代码健康：IdentifierNamingPostForWorldWideWebBlog

本文源于Google公司的马桶上的健康代码，作者：Chris Lewis & Bob Nystrom

创建长标识符很容易被采纳。 较长的名称通常会使内容更具可读性。 但是**名称太长会降低可读性**。 在GitHub和其他地方，有很多变量名超过60个字符的示例。 我们**以58个字符组成了此句，以供考虑**：

* 变量名称
* 使用一些简单的准则
* 漂亮的源代码
                      
名称应该具备两件事：**清晰**（知道它指的是什么）和**准确**（知道它不指的是什么）。 以下是一些有帮助的参考准则：

* **忽略变量类型声明中显而易见的词**

```java 
// Bad, the type tells us what these variables are:
String nameString; 
List<datetime> holidayDateList;

// Better:
String name; 
List<datetime> holidays;

```

* **忽略无关细节**

```java
// Overly specific names are hard to read:
Monster finalBattleMostDangerousBossMonster; 
Payments nonTypicalMonthlyPayments;

// Better, if there's no other monsters or payments that need disambiguation:
Monster boss; 
Payments payments;
```

* **忽略与周围上下文清晰相关的词**

```java
// Bad, repeating the context:
class AnnualHolidaySale {
    int annualSaleRebate; 
    boolean promoteHolidaySale() {...}
}

// Better:
class AnnualHolidaySale {
    int rebate; 
    boolean promote() {...}
}
```

* **省略可能适用于任何标识符的词**

通常的可疑对象如：

`数据，状态，数量，数量，值，管理器，引擎，对象，实体，实例，助手，util，util，代理，元数据，进程，句柄，上下文。 `


请删除它们。

这些规则有一些例外。 用你的判断。 太长的名称仍然比太短的名称好。 但是，遵循这些准则，你的代码将保持明确并且易于阅读。 包括“未来的你”在内的读者将因为清晰的代码而感谢你。


