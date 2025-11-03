#### css创建一个自适应的九宫格
1、flex

flex容器：
display:flex;
flex-driction:rows;
flex-warp:warp;

flex子项：
width:33.3%;

2、grid
grid容器：
display:grid;
grid-template-columns: repeat(3, 1fr);

grid子项：
aspect-ratio: 1


1. flex: 1 代表什么？
答案：flex: 1是 flex: 1 1 0%的简写，意思是：
flex-grow: 1- 可以放大
flex-shrink: 1- 可以缩小
flex-basis: 0%- 基准大小为0
2. flex-basis和width的区别？
答案：
flex-basis定义项目在主轴方向的空间
width定义项目的固定宽度
当同时设置时，flex-basis优先级高于 width

### grid布局
grid是二维布局，将布局分为行和列，分别控制在行和列上面的布局
flex是一维布局，只在一个维度控制布局，

display:grid;块级元素
display:inline-grid;行内元素

子元素成为网格项目，
grid-template-columns和grid-template-rows来控制行或者列，水平行，垂直列，


grid-template-columns：设置列宽
grid-template-rows：设置行高

```
.wrapper {
  display: grid;
  /*  声明了三列，宽度分别为 200px 100px 200px */
  grid-template-columns: 200px 100px 200px;
  grid-gap: 5px;
  /*  声明了两行，行高分别为 50px 50px  */
  grid-template-rows: 50px 50px;
}
```
repeat() 函数：可以简化重复的值。该函数接受两个参数，第一个参数是重复的次数，第二个参数是所要重复的值。比如上面行高都是一样的，我们可以这么去实现，实际效果是一模一样的

```
.wrapper-1 {
  display: grid;
  grid-template-columns: 200px 100px 200px;
  grid-gap: 5px;
  /*  2行，而且行高都为 50px  */
  grid-template-rows: repeat(2, 50px);
}
```
auto-fill自动填充，让一行中尽量填充更多的单元格，
##### grid-template-columns: repeat(auto-fill, 200px) 表示列宽是 200 px，但列的数量是不固定的，只要浏览器能够容纳得下，就可以放置元素


##### fr关键字:
表示网格可用空间的一等分，grid-template-columns: 200px 1fr 2fr 表示第一个列宽设置为 200px，后面剩余的宽度分为两部分，宽度分别为剩余宽度的 1/3 和 2/3

##### minmax函数：
设置最大最小尺寸，表示长度就在这个范围之中都可以应用到网格项目中。它接受两个参数，分别为最小值和最大值。grid-template-columns: 1fr 1fr minmax(300px, 2fr) 的意思是，第三个列宽最少也是要 300px，但是最大不能大于第一第二列宽的两倍。

##### auto关键字：
由浏览器决定长度，grid-template-columns: 100px auto 100px 表示第一第三列为 100px，中间由浏览器决定长度，可以用作自适应宽度


##### gap:
grid-row-gap 属性、grid-column-gap 属性分别设置行间距和列间距。grid-gap 属性是两者的简写形式。

grid-row-gap: 10px 表示行间距是 10px，grid-column-gap: 20px 表示列间距是 20px。grid-gap: 10px 20px 实现的效果是一样的.

##### grid-template-areas 属性
grid-template-areas 属性用于定义区域，一个区域由一个或者多个单元格组成

一般这个属性跟网格元素的 grid-area 一起使用，我们在这里一起介绍。grid-area 属性指定项目放在哪一个区域

.wrapper {
  display: grid;
  grid-gap: 10px;
  grid-template-columns: 120px  120px  120px;
  grid-template-areas:
    ". header  header"
    "sidebar content content";
  background-color: #fff;
  color: #444;
}


上面代码表示划分出 6 个单元格，其中值得注意的是 . 符号代表空的单元格，也就是没有用到该单元格。

.sidebar {
  grid-area: sidebar;
}

.content {
  grid-area: content;
}

.header {
  grid-area: header;
}

以上代码表示将类 .sidebar .content .header所在的元素放在上面  grid-template-areas 中定义的 sidebar content header 区域中


##### grid-auto-flow 属性
grid-auto-flow  属性控制着自动布局算法怎样运作，精确指定在网格中被自动布局的元素怎样排列。默认的放置顺序是"先行后列"，即先填满第一行，再开始放入第二行，即下图英文数字的顺序 one,two,three...。这个顺序由 grid-auto-flow 属性决定，默认值是 row

grid-auto-flow: row dense，表示尽可能填满表格

grid-auto-flow: column，表示先列后行

##### justify-items,align-items
justify-items 属性设置单元格内容的水平位置（左中右），align-items 属性设置单元格的垂直位置
.container {
  justify-items: start | end | center | stretch;
  align-items: start | end | center | stretch;
}

start:对齐网格起始边缘
end:对齐单元格的结束边缘
center:单元格内部居中
stretch:拉伸，占满单元格的整个宽度（默认值）

##### justify-content 属性、align-content 属性
justify-content 属性是整个内容区域在容器里面的水平位置（左中右），align-content 属性是整个内容区域的垂直位置（上中下）

.container {
  justify-content: start | end | center | stretch | space-around | space-between | space-evenly;
  align-content: start | end | center | stretch | space-around | space-between | space-evenly;  
}

start - 对齐容器的起始边框
end - 对齐容器的结束边框
center - 容器内部居中
space-around - 每个项目两侧的间隔相等。所以，项目之间的间隔比项目与容器边框的间隔大一倍
space-between - 项目与项目的间隔相等，项目与容器边框之间没有间隔
space-evenly - 项目与项目的间隔相等，项目与容器边框之间也是同样长度的间隔
stretch - 项目大小没有指定时，拉伸占据整个网格容器


##### grid-auto-columns 属性和 grid-auto-rows 属性
隐式和显示网格：显式网格包含了你在 grid-template-columns 和 grid-template-rows 属性中定义的行和列。如果你在网格定义之外又放了一些东西，或者因为内容的数量而需要的更多网格轨道的时候，网格将会在隐式网格中创建行和列

假如有多余的网格（也就是上面提到的隐式网格），那么它的行高和列宽可以根据 grid-auto-columns 属性和 grid-auto-rows 属性设置。它们的写法和grid-template-columns 和 grid-template-rows 完全相同。如果不指定这两个属性，浏览器完全根据单元格内容的大小，决定新增网格的列宽和行高

##### grid-column-start 属性、grid-column-end 属性、grid-row-start 属性以及grid-row-end 属性
可以指定网格项目所在的四个边框，分别定位在哪根网格线，从而指定项目的位置

grid-column-start 属性：左边框所在的垂直网格线
grid-column-end 属性：右边框所在的垂直网格线
grid-row-start 属性：上边框所在的水平网格线
grid-row-end 属性：下边框所在的水平网格线


##### justify-self 属性、align-self 属性以及 place-self 属性
justify-self 属性设置单元格内容的水平位置（左中右），跟 justify-items 属性的用法完全一致，但只作用于单个项目

align-self 属性设置单元格内容的垂直位置（上中下），跟align-items属性的用法完全一致，也是只作用于单个项目



##### css3的animation
```
@keyframes animationName {
  0% { opacity: 0; }
  50% { opacity: 0.5; }
  100% { opacity: 1; }
}

.element {
  /* 语法：name duration timing-function delay iteration-count direction fill-mode play-state */
  animation: slideIn 2s ease 1s infinite alternate both;
  
  /* 多个动画 */
  animation: 
    fadeIn 1.5s ease-in-out,
    slideIn 2s ease-out 0.5s;
}
```

##### position属性
```
position有五个主要的值：
static（静态定位）
relative（相对定位）
absolute（绝对定位）
fixed（固定定位）
sticky（粘性定位）
```
1、static
特点：这是元素的默认值。元素按照正常的文档流进行排列。top, right, bottom, left和 z-index属性对静态定位的元素无效
2、relative
元素相对于其原本在正常文档流中的位置进行偏移,它原本在文档流中占据的空间会被保留，不会被其他元素填充。
3、absolute
素会脱离正常的文档流。它原本占据的空间会被后续元素填充,元素的位置是相对于最近的非 static（通常是 relative, absolute, fixed, sticky）定位的祖先元素来确定的,通常需要手动设置 width，因为脱离文档流后，块级元素不再默认 100% 宽度
4、fixed
元素会脱离正常的文档流,元素的位置是相对于浏览器视口（viewport）来确定的。这意味着即使页面滚动，它也会始终停留在屏幕的同一个位置。
常用来制作导航栏、返回顶部按钮、弹窗蒙层、页脚信息等需要始终可见的元素。
5、sticky
可以看作是 relative和 fixed的混合体。
元素在跨越特定阈值前表现为相对定位（relative），之后变为固定定位（fixed）。
必须指定 top, right, bottom, left四个阈值之一，粘性定位才会生效。这个值表示“离视口边缘还有多少距离时，触发固定效果”。
这个定位的容器是其最近的滚动祖先（滚动条所在的祖先元素），如果祖先都不可滚动，则相对于视口固定
##### css3最新属性
