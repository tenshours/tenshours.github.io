---
layout: post
title: "Spring AOP"
date: 2019-03-29
Tag: 
- Spring AOP
---

#### 静态代理

{% highlight css %}
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
{% highlight css %}

静态代理缺点：如果一旦类似于Hello需要代理的对象太多，代码量太大，维护成本高。

#### JDK 动态代理

