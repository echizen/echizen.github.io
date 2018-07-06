# 代码重复度检测

代码重复度是代码质量的一个重要方面。重复意味着维护更改时需要改动多处，也意味着抽象度不够，可以公用的部分没有抽取出来。

## js重复度检测

目标：能够发现一个文件和多个文件中重复的片段，并且能够识别替换变量名这样的混淆操作

通过参数threshold调控重合度检测的严格程度，即重复node数。

记录节点- 抓取重复片段并记录 - 扩展重复内容到最大值 - 分析包装结果

### AST结构

[https://babeljs.io/docs/en/next/babel-parser.html](https://babeljs.io/docs/en/next/babel-parser.html)

[https://astexplorer.net/](https://astexplorer.net/)

[AST format](https://github.com/babel/babel/blob/master/packages/babel-parser/ast/spec.md)

每个node都有type属性

程序多是在以一定的逻辑操作变量和数据，变量赋值、变量比较等

- Identifier：变量
- Literals: 数据

### 检测方案

- 获取：以文件为单位，通过`babylon`获取一般js及jsx文件AST节点，通过`vue-template-compiler`获取vue文件AST节点
- 记录：记录AST节点，以文件路径为key, 遍历平铺每个节点到value数组中，注意这一步是要将所有子节点也一并分析出平铺。平铺是为了减少树形层级结构带来的复杂度
- 分析：遍历节点，以所设置的重复度节点n个为一组，记录`_map`, 以`node.type`的拼接算出的hash值为key，n个的node组为值记录到_map里。**`node.type`反应的是一种逻辑结构**
- 分析：再以每个`_map[key]`为单位，分析其记录的重复node数组，之前记录的是以type为单位的可能重复的值(只要是相同的逻辑结构都被记录到了一组中)，再以identifiers和literals为维度分析出更准确的重复节点。具体方法是，如果要求严格匹配，即变量名也匹配，则针对类型为`identifiers`的节点，以`node.name`拼接出新的obj的 key值，划分分组，确保变量名称一致。然后类型为`literals`的节点以`node.value`拼接后划分分组，这一步是确保操作的数据一致。
- 分析：以上划分结果都是以设定的重复片段参数`threshold`分组的，在汇报结果时需要将这个重复片段扩展到最大，所以需要对每组结果进行扩充合并。具体做法是拿到每个重复的数组[]所在的文件的所有节点，从重节点数组的首尾开始向2边扩展，对每个重复的集合[[],[],...]的每一项都同事做扩展和比较，直到出现不相等时停止，记录新的最大重复片段的首尾
- 封装结果格式：将分析的重复片段封装成需要的数据格式

![image](https://s10.mogucdn.com/mlcdn/c45406/180618_6iki682jd49abl4k2e8di03ale573_404x818.png)

综上，主要思想是先以`node.type`为分析点找出逻辑相同可能重复的片段分组数组`typeGroupArr`，然后进一步在前面`typeGroupArr`的记录结果中近一步分析`identifiers`和`literals`。比较巧妙的确定重复片段的手段，是根据关键信息如`type、identifiers、literals`，以传入的重复量`threshold`为切割点，将相同内容记录到map里同一个key值的数组中。方便后面对比及提取和封装结果

## css重复度检测

js的重复原因基本就是复制粘贴导致。而css直接复制粘贴导致的重复场景并不多，但是由于大家对于css重视度不够，css的重复场景更多，譬如没有将公用样式抽成一个单独的选择器样式，而是在每个选择器里都重复声明；譬如维护阶段有新增样式要加时，重复声明选择器再重复写了部分样式。这些一方面是影响浏览器渲染效率，另一方面对于微信小程序这样对包大小有严格控制的平台更是对重复度更敏感。

相比于js直接比对重复节点，css的重复度检测立足于解决更多的问题，且不受样式声明顺序影响，检测方案也是更为复杂。

css我考虑了2种情况：

1. 相同选择器里重复声明同一属性样式，无论这个选择器层级权重，都要检测出来，譬如下面这个例子

    ```css
        .selector{
            font-size: 14px;
        }
        body .selector{
            font-size: 14px;
        }
    ```
    
2. 不同选择器之间重复样式片段，默认设置样式重复数大于等于3判定重复，这种情况是建议开发者将重复的样式抽离出单独的selector来减小文件体积，便于维护。如：
    
    ```css
        .containerLine {
            position: absolute;
            left: 0;
            bottom: 0;
            width: 750px;
            height: 1px; /* no2rem */
            background-color: #E5E5E5;
          }
          .line {
            z-index: 1;
            position: absolute;
            top: 90px;
            left: 0;
            background-color: #E5E5E5;
            width: 750px;
            height: 1px; /* no2rem */
          }
    ```
    
css ast结构： [http://astexplorer.net/#/2uBU1BLuJ1](http://astexplorer.net/#/2uBU1BLuJ1)

### 检测方案

- 获得所有`css selector`的AST节点，以selector为key值的obj存储。这个selector的取值是多个层级中最精确的那一层，譬如`.parent .child{}`的样式则记录在`.child`里
- 获取同一个`selector`下的重复样式节点，并记录
- 筛选重复的样式片段：
    - 在同一个`selector`中降维平铺所有的样式`decl`节点，比较2个`selector`之间的所有样式节点，如果相同就记录到`tempDupliRecored[locId][declToKey]`中，并一同记录下`selector`信息。（`locId`为selector为`file+startLoc`的hash）
    - 在`tempDupliRecored`里筛选，相同的`props`和`value`的`declNode`记录的重复node结果可能来源于不同selector的集合，单个selector的重复度不一定大于所设置的重复片段数`threshold`，需要剔除获得最优结果，最优结果是来源于不同的2个selector中的最大重复片段的记录。
    - 以重复片段的`declNode`的key值拼接 为key记录重复片段的obj `dupliRecored`
- 封装结果格式：包括重复node的顺序重排为`start line`大小顺序，囊括进位置信息、代码片段、文件路径信息

![image](https://s10.mogucdn.com/mlcdn/c45406/180618_61ie9haklkd2ih9kh0j45fc3ee61f_476x790.png)

综上，先将ast节点以selector为特征分组，然后找出相同selector里的重复样式属性。再两两比较selector，记下不同位置下重复的样式片段，然后通过位置信息，过滤出对于某一位置重复度大于所设置`threshold`值的样式数据。



