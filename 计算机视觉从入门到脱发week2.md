# 计算机视觉从入门到脱发

2019-10 @武折口

[toc]

## Week2. Python语言培训

上礼拜我们大致介绍了对象和类是怎么回事，本周我们将继续了解对象在Python中的神秘力量。

> Guido对语言设计美学的深入理解让人震惊。我认识不少很不错的编程语言设计者，他们设计出来的东西确实很精彩，但是从来都不会有用户。Guido知道如何在理论上做出一定妥协，设计出来的语言让使用者觉得如沐春风，这真是不可多得。
>
> ——Jim Hugunin (Jython作者)

### 魔法方法

不知道大家有没有想过，为什么C语言里的字符串`char[]`是不可以相加的，而Python里却可以呢？比如说`"I am "+"rich"`这样一个表达式在我们看来似乎是合情合理的，理应会输出`I am rich`，但是对于C语言而言你试图做的事情是把三个数组（指针）加在一起，这是不阔以的。那Python是如何实现我们直觉上的那种操作的呢？

聪明的你一定想到了，既然Python中一切都是对象，`"I am "`这样一个字符串其实正是一个`str`对象，而我们在写出`"I am "+"rich"`这样的语法时，实际调用的是`"I am ".__add__("rich")`[^1]，你可以在控制台里自己尝试一下。这里这个用双下划线包裹起来的函数`__add__`，就是传说中的“魔法方法”了。许多材料中亦称其“特殊方法”，然二者相权，当取其中二者耳[^2]。下面展示了一个可以相加的对象`White`[^3]：

```python
class White():

    def __init__(self, isHappy, reason):
        self.isHappy = isHappy
        self.reason = reason

    def __add__(self, other):
        if self.isHappy and other.isHappy:
            return White(False, '第一次有了{}，又有了{}。但是，为什么…'.format(self.reason, other.reason))

    def why(self):
        print(self.reason)


happy1 = White(True, '喜欢的语言')
happy2 = White(True, '全世界最好的语言')
double_happy = happy1 + happy2
double_happy.why()  # '第一次有了喜欢的语言，又有了全世界最好的语言。但是，为什么…'

```
Python的魔法方法规范了这门语言构建自身各个模块的接口，也即，Python中的各种对象通过魔法方法来实现对于众多Python语言特性的支持，这其中就包括下文将介绍的迭代，上文提到的“加法”其实是运算符重载，还有暂时不会介绍的属性访问、集合类、对象创建和销毁等等语言特性。

[^1]: 你可以打开客户端现在就自己尝试一下。
[^2]: 选自《我的Python不可能那么神奇》。魔法方法这么中二的名字当然不是笔者取的！据说这个昵称来源于这些方法神奇的效果。
[^3]: ~~白学模拟器~~

### 迭代
迭代，即`Iteration`，换个说法大家可能更熟悉点，也就是`循环`。大家想过循环到底是怎么实现的吗？比如：

```python
for i in range(10):
    print(i)
```

好，现在请你猜测一下`range`是个什么东西？对啦！它又双叒叕是对象哦。`range(10)`表示一个范围从`[0,10)`的迭代器，配合使用`for...in`的语法即可将迭代器中生成的值每次赋值给一个变量。一个简单的迭代器看起来像这样：

```python
class SomeIterator:
  def __iter__(self):
    self.a = 1
    return self
 
  def __next__(self):
    if self.a <= 20:
      x = self.a
      self.a += 1
      return x
    else:
      raise StopIteration
```

其中`__iter__`方法返回一个对象，这个对象得实现`__next__`方法，在这里这个对象也就是自己；`__next__`方法要返回下一个值，或者抛出`StopIteration`的中断，告诉程序这个迭代器玩完了。所以其实只要一个对象实现了`__iter__`和`__next__`两个方法，它就成为了一个“迭代器”，自动地支持`for...in`语法。因此上面的迭代器就可以这么用：

```python
for i in SomeIterator():
    print(i) #1, 2, 3, ...
```

是不是挺神奇的？通过自己实现对象中的特殊方法，你似乎有能力改变Python语法对你所编写的对象的操作方式，而这就是Python的各种对象都能自然地融入语法的原因之一。你将发现，以迭代器为例，Python通过这种方式带给了无数人写的无数Python代码以高度的一致性与拓展性，你既可以决定你的对象应该有怎样的表现，也可以用统一的语法去使用别人定义的对象。理解这一点，是写出`Pythonic Code`[^4]的关键。

[^4]:Python有自己的设计哲学，符合这种哲学的代码才能称之为`Pythonic Code`。优雅，重要得很。

### 数据结构

这段笔者不准备展开详解，请你参照其它各种材料自行学习。这里要求能够完全掌握的类型有序列`list, tuple, namedtuple, str, bytes`和字典`dict`。

### 语法糖

>  语法糖 (Syntactic sugar)是由英国计算机科学家彼得·蓝丁发明的一个术语，指计算机语言中添加的某种语法，这种语法对语言的功能没有影响，但是更方便程序员使用。 语法糖让程序更加简洁，有更高的可读性。
> ——WikiPedia

Python的语法糖使得Python在处理很多基本操作时有着更简明的方式，看起来爽，用起来也爽。这里特别介绍一下：切片和列表生成式。

#### 切片 

切片的目的在于从一段序列中取出一段来，它调用的是对象的`__getitem__`方法，其语法形如：

```python
mylist = [2, 4, 6, 8]
mylist[1:2] #[4, 6]
```

请注意，切片和区间操作不包括范围里的最后一个元素，即`[start, stop)`[^5]。这样设计的意义首先是方便计算切片的长度，其次这样可以用同一个下标把切序列成两半而不会有重叠。所有区间都采用[start, stop)的形式的好处有[^6]：

1. 能用自然数做上下界取到任何自然数
2. 便于切割序列
3. 便于取单个元素
4. 便于计算长度：从0开始计数时[0, N)表示长度为N的序列

多维切片和省略看Tentative NumPy Tutorial[^7]吧，标准库里并没有这种操作。

#### 列表推导和生成器表达式

列表推导是一个快速从原序列中按某种要求生成新序列的语法，形如：

```python
mylist = [1, 2, 3, 4, 5]
odd_number = [x for x in mylist if x%2==0] #[2, 4]
large_number = [x for x in mylist if x>3]  #[4, 5]
possible_sum = [x+y for x in mylist for y in mylist]  #[2, 3, 4, 5, 6, 3, 4, 5, 6, 7, 4, 5, 6, 7, 8, 5, 6, 7, 8, 9, 6, 7, 8, 9, 10]
```

发现了没？原来需要你写一个for里面套一个if再做好多`push`操作的算法，现在只需要一行了。甚至如`possible_sum`那行所展示的那样，你可以嵌套好几个`for`或者好几个`if`，if you like. 而这时，另外一个非常类似却又不同的语法出现了：

```python
possible_sum_generator = (x+y for x in mylist for y in mylist)
```

什么嘛，只是方括号变成了圆括号？当然不是，圆括号包起来的，叫做生成器表达式，它实现的效果和上面的列表推导非常相似，只是有一点不同——它不是返回一个新的序列，而是返回一个生成器。

生成器讲简单点就是一个好好的函数不用`return`返回数据而是用`yield`，就像这样：

```python
def myGenerator():
    for i in range(10):
        yield 2*i
```

然后调用这个“生成器”会返回一个“迭代器”，也就是上面用来`for...in`的那个玩意。在每次迭代时，它会运行到`yield`的地方暂停并返回一个值[^8]，然后下一次迭代时再继续这个程序到下一个`yield`。比如下面的代码：

```python
even = myGenerator()
for i in even:
    print(i)# 0, 2, 4,...
```

看起来没什么特别的，但`0, 2, 4`这样的值都是在每次迭代的时候才被计算出来，而不是在迭代前就全部算出来了一个个轮流赋值给`i`。因此生成器就是每次只产出一个值，而不像普通的函数那样一次性就把所有的结果全计算好了。它的意义在于描述某种规律。比如无穷无尽的斐波那契数列，如果要用函数实现并要进行计算的话函数一次性就得全部计算完并输出，而生成器就可以按你的需求输出相应数列元素的值。

理解了生成器之后，生成器表达式不过是一个快速构建生成器的方式罢了。此外一提，虽然我们说生成器可以随取随用，随关随停，但是如果你就非得想一次性全去完，可以用`*`运算符：

```python
even = myGenerator()
mydict = [*even]
print(mydict)# [0, 2, 4,...]
```

[^5]:~~每次都要纠结半天然后决定随缘，看索引有没有超出上限来调试的日子结束了！~~
[^6]: Why numbering should start at zero. http://www.cs.utexas.edu/users/EWD/transcriptions/EWD08xx/EWD831.html
[^7]: Tentative NumPy Tutorial. https://scipy.github.io/old-wiki/pages/Tentative_NumPy_Tutorial
[^8]:如果你有机会接触协程，你会发现yield还可以接受一个值，但你目前还不需要掌握

### 作业

编写一个电子诗人`NormalPoet`类，要求其具有`vocabulary`和`grammar`属性，实现`get_random_word(self, wordlist)`、`nianshi(self)`、`__iter__(self)`、`__next__(self)`方法。程序的主体已经给出，请把`pass`改成相应的代码。

```python
# -*- coding: UTF-8 -*-
import random

class NormalPoet:
    def __init__(self, grammar, vocabulary):
        '''这里把grammar, vocabulary保存成对象的属性'''
        pass
    def get_random_word(self, wordlist):
        '''这里传入一个list，然后返回这个list中的任意元素'''
        pass
    def nianshi(self):
        '''这里通过get_random_word随机取一种grammar，然后用string.replace替换grammar中的成分'''
        pass
    def __iter__(self):
        '''返回一个迭代器'''
        pass
    def __next__(self):
        '''返回一句诗'''
        pass


if __name__ == '__main__':
    
    # init grammar and vocabulary
    grammar = ['MM在DD', 'XX的MM在DD', 'MM是XX的', 'TTXX的MM', '从MM到MM']
    vocabulary = {}
    vocabulary['DD'] = ['蒸发', '磁化', '撕杀', '氧化', '互相残杀', '升华', '雾化', '自杀', '挥发', '被磁化', '自杀', '谈话', '磨擦', '说梦话', '说胡话']
    vocabulary['DJ'] = ['刺杀', '喝下', '咒骂', '磨擦', '轰炸', '刮', '溶化', '痛打', '磁化', '淡化', '大骂', '谋杀', '锤打', '攻打', '冲刷', '润滑', '暗杀', '拧开']
    vocabulary['TT'] = ['啊！', '噢！', '嘘！', '唉！', '呀！', '天啊！', '哈哈哈哈！']
    vocabulary['XX'] = ['哗哗啦啦', '宏大', '无穷大', '高大', '光滑', '无限大']
    vocabulary['MM'] = ['爸爸', '犹大', '大坝', '火把', '蒙娜丽莎', '尾巴', '酒吧']

    # create a poet
    poet = NormalPoet(grammar, vocabulary)
    
    # use loop to nianshi
    for i in range(10):
        print(poet.nianshi())
```

#### 参考链接

下面给出的是你很可能会用到的一些函数的介绍。

> string.replace()
> https://www.geeksforgeeks.org/python-string-replace/ 
>
> dict.keys()
> https://www.runoob.com/python/python-dictionary.html 
>
> 更多词汇和语法（上面题里截取了一部分）
> https://github.com/iwestlin/lab/blob/master/poet/main.js 
>
> 想法及实现的思路来自于刘慈欣写的电子诗人
>  http://blog.sina.com.cn/s/blog_540d5e800101aikv.html 

#### 文学作品欣赏

> 高楼群被呼唤了
>
> 我看到有包装的染色体在打呼噜彩云在引诱着国境
>
> 我纤细得成长
>
> 有曲线美的球门和潮湿的铁人和黑暗的酒杯还有黑乎乎的星期日都在绷紧
>
> 大西州被嘲讽了
>
> 我面对着高电压的进行曲和低下的圆柱体
>
> 我感慨了
>
> 你是船夫你是黄金般的冰河时代
>
> 我面对着无希望的信徒和闷热的新生儿
>
> 失落的野马从罗浮宫飞来