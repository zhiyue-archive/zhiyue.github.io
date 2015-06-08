title: "用python验证蒙提霍尔问题"
date: 2015-05-19 02:55:49
categories: Python
tags: [概率论,三门问题,Python]
---
最初看到这个问题是初中的时候买了一本有关数学谜题的书里面概率论的一张的课后拓展就是说到三门问题，当时作为一个扩展阅读看了一下，里面说到了一个世界智商最高的女人秒杀了美国一大群的数学高材生的精彩故事（比较夸张），当时对这个问题也是似懂非懂。

<!--more-->

## 什么是蒙提霍尔问题？
![蒙提霍尔][1]
> 蒙提霍尔问题，亦称为蒙特霍问题或三门问题（英文：Monty Hall problem），是一个源自博弈论的数学游戏问题，大致出自美国的电视游戏节目Let's Make a Deal。问题的名字来自该节目的主持人蒙提·霍尔（Monty Hall）。

最初的表述是：

> 参赛者会看见三扇关闭了的门，其中一扇的后面有一辆汽车，选中后面有车的那扇门就可以赢得该汽车，而另外两扇门后面则各藏有一只山羊。当参赛者选定了一扇门，但未去开启它的时候，节目主持人开启剩下两扇门的其中一扇，露出其中一只山羊。主持人其后会问参赛者要不要换另一扇仍然关上的门。
问题是：换另一扇门会否增加参赛者赢得汽车的机会率？

这个古老的问题一经提出就引起了剧烈的争论，有人认为换与不换最终得到车的概率都是$\frac{1}{2}$，有人认为换门之后得到车的概率更大，应该选择换门之后得到车的概率为$\frac{2}{3}$在撰写这篇文章的时候在[果壳上][2]还有人在为此争吵，[知乎上][3]也有许多关于这方面的讨论，其实这些争论很多情况下都是因这个问题的模糊表述所引起的，关键点在于**主持人对于门后的情况是否了解**：

1. 如果主持人事先知道哪个门里有山羊并且他特意选择了有山羊的门打开了，那么参赛者应该换另一扇门，这可以将他胜利的概率从$\frac{1}{3}$升到$\frac{2}{3}$
2. 如果主持人事先不知道哪个门里有山羊或者他只是随机的选择了一个门，但事实发现里面恰好是山羊。这时候参赛者没有换门的必要，胜利概率总是$\frac{1}{2}$

为了后续的讨论，这里采用[维基百科][4]上对于这一个问题的不含糊的定义

严格的表述如下：

- 参赛者在三扇门中挑选一扇。他并不知道内里有什么。
- 主持人知道每扇门后面有什么。
- 主持人必须开启剩下的其中一扇门，并且必须提供换门的机会。
- 主持人永远都会挑一扇有山羊的门。
    - 如果参赛者挑了一扇有山羊的门，主持人必须挑另一扇有山羊的门。
    - 如果参赛者挑了一扇有汽车的门，主持人随机在另外两扇门中挑一扇有山羊的门。
- 参赛者会被问是否保持他的原来选择，还是转而选择剩下的那一道门。

那么这个问题这可以很好的理解了，引用维基的一幅图片解析：
![蒙提霍尔解答][5]

有三种可能的情况，全部都有相等的可能性（$\frac{1}{3}$）：

- 参赛者挑汽车，主持人挑两头羊的任何一头。转换将失败。
- 参赛者挑A羊，主持人挑B羊。转换将赢得汽车。
- 参赛者挑B羊，主持人挑A羊。转换将赢得汽车。

所以玩家选择换门之后获胜的概率应为$\frac{2}{3}$

## 证明？


![蒙提霍尔解答][6]
**定义**: 
- `事件A`为一开始玩家选择的一扇门
- `事件H`为最后门后的结果


- 如果是选择不换门的策略
$P \left(H=car \right) = P \left(A=car \right) = \frac{1}{3} $
因为选择的是不交换的策略，所有只有一开始选中的是汽车，最后才能选中汽车。

- 选择交换门的策略
$P \left(H=car \right) = P \left(A=sheep \right) = \frac{2}{3} $
因为选择的是交换的策略，所有只有一开始选中的是羊，最后才能选中汽车。

## 程序验证

实践是检验真理的唯一标准，在流言终结者看到他们人工重复这个实验区验证，发现这样很浪费时间。何通过计算机去去模拟这一段过程呢？
下面使用python程序来模拟这一段过程：

``` python
from __future__ import division
import logging
from matplotlib import pyplot as plt
import numpy as np
import random


class MontyHall(object):
    """docstring for MontyHall"""

    def __init__(self, num=3):
        """
    	创建一个door列表
    	0 代表关门
    	1 表示后面有车
    	-1 代表门被打开
        """
        super(MontyHall, self).__init__()
        self.doors = [0] * num
        self.doors[0] = 1
        self.choice = -1
        self.exclude_car = False
        self.shuffle()

    def shuffle(self):
        """  
        开始新游戏
        重新分配门后的东西
        """
        if self.exclude_car == True:
            self.doors[0] = 1
            self.exclude_car = False
        for i in xrange(len(self.doors)):
            if self.doors[i] == -1:
                self.doors[i] = 0
        random.shuffle(self.doors)

    def make_choice(self):
        """
        player随机选择一扇门
        """
        self.choice = random.randint(0, len(self.doors) - 1)
        logging.info("choice: %d" % self.choice)
        logging.info("original: %s" % self.doors)

    def exclude_doors(self):
        """
        主持人知道门后的情况排除门
        直到剩余两扇门
        """
        to_be_excluded = []
        for i in xrange(len(self.doors)):
            if self.doors[i] == 0 and self.choice != i:
                to_be_excluded.append(i)  
        random.shuffle(to_be_excluded)
        for i in xrange(len(self.doors) - 2):
            self.doors[to_be_excluded[i]] = -1
        logging.info("final: %s" % self.doors)

    def random_exclude_doors(self):
        """
        主持人并不知道门后面的情况随机的开门
        直到剩余两扇门
        """
        to_be_excluded = []
        for i in xrange(len(self.doors)):
            if self.doors[i] != -1 and i != self.choice:
                to_be_excluded.append(i)  
        random.shuffle(to_be_excluded)
        for i in xrange(len(self.doors) - 2):
            if self.doors[to_be_excluded[i]] == 1:
                self.exclude_car = True
            self.doors[to_be_excluded[i]] = -1
        logging.info("final: %s" % self.doors)

    def change_choice(self):
        """
        player改变选择
        """
        to_change = []
        for i in xrange(len(self.doors)):
            if self.doors[i] != -1 and i != self.choice:
                to_change.append(i)
        self.choice = random.choice(to_change)
        logging.info("choice changed: %d" % self.choice)

    def random_choice(self):
        """
        player 第二次随机选择门
        """
        to_select = []
        for i in xrange(len(self.doors)):
            if self.doors[i] != -1:
                to_select.append(i)
        self.choice = random.choice(to_select)
        logging.info("random choice : %d" % self.choice)


    def show_answer(self):
        """
        展示门后的情况
        """
        logging.info(self.doors)

    def check_result(self):
        """
        验证结果
        """
        got_it = False
        if self.doors[self.choice] == 1:
            got_it = True
        return got_it
```

### 模拟1000轮，每一轮重复试验1000次


- 不改变选择：

``` python
def unchange_choice_test(n):
    """
    不改变初始的选择
    """
    result = {}
    game = MontyHall()
    for i in xrange(n):
        game.shuffle()
        game.make_choice()
        game.exclude_doors()
        if game.check_result():
            result["yes"] = result.get("yes", 0) + 1
        else:
            result["no"] = result.get("no", 0) + 1
    for key in result:
        print "%s: %d" % (key, result[key])
    return result["yes"] / n

if __name__ == '__main__':
    logging.basicConfig(format='%(levelname)s:%(message)s', level=logging.WARNING)
    results = []
    test_num = 1000
    round_num = 1000
    for x in xrange(0,round_num):
        results.append(change_random_test(test_num) )

    y_mean = np.mean(results)
    y_std = np.std(results)
    x = range(0,round_num)
    y = results
    plt.figure(figsize=(8,4))
    
    plt.xlabel("round")
    plt.ylabel("frequency")
    plt.title("The frequency of the success")
    tx = round_num / 2
    ty = y_mean
    label_var = "$\sigma \left( X \\right)=$%f" % y_std
    label_mean = "$ X =$%f" % y_mean
    p1_label = "%s and %s" % (label_var,label_mean)
    p1 = plt.plot(x,y,"-",label=p1_label,linewidth=2)
    plt.legend(loc='upper left')
    

    pl2 = plt.figure(2)
    plt.figure(2)
    plt.hist(results,40,normed=1,alpha=0.8)
    plt.show()
```
结果：
![此处输入图片的描述][7]
概率分布：
![此处输入图片的描述][8]
成功的概率均值在 $\frac{1}{3}$ 附近

- 改变选择：

``` python
def change_choice_test(n):
    """
    交换选择的门
    """
    result = {}
    game = MontyHall()
    for i in xrange(n):
        game.shuffle()
        game.make_choice()
        game.exclude_doors()
        game.change_choice()
        if game.check_result():
            result["yes"] = result.get("yes", 0) + 1
        else:
            result["no"] = result.get("no", 0) + 1
    for key in result:
        print "%s: %d" % (key, result[key])
    return result["yes"] / n
```
同样的方法绘图得到结果：
![此处输入图片的描述][9]
概率分布:
![此处输入图片的描述][10]
成功的概率均值在 $\frac{2}{3}$ 附近

> 通过上面的分析与模拟可知最佳的策略当然就是换门。

## 更加深入的讨论


- **如果门的数量不止是3个，如果是50扇门呢？**

![此处输入图片的描述][11]
这种情况下，主持人打开48扇都是羊的门后，再给你选择，很多人这个时候应该就不会固守那$\frac{1}{2}$，而会选择换门
把门的数据增大到100,1000，这种情况会更加明显。
还是通过一段程序模拟说明：

``` python
def change_choice_test_large(n,m):
    """
    交换选择的门
    """
    result = {}
    game = MontyHall(m)
    for i in xrange(n):
        game.shuffle()
        game.make_choice()
        game.exclude_doors()
        game.change_choice()
        if game.check_result():
            result["yes"] = result.get("yes", 0) + 1
        else:
            result["no"] = result.get("no", 0) + 1
    for key in result:
        print "%s: %d" % (key, result[key])
    return result["yes"] / n
    
    
if __name__ == '__main__':
    logging.basicConfig(format='%(levelname)s:%(message)s', level=logging.WARNING)
    results = []
    test_num = 1000
    round_num = 1000
    for x in xrange(0,round_num):
        results.append(change_choice_test_large(test_num,50) )
```
结果：
![此处输入图片的描述][12]
![此处输入图片的描述][13]

这时候就要选择**交换门**。

- **遇到这种情况我很困惑，我决定抛硬币决定，这个时候成功的概率？**

这是第3种策略，成功的概率和硬币有关，也就是$\frac1 2$,这种情况就是从剩下的门中随机选择一扇，这个策略从上面分析来看不是最好的，但是比不改变的策略要好。
程序的模拟结果：
![此处输入图片的描述][14]
![此处输入图片的描述][15]

- 比如门意外打开的情况呢，也就是上面描述的第二种情况（主持在不知门后的情况下打开门呢）？

这种情况下其实就是一个条件概率，事件A是玩家最后开到的是车，事件B是主持人打开的门是羊。
$$
P(A|B) = \dfrac{P(B|A) \cdot P(A) }{P(B)}
$$
因为只有主持人开到是羊的情况下，玩家才有可能开到车所以 $P(B|A) = 1$
设玩家第一次选择的门为`事件C`则

- 不交换策略下的条件概率是：
$$
P(B) = P(C='汽车') + P(C='羊') \times \frac {1}{2}
\Rightarrow P(B)= \frac{1}{3} + \frac{1}{3} = \frac{2}{3}
$$

$$
P(A) = P(C='汽车') = \frac{1}{3}
$$

$$
P(A|B) = \dfrac{P(A) }{P(B) } = \dfrac{\frac{1}{3}}{\frac{2}{3}}= \frac{1}{2}
$$

- 交换策略下的条件概率是：
$$
P(B) = P(C='汽车') + P(C='羊') \times \frac {1}{2}
\Rightarrow P(B)= \frac{1}{3} + \frac{1}{3} = \frac{2}{3}
$$

$$
P(A) = P(C='羊') \times \frac{1}{2} = \frac{1}{3}
$$

$$
P(A|B) = \dfrac{P(A) }{P(B) } = \dfrac{\frac{1}{3}}{\frac{2}{3}}= \frac{1}{2}
$$
因此在主持人不知道门后的情况下打开一扇，然后发现门后是羊的情况下，换门与不换门最终的概率都是$\frac{1}{2}$
还是可以通过程序进行模拟：

``` 
def unknown_doors_choice_test(n):
    """
    主持人并不知道门后面的情况随机的开门
    交换选择的门
    """
    result = {}
    game = MontyHall()
    continue_count = 0
    for i in xrange(n):
        game.shuffle()
        game.make_choice()
        game.random_exclude_doors()
        game.change_choice()
        if game.exclude_car == False:
            continue_count += 1
        if game.check_result():
            result["yes"] = result.get("yes", 0) + 1
        else:
            result["no"] = result.get("no", 0) + 1
    #for key in result:
    #    print "%s: %d" % (key, result[key])
    logging.info("continue_count: %d" % continue_count)
    if continue_count == 0:
        return 0.0
    return result["yes"] / continue_count   
```
![此处输入图片的描述][16]
![此处输入图片的描述][17]
在这种情况下交换门也没有提升成功的概率

---
## 总结

今天写的这篇东西也算是了解我童年的一个遗憾，人的直觉有时候是很不可靠，要摆脱个人局限的认知才能拥抱更大的世界。
什么？看完这些解析，你还觉得不满意那么你还可以从下面的参考中寻找更好的解析，本文撰写过程有部分的图片引用自一下的参考，如果你还有疑问欢迎你联系我进一步的讨论。

## 练习
下面是三门问题的两个翻版，引用自[三门问题及相关][18]：

### 女孩的概率

- 你结交一位新朋友，问她是否有孩子。她说有，有两个。你问，有女孩吗？她说有。那么，两个都是女孩的概率是多少？
 > 答：三分之一。因为生两个孩子的可能性有四种等可能：BB、GG、BG、GB（即男男、女女、男女、女男）。 因为我们已知至少有一个女儿，所以BB是不可能的。因此GG是可能出现的三个等可能的结果之一，所以两个孩子都是女儿的概率为三分之一。这对应了三门问题的第一种情况。

-  你结交一位新朋友，问她是否有孩子。她说有，有两个。你问，有女孩吗？她说有。第二天，你看见她带了一个小女孩。你问她，这是你女儿吗？她说，是。她的两个孩子都是女孩的概率是多少？
>这个概率和生女孩的概率相同，二分之一。这似乎非常奇怪，因为我们所拥有的信息看起来并不比第一种情况时多，但概率却不同。但是这里的问题其实是，那个你没>见过的孩子是女孩的概率是多少？这个概率和生女孩的概率相同，二分之一。
这对应了三门问题的第二种情况。当然这里也有语言问题，必须假定这位母亲不是特定带出一个小女孩来给你看的。也就是说你只是碰巧发现了它是位小女孩。这取决于是判断选择 或q 随机选择。如果是被你碰巧撞见这是属于随机选择。这就对应了三门问题的第二种情况。这其实是增加了信息的。否则如果她主动带一个小女孩过来给你，则属于判断选择。
你得到的答案依赖于所讲的故事；它依赖于你是如何得知至少一个孩子是女孩的。

### 三囚犯问题

- 亚当、比尔和查尔斯被关在一个监狱里，只有监狱看守知道谁会被判死刑，另外两位将会获释。有1／3的概率会被处死刑的亚当，给他母亲写了一封信，想要获释的比尔或查尔斯帮忙代寄。当亚当问看守他应当把他的信交给比尔还是查尔斯时，这位富有同情心的看守很为难。他认为如果他把将要获释的人的名字告诉亚当，那么亚当就会有1／2的概率被判死刑，因为剩下的人和亚当这两人中一定有一个人被处死。如果他隐瞒这信息，亚当被处死的概率是1／3。既然亚当知道其他两人中必有一人会获释，那么亚当自己被处死的概率怎么可能会因为看守告诉他其他两人中被获释者的姓名后而改变呢？ 
> 正确的答案是：看守不用当心，因为即使把获释人的姓名告诉亚当，亚当被处死的概率仍然是1／3，没有改变。但是，剩下的那位没被点名的人就有2／3的概率被处死（被处死的可能性升高了）。如果这个问题换一种说法，就是看守无意间说出了查尔斯不会死。那么概率就会发生改变。

> 这个其实和三门问题是一致的。你可以把狱卒当成主持人，被处死当成是大奖，那么这个是对应于三门问题的第一种情况，就是主持人知道门后面的情况。狱卒说出谁会被释放，相当于主持人打开一扇门。但是因为三囚徒问题不能选择，也就相当于三门问题中的不换门的策略。最终的概率还是1/3是没有发生改变的。

> 为了避免产生歧义，规定一下：
1.如果（亚当，查尔斯）被释放，那么狱卒会告诉亚当："查尔斯被释放"。
2.如果（亚当，比尔）被释放，那么狱卒会告诉亚当："比尔被释放"
3.如果（查尔斯，比尔）被释放，那么狱卒会以1/2的概率告诉亚当："查尔斯被释放"或者"比尔被释放"

> 意思就很明显了，在狱卒说出比尔被释放的条件下，亚当被释放的概率是？用条件概率算一下。

> 定义事件：
A ：狱卒说出"比尔被释放"
B ：代表亚当被释放。

>$$
P(A) =  \frac{1}{2}
$$
$$
P(A \cap B) =  \frac{1}{3}
$$
$$
P(B|A)=\frac{P(A \cap B)}{P(A)}= \frac{2}{3}
$$
> 那什么时候才是1/2的概率呢？
规则3更改为：如果（查尔斯，比尔）被释放，那么狱卒会告诉亚当"比尔被释放"
这个时候计算就是： $$  P(B|A)=\frac{P(A \cap B)}{P(A)}= \frac{\frac{1}{3}}{\frac{2}{3}}   =\frac{1}{2} $$

>那如果规则3改为：如果（查尔斯，比尔）被释放，那么狱卒会告诉亚当"查尔斯被释放"
这个时候：亚当被释放的概率就会变为1
问题在于规则2和规则3下说"比尔被释放"不是等概率发生的。


### 类似的问题还有

- 抛两枚硬币其中有一枚硬币是正面，问两枚硬币都是正面的概率是？
- 抛两枚硬币其中第一枚硬币是正面，问两枚硬币都是正面的概率是？


the end.

---


## 参考:


1. [蒙提霍尔问题 - 维基百科，自由的百科全书](http://zh.wikipedia.org/wiki/%E8%92%99%E6%8F%90%E9%9C%8D%E7%88%BE%E5%95%8F%E9%A1%8C)

2. [三扇门问题 | 左岸读书](http://www.zreading.cn/archives/711.html)

3. [蒙提霍尔问题（又称三门问题、山羊汽车问题）的正解是什么？](http://www.zhihu.com/question/26709273?rf=22113980)

4. [趣味编程：三门问题](http://www.cnblogs.com/twocats/p/3440398.html)

5. [三门问题及相关](http://zhiqiang.org/blog/science/three-doors-related-problems.html)


6. [换还是不换？争议从未停止过的三门问题](http://www.guokr.com/post/9314/)

7. [在「三门问题」中，参与者应该选择「换」还是「不换」？主持人是否知道门后情形对结论有何影响？](http://www.zhihu.com/question/19825086)
8. [THE MONTY HALL PROBLEM][19]
9. [流言终结者第九季][20]
10. [某个家庭中有 2 个小孩，已知其中一个是女孩，则另一个是男孩的概率是多少？-知乎][21]
11. [从贝叶斯定律的角度理解“蒙提霍尔问题”和“三个囚犯问题”][22]
12. [三个囚犯问题，求解？][23]

---

更新日志：

- 2015-05-20  增加三囚徒问题的解答
- 2015-05-09 第一次撰写


  [1]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/24200658-c442dec3691a4e929938ba98a52d78c5.png
  [2]: http://www.guokr.com/post/9314/
  [3]: http://www.zhihu.com/search?q=%E4%B8%89%E9%97%A8%E9%97%AE%E9%A2%98&type=question
  [4]: http://zh.wikipedia.org/wiki/%E8%92%99%E6%8F%90%E9%9C%8D%E7%88%BE%E5%95%8F%E9%A1%8C
  [5]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/zhiyue_2015-05-09_164731.png
  [6]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/564.jpg
  [7]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/%E4%B8%BB%E6%8C%81%E4%BA%BA%E7%9F%A5%E9%81%93%E9%97%A8%E5%90%8E%E4%B8%8D%E6%94%B9%E5%8F%98.png
  [8]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/%E4%B8%BB%E6%8C%81%E4%BA%BA%E7%9F%A5%E9%81%93%E9%97%A8%E5%90%8E%E6%83%85%E5%86%B5%E4%B8%8D%E6%94%B9%E5%8F%98.png
  [9]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/%E4%B8%BB%E6%8C%81%E4%BA%BA%E7%9F%A5%E9%81%93%E6%94%B9%E5%8F%98.png
  [10]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/%E6%94%AF%E6%8C%81%E4%BA%BA%E7%9F%A5%E9%81%93%E6%94%B9%E5%8F%98.png
  [11]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/24200702-dd4e3b818eee4ae680cb357ce903d8e0.png
  [12]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/large_num.png
  [13]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/large_num2.png
  [14]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/%E4%B8%BB%E6%8C%81%E4%BA%BA%E7%9F%A5%E9%81%93%E9%9A%8F%E6%9C%BA%E9%80%89%E6%8B%A9.png
  [15]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/%E4%B8%BB%E6%8C%81%E4%BA%BA%E7%9F%A5%E9%81%93%E9%9A%8F%E6%9C%BA%E9%80%89%E6%8B%A9%E9%A2%91%E7%8E%87%E5%88%86%E5%B8%83.png
  [16]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/%E4%B8%BB%E6%8C%81%E4%BA%BA%E4%B8%8D%E7%9F%A5%E9%81%93%E9%97%A8%E5%90%8E%E7%9A%84%E6%83%85%E5%86%B5%E4%B8%8D%E6%94%B9%E5%8F%98.png
  [17]: http://7xilqm.com1.z0.glb.clouddn.com/Monty%20Hall%20problem/%E4%B8%BB%E6%8C%81%E4%BA%BA%E4%B8%8D%E7%9F%A5%E9%81%93%E9%97%A8%E5%90%8E%E9%9D%A2%E7%9A%84%E6%83%85%E5%86%B5%E4%B8%8D%E6%94%B9%E5%8F%98.png
  [18]: http://zhiqiang.org/blog/science/three-doors-related-problems.html
  [19]: http://www.letsmakeadeal.com/problem.htm
  [20]: http://www.bilibili.tv/video/av267091/index_21.html
  [21]: http://www.zhihu.com/question/27534611
  [22]: http://blog.sciencenet.cn/blog-624263-777817.html
  [23]: http://www.zhihu.com/question/24276164
