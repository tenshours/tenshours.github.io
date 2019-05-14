---
date: 2019-4-22
title: "Scope Bean 创建"
tags: spring
---

#### Scope Bean

```
org.springframework.context.annotation.AnnotatedBeanDefinitionReader#doRegisterBean
  definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
org.springframework.aop.scope.ScopedProxyUtils#createScopedProxy
在createScopedProx创建代理org.springframework.aop.scope.ScopedProxyFactoryBean的RootBeanDefinetion

这里向容器注册了代理的BeanDefinition
在创建bean的时候，会创建ScopedProxyFactoryBean的对象，然而ScopedProxyFactoryBean实现了BeanFactoryAware接口，调用重写的方法setBeanFactory
org.springframework.aop.scope.ScopedProxyFactoryBean#setBeanFactory
	this.proxy = pf.getProxy(cbf.getBeanClassLoader());
这里创建了scope对象的代理
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)中有方法会调用setBeanFactory
```

 