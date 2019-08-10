# Spring AOP的实现原理
## AOP介绍
AOP利用“横切”的技术将位于同一层面多个类的公共行为封装为一个可重用模块即“切面”，降低重复代码与模块耦合。
AOP将系统划分为两部分：核心关注点与横切关注点。
* 核心关注点使用业务逻辑处理
* 横切关注点常发生于核心关注点多处且基本相似，如日志、权限认证等。
## AOP实现原理
AOP实现与JAVA设计模式中的代理模式相关。
代理模式可划分为：1. 静态代理 2. 动态代理
- ### AOP静态代理实现  
AOP静态代理实现的关键在于代理对象与目标对象实现共同的接口，并且代理对象持有目标对象的引用。

```java
// 接口
public interface MagicWand {
    /*
    * 挥动魔法棒
    */
    void wave();
}
```

```java
// 接口实现
public class MagicWandImpl implements MagicWand {

    @Override
    public void wave(String words) {
        System.out.println(words + " The magic wand is waving!");
    }
}
```

```java
// 代理类增加日志功能
public class ProxyMagicWand implements MagicWand {
    private MagicWand magicWand;

    public ProxyMagicWand(MagicWand magicWand) {
        super();
        this.magicWand = magicWand;
    }

    @Override
    public void wave(String words) {
        System.out.println("The magic wand will wave!");
        magicWand.wave(words);
        System.out.println("The magic wand has been waved!");
    }
}
```

```java
// 测试代码
public class StaticTest {
    public static void main(String[] args) {
        MagicWand magicWand = new ProxyMagicWand(new MagicWandImpl());
        magicWand.wave("expecto patronum!");
    }
}
```

- ### AOP动态代理实现
静态代理存在缺陷：代理类与目标类一一对应，无法应对大量类需要代理的需求。  
JDK1.3后提供了一个API：java.lang.reflect.InvocationHandler，在JVM调用某个类的方法时动态地给该方法额外增加某些需求。  
动态代理实现主要是通过实现InvocationHandler，并且将目标对象注入到代理对象中，利用反射机制来执行目标对象的方法。

```java
public class DynamicProxyMagicWand implements InvocationHandler {
    // 目标对象
    private Object target;

    /*
    * 通过反射实例化目标对象
    * @param object
    * @return  
    */
    public Object bind(Object object) {
        this.target = object;
        return Proxy.newProxyInstance(this.target.getClass().getClassLoader(), this.target.getClass().getInterfaces(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) {
        Object result = null;
        System.out.println("The magic wand will wave!");
        // 通过反射机制运行目标对象的方法
        result = method.invoke(this.target, args);
        System.out.println("The magic wand has been waved!");
        return result;
    }
}
```

```java
// 测试代码
public class DynamicTest {
    public static void main(String[] args) {
        MagicWand magicWand = (MagicWand) new DynamicProxyMagicWand().bind(new MagicWandImpl());
        magicWand.wave("expecto patronum!");
    }
}
```

- ### AOP动态代理实现完善
解耦额外操作与被代理对象扩展功能

``` java
// Logger接口
public interface Logger {
    void start(Method method);
    void end(Method method);
}
```

```java
// Logger接口实现
public LoggerImpl implements Logger {

    @Override
    void start(Method method){
        System.out.println("The magic wand will wave!");
    }

    @Override
    void end(Method method){
        System.out.println("The magic wand has been waved!");
    }
}
```

```java
public class DynamicProxyBetterMagicWand implements InvocationHandler{
    //调用对象
    private Object proxy;
    //目标对象
    private Object target;
    
    public Object bind(Object target, Object proxy){
        this.target = target;
        this.proxy = proxy;
        return Proxy.newProxyInstance(this.target.getClass().getClassLoader(), this.target.getClass().getInterfaces(), this);
    }
    
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        Object result = null;
        //反射得到额外操作的类
        Class clazz = this.proxy.getClass();
        //反射得到额外操作类的start方法
        Method start = clazz.getDeclaredMethod("start", new Class[]{Method.class});
        //反射执行start方法
        start.invoke(this.proxy, new Object[] {method});
        //执行实际方法
        method.invoke(this.target, args);
        //反射得到额外操作类的end方法
        Method end = clazz.getDeclaredMethod("end", new Class[]{Method.class});
        //反射执行end方法
        end.invoke(this.proxy, new Object[] {method});
        return result;
    }
    
}
```

```java
public class DynamicBetterTest {
    public static void main(String[] args) {
        MagicWand hello = (MagicWand) new DynamicProxyBetterMagicWand().bind(new MagicWandImpl(),new LoggerImpl());
        hello.sayHello("expecto patronum!");
    }
}
```

通过上例，可以发现通过动态代理和反射技术，已经基本实现了AOP的功能，即一个简单的spring aop框架。
