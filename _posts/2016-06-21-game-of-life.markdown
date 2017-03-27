---
layout:     post
title:      "康威生命游戏（Game of Life）及其 Numpy 实现"
subtitle:   "Conway\'s Game of Life using Numpy"
date:       2016-06-21 05:00:00
author:     "Haojun, Undergraduate Research Scientist"
header-img: "img/in-post/game-of-life/header.jpg"
catalog:    true
tags:
    - Python
    - Game of Life
    - NumPy
    - Automata
    - Tutorial
---

## Numpy 简介
---
Numpy 是 Python 科学计算的基本包。它具有以下功能：

* 强大的多维度数组对象
* 复杂的广播（Broadcasting）方程
* 集成 C／C++ 和 Fortran 的代码工具
* 支持线性代数，傅立叶变换，以及随机数

Numpy 不仅可以用于科学计算，也可以作为高效的通用数据的多维容器。Numpy 可以无缝且迅速地集成任意的数据类型。我们用一个简单的生命游戏来探索 Numpy 的各种特性。

## 生命游戏（Game of Life）
---
生命游戏是一个零玩家游戏。它包括一个二维矩形世界，这个世界中的每个方格居住着一个活着的或死了的细胞。一个细胞在下一个时刻生死取决于相邻八个方格中活着的或死了的细胞的数量。如果相邻方格活着的细胞数量过多，这个细胞会因为资源匮乏而在下一个时刻死去；相反，如果周围活细胞过少，这个细胞会因太孤单而死去。实际中，玩家可以设定周围活细胞的数目怎样时才适宜该细胞的生存。如果这个数目设定过高，世界中的大部分细胞会因为找不到太多的活的邻居而死去，直到整个世界都没有生命；如果这个数目设定过低，世界中又会被生命充满而没有什么变化。

实际中，这个数目一般选取2或者3；这样整个生命世界才不至于太过荒凉或拥挤，而是一种动态的平衡。这样的话，游戏的规则就是：当一个方格周围有2或3个活细胞时，方格中的活细胞在下一个时刻继续存活；即使这个时刻方格中没有活细胞，在下一个时刻也会“诞生”活细胞。

在这个游戏中，还可以设定一些更加复杂的规则，例如当前方格的状况不仅由父一代决定，而且还考虑祖父一代的情况。玩家还可以作为这个世界的「上帝」，随意设定某个方格细胞的死活，以观察对世界的影响。

在游戏的进行中，杂乱无序的细胞会逐渐演化出各种精致、有形的结构；这些结构往往有很好的对称性，而且每一代都在变化形状。一些形状已经锁定，不会逐代变化。有时，一些已经成形的结构会因为一些无序细胞的“入侵”而被破坏。但是形状和秩序经常能从杂乱中产生出来。

这个游戏被许多计算机程序实现了。Unix 世界中的许多黑客喜欢玩这个游戏，他们用字符代表一个细胞，在一个计算机屏幕上进行演化。比较著名的例子是，GNU Emacs 编辑器中就包括这样一个小游戏。

![康威生命游戏中的一种可持续繁殖模式：Gosper 的机枪制造「滑翔机」](/img/in-post/game-of-life/glider_gun.gif)

## 游戏规则
---
生命游戏中，对于任意细胞，规则如下：

每个细胞有两种状态-存活或死亡，每个细胞与以自身为中心的周围八格细胞产生互动。（如图，黑色为存活，白色为死亡）

1. 当前细胞为存活状态时，当周围低于2个（不包含2个）存活细胞时， 该细胞变成死亡状态。（模拟生命数量稀少）
2. 当前细胞为存活状态时，当周围有2个或3个存活细胞时， 该细胞保持原样。
3. 当前细胞为存活状态时，当周围有3个以上的存活细胞时，该细胞变成死亡状态。（模拟生命数量过多）
4. 当前细胞为死亡状态时，当周围有3个存活细胞时，该细胞变成存活状态。 （模拟繁殖）

可以把最初的细胞结构定义为种子，当所有在种子中的细胞同时被以上规则处理后, 可以得到第一代细胞图。按规则继续处理当前的细胞图，可以得到下一代的细胞图，周而复始。

## 特殊的生命细胞
---

<table>
	<tr>
		<td colspan="4" style="text-align:center">静止细胞</td>
	</tr>
	<tr>
		<td style="text-align:center">Block</td>
		<td style="text-align:center">Beehive</td>
		<td style="text-align:center">Loaf</td>
		<td style="text-align:center">Boat</td>
	</tr>
	<tr>
		<td style="text-align:center"><img src="/img/in-post/game-of-life/block.png" alt=""></td>
		<td style="text-align:center"><img src="/img/in-post/game-of-life/beehive.png" alt=""></td>
		<td style="text-align:center"><img src="/img/in-post/game-of-life/loaf.png" alt=""></td>
		<td style="text-align:center"><img src="/img/in-post/game-of-life/boat.png" alt=""></td>
	</tr>
</table>

<table>
	<tr>
		<td colspan="5" style="text-align:center">周期细胞</td>
	</tr>
	<tr>
		<td style="text-align:center">Blinker（2周期）</td>
		<td style="text-align:center">Toad（2周期）</td>
		<td style="text-align:center">Beacon（2周期）</td>
		<td style="text-align:center">Pulsar（3周期）</td>
		<td style="text-align:center">Pentadecathlon（15周期）</td>
	</tr>
	<tr>
		<td style="text-align:center"><img src="/img/in-post/game-of-life/blinker.gif" alt=""></td>
		<td style="text-align:center"><img src="/img/in-post/game-of-life/toad.gif" alt=""></td>
		<td style="text-align:center"><img src="/img/in-post/game-of-life/beacon.gif" alt=""></td>
		<td style="text-align:center"><img src="/img/in-post/game-of-life/pulsar.gif" alt=""></td>
		<td style="text-align:center"><img src="/img/in-post/game-of-life/pentadecathlon.gif" alt=""></td>
	</tr>
</table>

<table>
	<tr>
		<td colspan="2" style="text-align:center">飞船细胞</td>
	</tr>
	<tr>
		<td style="text-align:center">Glider</td>
		<td style="text-align:center">Lightweight spaceship (LWSS)</td>
	</tr>
	<tr>
		<td style="text-align:center"><img src="/img/in-post/game-of-life/glider.gif" alt=""></td>
		<td style="text-align:center"><img src="/img/in-post/game-of-life/lightweight_spaceship.gif" alt=""></td>
	</tr>
</table>

## Numpy 数组
---
首先使用下列命令创建 Numpy 数组用于保存生命细胞：

```python
import numpy as np
Z = np.array([[0, 0, 0, 0, 0, 0],
              [0, 0, 0, 1, 0, 0],
              [0, 1, 0, 1, 0, 0],
              [0, 0, 1, 1, 0, 0],
              [0, 0, 0, 0, 0, 0],
              [0, 0, 0, 0, 0, 0]])
```

因为我们没有声明数组的数据类型，且数组里的数据为整型，Numpy 选择了整型数据类型。可通过下列命令查看数据类型：

```python
>>> print Z.dtype
int64
```

可通过下列命令查看数组的维度（6*6）：

```python
>>> print Z.shape
(6, 6)
```

可用 `row` 和 `column` 下标读取数组 `Z` 的数据：

```python
>>> print Z[0, 5]
0
```

我们也可以读取指定的数组块：

```python
>>> print Z[1:5, 1:5]
[[0 0 1 0]
 [1 0 1 0]
 [0 1 1 0]
 [0 0 0 0]]
```

需要注意的是上一步得出的数组块是 `Z` 的数组块指针，换句话说，对于这一数组块的任何改变都将被作用在 `z` 上：

```python
>>> A = Z[1:5, 1:5]
>>> A[0, 0] = 9
>>> print A
[[9 0 1 0]
 [1 0 1 0]
 [0 1 1 0]
 [0 0 0 0]]
>>> print Z
[[0 0 0 0 0 0]
 [0 9 0 1 0 0]
 [0 1 0 1 0 0]
 [0 0 1 1 0 0]
 [0 0 0 0 0 0]
 [0 0 0 0 0 0]]
```

我们将 `A[0, 0]` 设为9，可以看出 `Z[1, 1]` 也被设为了9，因为 `A[0, 0]` 对应的是 `Z[1, 1]`。通过下列命令可以看出数组 `A` 属于数组 `Z`。

```python
>>> print Z.base is None
True
>>> print A.base is Z
True
```

## 计算邻居细胞
---
我们可以使用非常笨拙的方式去一个一个数邻居细胞的数量，但是我们还是希望能够快速的进行这一步骤，因为细胞总数量的增加带来的却是程序运行时间的几何式增长。我们将使用 Numpy 的一个特性，矢量化，来实现这个要求。

首先，通过下列代码不难看出 `Z` 可以像纯量那样进行计算：

```python
>>> print 1 + (2 * Z + 3)
[[4 4 4 4 4 4]
 [4 4 4 6 4 4]
 [4 6 4 6 4 4]
 [4 4 6 6 4 4]
 [4 4 4 4 4 4]
 [4 4 4 4 4 4]]
```

不难看出，上一步所用到的公式被单独作用到 `Z` 的每一个元素。换句话说，对于任意的 `i` 和 `j`，`(1 + (2 * Z + 3))[i, j] == (1 + (2 * Z[i, j] + 3))`。

于是，我们可以通过矢量化的公式来计算邻居细胞数量。

```python
>>> N = np.zeros(Z.shape, dtype=int)
>>> N[1:-1, 1:-1] += (Z[ :-2, :-2] + Z[ :-2, 1:-1] + Z[ :-2, 2:] +
                      Z[1:-1, :-2]                 + Z[1:-1, 2:] +
                      Z[2:  , :-2] + Z[2:  , 1:-1] + Z[2:  , 2:])
```

## 遍历细胞
---
把我们前面的步骤整合起来，我们就得到了矢量化的遍历细胞方程：

```python
def iterate(Z):
    # Count neighbours
    N = (Z[0:-2, 0:-2] + Z[0:-2, 1:-1] + Z[0:-2, 2:] +
         Z[1:-1, 0:-2]                 + Z[1:-1, 2:] +
         Z[2:  , 0:-2] + Z[2:  , 1:-1] + Z[2:  , 2:])

    # Apply rules
    birth = (N == 3) & (Z[1:-1, 1:-1] == 0)
    survive = ((N == 2) | (N == 3)) & (Z[1:-1, 1:-1] == 1)
    Z[...] = 0
    Z[1:-1, 1:-1][birth | survive] = 1
    return Z
```

让我们来测试一下这个方程是否正确：

```python
>>> print Z
[[0 0 0 0 0 0]
 [0 0 0 1 0 0]
 [0 1 0 1 0 0]
 [0 0 1 1 0 0]
 [0 0 0 0 0 0]
 [0 0 0 0 0 0]]
>>> for i in range(4): iterate(Z)
>>> print Z
[[0 0 0 0 0 0]
 [0 0 0 0 0 0]
 [0 0 0 0 1 0]
 [0 0 1 0 1 0]
 [0 0 0 1 1 0]
 [0 0 0 0 0 0]]
```

## 使用更大的生命世界
---
之前，我们使用的数组 `Z` 是给定的，但我们可以随机生成更大的数组：

```python
>>> Z = np.random.randint(0, 2, (256, 512))
```

我们可以像之前那样遍历整个生命世界：

```python
>>> for i in range(100): iterate(Z)
```

我们可以通过下列命令查看这个生命世界：

```python
>>> size = np.array(Z.shape)
>>> dpi = 72.0
>>> figsize = size[1] / float(dpi), size[0] / float(dpi)
>>> fig = plt.figure(figsize=figsize, dpi=dpi, facecolor="white")
>>> fig.add_axes([0.0, 0.0, 1.0, 1.0], frameon=False)
>>> plt.imshow(Z, interpolation='nearest', cmap=plt.cm.gray_r)
>>> plt.xticks([]), plt.yticks([])
>>> plt.show()
```

![](/img/in-post/game-of-life/conway.png)

很简单，不是么？

你可以从<a href="/attach/game-of-life-numpy.py">这里（game-of-life-numpy.py）</a>下载完整的生命游戏代码。
