- 概念：函数式接口指是有且只有一个未实现的方法的接口，一般通过@FunctionalInterface表明某个接口是一个函数式接口。函数式接口是Java支持函数式编程的基础。
- 核心：以处理数据的方式处理代码。
- 用途：精简代码，允许在更高层次更自然的描述业务逻辑，表达能力更强，便于实现并行化。
- 语法：lamda表达式,即匿名函数，没有函数名只有函数体，可作为参数直接传递给相关调用者。
              输入：->前的()包围部分可以有0个、1个或多个入参
              函数体：->后的{}包围的部分即一段代码
              输出：有无返回值皆可，若需要返回则通过return。
              例：
  
**传统方式**
```java
Consumer c = new Consumer(){
    @Override
    public void say(Object o){
        System.out.println(o);
    }
};
```

 **函数式**
```
Consumer c = (o)->{
    System.out.println(o); 
};

```

方法引用可以用来简化lamda表达式，::前半部分是类名/实例名，::后半部分是方法名
静态方法引用：ClassName::methodName，例：

```
System.out::println

```

实例方法引用：instanceReference:methodName，例：Animal类中this::say
构造方法引用：Class::new
函数式接口有Consumer、Function、Predicate三种

Consumer有输入无输出
Function有一个输入一个输出
Predicate判断是否满足条件
Stream流式处理集合，filter过滤，map一一映射操作，collect转List，reduce通过一组值生成一个值。
Optional对象灵活应对各种空值、异常情况。
