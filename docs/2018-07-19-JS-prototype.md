# JavaScript 原型

## 原型对象及原型链

原型对象（prototype）是 JavaScript 特有的概念。在 JS 中，一切皆对象 （Object），而每个对象都有一个私有属性（_proto_）指向它的原型对象。该原型对象也有一个自己的原型对象，层层向上构成了一个**原型链**。直到一个对象的原型对象为 `null`，而 `null` 没有原型，是这个原型链中的最后一个节点。

## 基于原型链的继承

通过使用原型，JS可以建立其他传统 `OO` 语言中的继承，从而体现层次关系。

当试图访问一个对象的属性时，解析器需要从下往上的遍历整个原型链结构，直到找到一个名字匹配的属性或到达原型链的末尾。

下面有一个具体的例子，演示访问属性时会发生什么：

声明一个对象 `father`，有 `house` 和 `car` 两个属性：

```js
function Father(house, car) {
    this.house = house;
    this.car = car;

    this.makeMoney = function () {
        return "work hard and make money.";
    }
}
```

声明一个对象 `Son`, 什么属性也没有：

```js
function Son() {
    this.spendMoney = function () {
        return "spend money.";
    }
}
```

然后将 `Son` 的原型指向一个 `Father` 对象的实例：

```js
Son.prototype = new Father("2 houses", "3 cars");
```

这时候实例化一个 `Son` 对象：

```js
var son = new Son();

console.log(son.house);
console.log(son.car);
console.log(son.makeMoney());
```

得到结果：

```js
2 houses
3 cars
work hard and make money.
```

在定义时，`Son` 对象分明什么属性属性也没有，再将其原型指向一个 `Father` 对象的实例后，就有了 `Father` 的属性，就好像熊孩子继承了父亲财产一样。

下面我们来分析一下为什么会有上面的结果。

![](/images/proto.jpg)

根据前文所述，在遍历原型链的时候，是自上而下的，如果没有找到该属性，会继续顺着原型链往上，直到原型链的末端 `null`。 在上面例子中，`Son` 的原型指向 `Father` 的一个实例。所以在寻找 `son` 的属性时，寻找过程中会找到其值，从而实现了**继承**。 



