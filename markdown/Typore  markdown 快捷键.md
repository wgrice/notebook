## windows快捷键

- 无序列表：输入-之后输入空格
- 有序列表：输入数字+“.”之后输入空格
- 任务列表：-[空格]空格 文字
- 标题：ctrl+数字
- 表格：ctrl+t
- 生成目录：[TOC]按回车
- 选中一整行：ctrl+l
- 选中单词：ctrl+d
- 选中相同格式的文字：ctrl+e
- 跳转到文章开头：ctrl+home
- 跳转到文章结尾：ctrl+end
- 搜索：ctrl+f
- 替换：ctrl+h
- 引用：输入>之后输入空格
- 代码块：ctrl+alt+f
- 加粗：ctrl+b
- 倾斜：ctrl+i
- 下划线：ctrl+u
- 删除线：alt+shift+5
- 插入图片：直接拖动到指定位置即可或者ctrl+shift+i
- 插入链接：ctrl + k

## 给代码块设置快捷键

偏好设置 -> 打开高级设置 -> conf.user.json 文件

```json
Copy  "keyBinding": {
    // for example: 
    // "Always on Top": "Ctrl+Shift+P"
    "Always on Top": "Ctrl+Shift+P",  
    "Code Fences": "Ctrl+Shift+F",  
    "Ordered List":"Ctrl+Alt+o",  
    "Unordered List": "Ctrl+Alt+u"  
  },
```

Code Fences 代码块

Ordered List 数字有序列表

Unordered List 无序列表

## Mac中的快捷键

1. 最大标题：command + 1 或者：#
2. 大标题：command + 2 或者：##
3. 标准标题：command + 3 或者：###
4. 中标题：command + 4 或者：####
5. 小标题：command + 5 或者：#####
6. 插入表格：command + T
7. 插入代码：command + alt +c
8. 行间公式 command + Alt + b
9. 段落：command + 0
10. 竖线 ： command + Alt +q
11. 有序列表（1. 2.） ：输入数字+“.”之后输入空格 或者：command + Alt + o
12. 黑点标记：command + Alt + u
13. 隔离线shift + command + -
14. 超链接：command + Alt + l
15. 插入链接：command +k
16. 下划线：command +u
17. 加粗：command +b
18. 搜索：command +f

## 图片

[![img](C:\wg\project\git\notebook\markdown\Typore  markdown 快捷键.assets\443934-20181012170159282-378811511.png)](https://img2018.cnblogs.com/blog/443934/201810/443934-20181012170159282-378811511.png)
[![img](C:\wg\project\git\notebook\markdown\Typore  markdown 快捷键.assets\443934-20181012170211920-1988294604.png)](https://img2018.cnblogs.com/blog/443934/201810/443934-20181012170211920-1988294604.png)

## 表情

输出表情需要借助 `：`符号。

栗子：`:smile` 显示为 😄,记住是左右两边都要冒号。

使用者可以通过使用`ESC`键触发表情建议补全功能，也可在功能面板启用后自动触发此功能。同时，直接从菜单栏`Edit` -> `Emoji & Symbols`插入UTF8表情符号也是可以的。

或者使用下面的方法

访问网站 https://emojikeyboard.org/，找到需要的符号，鼠标左键单击，然后粘贴到需要的地方就行了！🆗

## 数学公式

你可以通过使用**MathJax**来实现*LaTeX*的数学符号的表达。

输入`$$`，然后按下`Enter`键就会弹出一个支持TeX/LaTeX语法的输入框，下面是一个栗子：

V1×V2=∣∣ijk ∂X∂u∂Y∂u0 ∂X∂v∂Y∂v0 ∣∣V1×V2=|ijk ∂X∂u∂Y∂u0 ∂X∂v∂Y∂v0 |


在Markdown源文件中，数学的公式块是通过利用标记借用语言来实现的：

```
Copy$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$
```

## HTML

Typora不能使用HTML元素，但是Typora可以解析和编译非常有限的HTML元素，作为Markdown功能的补充，这些有限的功能包括：

- 下划线： `<u>underline</u>`
- 图片：`<img src="http://www.w3.org/html/logo/img/mark-word-icon.png" width="200px" />`（HTML标签中的`width`, `height` 以及属于样式的`width`, `height`, `zoom`样式可以被识别和应用。）
- 评论：`<!-- This is some comments -->`
- 超链接： `<a href="http://typora.io" target="_blank">link</a>` 。

大多数这些属性、样式或分类会被忽略。对其他的标签，Typora会将它们以HTML片段的形式表达。

## 行内嵌数学符号

想要使用这个功能，需要在设置面板的 `Markdown`栏启用它。然后使用`$`来启动TeX命令，栗如：`$\lim_{x \to \infty} \exp(-x) = 0$` 会以LaTeX的命令形式表达出来。

为了触发行内内嵌数学符号的实时编译你需要：输入`$`然后按下`ESC`键之后输入TeX命令，之后就会弹出一个如图所示的工具提示栏：

[![img](C:\wg\project\git\notebook\markdown\Typore  markdown 快捷键.assets\v2-4033508b043cad96c59ec4edbca92f36_b.gif)](https://pic3.zhimg.com/v2-4033508b043cad96c59ec4edbca92f36_b.gif)

## 下标

想要使用这个功能，需要在设置面板的 `Markdown` 栏启动它，之后使用`~`来修饰下标文本。栗如：

`H~2~O` 和`X~long\ text~` 显示为 H~2~O 和X~long text~ 。

\#### 13.上标

想要使用这个功能，需要在设置面板的 `Markdown` 栏启动它，之后使用`^`来修饰下标文本。栗如：

`X^2^` 显示为 X^2^ 。

## 高亮

想要使用这个功能，需要在设置面板的`Markdown` 栏启动它，之后使用`==`来修饰高亮文本，栗如：

`==highlight==` 显示为 ==highlight== 。