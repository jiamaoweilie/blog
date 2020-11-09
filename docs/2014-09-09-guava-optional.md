# Guava Optional

兜兜转转，又开始拾起这篇曾经放弃了几百次的文章。时隔一个多月，依然清晰的记得第一次写完那个例子时，被某大师问了几个问题后尴尬地想撞墙的心情。查阅了一些乱七八糟的资料，试图找到一些能够帮我理解的例子，但未果，我糊涂的大脑还是无法理解这个奇葩的存在。直到某天，得到点化，似乎有了一些理解（很多疑问还在），希望写完不要被杀。

先说一说为什么要用[Guava Optional](https://code.google.com/p/guava-libraries/wiki/UsingAndAvoidingNullExplained)吧. Google的官方文档中说，它是用来避免使用null的，而且Google的code base中大约95%的collection中不该含有null值。看到这里，我不禁郁闷，null到底是一个什么样令人深恶痛绝的东西，令他们欲除之而后快。

看看他们是怎么解释的吧。文档中说，null之所以令人讨厌，最主要的原因是其**含义模糊**。比如，当Map.get(key)返回null时，可能是因为Map中的值为null，也有可能是Map中没有对应的值。null可以表示失败，可以表示成功，可以表示任何事情，所以如果要想表达的意思比较清晰，就要避免使用null。这样看着似乎是很有道理，那就这么着吧，在使用Map的时候不要把null作为key传入。可是该用什么呢？`Optional.absent()`呗。

可是`Optional.absent()`到底是个什么呢？其实它就是Optional提供的一个表示不存在的符号。Optional是一种使用非空值代替可以为空的值的方式，它可能存在（isPresent为ture），也可能不存在。例如：

```java
Optional<Integer> possible = Optional.of(5);
possible.isPresent(); // returns true
possible.get(); // returns 5
```

除了上面说的方式外，什么时候使用Optional合适呢？ 看了一堆例子，大多像这样：

```java
public class GoogleGuavaOptionalExample {
    public static void main(String[] args) {
        Optional<List<String>> fromNull = Optional.fromNullable(getList());
        if (fromNull.isPresent()) {
            // do something
        }
    }

    private static List<String> getList() {
        return null;
    }
}
```

看完这个例子，估计我们的内心都不禁呐喊，这是个啥啊！！！！那我用其他方法，判断一下返回值是不是null不也一样（某大师也是这么问我的）。后来发现Optional这家伙主要用于[空对象模式(Null Object pattern)](http://en.wikipedia.org/wiki/Null_Object_pattern)。那空对象模式又是什么时候用呢？资料上说它的用法主要有两种：

1. 一种是返回集合的时候。
2. 策略模式(Strategy Pattern)或者状态模式(State Pattern)。

第一种用法也许说的就是上面的那几行代码中显示的那样。那策略模式又是怎么用呢？从字面意思看，Optional的意思是可选择的，它就是用来表示一个可有可无的东西。例如命令行中的`ls`命令，它的格式是`ls [选项] [目录名]`，其中的后两项就是Optional，是可有可无的，如果有的话，会产生一些额外的作用。比如`ls -i`，会列出每个文件的inode号；`ls -l Documents`，会列出Documents目录下所有文件的详细信息；如果单纯的使用`ls`，则直接列出当前目录的的子目录和文件。

遗憾的是这个高大上的例子我没有真正实现，那么就用这个接地气的例子代替吧。

*话说今日天朗气清，惠风和畅，某大师用自己的午饭推荐神器 [chisha](http://icodeit.org/2014/09/simple-idea-and-simple-script/) 摇出今天中午要吃炒饭，于是来到餐厅愉快的点餐。*

根据上述需求，我们设计出一个订单类：

```java
public class Order {
    private  Food food;
    private  OptionalRequirement optionalRequirement;

    public Order(Food food, OptionalRequirement optionalRequirement) {
        this.food = food;
        this.optionalRequirement = optionalRequirement;
    }

    public Food getFood() {
        return food;
    }

    public void setFood(Food food) {
        this.food = food;
    }

    public OptionalRequirement getOptionalRequirement() {
        return optionalRequirement;
    }

    public void setOptionalRequirement(OptionalRequirement optionalRequirement) {
        this.optionalRequirement = optionalRequirement;
    }
}
```
在设计订单类时，我们加入了一个OptionalRequirement属性，以满足顾客的一些额外需求。如果大师想给炒饭里面额外加个鸡蛋，那就下单：

```java
Food friedRice = new Food("friedRice");
OptionalRequirement moreEgg = new ExtraEgg();

Order order = new Order(friedRice, moreEgg);
```

如果想多吃点肉，那也很随意：

```java
Food friedRice = new Food("friedRice");
OptionalRequirement moreMeat = new ExtraMeat();

Order order = new Order(friedRice, moreMeat);
```

可是天有不测风云，大师今天偏偏啥额外需求也没有，那么这个订单会长成这样：

```java
Food friedRice = new Food("friedRice");
Order order = new Order(friedRice, null);
```

显然，这null具有各自语义不明的缺点，而且可能会造成空指针异常。这时候Optional就可以优雅的出场了，订单类会被改成这样：

```java
public class Order {
    private  Food food;
    private  Optional<OptionalRequirement> optionalRequirement;

    public Order(Food food, Optional<OptionalRequirement> optionalRequirement) {
        this.food = food;
        this.optionalRequirement = optionalRequirement;
    }

    public Food getFood() {
        return food;
    }

    public Optional<OptionalRequirement> getRequirementOptional() {
        return optionalRequirement;
    }

    public void setFood(Food food) {
        this.food = food;
    }

    public void setRequirementOptional(Optional<OptionalRequirement> optionalRequirement) {
        this.optionalRequirement = optionalRequirement;
    }
}
```

这里的`OptionalRequirement`就像是命令行中那些可有可无的可选参数，如果有额外要求时，就这么使用如下方法点单：

```java
Food friedRice = new Food("friedRice");
OptionalRequirement moreEgg = new ExtraEgg();

Order order = new Order(friedRice, Optional.of(moreEgg));
```

如果没有其他要求时，就是下面的方式：

```java
Food friedRice = new Food("friedRice");

Order order = new Order(friedRice, Optional.<OptionalRequirement>absent());
```

`Optional.<OptionalRequirement>absent()`用来表示额外的需求不存在，顾客就需要一碗常规的炒饭就好了。这样就避免了使用null语义不明确的问题。
