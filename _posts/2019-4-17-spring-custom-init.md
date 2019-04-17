---
title: "Spring Bean启动"
date: 2019-4-17
tag: spring
---



#### 基于component

~~~java
org.springframework.beans.factory.config.BeanFactoryPostProcessor

org.springframework.beans.factory.config.BeanPostProcessor

org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor
~~~



#### 基于Bean

~~~java
org.springframework.beans.factory.InitializingBean

org.springframework.context.annotation.Bean#initMethod

org.springframework.beans.factory.BeanNameAware

org.springframework.beans.factory.BeanFactoryAware

org.springframework.context.ApplicationContextAware

org.springframework.beans.factory.FactoryBean

org.springframework.beans.factory.DisposableBean
~~~



~~~python
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
 
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
      //1️⃣
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
      //执行BeanPostProcessor.postProcessBeforeInitialization
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
      //2️⃣
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
		if (mbd == null || !mbd.isSynthetic()) {
      //执行BeanPostProcessor.postProcessAfterInitialization
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}


org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods
//1️⃣执行Aware的三个子类方法
private void invokeAwareMethods(final String beanName, final Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof BeanNameAware) {
				//BeanNameAware.setBeanName
				((BeanNameAware) bean).setBeanName(beanName);
			}
			if (bean instanceof BeanClassLoaderAware) {
				ClassLoader bcl = getBeanClassLoader();
				if (bcl != null) {
					((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
				}
			}
			if (bean instanceof BeanFactoryAware) {
				//BeanFactoryAware.setBeanName
				((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
			}
		}
	}
	
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeInitMethods
//2️⃣
protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
			throws Throwable {

		boolean isInitializingBean = (bean instanceof InitializingBean);
		if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
			if (logger.isTraceEnabled()) {
				logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
			}
      //执行InitializingBean.afterPropertiesSet
			if (System.getSecurityManager() != null) {
				try {
					AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
						((InitializingBean) bean).afterPropertiesSet();
						return null;
					}, getAccessControlContext());
				}
				catch (PrivilegedActionException pae) {
					throw pae.getException();
				}
			}
			else {
				((InitializingBean) bean).afterPropertiesSet();
			}
		}
		//执行Bean的注解initMethod指定的方法
		if (mbd != null && bean.getClass() != NullBean.class) {
			String initMethodName = mbd.getInitMethodName();
			if (StringUtils.hasLength(initMethodName) &&
					!(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
					!mbd.isExternallyManagedInitMethod(initMethodName)) {
				invokeCustomInitMethod(beanName, bean, mbd);
			}
		}
	}
~~~

~~~java
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean会调用initializeBean方法。

if (instanceWrapper == null) {
		instanceWrapper = createBeanInstance(beanName, mbd, args);
}

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
Object beanInstance = doCreateBean(beanName, mbdToUse, args);
		
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#resolveBeforeInstantiation
//执行InstantiationAwareBeanPostProcessor的postProcessBeforeInstantiation和postProcessAfterInstantiation
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
		Object bean = null;
		if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
			// Make sure bean class is actually resolved at this point.
			if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
				Class<?> targetType = determineTargetType(beanName, mbd);
				if (targetType != null) {
					bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
					if (bean != null) {
						bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
					}
				}
			}
			mbd.beforeInstantiationResolved = (bean != null);
		}
		return bean;
	}
~~~



~~~java
2019-04-17 11:05:12.600  INFO 1047 --- [           main] ustomInstantiationAwareBeanPostProcessor : postProcessBeforeInstantiation(): invoked... for beanName entityThree and class is class com.gavin.mybatis.LifeCyle.EntityThree
2019-04-17 11:05:12.602  INFO 1047 --- [           main] ustomInstantiationAwareBeanPostProcessor : postProcessAfterInstantiation(): invoked... for beanName entityThree and bean is com.gavin.mybatis.LifeCyle.EntityThree@2424686b
2019-04-17 11:05:12.602  INFO 1047 --- [           main] ustomInstantiationAwareBeanPostProcessor : postProcessProperties(): invoked... for beanName entityThree and bean is com.gavin.mybatis.LifeCyle.EntityThree@2424686b
2019-04-17 11:05:12.602  INFO 1047 --- [           main] com.gavin.mybatis.LifeCyle.EntityThree   : setBeanName(): invoked... bean name is entityThree
2019-04-17 11:05:12.602  INFO 1047 --- [           main] com.gavin.mybatis.LifeCyle.EntityThree   : setBeanFactory(): invoked...
2019-04-17 11:05:12.602  INFO 1047 --- [           main] com.gavin.mybatis.LifeCyle.EntityThree   : setApplicationContext(): invoked...
2019-04-17 11:05:12.602  INFO 1047 --- [           main] c.g.m.LifeCyle.CustomBeanPostProcessor   : postProcessBeforeInitialization(): bean name is entityThree and bean is com.gavin.mybatis.LifeCyle.EntityThree@2424686b
2019-04-17 11:05:12.603  INFO 1047 --- [           main] c.g.m.LifeCyle.CustomBeanPostProcessor   : postProcessAfterInitialization(): bean name is entityThree and bean is com.gavin.mybatis.LifeCyle.EntityThree@2424686b
~~~

