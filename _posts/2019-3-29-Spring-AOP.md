---
layout: post
title: "Spring AOP"
date: 2019-03-29
Tag: 
- Spring AOP
---

#### 静态代理

~~~ java
public interface Hello {
	void say();
}

public class World implements Hello {
	@Override
	public void say() {
		System.out.print("Hello World!");
	}
}

public class Proxy implements Hello() {
	Hello mHello;
	public Proxy(Hello pHello) {
		this.mHello = pHello;
	}
  @Override
  public void say() {
    System.out.print("Before say");	
    mHello.say();
    System.out.print("After say");	
  }
}
~~~

静态代理缺点：如果一旦类似于Hello需要代理的对象太多，代码量太大，维护成本高。

#### JDK 动态代理

```java
public class InvocationSub implements InvocationHandler {

    private Object target;

    public void setTarget(Object pTarget) {
        target = pTarget;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before invoke");
        method.invoke(target, args);
        System.out.println("After invoke");
        return null;
    }
}

public class Test {
    public static void main(String[] args) {
        World world = new World();
        InvocationSub sub = new InvocationSub();
        sub.setTarget(world);
        Hello hello = (Hello) Proxy.newProxyInstance(world.getClass().getClassLoader(), 	world.getClass().getInterfaces(), sub);
        hello.say();
    }
}
```

JDK用java.lang.reflect.InvocationHandler来实现动态代理。被代理的对象必须是实现了接口。当执行的时候，JVM就创建字节码class文件，动态的加载class，执行速度变慢，编译速度和原来一样。

#### CGLIB

```java
public class CglibProxy implements MethodInterceptor {

    Object target;

    public CglibProxy(Object pTarget) {
        target = pTarget;
    }

    @Override
    public Object intercept(Object pO, Method pMethod, Object[] pObjects, MethodProxy pMethodProxy) throws Throwable {
        System.out.println("Before invoke say");
        Object result = pMethodProxy.invoke(target, pObjects);
        System.out.println("After invoke say");
        return result;
    }
}

public class Test {
    public static void main(String[] args) {
        World world = new World();
        Enhancer enhancer = new Enhancer();
        CglibProxy proxy = new CglibProxy(world);
        enhancer.setCallback(proxy);
        enhancer.setSuperclass(world.getClass());
        Hello hello = (Hello) enhancer.create();
        hello.say();
    }
}
```

Cglib的实现，可以针对接口和具体类。在编译的时候生成class文件，执行的速度快，编译花更多时间。



Spring AOP实现的时候，就动态切换选择使用JDK的代理还是Cglib来动态代理。他们都会创建代理对象。