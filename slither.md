[TOC]

Slither是一个用Python 3编写的智能合约静态分析框架，提供如下功能：



- 自动化漏洞检测（Automated vulnerability detection）。提供超30多项的漏洞检查模型，模型列表详见：https://github.com/crytic/slither#detectors。
- 自动优化检测（Automated optimization detection）。Slither可以检测编译器遗漏的代码优化项并给出优化建议。
- 代码理解（Code understanding）。Slither能够绘制合约的继承拓扑图，合约方法调用关系图等，帮助开发者理解代码。
- 辅助代码审查（Assisted code review）。用户可以通过API与Slither进行交互。

Slither的工作方式如下：



1. Solidity compiler：智能合约源码经过solc编译后得到Solidity抽象语法树（Solidity Abstract Syntax Tree，AST）作为Slither的输入；可以指定Slither去调用一些常见的框架（包括Truffle，Embark和Dapp）去分析一份智能合约。
2. information recovery（数据整合）：Slither会生成一些重要的信息，比如合约的继承图（inheritance graph）、控制流图（CFG）以及合约中函数列表。
3. SlithIR conversion：Slither将合约代码转换为[SlithIR](https://github.com/crytic/slither/wiki/SlithIR)（一种内部表示语言），目的是通过简单的API实现高精度分析，支持污点和值的跟踪，从而支持检测复杂的模型。
4. 在代码分析阶段，Slither运行一组预定义的分析，包括合约中变量、函数的依赖关系；变量的读写和函数的权限控制。
5. 经过Slither的核心处理之后，就可以提供漏洞检测、代码优化检测和代码理解输出等。



#Slither


## A. 内置代码分析

- **读/写**

  s 标识变量的读写。对于控制流程图的每个契约、函数或者节点，可以检索读取或写入的变量，并且按照变量的类型(local or state) 进行过滤。例如，可以知道从特定函数 write 的状态变量，或者找到哪些函数wrie了给定的变量。一些检测器是基于这些信息的，比如未初始化的变量和 reentrancy漏洞

- **Protected functions**

  在智能合约的设计中，一个反复出现的模式是使用所有权来保护对函数的使用。这种模式背后的思想是，一个特定的用户(或者一组用户) 称为`所有者`，可以执行特权操作。例如，只有所有者才可以铸造新的代币.对函数保护进行建模可以降低误报的数量.为了检测未受保护的函数,Slither 会检查以下的情况:

  (1) 非构造函数的函数

  (2) msg.sender 的caller 的地址没有直接用于比较

  这种启发式的方法可能会有误报和漏报

- **数据相关性(Data dependency analysis)**

  Slither 使用 SSA 表示法计算所有变量的数据相关性.首先在每个函数的上下文分析依赖性,然后在合约的所有函数中计算 fixpoint 来确定在多交易的上下文中是否存在依赖关系. Slither 将一些变量分类为 *tainted* , 这表明该变量依赖于用户控制的变量,因此可能会受到用户的影响. 最后, 数据相关性考虑了受保护函数的启发.因此, 可以根据用户的权限级别选择依赖. 这种关于数据依赖型的粒度信息对于编写准确的漏洞检测器至关重要.



## B. 自动化漏洞给检测

​		Slither 的开源版本包含二十多个bug检测器，包括

- 



# SLITHIR

​		SlithIR是Slither中用来表示实代码的混合中间表示。控制流图的每个节点最多可以包含一个Solidity表达式，该表达式被转换为一组SlithIR指令。这种表示使得实现分析更加容易，而不会丢失包含在可靠源代码中的关键语义信息

## A. 指令集

​		SlithIR使用不到40条指令。它没有内部控制流表示，并且依赖于Slither的控制流图结构(SLITHIR代码与图中的每个节点相关联)。下面给出了一些值得注意的指令的高级描述；完整的描述见[26]。

1. **符号**: LV 和 RV 分别表示可赋值的变量(左值)和只读的变量(右值)。变量可以是Solidity 变量，也可以是由中间表示创建的临时变量
2. **算术运算**: 表示为二元或一元运算符:
   - LV = RV BINARY RV
   - LV = UNARY RV
3. **映射和数组**::Solidity 允许对通过解引用访问的映射和数组进行操作。SlithIR使用一个特定的变量类型，称为REF(引用变量)来存储解引用的结果。索引运算符允许对变量进行解引用:
   -  REF ← Variable [Index]
4. **结构体**:: 对结构体的访问是通过成员运算符完成的
5. ...

## B. SSA

​		静态单一赋值(SSA) 是编译和一般静态分析常用的表示方式，它要求每个变量只分配一次，并且在使用之前定义。比如:

```C
x = 3;
y = x++;
x = y;
```

​		变成 :

```c
x1 = 3;
y1 = x1;
x2 = x1 + 1;
x3 = y1 + x2;
```

​		SSA 的主要优势之一是允许定义链易于计算，是的依赖性分析变得简单.Slither存储两个SlithIR版本:带SSA和不带SSA





















