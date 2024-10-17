---
title: Haskell-函数的语法
date: 2017-12-08 20:51:36
tags:
- Haskell
categories:
-  Learn You a Haskell for Great Good! 
---

# 模式匹配
模式匹配通过检查数据的特定结构来检查其是否匹配，并按模式从中取得数据。
在定义函数时，可以为不同的模式分别定义函数体，可以匹配一切数据类型---数字，字符，List，元组，等等。

<!-- more -->

``` haskell?linenums
sayMe :: (Integral a) => a -> String   
sayMe 1 = "One!"   
sayMe 2 = "Two!"   
sayMe 3 = "Three!"   
sayMe 4 = "Four!"   
sayMe 5 = "Five!"   
sayMe x = "Not between 1 and 5"  
```
万能匹配：最后一个匹配可以匹配一切数值并将其绑定为x。
在定义模式时，一定要留一个万能匹配的模式，保证程序就不会因为不可预料的输入而崩溃。

## Tuple 匹配
``` haskell
addVectors :: (Num a) => (a, a) -> (a, a) -> (a, a)   
addVectors (x1, y1) (x2, y2) = (x1 + x2, y1 + y2)  
```

## List 匹配

``` haskell?linenums
tell :: (Show a) => [a] -> String   
tell [] = "The list is empty"   
tell (x:[]) = "The list has one element: " ++ show x   
tell (x:y:[]) = "The list has two elements: " ++ show x ++ " and " ++ show y   
tell (x:y:_) = "This list is long. The first two elements are: " ++ show x ++ " and " ++ show y  
```
## as模式
将一个名字和@置于模式前，可以在按模式分割什么东西时仍保留对其整体的引用。

``` haskell?linenums
capital :: String -> String   
capital "" = "Empty string, whoops!"   
capital all@(x:xs) = "The first letter of " ++ all ++ " is " ++ [x]
```

# 门卫
门卫（guard）则用来检查一个值的某项属性是否为真。

``` haskell?linenums
bmiTell :: (RealFloat a) => a -> String   
bmiTell bmi   
    | bmi <= 18.5 = "You're underweight, you emo, you!"   
    | bmi <= 25.0 = "You're supposedly normal. Pffft, I bet you're ugly!"   
    | bmi <= 30.0 = "You're fat! Lose some weight, fatty!"   
    | otherwise   = "You're a whale, congratulations!"  
```
## where关键字
where关键字跟在门卫后面（最好是与竖线缩进一致），可以定义多个名字和函数，这些名字对每个门卫都是可见的，这一来就避免了重复。
函数在where绑定中定义的名字只对本函数可见，因此我们不必担心它会污染其他函数的命名空间。
**注意，其中的名字都是一列垂直排开，如果不这样规范，haskell就搞不清楚它们在哪个地方了。**
where绑定也可以使用模式匹配

``` haskell?linenums
initials :: String -> String -> String   
initials firstname lastname = [f] ++ ". " ++ [l] ++ "."   
    where (f:_) = firstname   
          (l:_) = lastname  
```
where绑定可以定义名字，也可以定义函数

``` haskell?linenums
calcBmis :: (RealFloat a) => [(a, a)] -> [a]  
calcBmis xs = [bmi w h | (w, h) <- xs]  
    where bmi weight height = weight / height ^ 2  
```
## let 关键字
where绑定是在函数底部定义名字，对包括所有门卫在内的整个函数可见。
let绑定则是个表达式，允许你在任何位置定义局部变量，而对不同的门卫不可见，也可以使用模式匹配。
let的格式为let [bindings] in [expressions]。在let中绑定的名字仅对in部分可见。let里面定义的名字也得对齐到一列。

``` haskell?linenums
cylinder :: (RealFloat a) => a -> a -> a   
cylinder r h =  
    let sideArea = 2 * pi * r * h   
        topArea = pi * r ^2   
    in  sideArea + 2 * topArea 
```
let绑定本身是个表达式，可以随处安放；而where绑定则是个语法结构。

let也可以定义局部函数

``` haskell?linenums
[let square x = x * x in (square 5, square 3, square 2)] 
```
# case 表达式
模式匹配本质上不过就是case语句的语法糖而已。

``` haskell?linenums
head' :: [a] -> a   
head' xs = case xs of [] -> error "No head for empty lists!"   
                      (x:_) -> x  
```



``` haskell?linenums
describeList :: [a] -> String   
describeList xs = "The list is " ++ case xs of [] -> "empty."   
                                               [x] -> "a singleton list."    
                                               xs -> "a longer list."  
```



``` haskell?linenums
describeList :: [a] -> String   
describeList xs = "The list is " ++ what xs   
    where what [] = "empty."   
          what [x] = "a singleton list."   
          what xs = "a longer list."  
```
