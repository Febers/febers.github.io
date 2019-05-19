title: Markdown 语法详解
date: 2019-05-19 15:36:02
tags: 
- Markdown
categories: 
- Markdown
---

## 引言

这篇文章本应该在搭建博客之后就发布，一开始觉得 Markdown  的语法足够简单，熟能生巧，无需花费篇幅去记录；近来无事，反省了一下自己的错误认识，除去一些高级用法，了解这门用途广泛的标记语言的由来与发展，回顾它的基础语法如何将排版变成一件充满乐趣的事，完全值得专门写一篇文章。

![logo](Markdown-语法详解\markdown.png)

<!--more-->

## 关于 

Markdown 是一门轻量级的标记语言，由美国工程师 John Gruber 于2004年创造。这门语言的目的是让人们`使用易于阅读、易于撰写的纯文字格式，并选择性地转换成有效的 XHTML（或是 HTML）`。

Markdown 所谓的**易读**并不是指排版之后呈现的结果易读，而是指原始格式下的文件依然拥有优秀的可读性，不会像阅读原始 HTML 代码一样，满眼都是尖括号（可以通过 右键浏览器页面 -> 查看源代码 体验）。Markdown 的**易写**则体现在其语法足够简单，学习曲线平缓，并且在写作中基本可以脱离鼠标操作。

Markdown 的轻量级是相对于 LaTeX 来说的，这种基于 Tex 的排版系统广泛运用在高质量书籍印刷和复杂公式论文中。不过使用 Markdown 仍然可以使用一些基本的数学公式，比如`$ E = mc^2 $`、`$ \int_0^xf(x)dx $`

$ E = mc^2 $

$ \int_0^xf(x)dx $

这需要不同平台上的 Markdown 数学公式插件的支持，本博客使用`Hexo` + `Github Pages`搭建，可以通过安装 [MathJax](https://www.mathjax.org) 实现。

### 分类

跟早期的 HTML 类似，Markdown 在发展的过程中衍生出了不同的版本，它们的基本语法上相通，但是在诸如表格、锚点、时序图等实现上出现了不一致。在关于语法规范化的讨论中，作者 John Gruber 认为，`不同的网站（和人们）有不同的需求，没有一种语法可以让所有人满意`。

现今 Markdown 的主要分类如下

- CommonMark：由 Stack Exchange、Github、Reddit 等组织发起的标准化项目。一开始名称为`Standard Markdown`，由于遭到作者的反对，更名`CommonMark`
- GFM：Github Flavored Markdown，由 Github 于2017年发布，基于 CommonMark。相信很多开发者都是通过一份`README.md`文件认识 Markdown，这也是本博客采用的版本。
- Markdown Extra：基于 PHP、Python 和 Rudy 中实现的 Markdown。

### 编辑器

市面上优秀的 Markdown 编辑器层出不穷，这也有力推动了 Markdown 的发展。可以使用`Sublime Text`配合插件编辑写作，可以使用`Typora`等优秀的跨平台工具实现所见即所得，也可以通过 [Cmd Markdown ](https://www.zybuluo.com)在线书写并导出、发布。

博主使用的是`Typora。`

![编辑器](Markdown-语法详解\编辑器.png)

相关链接：[码字必备：18 款优秀的 Markdown 写作工具 | 2015 年度盘点](https://sspai.com/post/32483)、 [用 Markdown 写作用什么文本编辑器？ - 知乎](https://www.zhihu.com/question/19637157)

## 初级语法

### 标题

```markdown
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

---

### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题

---

另外在 GFM 中，任意 1-6 个 **#** 标注的标题都会被添加上同名的锚点链接，比如`# First Title `会被标注成`[First Title](#first-title)`（注意小写转换），因此我们可以在文章的其他地方，使用标注之后的格式跳转到任何标题，比如`[跳转至引言](#引言)`



### 文字

| 格式                                                 | 效果                   |
| ---------------------------------------------------- | ---------------------- |
| `*斜体1*`                                            | *斜体1*                |
| `_斜体2_`                                            | _斜体2_                |
| `**粗体1**`                                          | **粗体1**              |
| `__粗体2__`                                          | __粗体2__              |
| `~~删除线~~`                                         | ~~删除线~~             |
| `***斜粗体1***`                                      | ***斜粗体1***          |
| `___斜粗体2___`                                      | ___斜粗体2___          |
| `***~~斜粗体删除线1~~***`、`~~***斜粗体删除线2***~~` | ***~~斜粗体删除线~~*** |



### 表情

GFM 语法支持添加 emoji 表情，输入不同的符号码（两个冒号包围的字符）可以显示出不同的表情。比如`:stuck_out_tongue_winking_eye:`：:stuck_out_tongue_winking_eye:

:ghost: :dog: :poop: :fire: :bow:

:interrobang: :rowboat: :watermelon: :swimmer: :fallen_leaf:

可以在此找到不同表情对应的符号码：[Emoji cheat sheet for GitHub, Basecamp, Slack & more](https://www.webfx.com/tools/emoji-cheat-sheet/)



Hexo 默认不支持 emoji 表情，可以通过安装 [hexo-filter-github-emojis](https://github.com/crimx/hexo-filter-github-emojis) 实现



### 分割线

使用三个（或多个连续）的`-`、`*`、`-`实现分割线效果

```markdown
---
___
******
```

------

------

------



### 链接

链接分为文字链接和图片链接B

#### 文字链接

`[ReBe](https://febers.github.io "鼠标悬停显示")`：[ReBe

支持使用标识符标志地址，将真正的URL地址放在文末，比如

```markdown
[Github][Github URL]
[Github URL]:https://github.com/Febers
```

效果如下:

[Github][Github URL]

[Github URL]:https://github.com/Febers

#### 图片链接

基本格式为`![title](url)`，其中`title`可省略，`![](https://camo.githubusercontent.com/abf9d87ce112444bca1ddfffaa2063f02a2c26d0/68747470733a2f2f7261772e6769746875622e636f6d2f6164616d2d702f6d61726b646f776e2d686572652f6d61737465722f73746f72652d6173736574732f646f732d65717569732d4d44482e6a7067)`

![](https://camo.githubusercontent.com/abf9d87ce112444bca1ddfffaa2063f02a2c26d0/68747470733a2f2f7261772e6769746875622e636f6d2f6164616d2d702f6d61726b646f776e2d686572652f6d61737465722f73746f72652d6173736574732f646f732d65717569732d4d44482e6a7067)



### 列表

#### 有序列表

看起来并不明显

```markdown
1. PHP是最好的语言？
2. PHP是最好的语言！
```

1. PHP是最好的语言？

2. PHP是最好的语言！

   

#### 无序列表

可以使用`-`、`*`、`+`开头接空格，但在多级列表中最好使用`-`

```markdown
- PHP是最好的语言？
* PHP是最好的语言。
+ PHP是最好的语言！
	- 毫无疑问
		- 众所周知
```

- PHP是最好的语言？
* PHP是最好的语言。
+ PHP是最好的语言！
  - 毫无疑问
    - 众所周知

#### 复选框列表

```markdown
- [x] 大一
- [x] 大二
- [ ] 大三
- [ ] 大四
```

- [x] 大一
- [x] 大二
- [ ] 大三
- [ ] 大四

Hexo 默认的渲染引擎 Marked 不支持 TODO list，可以更换为 markdown-it，之后实现的效果如 Typora 预览

![todo-list](Markdown-语法详解\todo-list.png)



### 引用与高亮

#### 引用

使用`>`实现引用，多个`>`实现引用层级

```markdown
> PHP
>> 是
>>> 最好的
>>>> 语言
```

> PHP
> > 是
> > > 最好的
> > >
> > > > 语言

一般用在引用原文内容中

> 一语未了，只听后院中有人笑声，说：“我来迟了，不曾迎接远客！”黛玉纳罕道：“这些人个个皆敛声屏气，恭肃严整如此，这来者系谁，这样放诞无礼？”心下想时，只见一群媳妇丫鬟围拥着一个人从后房门进来。这个人打扮与众姑娘不同，彩绣辉煌，恍若神妃仙子：头上戴着金丝八宝攒珠髻，绾着朝阳五凤挂珠钗；项上戴着赤金盘螭璎珞圈，裙边系着豆绿宫绦，双衡比目玫瑰佩；身上穿着缕金百蝶穿花大红洋缎窄褃袄，外罩五彩刻丝石青银鼠褂；下着翡翠撒花洋绉裙。一双丹凤三角眼，两弯柳叶吊梢眉，身量苗条，体格风骚，粉面含春威不露，丹唇未起笑先闻。黛玉连忙起身接见。



#### 高亮

使用单个反引号实现单行文本高亮，三个反引号实现代码块高亮。可以在第一个```后添加语言名称实现不同的语法高亮

```markdown
PHP是`最好的`语言

​```Kotlin
fun main(args: Array<String>) {
    print("hello")
}
​```
```

PHP是`最好的`语言

```Kotlin
fun main(args: Array<String>) {
    print("hello")
}
```

## 高级进阶

在不同的 Markdown 版本中实现可能不同

### 表格

```markdown
| 序号 | 列名1 | 列名2 |
| - | - | - |
| 0 | 一一  | 一二  |
| 1 | 二一  | 二二  |
```

| 序号 | 列名1 | 列名2 |
| - | - | - |
| 0 | 一一  | 一二  |
| 1 | 二一  | 二二  |

在分隔行（第二行）中的`-`右边添加`:`，表格内容实现右对齐效果，两边都加则为居中对齐，默认为左对齐

```markdown
| 序号 | 列名1 | 列名2 |
| :-: | :-: | :-: |
| 0 | 一一  | 一二  |
```

| 序号 | 列名1 | 列名2 |
| :--: | :---: | :---: |
|  0   | 一一  | 一二  |



### 流程图

分为两部分，第一部分定义元素，第二部分定义元素走向。定义元素语法为`tag=>type: content:>url`，其中`tag`为元素名称，`type`为元素类型，有以下6种

| type        | 含义       |
| ----------- | ---------- |
| start       | 开始       |
| end         | 结束       |
| operation   | 操作       |
| subroutine  | 子程序     |
| condition   | 条件       |
| inputoutput | 输入或输出 |

`content`为在流程图方框中显示的内容

```markdown
st=>start: 开始:>https://www.markdown-syntax.com
io=>inputoutput: 输入或输出
op=>operation: 操作
cond=>condition: Yes or No?
sub=>subroutine: 子程序
e=>end: 结束

st->io->op->cond
cond(yes)->e
cond(no)->sub->io
```



```flow
st=>start: 开始:>https://www.markdown-syntax.com
io=>inputoutput: 输入或输出
op=>operation: 操作
cond=>condition: Yes or No?
sub=>subroutine: 子程序
e=>end: 结束

st->io->op->cond
cond(yes)->e
cond(no)->sub->io
```



Hexo 原生并不支持流程图，需要安装[hexo-filter-flowchart](
https://github.com/bubkoo/hexo-filter-flowchart)

### 时序图

`title`为时序图标题，`participant`定义时序图对象，`note`定义时序图中的说明，有三种方位控制

- left of, 表示说明位于当前对象的左侧
- right of, 表示说明位于当前对象的右侧
- over, 表示说明覆盖在当前对象（们）上

不同对象之间使用箭头控制指向

- ->：实线实箭头
- -->：虚线实箭头
- ->>：实线虚箭头
- -->>：虚线虚箭头

```markdown
title: 时序图标题
participant 大一
participant 大二
participant 大三

note left of 大一: 大一好好学习
note over 大二: 大二课程很多
note right of 大三: 大三面临毕业

大一->大一:大一留级
大一->大二:大一迟早要到大二
大二-->大三:大二不一定能升大三
大二->>大三:大二不一定能升大三
大三-->>大一:大三也可能回炉重造
```



```sequence
title: 大学生活
participant 大一
participant 大二
participant 大三

note left of 大一: 大一好好学习
note over 大二: 大二课程很多
note right of 大三: 大三面临毕业

大一->大一:大一惨遭留级
大一->大二:大一迟早要到大二
大二-->大三:大二不一定能升大三
大二->>大三:大二不一定能升大三
大三-->>大一:大三也可能回炉重造
```



Hexo 默认同样不支持时序图，使用`[hexo-filter-sequence](https://github.com/bubkoo/hexo-filter-sequence)。具体的做法参考 [为 Hexo 增加时序图解析功能](http://wewelove.github.io/fcoder/2017/09/06/markdown-sequence/)