---
layout: post-cn
title:  "数据结构课程 Project 3 重写记"
date:   2026-06-27 10:00:00 +0800
categories: zh-cn
tags: programming cpp notes
math: False
---

事情是这样的：我这学期在上一门数据结构的课程，虽然感觉难度不大，但是它的 Project 3 一堆条条框框，让我写得非常不爽；于是在*按照要求做完提交*之后，我又抽时间自己从零搭了一个能实现一模一样功能的程序，但是除了使用 C++17 标准之外**完全不设任何限制**。最后我做出了一个用**回调函数实现解耦**的架构。写这篇帖子是简要介绍一下我是**如何搓出这个架构**的。

重写过的项目源码在[这里](https://github.com/tomategg-101325/species-simulator)，实现的功能也写在 readme 里面了，这里不做过多赘述。简而言之，就是一个**高度可扩展的物种演化模拟程序**。

You can view the English translation of the post [here](/en-us/2026/06/27/ve280-p3-reformation-en.html), but it is only for reference.

## 痛点：模拟器层次的复杂度

由于项目要求限制[^1]，在交上去的代码里，模拟器层次有一个巨大的 `switch` 语句块，用来根据物种程序的指令类型来让底下的引擎执行不同的物理操作。下面是代码片段 (不完全是提交上去的代码，但是功能和结构一致)：

```cpp
void Simulator::SimulateCreature(unsigned int creature_index) {
  ...
  current_instruction = engine_.get_instruction(creature_index);  // obtain current instruction
  ...
  switch (current_instruction.opcode) {
    case Opcode::kHop:  // hop forward
      engine_.MoveCreature(creature_index);
      break;
    case Opcode::kLeft:  // turn left
      engine_.TurnCreatureLeft(creature_index);
      break;
    ...
  }
  ...
}
```

功能上虽然没有问题，但是这种做法的缺点有不少：

- 单个函数体**过于复杂**；
- 如果将来要增加新的指令，需要再往这个 `switch` 里面增添东西，更加**难以维护**；
- 指令类型区分和指令执行全部耦合在一起，**调试不方便**。

事实上我本地的 code quality 工具直接警告这个 `switch` 语句块所在的函数复杂度过高，所幸评测机没卡这个点扣我分。

但为什么一开始会把指令执行放在模拟器里面呢？因为引擎归模拟器管理，而指令的执行又依赖于引擎提供的各种物理操作（例如前进、左转、感染前方生物等等）。 

## 联想：嵌入式开发中学到的回调函数

在当初写完这个函数之后，我也一直在想能不能把指令执行从模拟器里面**解耦**出去。

某天洗澡的时候，我忽然回忆起嵌入式开发中学到的**回调函数 (callback function)**。回调函数一般是和**中断 (interrupt)** 搭配使用，用来在某个特定的事件发生时对其进行处理。老师教的基本架构是这样：

- 回调函数层：
  - 规定一个**函数类型 (function type)**，即函数的参数类型和个数以及函数的返回类型，*不包括参数名称、函数名称和具体实现*；
  - 开辟一个该类型函数的**函数指针**的数组作为**注册表 (registry)**；
  - 提供注册函数的**接口**以及执行所有已注册函数的接口。
- 应用层：
  - **实现**具体的事件处理函数，要满足规定的函数类型；
  - 将处理函数的函数指针**注册**进回调函数层；
  - 在中断函数里直接**调用**回调函数层给的执行接口。

> 💡 在嵌入式领域，中断是实现事件处理的方式之一。当某个事件发生的时候，对应的设备会让单片机 (MCU) 知晓，这个时候单片机会中断正在执行的其他程序，转而处理这个刚刚发生的事件。

这种架构能让回调函数层完全不用知晓应用层要干什么，它只机械地执行应用层传过来的回调函数，因而在需求发生变更时也不用改动。这就是一种解耦的方式。

## 解决：依赖倒置

在上述的架构中，回调函数层不依赖于应用层的具体实现，而它俩都依赖于回调函数的**函数类型**。这就是**依赖倒置原理 (Dependency Inversion Principle)**。它要求：

> 高层次不应依赖于低层次；两者都应**依赖于某个抽象 (abstraction)**。
> 
> 抽象不应依赖于实现细节；相反，**实现细节应依赖于这个抽象**。

在嵌入式的例子里，**回调函数的函数类型就是这个抽象**。而稍微迁移一下，依赖倒置原理也能应用在这个项目中。我是这样做的：

- 抽象出一个**回调函数列表** `WorldCallbacks`，包含各种基本的物理操作（获取、设置、判断）；
- 抽象出一个执行方法的抽象类 `IMethod`，重载其括号运算符用于具体执行，接受上面的这个回调函数列表作为参数，同时让其为虚函数；
- 从执行方法的抽象类中继承出具体的各种动作 (action) 和感知 (sensor)，实现具体的执行过程。
- 在模拟器里维护一个 `IMethod` 类型的指针哈希表，指令名称作为其键值，方便快速调用。
- 让引擎提供 `WorldCallbacks` 的具体实现。

这个回调函数列表就是统一的抽象，模拟器层次和引擎层次都依赖于它。现在，在模拟器的执行逻辑里，只需要根据指令名称定位到对应的执行方法，然后调用其括号运算符，传入准备好的回调函数列表和其他相关参数即可。逻辑简单很多，如果要扩展的话也可以直接继承出一个新的执行类。

```cpp
void Simulator::SimulateCreature(unsigned int creature_index) {
  ...
  current_instruction = engine_.get_instruction(creature_index);  // obtain current instruction
  ...
  std::shared_ptr<IMethod> executor = get_executor(current_instruction.opcode);  // obtain executor
  ...
  (*executor)(world_callbacks, creature_index, ...);  // dispatch the executor
  ...
}
```

重写过的这个 `SimulateCreature` 函数虽然看上去还会很复杂，但是其核心就是上面的这三行代码：**获取指令、找到执行方法，传入参数执行**。其它的都是一些检查判断之类的操作，是为了满足杂七杂八的其他需求。如果后续还要加新的功能的话，是**完全不用动它**的。

## 脚注

[^1]: 连头文件都要用白名单限制是何意味呢我请问了？连 `stdexcept` 和 `memory` 都不让引，害得我抛个异常都得用字符串字面量 (string literal)，智能指针也用不了，相当的不优雅……
