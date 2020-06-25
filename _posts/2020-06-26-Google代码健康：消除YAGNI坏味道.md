---
layout:     post
title:      "消除YAGNI坏味道"
subtitle:   "Google代码健康"
date:       2020-06-26 7:00:00
author:     "Kaelzhang"
header-img: "img/post-bg-js-module.jpg"
catalog:    true
tags:
    - DevOps
    - 代码质量
---


# Google代码健康：消除YAGNI坏味道

本文源于Google公司的马桶上的健康代码，作者：Marc Eaddy。

大多数的软件开发成本在于维护成本。降低维护成本的**一种方法是仅在真正需要时才实现某些东西，也就是“You Aren't Gonna Need It（你将不会需要它）” （YAGNI）设计原则。**如何发现不必要的代码？跟随你的“嗅觉”！

代码坏味道通常是表示存在设计缺陷的代码模式。例如：创建仅有一个子类的基类或接口，可能表明你在猜测将来会需要更多的子类。相反，请你**践行增量开发和设计**：在实际需要之前不要添加第二个子类。

** 以下C ++代码具有许多YAGNI坏味道**：


``` C++
class Mammal { ...
  virtual Status Sleep(bool hibernate) = 0;
};
class Human : public Mammal { ...
  virtual Status Sleep(bool hibernate) {
    age += hibernate ? kSevenMonths : kSevenHours;
    return OK;
  }
};
```

当真正仅需一个类时，维护人员就必须承担理解，记录和测试两个类的负担。即使所有调用方都传递false ，代码也必须处理休眠状态为真的情况，以及即使从未发生过的Sleep的情况，也必须处理Sleep 返回为错误的情况。这将导致不必要的代码永远不会执行。**消除这些坏味道可简化代码**：   

``` C++
class Human { ...
  void Sleep() { age += kSevenHours; }
};
```

**以下是其他一些YAGNI的坏味道**：

* 除测试外从未执行过的代码（又称：到达时已失效的代码）
* 设计为实际上不应为子类的子类（具有虚方法和/或保护的成员）的类
* 可以是私有的公开或受保护的方法或字段
* 总是具有相同的值的参数，变量或标记

值得庆幸的是，**通过寻找简单的模式，通常很容易发现YAGNI坏味道和一般的代码坏味道，并且使用简单的重构就可以轻松消除它们**。

你是否在考虑添加今天不会使用的代码？相信我，你将不会需要它！


