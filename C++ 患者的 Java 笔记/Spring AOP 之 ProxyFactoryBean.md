# Spring AOP 之 ProxyFactoryBean

Spring 里面的 AOP 实现有两种，一种是通过 ProxyFactoryBean 提供的 AOP 功能，一种是通过引入 aspectj 提供的 AOP 功能。本文先弄清楚 ProxyFactoryBean。

## AOP 中的几个概念

AOP 是面向切面编程，目前先了解一下几个概念。

**连接点**（joinpoint）

可以被拦截的点，目前 Spring 只支持方法类型的连接点，所以在 Spring 中连接点就是被拦截到的方法，广义上连接点还包括字段和构造器。

**切入点**（pointcut）

对连接点进行拦截的定义。在程序代码中体现为切入点表达式。

**通知**（advice）

拦截到连接点后要执行的代码，通知可分为：前置、后置、异常、最终、环绕五类。

**增强**（advisor）

通知（advice）的加强版，可以实现更灵活的功能。

**横切关注点**

对哪些连接点进行拦截，拦截后怎么处理，这些点就是横切关注点

**切面**（aspect）

切面是**横切关注点**的抽象，所以可以理解为：切面=切入点+通知

**目标对象**

被代理的目标对象

**其他**

AOP 中还有其他术语，暂不理会。

**个人理解**

程序中所有可以被拦截的地方成为连接点，我们真正指明的要拦截的地方就是切入点，拦截后要执行的逻辑（要执行的代码）叫做通知，我们在程序中加入了切入点和通知代码就是加入了一个切面，被加入了切面的程序单元叫目标对象。

这些概念的的解释参见：

https://www.cnblogs.com/hongwz/p/5764917.html

https://www.cnblogs.com/liuruowang/p/5711563.html

## ProxyFactoryBean 的几个重要属性

ProxyFactoryBean

- interfaces，需要拦截的接口
- target，代码的目标对象，一个 bean。
- interceptorNames，在目标对象中插入的通知（advice）或增强（advisor）列表。
- proxyTargetClass，boolen 变量，指明是代理一个接口还是代理整个目标类，默认为 false，代理一个接口。如果是代理接口，则只是代理接口中定义的方法，代理目标类即代理目标对象所属类的所有方法。如果前面的 interfaces 属性没有指定，也没有激活 autodetection 那就代理整个目标类。

## ProxyFactoryBean 使用示例

### 定义要拦截的接口

```java
public interface AService {
    public void fooA(String _msg);  
    public void barA(); 
}
```

### 定义目标对象类

```java
public class AServiceImpl implements AService{
    @Override
    public void fooA(String _msg) {
         System.out.println("AServiceImpl.fooA(msg:"+_msg+")");
    }
    @Override
    public void barA() {
         System.out.println("AServiceImpl.barA()");  
    }
 
```

目标对象可以不止实现了一个接口。

### 定义通知

定义一个前置通知，在拦截点之前打印一串字符。

```java
public class MyBeforeAdvice implements MethodBeforeAdvice{
    @Override
    public void before(Method method, Object[] args, Object target)
            throws Throwable {
        System.out.println("run my before advice");
    }
}
```

### 通过配置将它们组装起来

```xml
<!-- 目标对象 -->
<bean id="aServiceImpl" class="com.lg.aop.service.impl.AServiceImpl"/>
<!-- 通知 -->
<bean id="myBeforAdvice" class="com.lg.aop.MyBeforeAdvice"/>
<!-- ProxyFactoryBean -->
<bean id="aServiceImplProxy"     class="org.springframework.aop.framework.ProxyFactoryBean">
    <!-- 指定要拦截的接口，即切入点 -->
    <property name="interfaces" value="com.lg.aop.service.AService"/>
    <!-- 指定目标 bean -->
    <property name="target">
        <ref bean="aServiceImpl"/>
    </property>
    <!-- 指定通知 -->
    <property name="interceptorNames">  
        <list>  
            <value>myBeforAdvice</value>  
        </list>  
    </property>  
</bean>
```

### 然后就可以使用了

```java
@Autowired
private AServiceImplProxy aServiceImplProxy;
@Test
public void testAOP(){
    aService.barA();
}
```

实际上我们使用的是一个名为 aServiceImplProxy 的 ProxyFactoryBean 对象。

@Autowired 是按类型注入，我们还可以按名称注入。方法如下：

```java
@Resource(name="aServiceImplProxy")
private AService aService;
@Test
public void testAOP(){
    aService.barA();
}
```

执行结果如下：

```
run my before advice
AServiceImpl.barA()
```



参考：[Spring AOP源码分析(七)ProxyFactoryBean介绍](https://yq.aliyun.com/articles/38890)。感谢这篇博文，正是看了这篇文章我才开始理解厘清了一些概念术语。

