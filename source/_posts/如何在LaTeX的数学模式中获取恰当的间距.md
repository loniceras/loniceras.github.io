---
title: 如何在 LaTeX 的数学模式中获取恰当的间距?
date: 2020-07-17 14:05:20
tags: [排版,LaTeX]
categories: [排版]
mathjax: false
---
# 介绍
对于数学模式中间距的调整可以视作是对公式的精细调整的一部分, 这需要敏锐的观察和一定的审美. 了解 LaTeX 的数学模式中自动生成间距的机制对于调整公式内部间距是十分有益的, 而本文主要对此方面进行介绍, 并提出一些有关间距精细调整的建议, 可视作对[该材料](https://zhuanlan.zhihu.com/p/19683504)的补充. 本文原发于[知乎](https://zhuanlan.zhihu.com/p/159488344).
<!--more-->

# 原理
我们知道, 在数学模式中所有用 space 键直接输入的空格都会被忽略. 如果希望插入空格, 则需要在空格前插入反斜杠号, 如 `$a\ b$`, 可强制在 a 与 b 中插入空格. 通常我们无需进行这一操作, 更多依赖于 LaTeX 对数学符号作用的识别来自动插入适当的间距. 而当 LaTeX 由于各种原因无法正确生成恰当的间距或者定义新的数学符号的时候, 我们就应该进行适当的操作使恰当的间距被生成.

如下列表所示, 在 LaTeX 中的参与生成间距的符号分为 8 类, 括号内的指令为数学符号的类型声明方式.
- 普通符 (Ord, `\mathord`) : 如 `0 a A \varGamma \gamma \aleph` 等, 包括上下划线, 重音以及根式.
- 大型运算符 (Op, `\mathop`) : 如 `\int \sum \coprod \bigcup \bigvee \max` 等.
- 二元运算符 (Bin, `\mathbin`) : 如 `+ \cdot \cup \cap \sqcup \vee \wedge` 等.
- 关系符 (Rel, `\mathrel`) : 如 `= < \leq \in \subseteq \prec \sim \rightarrow \mid` 等.
- 开始符 (Open, `\mathopen`) : 如 `( [ \{ \lceil \lfloor \langle \lvert` 等.
- 结束符 (Close, `\mathclose`) : 如 `) ] \} \rceil \rfloor \rangle \rvert` 等.
- 标点符 (Punct, `\mathpunct`) : 如 `, . ; \cdot \ldot \ddot` 等 .
- 内部符 (Inner, `\mathinner`) : 视作整体公式的子公式, 包括分式以及由 `\left` 和 `\right` 命令生成的公式片段.

我们在下表中给出 LaTeX 中自动生成的空白宽度. 表的行标题为左侧符号, 列标题为右侧符号, 两者交汇处就是所插入间距的宽度. 其中, 0 表示不插入间距, 1 表示 thin space (即 `\,` ), 2 表示 medium space (即 `\:` ), 3 表示 thick space (即 `\;` ), * 表示不可能出现该情形 (此时通常会将 Bin 解释为 Ord, 细节参考 The TeXbook 的附录 G). 处于圆括号中的数字表示如果该情况使用 `\scriptstyle` (包括上下标, `\textstyle` 下的行内分式) 或 `\scriptscriptstyle` (包括上下标的上下标, `\textstyle` 下的行内分式的上下标) 样式则不插入间距. 事实上, 命令 `\mathord{}` 是不必要的, 我们使用 `{}` 可以更灵活地实现想要的效果, TeX 将 `{}` 及其内部作为整体视作 Ord, 然后根据前后紧邻的符号类型自动插入相应的间距, 而不会对 `{}` 内部的符号类型产生影响.

|  | Ord | Op | Bin | Rel | Open | Close | Punct | Inner|
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| **Ord** | 0 | 1 | (2) | (3) | 0 | 0 | 0 | (1) |
| **Op** | 1 | 1 | * | (3) | 0 | 0 | 0 | (1) |
| **Bin** | (2) | (2) | * | * | (2) | * | * | (2) |
| **Rel** | (3) | (3) | * | 0 | (3) | 0 | 0 | (3) |
| **Open** | 0 | 0 | * | 0 | 0 | 0 | 0 | 0 |
| **Close** | 0 | 1 | (2) | (3) | 0 | 0 | 0 | (1) |
| **Punct** | (1) | (1) | * | (1) | (1) | (1) | (1) | (1) |
| **Inner** | (1) | 1 | (2) | (3) | (1) | 0 | (1) | (1) |

我们用具体的例子来进一步解释这个机制. 由于机制中关系符接二元运算符的情形并不存在, 可以观察到例子中等号 (Rel) 后的减号 (Bin) 被解释为了负号 (Ord). LaTeX 的根据类型生成空格的特性使我们在大部分情况下不需要操心过多的间距细节, 但有时候我们不得不亲力亲为才能获得较好的效果.
```
a        -        b        =        -      \max     \{       c        ,         d       \}
Ord  \:  Bin  \:  Ord  \;  Rel  \;  Ord  \,  Op  0  Open  0  Ord  0  Punct  \,  Ord  0  Close
```

# 应用
在简单了解了机制以后, 我们来看一些建议手动调整的情形.
- 定义新符号时应声明类型. 声明的方式可参考其作为前缀记法/中缀记法/后缀记法的类别或结合语义来确定. 这些声明不会设置字体, 因此有需要时应加上相应的字体设置命令.
- 修正错误的类型识别. 产生错误识别的原因前文已解释, 此处举例说明. 例一: 在表示含有绝对值的等式时, 加号 + 和减号 - 识别为了 Bin, 而非 Ord (可选用 `amsmath` 提供的 `\lvert \rvert` 避免该问题); 对于范数的双竖线 `\|` 也会产生类似问题 (可选用 `amsmath` 提供的 `\lVert \rVert` 避免该问题). 例二: 在表示等价关系间的包含关系时, 等价关系 `\sim` 和 `\approx` 识别为了 Rel, 而非 Ord; 对于其他关系间的集论表达也可能产生类似问题.
```
$ |   -   x   |   =   |   +   x   | $
 Ord Bin Ord Ord Rel Ord Bin Ord Ord
$ |  {-   x}  |   =   |   {+  x}  | $
 Ord Ord Ord Ord Rel Ord Ord Ord Ord
$ \sim \subseteq \approx $
   Rel     Rel     Rel   
$ {\sim} \subseteq {\approx} $   
   Ord      Rel      Ord
```
- 结合语义修改字符的默认类型. 有时, 我们所使用的符号的由语义确定的类型与 TeX 中的默认类型是不一样的. 例如, `\vee` 作为 Bin 时可以表示形式系统中的析取, 也可以表示格中元素的上确界. 而有时人们希望对析取记号的应用范围进行延伸, 用 `\vee` 的记号表示元语言中的"或", 此时 Bin 类型对于 `\vee` 来说就不合适了 (否则 `\in` 等 Rel 符号产生的间距都比表示"或"的 `\vee` 产生的间距大), 最少应该是 Rel 类型. 类似地, 一些集论工作者喜欢使用 `\upharpoonright` 来表示二元关系上的限制, 而在 TeX 中 `\upharpoonright` 是 Rel 类型, 应当仿照其他的二元关系运算符号的类型将其修改为 Bin 类型.
```
$ \{\,x\;|\;x^2 \in A \vee x \geq c\,\} $
$ \{\,x\;|\;x^2 \in A \mathrel{\vee} x \geq c\,\} $
$ S \subseteq R \upharpoonright A $
$ S \subseteq R \mathbin{\upharpoonright} A $
```
- 大型运算符的声明和使用. 使用 `\mathop` 声明大型运算符时, 如不另加指定则在 `\displaystyle` 下显示 `\limits` 式的上下标. 另外, `amsmath` 提供的 `\operatorname` 和 `\operatorname*` 也可用于声明大型运算符, 前者在 `\displaystyle` 下显示 `\nolimits` 式的上下标, 后者在 `\displaystyle` 下显示 `\limits` 式的上下标. 这两个命令不仅要求了 `\mathop`, 而且对字体进行了设置 (如不另加指定则为罗马体), 并修复了单字符作大型运算符时基线不准的问题 (见下例, 参考[此处](https://tex.stackexchange.com/questions/41261/mathop-shifts-the-baseline-declaremathoperator-doesnt/41267#41267)). 至于在实践中, 选择 `\mathop` 或 `\operatorname` 则仁者见仁智者见智.
```
$ \operatorname{P} $ is different from $ \mathop{\mathrm{P}} $
```
- 定界符缩放控制命令对间距的影响. `\big \bigl \bigr \bigm` 均可以得到同样大小的定界符, 但会影响定界符的类型, `\big` 得到的是 Ord 类型, `\bigl` 得到的是 Open 类型, `\bigr` 得到的是 Close 类型, `\bigm` 得到的是 Rel 类型, 其余定界符缩放控制命令如 `\bigg \Big \Bigg` 同理. 而 `\left` 和 `\right` 确定的定界符则会得到一个 Inner 类型, 因此有时可能产生比 `\bigl \bigr` 控制的定界符更大的空格, 如下例所示. 所以除了通常说的 `\left \right` 不易灵活控制定界符高度外, 还可能对定界符周围的间距产生影响 (思考: 如何消除 `\left \right` 对间距的影响?).
```
$ a\bigl(b + c\bigr) $
$ a\left(b + c\right) $
```
- 将文本插入数学模式. 这一话题与先前讨论的内容略有区别. 有一种回避问题的观点认为数学模式中不应该出现文本, 尽可能用符号变元替代文本并在非数学模式内对符号变元进行解释. 虽然这一理念值得提倡, 但有时却会导致描述变得繁琐复杂. 若能找到可生成恰当的间距的插入方式并适度使用, 则能得到平衡. 通常来说有两种较为可行的方式, 均采用了 `amsmath` 提供的 `\text` 指令, 两种方式在大部分情形下可获得相同的输出, 如下例所示. 前者将整个有文字连续出现的完整区域都放入 `\text{}` 中并在 `\text{}` 内再次使用数学模式, 后者则是使用 `\<space>` 来强制插入空格, 当 `\text{}` 中内容是英文时后者也可省略为 `\text{<space> ... <space>}` 的形式而不需要使用 `\<space>`. 应当注意, 如果是 `\text{}` 内是中文, 则 `\<space>` 是不能省略为 `\text{<space> ... <space>}` 的形式, 否则生成的间隔会不正确 (try it ?).  另外, 当 `\text{}` 处于 `\scriptstyle` 或 `\scriptscriptstyle` 下, 这两种方法会生成不同长度的间隔, 后者的间隔长度偏长 (try it ?).
```
$ \sup\{\,\operatorname{card}(B)\;|\;\text{$ B $ is a chain of $ A $}\,\} $
$ \sup\{\,\operatorname{card}(B)\;|\;B\ \text{is a chain of}\ A\,\} $
```
- 精细调整的限度. Knuth 本人也在 The TeXbook 中给出了一些关于精细调整的使用建议, 我们简化摘选了一部分列在下面, 可以发现精细调整无处不在. 如何在专注于内容和精细化排版细节间进行取舍 (或是选择花费更长时间两个都要) 也是每一位 LaTeX 使用者需要思考的.
```
$ 13!26! $     $ x^2/2 $     $ n/\log n $     $ \sqrt{2} x $      $ [0,1) $     $ \ln n(\ln\ln n)^2 $     $ \sin^2 x $\\
$ 13!\,26! $   $ x^2\!/2 $   $ n/\!\log n $   $ \sqrt{2} \, x $   $ [\,0,1) $   $ \ln n\,(\ln\ln n)^2 $   $ \sin^2\! x $
```

# 总结
在 LaTeX 的数学模式下的间距控制是一个复杂的问题, 远不是本文可以全部阐释清楚的. 本文中提及的一些方法和建议仅供参考, 如有不正确之处还请指正. 除了间距的调整, LaTeX 的数学模式下还有大量的可以精细调整之处, 诸如对分式高度长度, 根式高度斜率, 角标位置等的调整 (可能以后会写...