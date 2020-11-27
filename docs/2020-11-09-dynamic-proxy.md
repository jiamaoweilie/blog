# Java 动态代理

之前学习了`Java`反射，那么反射有哪些应用呢？一般来讲，反射的主要应用场景有：

* 开发通用框架 - 反射最重要的用途就是开发各种通用框架。很多框架（比如`Spring`）都是配置化的（比如通过`XML`文件配置`JavaBean`、`Filter` 等），为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射——运行时动态加载需要加载的对象。
* 注解 - 注解本身仅仅是起到标记作用，它需要利用反射机制，根据注解标记去调用注解解释器，执行行为。如果没有反射机制，注解并不比注释更有用。
可扩展性功能 - 应用程序可以通过使用完全限定名称创建可扩展性对象实例来使用外部的用户定义类。
* 动态代理 - 在切面编程（`AOP`）中，需要拦截特定的方法，通常，会选择动态代理方式。这时，就需要反射技术来实现了。

关于前两个点，本文不在详细说明，这里我想通过**动态代理**来学习一个反射的使用。动态代理在Java中有着广泛的应用，比如`Spring AOP`、`Hibernate`数据查询、测试框架的后端`mock`、`RPC`远程调用、`Java`注解对象获取、日志、用户鉴权、全局性异常处理、性能监控，甚至事务处理等。

## 什么是代理模式？

`Java`中的动态代理与设计模式中的代理模式有关，所以我们先来学习一下什么是代理模式？

> 给目标对象提供一个代理对象，并由代理对象控制对目标对象的引用。

代理模式角色分为 3 种：

1. `Subject`(抽象对象）：定义代理类和真实主题的公共对外方法，也是代理类代理真实主题的方法；
2. `RealSubject`（真实对象）：真正实现业务逻辑的类；
3. `Proxy`（代理对象）：用来代理和封装真实主题；

![代理模式图解](/images/proxy.jpg)

其中，代理对象起到了中介的作用，连接着客户端和真实的目标对象，又可以在访问前后增加一些额外的操作，比如记录一下操作日志。就像我们需要买火车票，直接去火车站买特别的不方面，我们就会选择在网上代理买。代理提供给我们买车票的服务，并且还能提供一些自己额外的服务。

根据字节码的创建时间，代理模式可以分为静态代理和动态代理：

* 静态代理 - 程序运营钱已经存在代理类的字节码文件，代理类和真实对象的关系在运行前就确定了。
* 动态代理 - 源码在程序运行期间由`JVM`根据反射等机制动态生成，在运行前并不存在代理类的字节码文件。

## 静态代理

我们通过一个实例来学习一下静态代理。

```java
// 抽象对象
public interface Subject {
    void hello(String name);

    String bye();
}

// 真实对象
public class RealSubject implements Subject {
    @Override
    public void hello(String name) {
        System.out.println("Hello  " + name);
    }

    @Override
    public String bye() {
        System.out.println("Goodbye");
        return "Over";
    }
}
```
上面代码定义了抽象对象，和真实对象，那么我们该怎么实现一个静态代理呢。不要着急，我们继续看下面的代码。

```java
// 代理对象
public class SubjectProxy {
    private Subject vendor;

    public SubjectProxy(Subject vendor) {
        this.vendor = vendor;
    }

    public void hello(String name) {
        System.out.println("Before method, I want to do something.");
        vendor.hello(name);
        System.out.println("After method, I want to do something.");
    }

    public void bye() {
        System.out.println("Before method, I want to do something.");
        vendor.bye();
        System.out.println("After method, I want to do something.");
    }
}
```

上面代码就是我们实现的静态代理对象，我们在该对象的构造函数中传入需要代理的真实对象，然后在下面的函数中调用真实对象的函数，并且可以在函数执行前后各做一些操作。执行上面代码，输出如下：

```
Before method, I want to do something.
Hello  Jimmy
After method, I want to do something.

Before method, I want to do something.
Goodbye
After method, I want to do something.
```

这样做就实现了客户与真实类之间的解耦，在不修改真实类代码的情况下可以做一些额外操作。

但是这样做有哪些缺点呢？

1. 当需要代理多个类时，我们需要一一写实现代理
2. 当结果出现增删改的时候，代理类也需要修改

怎么做才能避免出现这些问题呢，铛铛铛铛，动态代理就可以闪亮登场了。

## 动态代理

刚刚我们了解到，动态代理就是**在运行是动态生成的代理类**。那么`Java`动态代理到底是怎么生成代理类的呢？首先我们需要`InvocationHandler`接口。

```java
public interface InvocationHandler { 
    Object invoke(Object proxy, Method method, Object[] args); 
} 
```

然后我们需要实现这个接口做为`调用处理器`。当我们调用代理类对象的方法时，这个`调用`会转送到`invoke`方法中，代理类对象作为`proxy`参数传入，参数`method`标识了我们具体调用的是代理类的哪个方法，`args`
为这个方法的参数。这样一来，我们对代理类中的所有方法的调用都会变为对invoke的调用，这样我们可以在invoke方法中添加统一的处理逻辑。

下面我们就来实现一个动态代理吧。

```java
public class DynamicProxy implements InvocationHandler {
    // 这个就是我们要代理的真实对象
    private Object object;

    // 构造方法，给我们要代理的真实对象赋初值
    public DynamicProxy(Object object) {
        this.object = object;
    }

    @Override
    public Object invoke(Object object, Method method, Object[] args) throws Throwable {
        // 在代理真实对象前我们可以添加一些自己的操作
        System.out.println("Before method");
        System.out.println("Call Method: " + method);
        // 当代理对象调用真实对象的方法时，其会自动的跳转到代理对象关联的handler对象的invoke方法来进行调用
        Object obj = method.invoke(this.object, args);
        // 在代理真实对象后我们也可以添加一些自己的操作
        System.out.println("After method");
        System.out.println();
        return obj;
    }
}
```

从上面代码可以看出，代理类有一个真是类的对象引用，并在`invoke`方法中调用了真是类的响应方法，和前面的静态代理的功能完全一样。但是不需要在代理类中显式调用真实类，当真实类发生更改时，这里也完全不需要任何修改。完美的解决了静态代理的问题。

下面我们看看如何使用上面定义的动态代理：

```java
// 我们要代理的真实对象
Subject realSubject = new RealSubject();
// 我们要代理哪个真实对象，就将该对象传进去，最后是通过该真实对象来调用其方法的
InvocationHandler handler = new DynamicProxy(realSubject);
/*
  * 通过Proxy的newProxyInstance方法来创建我们的代理对象，我们来看看其三个参数
  * 第一个参数 handler.getClass().getClassLoader() ，我们这里使用handler这个类的ClassLoader对象来加载我们的代理对象
  * 第二个参数realSubject.getClass().getInterfaces()，我们这里为代理对象提供的接口是真实对象所实行的接口，表示我要代理的是该真实对象，这样我就能调用这组接口中的方法了
  * 第三个参数handler， 我们这里将这个代理对象关联到了上方的 InvocationHandler 这个对象上
  */
Subject subject = (Subject) Proxy.newProxyInstance(
        handler.getClass().getClassLoader(),
        realSubject.getClass().getInterfaces(),
        handler);

subject.hello("Jimmy");
subject.bye();
```

上面代码中，我们调用`Proxy`类的`newProxyInstance`方法来获取一个代理类实例。这个代理类实现了我们指定的接口并且会把方法调用分发到指定的调用处理器。上面代码的注释中，详细的介绍了该方法。

执行上述代码的结果如下：

```
Before method, I want to do something.
Hello  Jimmy
After method, I want to do something.

Before method, I want to do something.
Goodbye
After method, I want to do something.
```

可以看出，结果完全相同。至此，我们已经解决了使用静态代理的缺点。本文示例代码，可以[点击此链接](https://github.com/jiamaoweilie/my-learn/tree/master/dynamic-proxy/src/com/tw)查看。









