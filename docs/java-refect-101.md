# Java 反射简介

## 什么是反射？

反射是java提供的一个重要功能，可以在运行时检查类、接口、方法和变量等信息，无需知道类的名字，方法名等。还可以在运行时实例化新对象，调用方法以及设置和获取变量值。

一般情况下，我们使用某个类时必定知道它是什么类，是用来做什么的。于是我们直接对这个类进行实例化，之后使用这个类对象进行操作。

```java
IPhone iPhone = new IPhone(); //直接初始化，「正射」
iPhone.setPrice(8499);
```

上面这样子进行类对象的初始化，我们可以理解为「正」。

而反射则是一开始并不知道我要初始化的类对象是什么，自然也无法使用 new 关键字来创建对象了。

这时候，我们使用 JDK 提供的反射 API 进行反射调用：

```java
Class clz = Class.forName("com.tw.reflect.IPhone");
Method method = clz.getMethod("setPrice", int.class);
Constructor constructor = clz.getConstructor();
Object object = constructor.newInstance();
method.invoke(object, 8499);
```

上面两段代码的执行结果，其实是完全一样的。但是其思路完全不一样，第一段代码在未运行时就已经确定了要运行的类（IPhone），而第二段代码则是在运行时通过字符串值才得知要运行的类（com.tw.reflect.IPhone）。

所以就说**反射就是在运行时才知道要操作的类是什么，并且可以在运行时获取类的完整构造，并调用对应的方法**。

## 基本用法

我们再来看看上面的例子：

```java
public class IPhone {
    private int price;

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public static void main(String[] args) throws Exception{
        //正常的调用
        IPhone iPhone = new IPhone();
        iPhone.setPrice(8499);
        System.out.println("iPhone Price:" + iPhone.getPrice());

        //使用反射调用
        Class clz = Class.forName("com.tw.IPhone"); //获取 Class 对象实例
        Constructor iPhoneConstructor = clz.getConstructor(); //根据 Class 对象实例获取 Constructor 对象
        Object iPhoneObj = iPhoneConstructor.newInstance(); //使用 Constructor 对象的 newInstance 方法获取反射类对象

        Method setPriceMethod = clz.getMethod("setPrice", int.class); //获取方法的 Method 对象
        setPriceMethod.invoke(iPhoneObj, 8499); //利用 invoke 方法调用方法
        Method getPriceMethod = clz.getMethod("getPrice"); //获取方法的 Method 对象
        System.out.println("iPhone Price:" + getPriceMethod.invoke(iPhoneObj)); //利用 invoke 方法调用方法
    }
}
```
在上面的例子中，我们首先使用正常的方式调用，然后再使用反射调用方法`setPrice`，将价格设置为`8499`再使用反射调用`getPrice`方法打印价格。运行结果如下：

```java
iPhone Price: 8499
iPhone Price: 8499
```
可以看出，两者的执行结果完全相同。从这个简单的例子中，我们也可以总结出使用反射获取对象的一般步骤：

#### 获取`Class`对象实例

```java
Class clz = Class.forName("com.tw.IPhone");
```
#### 根据 Class 对象实例获取 Constructor 对象

```java
Object iPhoneObj = iPhoneConstructor.newInstance();
```
至此，我们已经利用反射生成了对象的实例。如果还需要调用对象的方法，则还需要下面的步骤：

#### 获取方法的 Method 对象

```java
Method setPriceMethod = clz.getMethod("setPrice", int.class);
```

#### 利用 invoke 方法调用方法

```java
setPriceMethod.invoke(appleObj, 14);
```

到这里，我们已经能够掌握反射的基本使用。

## 常用反射API

### 获取反射中的Class对象

在反射中，要获取一个类或调用一个类的方法，我们首先需要获取到该类的`Class`对象。

在`Java API`中，获取`Class`类对象有三种方法：

* 使`Class.forName`静态方法。当你知道该类的全路径名时，你可以使用该方法获取`Class`类对象。

```java
Class clz = Class.forName("com.tw.IPhone");
```
* 使用`.class`方法。

这种方法只适合在编译前就知道操作的`Class`。

```java
Class clz = String.class;
```

* 使用类对象的`getClass()`方法。

```
IPhone iPhone = new IPhone();
Class clz = iPhone.getClass();
```
### 通过反射创建类对象

通过反射创建类对象主要有两种方式：

#### 通过`Class`对象的`newInstance()`方法

```java
Class clz = IPhone.class;
IPhone iPhone = (IPhone) clz.newInstance();
```
#### 通过`Constructor`对象的`newInstance()`方法。

```java
Class clz = IPhone.class;
Constructor constructor = clz.getConstructor();
IPhone iPhone = (IPhone) constructor.newInstance();
```

通过`Constructor`对象创建类对象可以选择特定构造方法，而通过 `Class`对象则只能使用默认的无参数构造方法。下面的代码就调用了一个有参数的构造方法进行了类对象的初始化。

```java
Class clz = IPhone.class;
Constructor constructor = clz.getConstructor(int.class);
IPhone iPhone = (IPhone) constructor.newInstance(8499);
```
#### 通过反射获取类属性、方法、构造器

我们通过`Class`对象的`getFields()`方法可以获取`Class` 类的属性，但无法获取私有属性。

```java
Class clz = IPhone.class;
Field[] fields = clz.getFields();
for (Field field : fields) {
    System.out.println(field.getName());
}
```
因为不能获得私有属性，所以上述代码输出了个寂寞。

那么怎么样才能获得所有属性呢，`getDeclaredFields()`方法就可以出场了：

```java
Class clz = IPhone.class;
Field[] fields = clz.getDeclaredFields();
for (Field field : fields) {
    System.out.println(field.getName());
}
```
这次的输出结果是：

```java
price
```
与获取类属性一样，当我们去获取类方法、类构造器时，如果要获取私有方法或私有构造器，则必须使用有`declared`关键字的方法。