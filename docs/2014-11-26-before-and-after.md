# 伪元素的使用

之前在学习CSS VOCABULARY的时候看到过有关伪元素的知识，知道它可以方便开发，但是具体用途和用法并没有真正掌握。最近有机会再次接触，对其用途有了一些新认识。

## 完美的画个衬线

在写页面的时候经常会遇到下图所示的情况，需要在一个标题或一段文字的下方显示一条定长的衬线。

![](/images/img_for_before_after/how-it-works.png)

我们可以通过给该元素的下面加另一个元素，然后设置一些样式来实现这种效果。但是如果使用伪元素就可以在不添加任何额外元素的情况下方便的画出这条衬线。

假设，有这样一个标题需要设置上图所示的样式：

``` html
<h2 class="title">HOW IT WORKS</h2>	
```

我们来看看如何使用伪元素来为该标题设置样式：

``` css
h2{
    text-align: center;
    color: $gray-black-color;
    position: relative;
    
    &::after {
    content: "";
    position: absolute;
    top: 35px;
    left: 50%;
    width: 50px;
    height: 3px;
    background-color: rgba(39, 39, 43, 0.1);
    margin: 0 0 0 -23px;
  }
}
```

可以看到，我们将元素本身的`position`设为`relative`，然后将其伪元素after的`positon`设为`absolute`，这样after的位置就相对于其父元素是绝对位置，我们可以通过设置它的`top`，`left`，或者`right`属性来调整它的位置。上面代码中设定该元素的`width`为`50px`,`height`为`30px`，并设置了`background-color`，这样它的样子就是一条长`50px`，高`3px`的线。我们再让其`left`等于`50%`，`top`等于`35px`，它就会位于其父元素上边框下方35px处，并居中。至此，我们就实现了上图所示的衬线效果。

## 让这些圈圈优雅的连起来

下面这张图也是页面上的一个常见情况，一些元素从中间被一条线连起来，像糖葫芦一样这条线在图片的中间。

![](/images/img_for_before_after/numbers.png)

同样，我们可以使用伪元素实现这条线。假设有下面一段代码需要实现上图的样式：

``` html
<ul>
    <li>
        <h2 class="number first">1</h2>
        <div class="description">
            <i class="fa fa-list-alt fa-7x"></i>
            <h2 class="descSummary">Compare</h2>
            <p>Magni dolores eoos qui retione voluptatem sequi nesciunt neque porro quisquam est</p>
        </div>
    </li>
    <li>
        <h2 class="number">2</h2>
        <div class="description">
            <i class="fa fa-pencil fa-7x"></i>
            <h2>Receive Proposals</h2>
            <p>Magni dolores eoos qui retione voluptatem sequi nesciunt neque porro quisquam est</p>
        </div>
    </li>
    <li>
        <h2 class="number">3</h2>
        <div class="description">
            <i class="fa fa-users fa-7x"></i>
            <h2>Sell with the Best</h2>
            <p>Magni dolores eoos qui retione voluptatem sequi nesciunt neque porro quisquam est</p>
        </div>
    </li>
</ul>
```

我们只关注于数字和其衬线部分的样式：

``` css
li {
  width: 33.33%;
  float: left;
  position: relative;
  .number {
    border: 2px $border-color solid;
    height: 60px;
    width: 60px;
    margin: auto;
    text-align: center;
    line-height: 60px;
    border-radius: 50%;
    color: $gray-color;
    background-color: $background-color;
  }
  .number::before {
    content: '';
    position: absolute;
    top: 30px;
    right: 50%;
    width: 100%;
    height: 2px;
    background: $border-color;
    z-index: -1;
  }
  .first::before {
    display: none;
  }
}
```

可以看到，每个`li`向左浮动，并且宽度为33.33%，这样这三个数字就可以平均分布在页面中的一行。然后为数字元素设置了宽和高都是`60px`，并让其`border-radius`为50%，设置边框之后，三个圆圈就实现了。我们设置数字元素的`position`为`relative`，其伪元素before的`position`为`absolute`，然后设置其`top`为圆圈半径，`right`为`50%`，`width`为`100%`，这样我们就成功的在这些圈圈中间画了一条线。但是这样下来，你会发现第一个元素不需要有这条线，而且线有可能跑到圈圈里面。别着急，我们可以让第一个数字的伪元素不显示，并给圆圈设置一个背景色来覆盖其里面多余的线。

## 立体的阴影也没有多麻烦

看到下面这几个图标，很多人想必跟我一样一头雾水，不知其然也不知其所以然。后来经过研究，虽不知其完美的实现方法，但是发现使用伪元素可以实现如图效果。

![](/images/img_for_before_after/cycle.png)

先来看看我的html代码：

``` html
<ul>
    <li class="like">
        <a href="#" class="icon-heart"></a></li>
    <li class="trash">
        <a href="#" class="icon-trash"></a>
    </li>
    <li class="continue">
        <a href="#" class="icon-play"></a>
    </li>
    <li class="sync">
        <a href="#" class="icon-arrow-sync"></a>
    </li>
    <li class="fast-forward">
        <a href="#" class="icon-media-fast-forward"></a>
    </li>
</ul>
```

假设这些`icon`是我的音乐播放器里面的link，我需要让它们变成上图所示的样子。这里的icon采取[icomoon](https://icomoon.io/)里面的字体。

其样式代码如下:

``` css
ul {
  li {
    display: inline-block;
    width: 10%;
    a {
      position: relative;
      text-decoration: none;
      text-align: center;
      display: block;
      line-height: 60px;
      font-size: 4em;
      width: 60px;
      height: 60px;
      border-radius: 50%;
      color: $ font-gray-color;
      background-color: white;
      @include box-shadow(rgba(black, 0.5)0 0 20 px);
      &::after {
        z-index: -100;
        content: "";
        position: absolute;
        width: 80px;
        height: 80px;
        right: -10px;
        top: -10px;
        border-radius: 50%;
        background-color: rgba($ font-gray-color, 0.5);
      }
    }
  }
}

```

通过代码可以看到，我们给 `a` 元素设置了一定的宽、高、边框、阴影，并通过 `after` 在其下面画一个比它本身的直径大 `20px` 的圆，这样二者重叠起来就是一个完美的有阴影的图标。

当然，伪元素的用法远不止这些，我们以后再慢慢研究吧。

