---
layout: post
title: "css玄学之一二"
description: "《css世界》读后收获"
category: tech
tags: [css]
---
{% include JB/setup %}

css在我心中一直是玄学，因此一直希望遇见一本可以系统的介绍css规则。所以看到张鑫旭出了《css世界》，好不犹豫的买来读了。虽然这本书并没有如愿让我对css有系统的解惑，但是很多方面有了进一步认识。另外，张鑫旭大大真的是匠心，非常钦佩其十年如一日的深钻css世界，特别是他非常细致深入的不间断更新的博客。（个人不成熟的小感受是书中虽然有在按章节介绍内容，但是经常从a到b，导致读者主线丢失，感觉还是在一个个堆知识点，难以连成片。另外书中出现不少为了理解方便而取的名词，如“空白幽灵节点”，我还是希望书中的专业名词都跟标准里统一，不要自定义，否则出了这本书，我们可能就对不上了）。

非常认同，css玄在最终表现是多个属性共同作用的结果，这多个属性常上达几十个，所以单从某个属性角度很多时候无法理解其表现

# 内联

相比于块级元素，内联元素出现的不少，但是属性表现经常不受重视，譬如我就发现很多现象之前没有认知到。

先来介绍一波之前不被我理解的现象

## 诡异的案例系列

### 1.空元素高度不为0

[https://codepen.io/anon/pen/ZxRWjd?editors=1100](https://codepen.io/anon/pen/ZxRWjd?editors=1100)

![image](https://s10.mogucdn.com/mlcdn/c45406/180331_43hbfih4i05j7725f4be2c833f8ck_948x1130.png)


### 块状元素line-height设置的高度不是最终高度

[https://codepen.io/anon/pen/NYzrmq](https://codepen.io/anon/pen/NYzrmq)

```
.box { line-height: 32px; }
.box > span { font-size: 24px; }
<div class="box">
　 <span>文字</span>
</div>
```

解释：

其中有一个很关键的点，那就是24px的font-size大小是设置在`<span>`元素上的，这就导致了外部`<div>`元素的字体大小和`<span>`元素有较大出入。

`<span>`标签前面实际上有一个看不见的类似字符的“幽灵空白节点”。看不见的东西不利于理解，因此我们不妨使用一个看得见的字符x占位，同时“文字”后面也添加一个x，便于看出基线位置，于是就有如下HTML：

```
<div class="box">
　 x<span>文字x</span>
</div>
```

此时，我们可以明显看到两处大小完全不同的文字。一处是字母x构成了一个“匿名内联盒子”，另一处是“文字 x”所在的<span>元素，构成了一个“内联盒子”。由于都受line- height:32px影响，因此，这两个“内联盒子”的高度都是32px。下面关键的来了，对字符而言，font-size越大字符的基线位置越往下(需要将line-height的行距上下平分)，因为文字默认全部都是基线对齐，所以当字号大小不一样的两个文字在一起的时候，彼此就会发生上下位移，如果位移距离足够大，就会超过行高的限制，而导致出现意料之外的高度。

![image](https://s10.mogucdn.com/mlcdn/c45406/180401_088eehb5aal370gcb7287f37cdg00_928x262.png)

解决方案：给box设置和`span`一样的`font-size`

### 2.vertical-align:middle 不起作用

```
.box {
　 height: 128px;
}
.box > img {
　 height: 96px;
　 vertical-align: middle;
 }
<div class="box">
　 <img src="1.jpg">
</div>
```

此时图片顶着.box元素的上边缘显示，根本没垂直居中，完全没起作用！

解释： 

这种情况看上去是vertical-align:middle没起作用，实际上，vertical-align是在努力地渲染的，只是行框盒子前面的“幽灵空白节点”高度太小，如果我们通过设置一个足够大的行高让“幽灵空白节点”高度足够，就会看到vertical-align:middle起作用了。可以给`.box`设置个`line-height: 128px`

### 3.margin值无效

```
<div class="box">
　 <img src="mm1.jpg">
</div>
.box > img {
　 height: 96px;
　 margin-top: -200px;
} 
```

-200px远远超过图片的高度，图片应该完全跑到容器的外面，但是，图片依然有部分在.box元素中，而且就算margin-top设置成-99999px，图片也不会继续往上移动，完全失效。

解释： 

的前面有个“幽灵空白节点”，而在CSS世界中，非主动触发位移的内联元素是不可能跑到计算容器外面的，导致图片的位置被“幽灵空白节点”的vertical-align:baseline给限死了。我们不妨把看不见的“幽灵空白节点”使用字符x代替。

![image](https://s10.mogucdn.com/mlcdn/c45406/180401_6l7235b9il0ki532fi1agf36g216a_906x350.png)

因为字符x下边缘和图片下边缘对齐，字符x非主动定位，不可能跑到容器外面，所以图片就被限死在此问题，margin-top失效。

### 4.图片底部留白

[http:// demo.cssworld.cn/5/3-5.php](http://demo.cssworld.cn/5/3-5.php)

解释：

常见的图片底部留白问题也是一样，间隙产生的三大元凶就是“幽灵空白节点”、line-height和vertical-align属性。

解决：

- 图片块状化。可以一口气干掉“幽灵空白节点”、line-height和vertical-align。

- 容器line-height足够小。只要半行间距小到字母x的下边缘位置或者再往上，自然就没有了撑开底部间隙高度空间了。比方说，容器设置line-height:0。

- 容器font-size足够小。此方法要想生效，需要容器的line-height属性值和当前font-size相关，如line-height:1.5或者line-height:150%之类；否则只会让下面的间隙变得更大，因为基线位置因字符x变小而往上升了。

- 图片设置其他vertical-align属性值。间隙的产生原因之一就是基线对齐，所以我们设置vertical-align的值为top、middle、bottom中的任意一个都是可以的。

------

再来介绍下内联元素相关的概念和知识

## 外部盒子和容器盒子

“每个元素都两个盒子，外在盒子和容器盒子。外在盒子负责元素是可以一行显示，还是只能换行显示；容器盒子负责宽高、内容呈现什么的”

按照`display`的属性值不同，值为`block`的元素的盒子实际由外在的“块级盒子”和内在的“块级容器盒子”组成，值为`inline-block`的元素则由外在的“内联盒子”和内在的“块级容器盒子”组成，值为`inline`的元素则内外均是“内联盒子”。

“遵循这种理解，`display:block`应该脑补成`display:block-block`，`display:table`应该脑补成`display:block-table`”


“内联元素”指“外在盒子”为`inline`的元素。

## 内联元素影响区域尺寸的一些概念

- 内容区域（content area）。内容区域指一种围绕文字看不见的盒子，其大小仅受字符本身特性控制，本质上是一个字符盒子（character box）；但是有些元素，如图片这样的替换元素，其内容显然不是文字，不存在字符盒子之类的，因此，对于这些元素，内容区域可以看成元素自身。我们可以把文本选中的背景色区域作为内容区域

- 内联盒子（inline box）。“内联盒子”不会让内容成块显示，而是排成一行，这里的“内联盒子”实际指的就是元素的“外在盒子”，用来决定元素是内联还是块级。该盒子又可以细分为“内联盒子”和“匿名内联盒子”两类：　如果外部含内联标签（`<span>`、`<a>`和`<em>`等），则属于“内联盒子”；如果是个光秃秃的文字，则属于“匿名内联盒子”。需要注意的是，并不是所有光秃秃的文字都是“匿名内联盒子”，其还有可能是“匿名块级盒子”，关键要看前后的标签是内联还是块级。

- 行框盒子（line box）。例如：

    < p> 这是一行普通的文字，这里有个 < em>em< /em> 标签。 < /p>

  每一行就是一个“行框盒子”，每个“行框盒子”又是由一个一个“内联盒子”组成的。

- 包含块（containing block）。例如：

    < p>这是一行普通的文字，这里有个 < em>em< /em> 标签。< /p>

  `<p>`标签就是一个“包含盒子”，此盒子由一行一行的“行框盒子”组成。

## strut - 幽灵空白节点

在HTML5文档声明中，内联元素的所有解析和渲染表现就如同每个行框盒子的前面有一个“空白节点”一样。这个“空白节点”永远透明，不占据任何宽度，看不见也无法通过脚本获取，就好像幽灵一样，但又确确实实地存在，表现如同文本节点一样，因此，称之为“幽灵空白节点”

“幽灵空白节点”实际上也是一个盒子，不过是个假想盒，名叫“strut”，是一个存在于每个“行框盒子”前面，同时具有该元素的字体和行高属性的0宽度的内联盒。

## 基线

字母x的下边缘（线）就是我们的基线
`line-height`行高的定义就是两基线的间距，`vertical-align`的默认值就是基线
`x-height`，指的是字母x的高度，术语描述就是基线和等分线（meanline）（也称作中线，midline）之间的距离

![image](https://s10.mogucdn.com/mlcdn/c45406/180331_01f9g86a5719569dg53hd0gk1a013_906x358.png)

`vertical-align:middle`。这里的middle是中间的意思。注意，跟上面的median（中线）不是一个意思。在CSS世界中，middle指的是基线往上1/2 x-height高度。我们可以近似理解为字母x交叉点那个位置。

## line-height

对于块级元素，它指定元素中线框的最小高度。在未替换的内联元素中，它指定用于计算线框高度的高度。对于替代行内元素，如 button 或其他 input 元素，line-height 没有影响（原文未提到，对于部分替代元素，line-height 依然可以影响元素的样式布局）

默认值normal, 取决于用户代理。桌面浏览器（包括火狐浏览器）使用默认值，约为1.2，这取决于元素的 font-family。

如果使用数值作为line-height的属性值，那么所有的子元素继承的都是这个值；但是，如果使用百分比值或者长度值作为属性值，那么所有的子元素继承的是最终的计算值

对于非替换元素的纯内联元素，其可视高度完全由line-height决定。注意这里的措辞——“完全”，什么padding、border属性对可视高度是没有任何影响的，这也是我们平常口中的“盒模型”约定俗成说的是块级元素的原因。

内联元素的高度由固定高度和不固定高度组成，这个不固定的部分就是这里的“行距”。换句话说，line-height之所以起作用，就是通过改变“行距”来实现的。

行距 = 行高−em-box。转换成CSS语言就是：行距 = line-height - font-size。em是一个相对font-size大小的CSS单位，因此1em等用于当前一个font-size大小

内容区域可以近似理解为Firefox/IE浏览器下文本选中带背景色的区域

- `line-height`不能影响内联替换元素，有时候看到的表象的影响其实是影响了行框盒子里的“幽灵空白节点”
- 替换元素和内联非替换元素在一起时，由于同属内联元素，因此，会共同形成一个“行框盒子”，line-height在这个混合元素的“行框盒子”中扮演的角色是决定这个行盒的最小高度。例子：明明文字设置了line-height为20px，但是，如果文字后面有小图标，最后“行框盒子”高度却是21px或是22px。这种现象背后最大的黑手其实是vertical-align属性
- 对于块级元素，line-height对其本身是没有任何作用的，我们平时改变line-height，块级元素的高度跟着变化实际上是通过改变块级元素里面内联级别元素占据的高度实现的。

**`display:inline`的内联元素的`line-height`对最终内容盒子的高度没有显式影响，因为行高影响的是行框，并不是内联元素的高度，高度由font-size和font-family影响。而`display:inline-block`有，因为其内部盒子是块状盒子，块状盒子的内容高度由line-height、padding、border等共同决定**

### 使用

- 单行文字垂直居中：只需要`line-height`属性就可以，无需同时设置height并与line-height相等
- 多行文字
    - 多行文字使用一个标签包裹，然后设置display为inline-block。好处在于既能重置外部的line-height为正常的大小，又能保持内联元素特性，从而可以设置vertical-align属性，以及产生一个非常关键的“行框盒子，“每个“行框盒子”都会附带的一个产物——“幽灵空白节点”。“有了这个“幽灵空白节点”，我们的`line- height`就有了作用的对象
    - 因为内联元素默认都是基线对齐的，所以我们通过对元素设置vertical- align:middle来调整多行文本的垂直位置，从而实现我们想要的“垂直居中”效果。如果是要借助line-height实现图片垂直居中效果，也是类似的原理和做法。 
    eg: 

    ```
    .box {
    　 line-height: 120px;
    　 background-color: #f0f3f9;
    }
    .content {
    　 display: inline-block;
    　 line-height: 20px;
    　 margin: 0 20px;
    　 vertical-align: middle;
    }
    <div class="box">
    　 <div class="content">基于行高实现的...</div>
    </div>
    ```

## vertical-align

vertical-align属性的默认值baseline在文本之类的内联元素那里就是字符x的下边缘，对于替换元素则是替换元素的下边缘。但是，如果是inline-block元素，则规则要复杂了：一个inline-block元素，如果里面没有内联元素，或者overflow不是visible，则该元素的基线就是其margin底边缘；否则其基线就是元素里面最后一行内联元素的基线。

单行文本最终高度并不是由line-height决定。还受font-size和vertical-align影响

```
.box { line-height: 32px; }
.box > span { font-size: 24px; }
<div class="box">
　 <span>文字</span>
</div>
```

[http://demo.cssworld.cn/5/3-1.php](http://demo.cssworld.cn/5/3-1.php)

`.box`最终高度并不是32px.

### 值

top/bottom和baseline/middle却是完全不同的两个帮派，前者对齐看边缘看行框盒子，而后者是和字符x打交道

### 数字值

数字值根据计算值的不同，相对于基线往上或往下偏移，到底是往上还是往下取决于vertical- align的计算值是正值还是负值，如果是负值，往下偏移，如果是正值，往上偏移。

由于vertical-align的默认值是baseline，即基线对齐，而基线的定义是字母x的下边缘。因此，内联元素默认都是沿着字母x的下边缘对齐的。对于图片等替换元素，往往使用元素本身的下边缘作为基线

[http://demo.cssworld.cn/5/3-2.php](http://demo.cssworld.cn/5/3-2.php)

### 百分比值

凡是百分比值，均是需要一个相对计算的值，例如，margin和padding是相对于宽度计算的，line-height是相对于font-size计算的，而这里的vertical- align属性的百分比值则是相对于line-height的计算值计算的。

### 文本值

- vertical-align:text-top：盒子的顶部和父级内容区域的顶部对齐。
- vertical-align:text-bottom：盒子的底部和父级内容区域的底部对齐。
其中，理解的难点在于“父级内容区域”，这是个什么东西呢？

其可以看成是Firefox/IE浏览器文本选中的背景区域，或者默认状态下的内联文本的背景色区域。而所谓“父级内容区域”指的就是在父级元素当前font-size和font-family下应有的内容区域大小。

因此，这个定义又可以理解为（以text-top举例）：假设元素后面有一个和父元素font- size、font-family一模一样的文字内容，则vertical-align:text-top表示元素和这个文字的内容区域的上边缘对齐。


### 作用域

vertical-align起作用是有前提条件的，这个前提条件就是：只能应用于内联元素以及display值为table-cell的元素。

浮动和绝对定位会让元素块状化，此时`vertical-align`无效


### 复杂的案例

[http://demo.cssworld.cn/5/3-6.php](http://demo.cssworld.cn/5/3-6.php)


### vertical-align: middle的近似垂直居中

vertial-align:middle可以让内联元素的真正意义上的垂直中心位置和字符x的交叉点对齐。

基本上所有的字体中，字符x的位置都是偏下一点儿的，font-size越大偏移越明显，这才导致默认状态下的vertial-align:middle实现的都是“近似垂直居中”。

[http://demo.cssworld.cn/5/3-8.php](http://demo.cssworld.cn/5/3-8.php)

line-height的高度大于内容高度，空白行距需要均匀分布在em-box的上下方，但是em-box和文字内容区域并不重合，无法感知(高度是m的高度)

如果想要实现真正意义上的垂直居中对齐，只要想办法让字符x的中心位置就是容器的垂直中心位置即可，通常的做法是设置font-size:0，整个字符x缩小成了一个看不见的点，根据line-height的半行间距上下等分规则，这个点就正好是整个容器的垂直中心位置，这样就可以实现真正意义上的垂直居中对齐了。

---

# 高度的决定因素

高度对于我来说也是很魔幻的，譬如`line-height`能决定块状元素的高度，却完全不能左右内联元素的高度。`font-size`也是只对内联元素高度有影响，却不是唯一的决定因素。

[https://codepen.io/anon/pen/KoeNZr](https://codepen.io/anon/pen/KoeNZr)

- 块状元素：line-height、height、padding、border共同作用
- 内联元素：内联盒子共同作用后的行框盒子结果，每个内联盒子，如果是纯文本，又是font-size和font-family作用的结果

看到一篇文章从字体设计来分析字体、字号和行高的关系的：[https://zhuanlan.zhihu.com/p/27381252](https://zhuanlan.zhihu.com/p/27381252)

---

# 流

## float元素特性

float元素破坏了正常流

- 包裹性：浮动元素内部宽度小于父元素时，最终宽度表现为内部宽度。如果内部内容能铺满父元素时，表现宽度为父元素宽度。改变默认的宽度显示类型就可以，添加`white-space:nowrap`，可以让宽度表现从“包裹性”变成“最大可用宽度
- 块状化并格式化上下文；
- 破坏文档流；
- 没有任何margin合并；

行框盒子和浮动元素的不可重叠性”，也就是“行框盒子如果和浮动元素的垂直高度有重叠，则行框盒子在正常定位状态下只会跟随浮动元素，而不会发生重叠”。float元素的“浮动参考”是“行框盒子”

## position:absolute

元素一旦position属性值为absolute或fixed，其display计算值就是block或者table

## clear

clear属性：元素盒子的边不能和前面的浮动元素相邻。

clear属性只有块级元素才有效的，而::after等伪元素默认都是内联水平，这就是借助伪元素清除浮动影响时需要设置display属性值的原因。

```
.clear:after {
　 content: '';
　 display: table;　 // 也可以是'block'，或者是'list-item'
　 clear: both;
}
```

（1）如果clear:both元素前面的元素就是float元素，则margin-top负值即使设成-9999px，也不见任何效果。
（2）clear:both后面的元素依旧可能会发生文字环绕的现象

## 块级格式化上下文BFC

如果一个元素具有BFC，内部子元素不会影响外部的元素，也不会受外部元素影响。所以，BFC元素是不可能发生margin重叠的，因为margin重叠是会影响外面的元素的；BFC元素也可以用来清除浮动的影响，因为如果不清除，子元素浮动则父元素高度塌陷，必然会影响后面元素布局和定位，这显然有违BFC元素的子元素不会影响外部元素的设定。

那什么时候会触发BFC呢？常见的情况如下：

- 根元素；
- float的值不为none；现在一些style lint规则的提示相当智能，`display:inline-block; float: right`会被提示`float`不为none时，`dlisplay`表现为`block`
- overflow的值为auto、scroll或hidden；
- display的值为table-cell、table-caption和inline-block中的任何一个；
- position的值不为relative和static。

## 包含块

（1）根元素（很多场景下可以看成是<html>）被称为“初始包含块”，其尺寸等同于浏览器可视窗口的大小。

（2）对于其他元素，如果该元素的position是relative或者static，则“包含块”由其最近的块容器祖先盒的content box边界形成。

（3）如果元素position:fixed，则“包含块”是“初始包含块”。

（4）如果元素position:absolute，则“包含块”由最近的position不为static的祖先元素建立，具体方式如下。

如果该祖先元素是纯inline元素，则规则略复杂：

假设给内联元素的前后各生成一个宽度为0的内联盒子（inline box），则这两个内联盒子的padding box外面的包围盒就是内联元素的“包含块”；
如果该内联元素被跨行分割了，那么“包含块”是未定义的，也就是CSS2.1规范并没有明确定义，浏览器自行发挥。
否则，“包含块”由该祖先的padding box边界形成。

对于绝对定位元素。height:100%是第一个具有定位属性值的祖先元素的高度，而height:inherit则是单纯的父元素的高度继承

---

# css世界的层叠规则

我经历过很多设置`z-index:99999`元素还是被挡住的悲剧，曾不能理解为啥`z-index`更大值的元素被更小值元素覆盖的现象。

[https://codepen.io/anon/pen/KoeNZr](https://codepen.io/anon/pen/KoeNZr)

z-index属性只有和定位元素（position不为static的元素）在一起的时候才有作用，flex盒子的子元素也可以设置z-index属性

- 层叠上下文，英文称作stacking context，是HTML中的一个三维的概念。如果一个元素含有层叠上下文，我们可以理解为这个元素在z轴上就“高人一等”。可以把层叠上下文理解为一种“层叠结界”，自成一个小世界。这个小世界中可能有其他的“层叠结界”，而自身也可能处于其他“层叠结界”中。

- 层叠水平，英文称作stacking level，决定了同一个层叠上下文中元素在z轴上的显示顺序

所有的元素都有层叠水平，包括层叠上下文元素，也包括普通元素。然而，对普通元素的层叠水平探讨只局限在当前层叠上下文元素中。为什么呢？因为不如此就没有意义。层叠上下文本身就是一个强力的“层叠结界”，而普通元素的层叠水平是无法突破这个结界和结界外的元素去较量层叠水平的。

## 层叠顺序

![image](https://s10.mogucdn.com/mlcdn/c45406/180401_7j8941ijfkk3agib33i245232h10h_936x746.png)

（1）位于最下面的background/border特指层叠上下文元素的边框和背景色。每一个层叠顺序规则仅适用于当前层叠上下文元素的小世界。

（2）inline水平盒子指的是包括inline/inline-block/inline-table元素的“层叠顺序”，它们都是同等级别的。

（3）单纯从层叠水平上看，实际上z-index:0和z-index:auto是可以看成是一样的。注意这里的措辞——“单纯从层叠水平上看”，实际上，两者在层叠上下文领域有着根本性的差异。

## 层叠规则

（1）谁大谁上：当具有明显的层叠水平标识的时候，如生效的z-index属性值，在同一个层叠上下文领域，层叠水平值大的那一个覆盖小的那一个。

（2）后来居上：当元素的层叠水平一致、层叠顺序相同的时候，在DOM流中处于后面的元素会覆盖前面的元素。

## 层叠特性

- 层叠上下文的层叠水平要比普通元素高。
- 层叠上下文可以阻断元素的混合模式（参见http://www.zhangxinxu.com/wordpress/?p=5155的这篇文章的第二部分说明），隔离mix-blend-mode元素的混合。
- 层叠上下文可以嵌套，内部层叠上下文及其所有子元素均受制于外部的“层叠上下文”。
- 每个层叠上下文和兄弟元素独立，也就是说，当进行层叠变化或渲染的时候，只需要考虑后代元素。
- 每个层叠上下文是自成体系的，当元素发生层叠的时候，整个元素被认为是在父层叠上下文的层叠顺序中。

## 层叠上下文的创建

- 页面根元素天生具有层叠上下文，称为根层叠上下文
- z-index值为数值的定位元素。对于position值为relative/absolute/fixed声明的定位元素，当其z-index值不是auto的时候，会创建层叠上下文。部分浏览器如chrome fixed时无需声明z-index就是层叠上下文
- css3属性
    - 元素为flex布局元素（父元素display:flex|inline-flex），同时z-index值不是auto。
    - 元素的opacity值不是1。
    - 元素的transform值不是none。
    - 元素mix-blend-mode值不是normal。
    - 元素的filter值不是none。
    - 元素的isolation值是isolate。
    - 元素的will-change属性值为上面2～6的任意一个（如will-change:opacity、will-change:transform等）。
    - 元素的-webkit-overflow-scrolling设为touch。

```
<div style="position:relative; z-index:auto;">
　<!-- 美女 -->
　<img src="1.jpg" style="position:absolute; z-index:2;">　
</div>
<div style="position:relative; z-index:auto;">
　<!-- 美景 -->
　<img src="2.jpg" style="position:relative; z-index:1;">　
</div>
```

结果是美女在美景上

```
<div style="position:relative; z-index:0;">
　<!-- 美女 -->
　<img src="1.jpg" style="position:absolute; z-index:2;">　
</div>
<div style="position:relative; z-index:0;">
　<!-- 美景 -->
　<img src="2.jpg" style="position:relative; z-index:1;">　
</div>
```

结果是美景在美女上。

[http://demo.cssworld.cn/7/5-1.php](http://demo.cssworld.cn/7/5-1.php)

我们有时候会发现z-index嵌套错乱，这时要看看是不是受父级的层叠上下文元素干扰了，可能就豁然开朗了

## 层叠上下文与层叠顺序

一旦普通元素具有了层叠上下文，其层叠顺序就会变高。那它的层叠顺序究竟在哪个位置、哪个级别呢？

这里需要分两种情况讨论：

（1）如果层叠上下文元素不依赖z-index数值，则其层叠顺序是z-index:auto，可看成z:index:0级别
（2）如果层叠上下文元素依赖z-index数值，则其层叠顺序由z-index值决定。

定位元素会层叠在普通元素的上面，其根本原因就是：元素一旦成为定位元素，其z-index就会自动生效，此时其z-index就是默认的auto，也就是0级别，根据上面的层叠顺序表，就会覆盖inline或block或float元素。而不支持z-index的层叠上下文元素天然是z-index:auto级别，也就意味着，层叠上下文元素和定位元素是一个层叠顺序的，于是当它们发生层叠的时候，遵循的是“后来居上”准则。

---

其他的一些笔记记录...

# 其他易被忽略的知识点

## 块级元素与内联元素

块级盒子就负责结构，内联盒子就负责内容

### 块级盒子和内联盒子

“块级元素的基本特征，也就是一个水平流上只能单独显示一个元素，多个块级元素则换行显示。”

正是由于“块级元素”具有换行特性，因此理论上它都可以配合clear属性来清除浮动带来的影响。例如：

```
.clear:after {
　 content: '';
   display: table;　// 也可以是block，或者是list-item
　 clear: both;
}
``` 
   
[http://demo.cssworld.cn/3/1-1.php](http://demo.cssworld.cn/3/1-1.php)


### 标记盒子

li元素默认会有个圆点在首部

“list-item元素会出现项目符号是因为生成了一个附加的盒子，学名“标记盒子”（marker box），专门用来放圆点、数字这些项目符号”

### 外部盒子和容器盒子

“每个元素都两个盒子，外在盒子和容器盒子。外在盒子负责元素是可以一行显示，还是只能换行显示；容器盒子负责宽高、内容呈现什么的”

按照`display`的属性值不同，值为`block`的元素的盒子实际由外在的“块级盒子”和内在的“块级容器盒子”组成，值为`inline-block`的元素则由外在的“内联盒子”和内在的“块级容器盒子”组成，值为`inline`的元素则内外均是“内联盒子”。

“遵循这种理解，`display:block`应该脑补成`display:block-block`，`display:table`应该脑补成`display:block-table`”

### 元素尺寸

width/height作用在容器盒子上。content box环绕着width和height给定的矩形。

#### 深藏不露的width:auto

（1）充分利用可用空间。比方说，`<div>`、`<p>`这些元素的宽度默认是100%于父级容器的。fill-available。

（2）收缩与包裹。典型代表就是浮动、绝对定位、inline-block元素或table元素，英文称为shrink-to-fit，直译为“收缩到合适”，“包裹性”。CSS3中的fit-content指的就是这种宽度表现。

（3）收缩到最小。这个最容易出现在table-layout为auto的表格中。当每一列空间都不够的时候，文字能断就断，这种行为在规范中被描述为“preferred minimum width”或者“minimum content width。后来还有了一个更加好听的名字min-content。[http://demo.cssworld.cn/3/2-1.php](http://demo.cssworld.cn/3/2-1.php)

（4）超出容器限制。除非有明确的width相关设置，否则上面3种情况尺寸都不会主动超过父级容器宽度的，但是存在一些特殊情况。例如，内容很长的连续的英文和数字，或者内联元素被设置了white-space:nowrap。max-content

### 外部元素流体特性

#### 正常流宽度

block容器的流特性，其尺寸表现就会铺满容器。

冗余的写法：
```
a {
　　display: block;
　　width: 100%;
}
```   
保持流动性：“无宽度，无图片，无浮动”

流动性，并不是看上去的宽度100%显示这么简单，而是一种margin/border/padding和content内容区域自动分配水平空间的机制。

[http://demo.cssworld.cn/3/2-3.php](http://demo.cssworld.cn/3/2-3.php)

**不要给外层容器元素随意设置宽度，然后再通过各种计算设置内部元素宽度，导致难以维护，一个变更则要变动一片**

#### 格式化宽度

格式化宽度。格式化宽度仅出现在“绝对定位模型”中，也就是出现在position属性值为absolute或fixed的元素中。在默认情况下，绝对定位元素的宽度表现是“包裹性”，宽度由内部尺寸决定，但是，有一种情况其宽度是由外部尺寸决定的，对于非替换元素，当left/top或top/bottom对立方位的属性值同时存在的时候，元素的宽度表现为“格式化宽度”，其宽度大小相对于最近的具有定位特性（position属性值不是static）的祖先元素计算

### 内部元素与流体特性

判断一个元素使用的是否为“内部尺寸”呢？很简单，假如这个元素里面没有内容，宽度就是0，那就是应用的“内部尺寸”。

内部尺寸”有下面3种表现形式。

（1）包裹性。“shrink-to-fit”。除了“包裹”，还有“自适应性”。“自适应性”是区分后面两种尺寸表现很重要的一点。所谓“自适应性”，指的是元素尺寸由内部元素决定，但永远小于“包含块”容器的尺寸（除非容器尺寸小于元素的“首选最小宽度”）。换句话说就是，“包裹性”元素冥冥中有个max-width:100%罩着的感觉。因此，对于一个元素，如果其display属性值是inline-block，那么即使其里面内容再多，只要是正常文本，宽度也不会超过容器。于是，图文混排的时候，我们只要关心内容，除非“首选最小宽度”比容器宽度还要大，否则我们完全不需要担心某个元素内容太多而破坏了布局。
除了inline-block元素，浮动元素以及绝对定位元素都具有包裹性，均有类似的智能宽度行为。
[http://demo.cssworld.cn/3/2-4.php](http://demo.cssworld.cn/3/2-4.php)

（2）首选最小宽度。所谓“首选最小宽度”，指的是元素最适合的最小宽度。
  
- 东亚文字（如中文）最小宽度为每个汉字的宽度。
- 西方文字最小宽度由特定的连续的英文字符单元决定。并不是所有的英文字符都会组成连续单元，一般会终止于空格（普通空格）、短横线、问号以及其他非英文字符等
- 类似图片这样的替换元素的最小宽度就是该元素内容本身的宽度

（3）最大宽度。

最大宽度就是元素可以有的最大宽度。我自己是这么理解的，“最大宽度”实际等同于“包裹性”元素设置white-space:nowrap声明后的宽度。如果内部没有块级元素或者块级元素没有设定宽度值，则“最大宽度”实际上是最大的连续内联盒子的宽度

### 高度height

height:auto的表现也基本上就是这个套路：有几个元素盒子，每个多高，然后一加，就是最终的高度值了。height:auto也有外部尺寸特性。其仅存在于绝对定位模型中，也就是“格式化高度”

height和width还有一个比较明显的区别就是对百分比单位的支持。对于width属性，就算父元素width为auto，其百分比值也是支持的；但是，对于height属性，如果父元素height为auto，只要子元素在文档流中，其百分比值完全就被忽略了。百分比高度值要想起作用，其父级必须有一个可以生效的高度值

如果包含块的高度没有显式指定（即高度由内容决定），并且该元素不是绝对定位，则计算值为auto

### 替换元素

通过修改某个属性值呈现的内容就可以被替换的元素就称为“替换元素”。因此，`<img>`、`<object>`、`<video>`、`<iframe>`或者表单元素`<textarea>`和`<input>`都是典型的替换元素。

（1）内容的外观不受页面上的CSS的影响。用专业的话讲就是在样式表现在CSS作用域之外。如何更改替换元素本身的外观？需要类似`appearance`属性，或者浏览器自身暴露的一些样式接口，例如`::-ms-check{}`可以更改高版本IE浏览器下单复选框的内间距、背景色等样式，但是直接`input[type='checkbox']{}`却无法更改内间距、背景色等样式。

（2）有自己的尺寸。在Web中，很多替换元素在没有明确尺寸设定的情况下，其默认的尺寸（不包括边框）是300像素×150像素，如`<video>`、`<iframe>`或者`<canvas>`等，也有少部分替换元素为0像素，如`<img>`图片，而表单元素的替换元素的尺寸则和浏览器有关，没有明显的规律。

（3）在很多CSS属性上有自己的一套表现规则。比较具有代表性的就是`vertical-align`属性，对于替换元素和非替换元素，`vertical-align`属性值的解释是不一样的。比方说`vertical-align`的默认值的`baseline`，被定义为字符x的下边缘，但是到了替换元素那里就不适用了。因为替换元素的内容往往不可能含有字符x，于是替换元素的基线就被硬生生定义成了元素的下边缘。

“替换元素和非替换元素之间只隔了一个src属性”

“替换元素和非替换元素之间只隔了一个CSS content属性！”

“content属性生成的内容和普通元素内容才会有很多不同的特性表现”

（1）我们使用content生成的文本是无法选中、无法复制的，好像设置了user- select:none声明一般，但是普通元素的文本却可以被轻松选中。同时，content生成的文本无法被屏幕阅读设备读取，也无法被搜索引擎抓取

（2）不能左右:empty伪类。:empty是一个CSS选择器，当元素里面无内容的时候进行匹配

（3）content动态生成值无法通过js获取，如`counter()`的值

### content的使用

- 辅助元素生成
```
.element:before {
　  content: '';
}
```

- 清除浮动
```
.clear:after {
　 content: '';
　 display: table;　/* 也可以是'block' */
　 clear: both;
}
```

- 图形生成

- 字体图标

- 图片生成

### margin:auto

- 有时候元素就算没有设置width或height，也会自动填充

    <div></div>

    此`<div>`宽度就会自动填满容器。

- 有时候元素就算没有设置width或height，也会自动填充对应的方位。例如：
    ```
    div {
    　  position: absolute;
    　  left: 0; right: 0;
    }
    ```
    此时`<div>`宽度就会自动填满包含块容器。


margin:auto的填充规则如下。

（1）如果一侧定值，一侧auto，则auto为剩余空间大小。

（2）如果两侧均是auto，则平分剩余空间。

**触发margin:auto计算有一个前提条件，就是width或height为auto时，元素是具有对应方向的自动填充特性的**

### margin 无效

- display计算值inline的非替换元素的垂直margin是无效的
- 表格中的<tr>和<td>元素或者设置display计算值是table-cell或table-row的元素的margin都是无效的
- margin合并的时候，更改margin值可能是没有效果的
- 绝对定位元素非定位方位的margin值“无效”
    很多时候，我们对元素进行绝对定位的时候，只会设置1～2个相邻方位。例如：

        img { top: 10%; left: 30%;}
    此时right和bottom值属于auto状态，也就是右侧和底部没有进行定位，此时，这两个方向设置margin值我们在页面上是看不到定位变化的

- 定高容器的子元素的margin-bottom或者宽度定死的子元素的margin-right的定位“失效” 
- 鞭长莫及导致的margin无效
    ```
    <div class="box">
　     <img src="mm1.jpg">
　     <p>内容</p>
　  </div>

    .box > img {
    　  float: left;
    　  width: 256px;
    }
    .box > p {
    　  overflow: hidden;
    　  margin-left: 200px;
    }
    ```
    此时的`<p>`的`margin-left`从负无穷到256px都是没有任何效果的
    
- 内联特性导致的margin无效
    ```
    <div class="box">
    　 <img src="mm1.jpg">
    </div>
    .box > img {
    　  height: 96px;
    　  margin-top: -200px;
    } 
    ```

### absolute与overflow

绝对定位元素不总是被父级overflow属性剪裁，尤其当overflow在绝对定位元素及其包含块之间的时候。即**如果overflow不是定位元素，同时绝对定位元素和overflow容器之间也没有定位元素，则overflow无法对absolute元素进行剪裁**

```
<div style="overflow: hidden;">
　 <img src="1.jpg" style="position: absolute;">
</div>
```
overflow元素父级是定位元素也不会剪裁，例如：

```
<div style="position: relative;">
　<div style="overflow: hidden;">
　　 <img src="1.jpg" style="position: absolute;">
　</div>
</div>
```

但是，如果overflow属性所在的元素同时也是定位元素，里面的绝对定位元素会被剪裁：

```
<div style="overflow: hidden; position: relative;">
　 <img src="1.jpg" style="position: absolute;">　 <!-- 剪裁 -->
</div>
```

如果overflow元素和绝对定位元素之间有定位元素，也会被剪裁：

```
<div style="overflow: hidden;">
　 <div style="position: relative;">
　　 <img src="1.jpg" style="position: absolute;">　 <!-- 剪裁 -->
　 </div>
</div>
```

如果overflow的属性值不是hidden而是auto或者scroll，即使绝对定位元素高宽比overflow元素高宽还要大，也都不会出现滚动条。示例：[http://demo.cssworld.cn/6/5-11.php](http://demo.cssworld.cn/6/5-11.php)

由于position:fixed固定定位元素的包含块是根元素，因此，除非是窗体滚动，否则上面讨论的所有overflow剪裁规则对固定定位都不适用

ps: transform属性对overflow剪裁规则有影响，overflow元素自身transform的时候，Chrome和Opera浏览器下的overflow剪裁是无效的。 当大家遇到absolute元素被剪裁或者fixed固定定位失效时，可以看看是不是transform属性在作祟

### absolute的流体特性

绝对定位元素也具有类似块级元素的流体特性，条件是“对立方向同时发生定位的时候”。

如果只有left属性或者只有right属性，则由于包裹性，此时.box宽度是0。但是在本例中，因为left和right同时存在，所以宽度就不是0，而是表现为“格式化宽度”，宽度大小自适应于.box包含块的padding box，也就是说，如果包含块padding box宽度发生变化，.box的宽度也会跟着一起变

绝对定位元素的这种流体特性比普通元素要更强大，普通元素流体特性只有一个方向，默认是水平方向，但是绝对定位元素可以让垂直方向和水平方向同时保持流动性。

## position: relative

相对定位元素的left/top/right/bottom的百分比值是相对于包含块计算的，而不是自身。注意，虽然定位位移是相对自身，但是百分比值的计算值不是。

top和bottom这两个垂直方向的百分比值计算跟height的百分比值是一样的，都是相对高度计算

当相对定位元素同时应用对立方向定位值的时候，也就是top/bottom和left/right同时使用的时候，其表现和绝对定位差异很大。绝对定位是尺寸拉伸，保持流体特性，但是相对定位却是“你死我活”的表现。**top/bottom同时使用的时候，bottom被干掉；left/right同时使用的时候，right毙命。**


**本文大量内容来源于张鑫旭的《css 世界》一书，是一篇读书笔记。非原创内容。**