# CSS VOCABULARY

CSS词汇是前端开发经常需要用到的东西，但是各种各样的CSS词汇经常令人眼花缭乱。本文主要总结一些CSS词汇的主要功能和用法。

## **@import**

它是一种引入CSS的方式，可以在一个CSS文件中引入另一个CSS文件，使用方法如下：

```css
@import url(my.css);
```
## **Media Queries**

根据字面意思，“Media Queries”就是媒体查询。在CSS中，我们可以设置不同类型的媒体条件，并根据相应的条件，为符合条件的媒体调用与之对应的样式表。它主要用于响应式网页设计（Responsive web Design），该设计可使网站在多种浏览设备上完美显示。

Media Queries的简单用法如下：

```css
@media only screen and (max-width: 700px) {
    #content {
        width: 50%;
    }
}
```

以上media语句表示，当屏幕的最大宽度小于700px时，#content的宽度就变为原来的50%。这个例子包含了几种概念：

* 媒体类型（Media Type）

通过媒体类型可以对不同的设备指定不同的样式，如上例中的screen。[常见的媒体类型](http://www.w3.org/TR/CSS2/media.html#media-types)有数十种。

* 媒体特性（Media Features）

媒体特性就是媒体类型的特性，如上例中的mix-width，是指屏幕的最大宽度。[常见的媒体特性](http://www.w3.org/TR/css3-mediaqueries/#media1)有十三种。

* 关键词（Key Word）

关键词有三种：and， not， only。

and用于定义“与”条件，例如：

```css
@media screen (min-width:600px) and (max-width:900px)
```

是指屏幕的最小宽度大于600px并且最大宽度小于900px。

`not`用于排除某种特定的媒体类型，例如：

```css
@media not print and (max-width: 1200px)
```

是指排除print这种媒体类型。

only用来指定某种特定的媒体类型，例如：

```css
@media only screen and (max-device-width:240px)
```

用来特指screen这种媒体类型。

## **CSS选择器**

CSS选择器是CSS中最常用的东西，我们都不会感到陌生。但是它还有一些高端、神奇、不为常人所知的功能，下文将一一介绍。常用的CSS选择器有三种：
 
* 简单选择器（Simple Selector）
* 属性选择器（Attribute Selector）
* 伪类选择器（Pseudo-class Selector）


### **一. 简单选择器**

简单选择器是CSS世界使用最为广泛的选择器，我将通过一个实例来总结它的用法。实例的代码如下：

```html
<div class="demo">
    <ul>
        <li id="first" class="first">1</li>
        <li class="active important">2</li>
        <li class="important items">3</li>
        <li class="important">4</li>
        <li class="items">5</li>
        <li>6</li>
        <li>7</li>
        <li>8</li>
        <li>9</li>
        <li id="last" class="last">10</li>
    </ul>
</div>

```

在添加一些样式之后，实例的初步现实如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-status.png)

#### 1. 通配符选择器（Universal Selector）（*）

通配符选择器用来选择所有元素，或者某个元素下的所有元素。例如：

```css
* {
    margin: 0;
    padding: 0;
}
```

表示选择所有元素，并将其margin、padding都设置为0。

```css
.demo * {
    border: 1px solid blue;
}
```

表示选择了.demo下的所有所有元素。以上例子的效果图如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-tp.png)

从图中可以看出，div.demo下所有元素的边框都加上了上面代码中定义的样式。

#### 2. 元素选择器（Element Selector）（E）

元素就是html页面中的元素，例如本文中的div，ul，li等。元素选择器用来选择这些页面元素。例如：

```css
li {
    background-color: grey;
    color: orange;
}
```

这个例子选择html中的li元素，并将其背景色设为grey，前景色设为orange，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-e.png)

#### 3. 类选择器（Class Selector）（.className）

类选择器根据html文档中元素的类名来选择元素，也就是说元素必须有类名才能使用类选择器来选择。上文中的items，important等就是为相应元素添加的类，类选择器可以用来选择这些元素，例如：

```css 
.important {
    font-weight: bold;
    color: yellow;
}
```
表示选择类名为important的元素，并将其字体加粗、前景色设为yellow。值得注意的是，类选择器并不仅仅选择类名为important的元素，而是选择类名中包含important的元素，效果图如下：

![](/images/img_for_css_vocabulary/class.png)

#### 4. ID选择器（ID Selector）(#ID)

与类选择器类似，ID选择器根据html文档中元素的id来选择元素，例如：

```css
#first {
    background: lime;
    color: #000;
}

#last {
    background: #000;
    color: lime;
}
```

上例中，选择id为first的元素，将其背景色设为lime，前景色设为#000；选择id为last的元素，将其背景色设为#000，前景色设为lime，效果如下：

![](http://www.w3cplus.com/sites/default/files/css-select-id.png)

#### 5. 后代选择器（Descendant Selector）(E F)

后代选择器也叫包含选择器，它可以选择某元素的后代元素，例如：

```css
.demo li {color: blue;}
```
上例中，.demo是祖先元素，li是后代元素。该例子所表达的意思是，选择了.demo的所有后代li元素，不管li与.demo之间是子元素、孙元素或者更深的层级关系，li都会被选中。效果为：

![](http://www.w3cplus.com/sites/default/files/css3-selector-ef.png)

#### 6. 子选择器（Child Selector）(E>F)

与后代选择器不同，子元素指选择某元素的子元素。例如：E>F，其中E为父元素，F为子元素，该表达式表示选择E元素下的所有子元素F。

```css
ul > li {
    background: green;
    color: yellow;
}
```

这个例子表示，选择ul的所有子元素li，效果图如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-chirld.png)

如果将上例中的ul改为div则不能选中li元素，因为li不是div的子元素。


#### 7. 相邻选择器（Adjacent Sibling Selector）(E+F)

相邻选择器可以选择紧接在另一个元素后的元素，而且他们具有相同的父元素。例如：E+F，E元素和F元素具有相同的父元素，且F元素在的后面。

```css
li + li {
    background: green;
    color: yellow;
    border: 1px solid #ccc;
}
```

上例用来选择li之前有个li的元素，效果图如下：

![](http://www.w3cplus.com/sites/default/files/css-selector-xl.png)

从图中可以看出，第一个li元素没有被选中，因为它的前面没有li元素。

#### 8. 普通兄弟选择器（General Sibling Selector）(E~F) 

普通兄弟选择器用来选择元素后面所有的兄弟元素，它们可以不相邻，但必须具有相同的父元素。例如：

```css
.active ~ li {
    background: green;
    color: yellow;
    border: 1px solid #ccc;
}
```

用来选择.active元素后面的所有兄弟元素li，效果如下:

![](http://www.w3cplus.com/sites/default/files/css3-selector-tongyong.png)

#### 9. 群组选择器（Group Selector）(selector1,selector2,...,selectorN)

群组选择器将具有相同样式的元素放在一起，中间以逗号隔开，例如：

```css
.first, .last {
    background: green;
    color: yellow;
    border: 1px solid #ccc;
}
```

用来为.first和.last元素添加一些相同的样式，效果如下:

![](http://www.w3cplus.com/sites/default/files/css3-selector-group.png)

### **二. 属性选择器**

属性选择器用来对带有指定属性的html元素设置样式。同样，我将通过一个实例来总结属性选择器的用法，实例的代码如下：

```html
<div class="demo clearfix">
    <a href="http://www.xxx.com" target="_blank" class="links item first" id="first" title="first">1</a>
    <a href="" class="links active item" title="test website" target="_blank" lang="zh">2</a>
    <a href="sites/file/test.html" class="links item" title="this is a link" lang="zh-cn">3</a>
    <a href="sites/file/test.png" class="links item" target="_balnk" lang="zh-tw">4</a>
    <a href="sites/file/image.jpg" class="links item" title="zh-cn">5</a>
    <a href="mailto:xxx@hotmail" class="links item" title="website link" lang="zh">6</a>
    <a href="" class="links item" title="open the website" lang="cn">7</a>
    <a href="" class="links item" title="close the website" lang="en-zh">8</a>
    <a href="" class="links item" title="http://www.sina.com">9</a>
    <a href="" class="links item last" id="last">10</a>
</div>
```

在添加一些样式之后，实例的初步现实如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-status.png)

#### 1. E[attr]

E[attr]用来选择具有某种属性的元素，例如：

```css
.demo a[id] {
    background: blue;
    color: yellow;
    font-weight: bold;
}
```
表示，.demo下所有带id属性的a元素，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-attr.png)

#### 2. E[attr = "value"]

E[attr = "value"]用来选择具有某种属性，并且属性的值为“value”的元素，例如：

```css
.demo a[id="first"] {
    background: blue;
    color: yellow;
    font-weight: bold;
}
```
表示，.demo下带有id属性，并且id的值为“first”的元素，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-attr2.png)

#### 3. E[attr ~= "value"]

`E[attr ~= "value"]`根据属性值中的词列表来进行选择，选择属性值中具有“value”这个词的元素，例如：

```css
.demo a[title~="website"] {
    background: orange;
    color: green;
}
```
表示，.demo下的a元素的title属性中，只有其属性值包含“website”这个词就会被选择，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-attr4.png)

#### 4. E[attr ^= "value"]

`E[attr ^= "value"]`选择attr属性值以“value”开头的所有元素，例如：

```css
.demo a[href^="http://"] {
    background: orange;
    color: green;
}

.demo a[href^="mailto:"] {
    background: green;
    color: orange;
}
```

表示，选择.demo下具有href属性的a元素，并且href属性的值是以`http://`和`mailto:`开头，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-attr5.png)

#### 5. E[attr $= "value"]

与 `E[attr ^= "value"]` 相反，`E[attr $= "value"]` 选择 `attr`属性值以“value”结尾的元素，例如：

```css
.demo a[href$="png"] {
    background: orange;
    color: green;
}
```
表示，选择.demo下具有href属性的a元素，并且属性href的值以“png”结尾，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-attr6.png)

#### 6. E[attr *= "value"]

`E[attr *= "value"]`选择attr属性值包含子串“value”的所有元素，例如：

```css
.demo a[title*="site"] {
    background: black;
    color: white;
}
```
表示，选择.demo下具有title属性的a元素，并且title属性值包含子串“site”，效果如下：


![](http://www.w3cplus.com/sites/default/files/css3-selector-attr7.png)

#### 7. E[attr |= "value"]

`E[attr |= "value"]`选择`attr`属性值等于“value”或以“value-”开头的所有元素，例如：

```css
.demo a[lang|="zh"] {
    background: gray;
    color: yellow;
}
```
表示，选择`.demo`下具有`lang`属性的`a`元素，并且属性lang的值为“zh”或以“zh-”开头，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-attr8.png)

### **三. 伪类选择器**

#### 动态伪类

动态伪类并不存在与html文档中，只有当用户和网站交互的时候它才能体现出来。动态伪类包含两种：锚点伪类（Anchor Pseudo-Classes）和用户行为伪类（User Action Pseudo-Class）。

* **锚点伪类**

锚点伪类常用于链接的样式设置，例如：

```css
.demo a:link {
    color: gray;
}

.demo a:visited {
    color: yellow;
}

.demo a:hover {
    color: green;
}

.demo a:active {
    color: blue;
} 
```
表示，链接没有被访问时前景色为灰色，链接被访问过后前景色为黄色，鼠标悬浮在链接上时前景色为绿色，鼠标点中激活链接那一下前景色为蓝色。
注意这四个锚点伪类的设置顺序，如果顺序不对会使我们设置的样式不生效。

* **用户行为伪类**

用户行为伪类用于设置用户和页面交互时的一些样式设置，锚点伪类中的:hover和:active同时又是用户行为伪类。

* :hover，用于设置当用户把鼠标移动到元素上面时的效果；
* :active，用于设置用户点击元素时的效果（动作发生在点击的当时，松开鼠标此动作就会消失）；
* :focus，用户设置某元素成为焦点时的样式（例如选择一个输入框）。

#### UI状态伪类

常见的UI状态伪类有：

* :enabled
* :disabled
* :checked

这些伪类主要针对html文档中的Form元素操作，最常见的有"type='text'"有enabled和disabled两种状态，前者表示可写，后者表示不可写。那么使用UI状态伪类就可以为同一个元素的不同状态设置不同的样式，例如：

```
input[type="text"]:disabled {border:1px solid #999;background-color: #fefefe;}
```
表示，为页面中禁用的文本框设置一个样式。

#### :nth选择器

:nth选择器是伪类选择器中最复杂的一个，我将通过一个实例总结其用法，实例的代码为：

```html
<div class="demo clearfix">
    <ul class="clearfix">
        <li class="first odd" id="first"><a href="">1</a></li>
        <li class="even"><a href="">2</a></li>
        <li class="odd"><a href="">3</a></li>
        <li class="even"><a href="">4</a></li>
        <li class="odd"><a href="">5</a></li>
        <li class="even"><a href="">6</a></li>
        <li class="odd"><a href="">7</a></li>
        <li class="even"><a href="">8</a></li>
        <li class="odd"><a href="">9</a></li>
        <li class="even last" id="last"><a href="">10</a></li>
    </ul>
</div>
```

在添加一些样式之后，实例的初步现实如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-status.png)

##### 1. :first-child() 

:first-child()用来选择某个元素的第一个元素，例如：

```css
.demo li:first-child {
    background: green;
    border: 1px dotted blue;
}
```

表示，选择.demo的第一个li子元素，并为其加上一些样式，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-nth1.png)

##### 2. :last-child()

顾名思义，:last-child()用来选择某个元素的最后一个子元素，例如：

```css
.demo li:last-child {
    background: green;
    border: 2px dotted blue;
}
```

表示，选择.demo的最后一个li子元素，并为其加上一些样式，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-nth2.png)

##### 3. :nth-child()

:nth-child()可以选择一个元素的某个子元素，例如：

```css
.demo li:nth-child(3) {background: lime;}
```

表示.demo的第三个li子元素，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-nth3.png)

也可以选择某些子元素，有如：

```css
.demo li:nth-child(n) {background: lime;}
```

表示，选择.demo下所有的子元素，这里的n是一个从0开始计数的变量，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-nth4.png)

```css
.demo li:nth-child(-n+5) {background: lime;} 
```

因为n是从0开始计数的，所以以上代码会选择.demo的前5个li子元素。

![](http://www.w3cplus.com/sites/default/files/css3-selector-nth8.png)

:nth-child()的用法还有很多，具体总结如下：

```css
:nth-child(length);/*参数是具体数字*/
:nth-child(n);/*参数是n,n从0开始计算*/
:nth-child(n*length)/*n的倍数选择，n从0开始算*/
:nth-child(n+length);/*选择大于length后面的元素*/
:nth-child(-n+length)/*选择小于length前面的元素*/
:nth-child(n*length+1);/*表示隔几选一*/
```

对于:nth-child()有个很好玩，也特别容易出错的例子：

```html
<ul>
    <a>link</a>
    <p>some text</p>
    <li>apple 1</li>/*这一行会被选出*/
    <li>apple 2</li>
    <li>apple 3</li>
    <li>apple 4</li>
    <li>apple 5</li>
</ul>
```

对于这段代码，使用CSS选择器：

```css
ul li:nth-child(3) {
    color: red;
}
```

会选中apple 1那一行。这是因为nth-child()并不是从前面的选择符筛选后的结果中选择第n个。它的定义是，选择在其parent下的第n个子元素。而这个“第n个”的计算，是把它parent下所有的子元素都包括在内的。

##### 4. :nth-last-child()

:nth-last-child()与:nth-child()相似，只是它是由最后一个开始元素开始计数的，例如：

```
.demo li:nth-last-child(4) {background: lime;}
```
表示，选择.demo下的倒数第四个li元素，效果如下：

![](http://www.w3cplus.com/sites/default/files/css3-selector-nth10.png)

##### 5. :empty

:empty用于设置当元素没有任何内容时的样式。例如，页面中有一个box是用来输出错误信息的，那么当没有错误信息时，我们就可以通过:empty使之不显示。

##### 6. other nth selector

:nth选择器还有很多与:nth-child()相类似的其他用法，这里只将它们列举出来，不再一一赘述。

* :first-of-type
* :last-of-type
* :nth-of-type
* :nth-last-type
* :only-child
* :only-of-type

## **伪元素**

所谓伪元素，可以理解为浏览器为某元素附加的元素。常用的伪元素有：

* ::first-line
* ::first-letter
* ::before
* ::after

::first-line和::first-letter可以用于方便的为文本添加一些特殊的样式，例如：

```html
<div class="line">
    <p>The Coming of Wisdom with Time; <br>
        Though leaves are many, the root is one; <br>
        Through all the lying days of my youth; <br>
        I swayed my leaves and flowers in the sun; <br>
        Now I may wither into the truth.
    </p>
</div>
```

可以通过伪元素为其添加一些特殊样式：

```css
p::first-line {
    font-style: italic;
    color: red;
}

p::first-letter {
    font-size: 70px;
    float: left;
    margin-right: 3px;
}
```

效果为：

![](/images/img_for_css_vocabulary/firstline.png)

同理，::before和::after用于向元素的前面和后面添加一些特殊样式，它们需要与content属性一起使用：

```css
.css-class:before {
    content: " ";
}

.css-class:after {
    content: " ";
}
```

通过伪元素，可以方便的实现一些使用其他方法需要花费很大力气的工作。可以通过**[这里](http://icodeit.org/2013/05/before-and-after-selector-in-css/
)**了解它的用法和一些实例.