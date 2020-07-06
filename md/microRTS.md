---
title: "microRTS"
date: 2019-11-14T14:31:27+08:00
lastmod: 2019-11-14T15:57:25+08:00
draft: true
tags: ["ai", "game"]
categories: ["code"]
author: "Rouzip"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

> microRTS 是一个精简版的 Real-Time Strategy 游戏，我这学期的算法课程的大作业就是完成一个 AI。我会把期间遇到的所有问题、想法与进展记录下来。

## 开发环境 🏠

Mac 10.15.1

Ubuntu 18.04 LTS

Intellij IDEA

openjdk 13

## 项目简介 🌟

microRTS 是如星际、魔兽这样大型 RTS 游戏的一个精简版，所以很多操作都有所省略。环境也由很多复杂的地形精简到了现在的状态，游戏中的单位有：base、barracks、resources、worker、light、heavy 和 ranged。最终游戏的胜利是消灭一方玩家所有的单位。许多像维修建筑、出售建筑等操作都被省略了。

### 单位介绍

#### resouces

这是玩家所要进行采集的资源，worker 通过向 resources 采集资源后向 base 运回使得玩家的资源数量增加。

#### base

玩家生产 worker 的地方，采集到的资源也需要运回这里才算真正地采集结束。血量为 10，建造代价为 10 。在游戏中的图形化界面中可以显示当前玩家的资源数量，但是摧毁 base，资源仍旧存在。

#### barracks

这里是生产更为高级的兵种的单位，在较大的地图中要尽快建造出兵营来增加自己的攻击力。血量为 4，建造代价为 5 。

#### worker

这是游戏中最为基础的单位，其可以采集资源，建造 base 和 barracks，也可以进攻敌人。血量为 1， 建造代价为 1，攻击范围为 1，生产时间为 50，攻击力为 1，移动时间为 10，攻击时间为 5，采集时间为 20，运回资源时间为 10 。

#### light

较高级的兵种。血量为 4，建造代价为 2，攻击范围为 1，生产时间为 80，移动时间为 8，攻击时间为 5，攻击力为 2 。

#### heavy

最重型的高级兵种。血量为 4，建造代价为 2，攻击范围为 1，生产时间为 120，移动时间为 12，攻击时间为 5，攻击力为 4 。

#### ranged

远程攻击的高级兵种。血量为 1，生产代价为 2，攻击范围为 3，生产时间为 100，移动时间为 10，攻击时间为 5，攻击力为 1 。

## 战略 📖

### 硬编码算法

#### 自动采矿算法

硬编码，在自己基地附近的区域内进行采矿，合适的采矿的 worker 暂定为$ n(resources)+1 $，附近的矿定义为距离 base 不超过 5 个单位距离。采矿的 worker 为当前无任务的 worker 和已在采矿队列中的 worker 中选取，如果该 worker 在采矿中会自动

#### 编写固定脚本

较为简单但是较为死板，需要为每种情况都编写对应的脚本，同时需要较高的 domain knowledge。

#### 有限状态机

对于简单的 AI 来说足够好了，但是无法处理复杂的任务，很难保证一直处于理想的状态中。

> 混合其他的有限状态机，可以一定成都上地扩展可用性。

#### 决策树

相比于有限状态机，它提供了一个更加简单的决策判断，但是考虑情况过多就会变得代码特别臃肿，条件也会越来越复杂。

#### 有限状态机与决策树相结合

略

### 决策论算法

有更加高级的蒙特卡洛搜索树算法等，没有过多了解，略。

## 战术 🏹

### 寻路算法

AI 需要提供给主程序的，是当前时间应该行动的一系列 actions，但是我们在程序中需要设计的是一系列更为抽象的行为，例如 `attck`，`harvest`，`build`，`train` 等。对于这些更为抽象的行为，我们需要提供一个寻路算法，供其计算出当前应该采取的行为。目前游戏中内置的寻路算法有 BFS、FloodFill、Greedy 寻路算法。

## 损失函数

如何决定一个有效的损失函数能够更好更正确地决定下一步的动作。这里损失函数的具体定义由主观的游戏经验所决定，所以存在不合理性，一开始我希望通过随机梯度下降等算法对于该函数进行调整，但是 **合适的训练样本极难获取**，因为我们无法判断在对局的过程之中哪方处于优势，如果由人工标定仍旧引入了主观因素，那么意义不大。

## 初版决策函数（2019.11.14）

我决定使用别人已有的 AI 进行 action 的提供，同时根据下面的决策函数决定出下一步应该执行的最佳操作。目前仍旧存在的问题是仍旧无法在小地图中打败 workRush，根据多次观察，是由于兵营位置选择不恰当，这样的话，本来快要建造好的兵营就被赶来的 worker 打掉了，所以说要有 defense 机制来对于进攻的兵力有所防备。

$$
    Action(gs) = \argmax(\sum_{i=1}^n dis(myActiveUnit_i, enemyBaseBarrack) - \sum_{j=1}^m dis(enemyActiveUnit_j, myBaseBarrack) + \sum_{k=1}^n hitNum(myUnit_k), -\sum_{l=1}^m hitNum(enemyUnit_l))
$$
