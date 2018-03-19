# 《每日TLA+》系列之“限流”问题

## 问题描述
工程中需要调用第三方的API接口，有两种调用类型：
1. 一个GET调用一个对象集合，每次调用都会返回一些结果，并可能返回分页标记
2. 一个GET调用一个对象，如果需要，接着调用PUT更新该对象

该API有单位时间N次调用的流量限制。第一类调用可以任意调用，可以在两个调用间等待。第二类则可能调用1或者2次，然而如有需要高优先级可能会立即更新对象。考虑这些约束，如何保证流量不超标？

## 初始设计
让我们从最小可能的案例入手：

```
-------------------------------- MODULE api --------------------------------
EXTENDS Integers, TLC

(* --algorithm api
{
variables made_calls = 0, max_calls \in 5..10;

macro make_calls(n)
{
    made_calls := made_calls + n;
    assert made_calls <= max_calls;
}

process(get_collection = 0)
{
Request:
    make_calls(1);
    either
    {
        goto Request;
    }
    or
    {
        skip;
    }
 }
    
process(get_put \in 1..3)
{
Call:
    with(c \in {1, 2})
    {
        make_calls(c);
    }    
}

}
```

由于大多数业务逻辑与用例无关，因此可将其忽略。 get_collection进行任意数量的调用，由either goto代表。 每次TLC到达该分支点时，它都会探索两个分支，所以这可以是任意次数的调用。 同样，get_put在事务中进行一个或两个调用。 最后，每当made_calls改变，检查其值有没有超过最大值。

如果运行该代码生成的TLA+，会得到一个错误：断言失败。 可以通过在宏中添加一个检查来解决该问题，这样我们只会在剩下足够的速度时才调用。 一个解决方法是：


```
macro make_calls(n)
{
    if(made_calls < max_calls - n) \*new add
    {
        made_calls := made_calls + n;
    };
    assert made_calls <= max_calls;
}
```
如果超过限制，会使调用失败。

## 死锁
解决方案很有效，但却忽略了业务要求：我们不希望不调用，而是希望等到流量限制更新。 可以用等待来模拟：


```
macro make_calls(n)
{
    await made_calls < max_calls - n;  \* modify
    made_calls := made_calls + n;
    assert made_calls <= max_calls;
}
```

如果以上述算法运行，会出现死锁。 因为所有的进程都在等待限制更新，而目前尚未实现限制更新。 下面是其中一种实现：


```
process(reset_limit = 1)
{
Reset:
    while(TRUE)
    {
        made_calls := 0;
    } 
}
```
增加上述代码后，检测通过。

## 异步工作者
让我们添加些复杂度：所有进程都在不同的worker上运行，所以他们不会自动知道所做的调用次数。 为了避免超出限制，需要分享那些需要时间的信息。 一种方式是让每个进程查询一个中央缓存，只有在剩下足够多的调用次数时才会执行：


```
-------------------------------- MODULE api --------------------------------
EXTENDS Integers, TLC

(* --algorithm api
{
variables made_calls = 0, max_calls \in 5..10;

macro make_calls(n)
{
    await made_calls < max_calls - n;  \* modify
    made_calls := made_calls + n;
    assert made_calls <= max_calls;
}

process(get_collection = 0)
{
GCGetCalls:
    await made_calls <= max_calls - 1;
Request:
    make_calls(1);
    either
    {
        goto Request;
    }
    or
    {
        skip;
    }
 }
    
process(get_put \in 1..3)
{
GPGetCalls:
    await made_calls <= max_calls - 2;
Call:
    with(c \in {1, 2})
    {
        make_calls(c);
    }    
}

process(reset_limit = 1)
{
Reset:
    while(TRUE)
    {
        made_calls := 0;
    } 
}

}
*)
```

运行后会发现它是失败的，因为在获取和请求之间，另一个进程可以调用。 解决这个问题的方法之一是引入某种形式的锁或优先级。 这将能够工作，但它也大大增加了复杂度。 每个进程都需要等待所有其他进程，即使它有很多调用。 应该寻找一种更好的办法来解决这个问题。

## 信号量
可以使用的一个解决方案是调用保留：当进程检查还余有调用次数时，它保留N次调用，这些调用是在本地缓存的，而不是在API端中。 然后，一旦我们进行了适当的调用，我们将返回必要的保留调用。


```
-------------------------------- MODULE api --------------------------------
EXTENDS Integers, TLC

(* --algorithm api
{
variables made_calls = 0, 
          max_calls \in 5..10,
          reserved_calls = 0;

macro make_calls(n)
{
    made_calls := made_calls + n;
    assert made_calls <= max_calls;
}

macro reserve(n)
{
    await made_calls + reserved_calls + n <= max_calls;  \* modify 
    reserved_calls := reserved_calls + n;
}

process(get_collection = 0)
{
GCGetCalls:
    reserve(1);
Request:
    make_calls(1);
    reserved_calls := reserved_calls - 1;
    either
    {
        goto GCGetCalls;
    }
    or
    {
        skip;
    }
 }
    
process(get_put \in 1..3)
{
GPGetCalls:
    reserve(2);
Call:
    with(c \in {1, 2})
    {
        make_calls(c);
    };  
    reserved_calls := reserved_calls - 2;  
}

process(reset_limit = 1)
{
Reset:
    while(TRUE)
    {
        made_calls := 0;
    } 
}

}
*)
```


