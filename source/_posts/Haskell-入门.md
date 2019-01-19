---
title: Haskell-入门
date: 2017-12-01 22:19:04
tags:
- Haskell
categories:
-  Learn You a Haskell for Great Good! 
---

# 简单运算

 1. 一行可以使用多个运算符，按照运算符优先等级机选，可以使用括号改变优先次序。
 2. 使用负数时最好将其置于括号中。
 3. **&&** 逻辑与，**||** 逻辑或，**not** 逻辑否，**==** 相等判断，**/=** 不等判断。
 4. 函数调用拥有最高的优先级。
 5. 如果某函数有两个参数，也可以用 **\`** 符号将它括起，以中缀函数的形式调用它。div 92 10等价于92 \`div\` 10。
 6. 使用**let**关键字定义常量。
<!-- more -->

# 函数

 1. 函数的声明形式：先函数名，后跟由空格分隔的参数表。但在声明中一定要在 = 后面定义函数的行为。
 2. if语句，haskell中if语句的else部分是不可省略。if(condition) then  else

# List

 1. List是一种单类型的数据结构，可以用来存储多个类型相同的元素。一个List由方括号括起，其中的元素用逗号分隔开来。
 2. 使用**++**运算符可以将两个List合并。Haskell会遍历整个的List(++符号左边的那个)。当左侧List长度很长时，执行效率较低。
 3. **:** 运算符可以连接一个元素到一个List或者字符串之中，而\++运算符则是连接两个List。若要使用\++运算符连接单个元素到一个List之中，就用方括号把它括起使之成为单个元素的List。
 4. 使用 **!!** 运算符可以按照索引取得List中的元素，索引的下标从0开始。
 5. List同样也可以用来装List，甚至是List的List的List。List中的List可以是不同长度，但必须得是相同的类型。

 # List常用函数





| head    | 返回List的首个元素                                       |
| ------- | -------------------------------------------------------- |
| tail    | 返回List除去头部之后的部分                               |
| last    | 返回LIst的最后一个元素                                   |
| init    | 返回LIst除去最后一个元素的部分                           |
| length  | 返回List的长度                                           |
| null    | 检查List是否为空                                         |
| reverse | 将List反转                                               |
| take    | 返回List的前几个元素                                     |
| drop    | 删除List中的前几个元素                                   |
| maximum | 返回List中最大的那个元素                                 |
| minimum | 返回List中最小的那个元素                                 |
| sum     | 返回List中所有元素的和                                   |
| product | 返回List中所有元素的积                                   |
| elem    | 判断一个元素是否在包含于List，通常以中缀函数的形式调用它 |


# 区间

 1. [start..end]构造连续List。
 2. [start,start+d..end]构造步长为d的List.
 3. cycle接受一个List做参数并返回一个无限List。
 4. repeat接受一个值作参数，并返回一个仅包含该值的无限List。
 5. replicate可以得到相同元素的List，如replicate 3 10，得[10,10,10]。

# List Comprehension
 List Comprehension可以从既有的List中按照规则产生一个新List。
 1. 从多个List中取元素时
 [ x | x <- [10..20], x /= 13, x /= 15, x /= 19]   
 2. 从多个List中取元素时，comprehension会把所有的元素组合交付给输出函数。
 [ x*y | x <- [2,5,10], y <- [8,10,11]]   
 3. 操作含有List的List，使用嵌套的List comprehension
 [ [ x | x <- xs, even x ] | xs <- xxs]   
 4. List Comprehension可以分成多行

# Tuple
Tuple中的项不必为同一类型，在Tuple里可以存入多类型项的组合。
Tuple的类型取决于其中项的数目与其各自的类型。
Tuple中的项 由括号括起，并由逗号隔开。

## 序对
一个长度为2的Tuple，也可以称作序对，Pair。

 1. fst返回一个序对的首项。
 2. snd返回序对的尾项。
 3. 这两个函数仅对序对有效，而不能应用于三元组，四元组和五元组之上。
 4. zip可以用来生成一组序对(Pair)的List。它取两个List，然后将它们交叉配对，形成一组序对的List。较长的那个会在中间断开，去匹配较短的那个。