## 1. 序列图（*关键字：sequence*）

``` sequence
title: 三个臭皮匠的故事
participant 小王
participant 小李
participant 小异常

note left of 小王: 我是小王
note over 小李: 我是小李
note right of 小异常: 大家好！\n我是小异常

小王->小王: 小王想：今天要去见两个好朋友咯~
小王->小李: 嘿，小李好久不见啊~ 
小李-->>小王: 是啊
小李->小异常: 小异常，你好啊
小异常-->小王: 哈，小王！\n最近身体怎么样了？
小王->>小异常: 还可以吧
```
### 1、关键字
　　1) title
　　表示该序列图中的标题。

　　2) participant
　　表示该序列图中的对象。

　　3) note
　　表示该序列图中的部分说明。关于note以下三种关键字： 
　　	* left of：表示在当前对象的左侧;
　　	* right of：表示在当前对象的右侧;
　　	* over：表示覆盖在当前对象的上方;

### 2、箭头
　　1）->：实线实箭头
　　2）-->：虚线实箭头
　　3）->>：实线虚箭头
　　4）-->>：虚线虚箭头

### 3、换行
　　如果当前行中的文字过多想要换行，可以使用 \n 进行转义换行，效果如以上例子。

## 2. 流程图（*关键字：flow*）

```flow
sta=>start: 开始|past:>http://www.google.com[blank]
e=>end: 结束
op=>operation: 操作（处理块）
sub=>subroutine: 子程序
cond=>condition: 是或者不是（条件判断）?
cond2=>condition: 第二个判断（条件判断）?
io=>inputoutput: 输出

sta->op->cond
cond(yes)->e
cond(no)->cond2
cond2(yes,right)->sub(left)-op
cond2(no)->io(lef)->e
```
### 1、关键字
1) start, end
　　表示该流程图中的开始与结束。

2) operation
　　表示该流程图中的处理块。

3) subroutine
　　表示该流程图中的子程序块。

4) condition
　　表示该流程图中的条件判断。

5) inputoutput
　　表示该流程图中的输入输出。

6) right, left
　　表示该流程图中当前模块下一个箭头的指向（默认箭头指向下方）。

7) yes, no
　　表示该流程图中条件判断的分支（默认yes箭头向下no箭头向右；yes与no可以和right同时使用；yes箭头向右时，no箭头向下）

### 2、各模块之间的联系
1) 形式：
　基本形式：

```
模块标识=>模块关键字: 模块模块名称
```
　连接定义：
```
模块标识1->模块标识2
模块标识1->模块标识2->模块标识3
... ...
```
2) 说明：
　　通过模块与连接定义，可以组成一个完整的流程图。 
　　在模块定义中，模块标识与模块名称可以自定义，模块关键字不可以自定义！

### 3、注意事项
1) 在进行连接的时候，可以通过right, left确定箭头的指向；
2) 使用条件判断的连接时需要结合yes和no进行；
3) 在连接各模块之间不能有空格，在模块标识关键字时也不能有空格。

