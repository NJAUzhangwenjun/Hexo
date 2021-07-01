---
title: Spring源码-生命周期
top: false
cover: true
author: 张文军
date: 2021-06-29 13:04:41
tags: 
 - Spring
 - Spring源码
category: 
 - Spring源码
 - Spring
summary: Spring源码-生命周期
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

## Spring源码架构简单梳理



### 1. Spring核心：IOC和AOP

![image-20210629042908089](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025507.png)





### 2. Spring Bean的加载流程图解

![image-20210629044605747](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025508.png)

![image-20210629044033175](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025509.png)



## Spring启动加载Bean核心方法

### 1. refresh()

> org.springframework.context.support.AbstractApplicationContext#refresh

```java
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

			// Prepare this context for refreshing.
			/**
			 * 1、设置容器的启动时间
			 * 2、设置活跃状态为true
			 * 3、设置关闭状态为false
			 * 4、获取Environment对象， 并加载当前系统的属性值到Environment对象中
			 * 5、准备监听器和事件的集合对象，默认为空的集合
			 */
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			/**
			 * 创建容器对象: DefaultListabLeBeanFactory
			 * 加载xm配置文件的属性值到当前工厂中，最重要的就是BeanDefinition 
			 * 即：创建Beanfactory->读取Bean的配置信息->解析成BeanDefinition对象，存入Map
			 */
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			// beanFactory 准备工作，给各种属性初始化赋值
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				// 子类覆盖方法做额外的处理，此处我们自己一般不做任何扩展 工作，但是可以查看web中的代码，是有具体实现的
				postProcessBeanFactory(beanFactory);

				StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
				// Invoke factory processors registered as beans in the context.
				// 调用BeanFactory的各种后置处理器：对BeanDefinition增强、修改、替换
				invokeBeanFactoryPostProcessors(beanFactory);


				/**---------------------为Bean的实例化和初始化做准备工作-----开始----------------------------------*/
				

				// Register bean processors that intercept bean creation.
				// 注册bean的后置处理器，这里只是注册功能，为Bean的初始化做准备工作，真正调用的是getBean方法
				registerBeanPostProcessors(beanFactory);
				beanPostProcess.end();

				// Initialize message source for this context.
				// 为上下文初始化message源，即不同语言的消息体，用来做国际化
				initMessageSource();

				// Initialize event multicaster for this context.
				// 初始化事件监听多路广播器
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				// 注册监听器：在所有注册的bean中查找listener bean, 注册到消息广播器中
				registerListeners();

				/**---------------------为Bean的实例化和初始化做准备工作-----结束----------------------------------*/

				// Instantiate all remaining (non-lazy-init) singletons.
				// 实例化所有剩余的（非延迟初始化）单例
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
				contextRefresh.end();
			}
		}
	}
```





### 2. preInstantiateSingletons()

> org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons
>
> > 实例化和初始化

```java
/**
 * [preInstantiateSingletons 实例化和初始化所有单例对象]
 * @throws BeansException [description]
 */
public void preInstantiateSingletons() throws BeansException {
		if (logger.isTraceEnabled()) {
			logger.trace("Pre-instantiating singletons in " + this);
		}

		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
		// 将所有BeanDefinition的名字拷贝到一个集合中
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

		// Trigger initialization of all non-lazy singleton beans...
		// 触发所有非懒加载的单例对象的实例化
		for (String beanName : beanNames) {
			/*获取合并BeanDefinition后的BeanDefinition对象*/
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				/*判断当前Bean是否实现了FactoryBean接口*/
				if (isFactoryBean(beanName)) {
					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
					if (bean instanceof FactoryBean) {
						/*进行类型转换*/
						FactoryBean<?> factory = (FactoryBean<?>) bean;
					
						boolean isEagerInit;
						if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
							isEagerInit = AccessController.doPrivileged(
									(PrivilegedAction<Boolean>) ((SmartFactoryBean<?>) factory)::isEagerInit,
									getAccessControlContext());
						}
						else {
							isEagerInit = (factory instanceof SmartFactoryBean &&
									((SmartFactoryBean<?>) factory).isEagerInit());
						}
						/*判断是否需要立即初始化*/
						if (isEagerInit) {
							/*进行初始化*/
							getBean(beanName);
						}
					}
				}

				/*如果beanName对应的bean不是FactoryBean, 只是普通的bean,通过beanName获取bean实例*/
				/*即：当前Bean是首次进行实例化和初始化*/
				else {
					getBean(beanName);
				}
			}
		}

		// Trigger post-initialization callback for all applicable beans...
		for (String beanName : beanNames) {
			Object singletonInstance = getSingleton(beanName);
			if (singletonInstance instanceof SmartInitializingSingleton) {
				StartupStep smartInitialize = this.getApplicationStartup().start("spring.beans.smart-initialize")
						.tag("beanName", beanName);
				SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
				if (System.getSecurityManager() != null) {
					AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
						smartSingleton.afterSingletonsInstantiated();
						return null;
					}, getAccessControlContext());
				}
				else {
					smartSingleton.afterSingletonsInstantiated();
				}
				smartInitialize.end();
			}
		}
	}
```

### 3. getBean()

> org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean

```java
/**
 * [doGetBean 此方法是实际获取bean的方法，也是触发依赖注入的方法]
 * @param  name           [description]
 * @param  requiredType   [description]
 * @param  args           [description]
 * @param  typeCheckOnly  [description]
 * @return                [description]
 * @throws BeansException [description]
 */
protected <T> T doGetBean(
		String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)
		throws BeansException {

	String beanName = transformedBeanName(name);
	Object beanInstance;

	// Eagerly check singleton cache for manually registered singletons.
	// 检查缓存中是否已经有手动注册了该Bean
	Object sharedInstance = getSingleton(beanName);
	if (sharedInstance != null && args == null) {
		if (logger.isTraceEnabled()) {
			if (isSingletonCurrentlyInCreation(beanName)) {
				logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
						"' that is not fully initialized yet - a consequence of a circular reference");
			}
			else {
				logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
			}
		}
		beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, null);
	}

	else {
		// Fail if we're already creating this bean instance:
		// We're assumably within a circular reference.
		if (isPrototypeCurrentlyInCreation(beanName)) {
			throw new BeanCurrentlyInCreationException(beanName);
		}

		// Check if bean definition exists in this factory.
		// 检查当前BeanDefinition是否在当前的BeanFactory中，并且当前benaDefinition不是抽象的，即parentBeanDefinition
		BeanFactory parentBeanFactory = getParentBeanFactory();
		if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
			// Not found -> check parent.
			String nameToLookup = originalBeanName(name);
			if (parentBeanFactory instanceof AbstractBeanFactory) {
				return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
						nameToLookup, requiredType, args, typeCheckOnly);
			}
			else if (args != null) {
				// Delegation to parent with explicit args.
				return (T) parentBeanFactory.getBean(nameToLookup, args);
			}
			else if (requiredType != null) {
				// No args -> delegate to standard getBean method.
				return parentBeanFactory.getBean(nameToLookup, requiredType);
			}
			else {
				return (T) parentBeanFactory.getBean(nameToLookup);
			}
		}

		if (!typeCheckOnly) {
			markBeanAsCreated(beanName);
		}

		StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate")
				.tag("beanName", name);
		try {
			if (requiredType != null) {
				beanCreation.tag("beanType", requiredType::toString);
			}
			/*获取合并后的BeanDefinition*/
			RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
			/*检查BeanDefinition*/
			checkMergedBeanDefinition(mbd, beanName, args);

			// Guarantee initialization of beans that the current bean depends on.
			// 保证当前 bean 所依赖的 bean 的初始化
			// 即：实例化和初始化依赖对象
			String[] dependsOn = mbd.getDependsOn();
			if (dependsOn != null) {
				for (String dep : dependsOn) {
					if (isDependent(beanName, dep)) {
						throw new BeanCreationException(mbd.getResourceDescription(), beanName,
								"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
					}
					registerDependentBean(dep, beanName);
					try {
						/*递归调用实例化和初始化依赖Bean*/
						getBean(dep);
					}
					catch (NoSuchBeanDefinitionException ex) {
						throw new BeanCreationException(mbd.getResourceDescription(), beanName,
								"'" + beanName + "' depends on missing bean '" + dep + "'", ex);
					}
				}
			}

			// Create bean instance.
			// 创建Bean的实例
			
			// 单例
			if (mbd.isSingleton()) {
				sharedInstance = getSingleton(beanName, () -> {
					try {
						/*创建单例对象*/
						return createBean(beanName, mbd, args);
					}
					catch (BeansException ex) {
						// Explicitly remove instance from singleton cache: It might have been put there
						// eagerly by the creation process, to allow for circular reference resolution.
						// Also remove any beans that received a temporary reference to the bean.
						destroySingleton(beanName);
						throw ex;
					}
				});
				beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
			}

			else if (mbd.isPrototype()) {
				// It's a prototype -> create a new instance.
				Object prototypeInstance = null;
				try {
					beforePrototypeCreation(beanName);
					prototypeInstance = createBean(beanName, mbd, args);
				}
				finally {
					afterPrototypeCreation(beanName);
				}
				beanInstance = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
			}

			else {
				String scopeName = mbd.getScope();
				if (!StringUtils.hasLength(scopeName)) {
					throw new IllegalStateException("No scope name defined for bean ´" + beanName + "'");
				}
				Scope scope = this.scopes.get(scopeName);
				if (scope == null) {
					throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
				}
				try {
					Object scopedInstance = scope.get(beanName, () -> {
						beforePrototypeCreation(beanName);
						try {
							return createBean(beanName, mbd, args);
						}
						finally {
							afterPrototypeCreation(beanName);
						}
					});
					beanInstance = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
				}
				catch (IllegalStateException ex) {
					throw new ScopeNotActiveException(beanName, scopeName, ex);
				}
			}
		}
		catch (BeansException ex) {
			beanCreation.tag("exception", ex.getClass().toString());
			beanCreation.tag("message", String.valueOf(ex.getMessage()));
			cleanupAfterBeanCreationFailure(beanName);
			throw ex;
		}
		finally {
			beanCreation.end();
		}
	}

	return adaptBeanInstance(name, beanInstance, requiredType);
}
```

### 4. doGetBean()

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType, @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
		...
		// Eagerly check singleton cache for manually registered singletons.
		// 先去获取一次，如果不为null，此处就会走缓存了~~
		Object sharedInstance = getSingleton(beanName);
		...
		// 如果不是只检查类型，那就标记这个Bean被创建了~~添加到缓存里 也就是所谓的  当前创建Bean池
		if (!typeCheckOnly) {
			markBeanAsCreated(beanName);
		}
		...
		// Create bean instance.
		if (mbd.isSingleton()) {
		
			// 这个getSingleton方法不是SingletonBeanRegistry的接口方法  属于实现类DefaultSingletonBeanRegistry的一个public重载方法~~~
			// 它的特点是在执行singletonFactory.getObject();前后会执行beforeSingletonCreation(beanName);和afterSingletonCreation(beanName);  
			// 也就是保证这个Bean在创建过程中，放入正在创建的缓存池里  可以看到它实际创建bean调用的是我们的createBean方法~~~~
			sharedInstance = getSingleton(beanName, () -> {
				try {
					return createBean(beanName, mbd, args);
				} catch (BeansException ex) {
					destroySingleton(beanName);
					throw ex;
				}
			});
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
		}
	}
```



### 5. doCreateBean()：实例化和初始化

> :实例化和初始化

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean()

> 其中最重要的执行逻辑调用了三个方法：
>
> 1. 实例化Bean —–> *createBeanInstance()*
> 2. 属性填充  —> *populateBean()*
> 3. 执行初始化逻辑->执行AwareMethods和Bean后置处理器 —>  *initializeBean()*

```java
/**
 * [doCreateBean 实例化和初始化]
 * @param  beanName              [description]
 * @param  mbd                   [description]
 * @param  args                  [description]
 * @return                       [description]
 * @throws BeanCreationException [description]
 */
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args) throws BeanCreationException {
		...
		// 创建Bean对象，并且将对象包裹在BeanWrapper 中
		instanceWrapper = createBeanInstance(beanName, mbd, args);
		// 再从Wrapper中把Bean原始对象（非代理~~~）  这个时候这个Bean就有地址值了，就能被引用了~~~
		// 注意：此处是原始对象，这点非常的重要
		final Object bean = instanceWrapper.getWrappedInstance();
		...
		// earlySingletonExposure 用于表示是否”提前暴露“原始对象的引用，用于解决循环依赖。
		// 对于单例Bean，该变量一般为 true   但可以通过属性allowCircularReferences = false来关闭循环引用
		// isSingletonCurrentlyInCreation(beanName) 表示当前bean必须在创建中才行
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences && isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName + "' to allow for resolving potential circular references");
			}
			// 上面讲过调用此方法放进一个ObjectFactory，二级缓存会对应删除的
			// getEarlyBeanReference的作用：调用SmartInstantiationAwareBeanPostProcessor.getEarlyBeanReference()这个方法  否则啥都不做
			// 也就是给调用者个机会，自己去实现暴露这个bean的应用的逻辑~~~
			// 比如在getEarlyBeanReference()里可以实现AOP的逻辑~~~  参考自动代理创建器AbstractAutoProxyCreator  实现了这个方法来创建代理对象
			// 若不需要执行AOP的逻辑，直接返回Bean
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}
		Object exposedObject = bean; //exposedObject 是最终返回的对象
		...
		// 填充属于，解决@Autowired依赖~
		populateBean(beanName, mbd, instanceWrapper);
		// 执行初始化回调方法们~~~
		exposedObject = initializeBean(beanName, exposedObject, mbd);
		
		// earlySingletonExposure：如果你的bean允许被早期暴露出去 也就是说可以被循环引用  那这里就会进行检查
		// 此段代码非常重要
		if (earlySingletonExposure) {
			// 此时一级缓存肯定还没数据，但是呢此时候二级缓存earlySingletonObjects也没数据
			//注意，注意：第二参数为false  表示不会再去三级缓存里查了~~~

			// 此处非常巧妙的一点：：：因为上面各式各样的实例化、初始化的后置处理器都执行了，如果你在上面执行了这一句
			//  ((ConfigurableListableBeanFactory)this.beanFactory).registerSingleton(beanName, bean);
			// 那么此处得到的earlySingletonReference 的引用最终会是你手动放进去的Bean最终返回，完美的实现了"偷天换日" 特别适合中间件的设计
			// 我们知道，执行完此doCreateBean后执行addSingleton()  其实就是把自己再添加一次  **再一次强调，完美实现偷天换日**
			Object earlySingletonReference = getSingleton(beanName, false);
			if (earlySingletonReference != null) {
			
				// 这个意思是如果经过了initializeBean()后，exposedObject还是木有变，那就可以大胆放心的返回了
				// initializeBean会调用后置处理器，这个时候可以生成一个代理对象，那这个时候它哥俩就不会相等了 走else去判断吧
				if (exposedObject == bean) {
					exposedObject = earlySingletonReference;
				} 

				// allowRawInjectionDespiteWrapping这个值默认是false
				// hasDependentBean：若它有依赖的bean 那就需要继续校验了~~~(若没有依赖的 就放过它~)
				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
					// 拿到它所依赖的Bean们~~~~ 下面会遍历一个一个的去看~~
					String[] dependentBeans = getDependentBeans(beanName);
					Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
					
					// 一个个检查它所以Bean
					// removeSingletonIfCreatedForTypeCheckOnly这个放见下面  在AbstractBeanFactory里面
					// 简单的说，它如果判断到该dependentBean并没有在创建中的了的情况下,那就把它从所有缓存中移除~~~  并且返回true
					// 否则（比如确实在创建中） 那就返回false 进入我们的if里面~  表示所谓的真正依赖
					//（解释：就是真的需要依赖它先实例化，才能实例化自己的依赖）
					for (String dependentBean : dependentBeans) {
						if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
							actualDependentBeans.add(dependentBean);
						}
					}

					// 若存在真正依赖，那就报错（不要等到内存移除你才报错，那是非常不友好的） 
					// 这个异常是BeanCurrentlyInCreationException，报错日志也稍微留意一下，方便定位错误~~~~
					if (!actualDependentBeans.isEmpty()) {
						throw new BeanCurrentlyInCreationException(beanName,
								"Bean with name '" + beanName + "' has been injected into other beans [" +
								StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
								"] in its raw version as part of a circular reference, but has eventually been " +
								"wrapped. This means that said other beans do not use the final version of the " +
								"bean. This is often the result of over-eager type matching - consider using " +
								"'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
					}
				}
			}
		}
		
		return exposedObject;
	}

	// 虽然是remove方法 但是它的返回值也非常重要
	// 该方法唯一调用的地方就是循环依赖的最后检查处~~~~~
	protected boolean removeSingletonIfCreatedForTypeCheckOnly(String beanName) {
		// 如果这个bean不在创建中  比如是ForTypeCheckOnly的  那就移除掉
		if (!this.alreadyCreated.contains(beanName)) {
			removeSingleton(beanName);
			return true;
		}
		else {
			return false;
		}
	}

}
```

### 6. createBeanInstance()：实例化Benan



```java
/**
 * [createBeanInstance 实例化Benan]
 * @param  beanName [description]
 * @param  mbd      [description]
 * @param  args     [description]
 * @return          [description]
 */
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
		// Make sure bean class is actually resolved at this point.
		Class<?> beanClass = resolveBeanClass(mbd, beanName);

		if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
			throw new BeanCreationException(mbd.getResourceDescription(), beanName,
					"Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
		}

		Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
		if (instanceSupplier != null) {
			return obtainFromSupplier(instanceSupplier, beanName);
		}

		if (mbd.getFactoryMethodName() != null) {
			return instantiateUsingFactoryMethod(beanName, mbd, args);
		}

		// Shortcut when re-creating the same bean...
		boolean resolved = false;
		boolean autowireNecessary = false;
		if (args == null) {
			synchronized (mbd.constructorArgumentLock) {
				if (mbd.resolvedConstructorOrFactoryMethod != null) {
					resolved = true;
					autowireNecessary = mbd.constructorArgumentsResolved;
				}
			}
		}
		if (resolved) {
			if (autowireNecessary) {
				return autowireConstructor(beanName, mbd, null, null);
			}
			else {
				return instantiateBean(beanName, mbd);
			}
		}

		// Candidate constructors for autowiring?
		Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
		if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
				mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
			return autowireConstructor(beanName, mbd, ctors, args);
		}

		// Preferred constructors for default construction?
		ctors = mbd.getPreferredConstructors();
		if (ctors != null) {
			return autowireConstructor(beanName, mbd, ctors, null);
		}

		// No special handling: simply use no-arg constructor.
		return instantiateBean(beanName, mbd);
	}
```



### 7. populateBean()：属性填充

> : 属性填充(不会执行Aware方法)

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean()

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
   if (bw == null) {
      if (mbd.hasPropertyValues()) {
         throw new BeanCreationException(
               mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
      }
      else {
         // Skip property population phase for null instance.
         return;
      }
   }

   // Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
   // state of the bean before properties are set. This can be used, for example,
   // to support styles of field injection.
   if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
      for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
         if (!bp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
            return;
         }
      }
   }

   PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

   int resolvedAutowireMode = mbd.getResolvedAutowireMode();
   if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
      MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
      // Add property values based on autowire by name if applicable.
      if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
         autowireByName(beanName, mbd, bw, newPvs);
      }
      // Add property values based on autowire by type if applicable.
      if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
         autowireByType(beanName, mbd, bw, newPvs);
      }
      pvs = newPvs;
   }

   boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
   boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

   PropertyDescriptor[] filteredPds = null;
   if (hasInstAwareBpps) {
      if (pvs == null) {
         pvs = mbd.getPropertyValues();
      }
      for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
         PropertyValues pvsToUse = bp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
         if (pvsToUse == null) {
            if (filteredPds == null) {
               filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
            }
            pvsToUse = bp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
               return;
            }
         }
         pvs = pvsToUse;
      }
   }
   if (needsDepCheck) {
      if (filteredPds == null) {
         filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
      }
      checkDependencies(beanName, mbd, filteredPds, pvs);
   }

   if (pvs != null) {
       /**
       *属性填充
       */
      applyPropertyValues(beanName, mbd, bw, pvs);
   }
}
```

### 8 . initializeBean()

> :执行初始化逻辑->执行AwareMethods和Bean后置处理器

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean()

```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
   if (System.getSecurityManager() != null) {
      AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
         invokeAwareMethods(beanName, bean);
         return null;
      }, getAccessControlContext());
   }
   else {
       //执行Aware方法，给相应属性赋值
      invokeAwareMethods(beanName, bean);
   }

   Object wrappedBean = bean;
   if (mbd == null || !mbd.isSynthetic()) {
       // 执行Bean的处理器前置方法
      wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
   }

   try {
       //执行用户自定义的 init-method方法
      invokeInitMethods(beanName, wrappedBean, mbd);
   }
   catch (Throwable ex) {
      throw new BeanCreationException(
            (mbd != null ? mbd.getResourceDescription() : null),
            beanName, "Invocation of init method failed", ex);
   }
   if (mbd == null || !mbd.isSynthetic()) {
       // 执行Bean的处理器后置方法
      wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
   }

   return wrappedBean;
}
```



## Spring循环依赖

###  1. Spring循环依赖概述

> 在一个对象依赖关系中，A对象依赖B对象，B对象依赖A对象；造成循环依赖链

![image-20210630165136629](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025510.png)

### 2. 简单模仿解决循环引用：

```java
package cn.zhanghub.spring.circularDependency.circularDependency;

import lombok.Getter;
import lombok.SneakyThrows;

import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;

/**
 * 类描述：
 *
 * @ClassName Imitate
 * @Author 张文军
 * @Date 2021/7/1 0:00
 * @Version 1.0
 */
public class Imitate {
    /**
     * 放置创建好的bean Map,模拟单例池 
     */
    private static Map<String, Object> cacheMap = new HashMap<>(2);

    public static void main(String[] args) {
        // 假装扫描出来的对象       
        Class[] classes = {A.class, B.class};
        // 假装项目初始化实例化所有bean
        for (Class aClass : classes) {
            getBean(aClass);
        }
        // check
        System.out.println(getBean(B.class).getA() == getBean(A.class));
        System.out.println(getBean(A.class).getB() == getBean(B.class));
    }

    @SneakyThrows
    private static <T> T getBean(Class<T> beanClass) {
        // 类名小写简单代替bean的命名规则
        String beanName = beanClass.getSimpleName().toLowerCase();
        // 如果已经是一个bean，则直接返回
        if (cacheMap.containsKey(beanName)) {
            return (T) cacheMap.get(beanName);
        }
        // 将对象本身实例化
        Object object = beanClass.getDeclaredConstructor().newInstance();
        // 放入缓存
        cacheMap.put(beanName, object);
        // 把所有字段当成需要注入的bean，创建并注入到当前bean中
        Field[] fields = object.getClass().getDeclaredFields();
        for (Field field : fields) {
            field.setAccessible(true);
            // 获取需要注入字段的class            
            Class<?> fieldClass = field.getType();
            String fieldBeanName = fieldClass.getSimpleName().toLowerCase();
            // 如果需要注入的bean，已经在缓存Map中，那么把缓存Map中的值注入到该field即可            
            // 如果缓存没有 继续创建            
            field.set(object, cacheMap.containsKey(fieldBeanName) ? cacheMap.get(fieldBeanName) : getBean(fieldClass));
        }
        // 属性填充完成，返回
        return (T) object;
    }
}

class B {
    @Getter
    private A a;
}
class A {
    @Getter
    private B b;
}

```

### 3. Spring三级缓存

![image-20210701012914069](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025511.png)

```java
 /** Cache of singleton objects: bean name to bean instance. */

	// 从上至下 分表代表这“三级缓存”
	// 1、一级缓存放完整Bean对象
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256); //一级缓存
	// 2、二级缓存放Bean的早期暴露对象，即未完成初始化赋值的实例
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16); // 二级缓存
	// 3、放lambda表达式，来完成代理对象的覆盖过程-->即：存放Bean的原始工厂
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16); // 三级缓存
	...
	
	/** Names of beans that are currently in creation. */
	// 这个缓存也十分重要：它表示bean创建过程中都会在里面呆着~
	// 它在Bean开始创建时放值，创建完成时会将其移出~
	private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));

	/** Names of beans that have already been created at least once. */
	// 当这个Bean被创建完成后，会标记为这个 注意：这里是set集合 不会重复
	// 至少被创建了一次的  都会放进这里~~~~
	private final Set<String> alreadyCreated = Collections.newSetFromMap(new ConcurrentHashMap<>(256));
```



#### 1. getBean()方法中三级缓存

```java
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {

   Object singletonObject = this.singletonObjects.get(beanName);
   // 这个bean 正处于 创建阶段
   if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
      // 并发控制
      synchronized (this.singletonObjects) {
         // 单例缓存是否存在
         singletonObject = this.earlySingletonObjects.get(beanName);
         // 是否运行获取 bean factory 创建出的 bean
         if (singletonObject == null && allowEarlyReference) {
            // 获取缓存中的 ObjectFactory
            ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
            if (singletonFactory != null) {
               singletonObject = singletonFactory.getObject();
               // 将对象缓存到 earlySingletonObject中
               this.earlySingletonObjects.put(beanName, singletonObject);
               // 从工厂缓冲中移除
               this.singletonFactories.remove(beanName);
            }
         }
      }
   }
   return singletonObject;
```

1. 先从`一级缓存singletonObjects`中去获取。（如果获取到就直接return）
2. 如果获取不到或者对象正在创建中（`isSingletonCurrentlyInCreation()`），那就再从`二级缓存earlySingletonObjects`中获取。（如果获取到就直接return）
3. 如果还是获取不到，且允许singletonFactories（allowEarlyReference=true）通过`getObject()`获取。就从`三级缓存singletonFactory`.getObject()获取。**（如果获取到了就从**`singletonFactories`**中移除，并且放进**`earlySingletonObjects`**。其实也就是从三级缓存**`移动（是剪切、不是复制哦~）`**到了二级缓存）**

>  **加入`singletonFactories`三级缓存的前提是执行了构造器，所以构造器的循环依赖没法解决** 

#### 缓存添加过程

单例模式下，第一次获取Bean时由于Bean示例还未实例化，因此会先创建Bean然后放入缓存中，以后再次调用获取Bean方法将直接从缓存中获取，不再重新创建。Spring中，获取Bean的流程：

![image-20210701023710256](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025512.png)

 对Bean的创建最为核心三个方法解释如下：

- `createBeanInstance`：例化，其实也就是调用对象的**构造方法**实例化对象
- `populateBean`：填充属性，这一步主要是对bean的依赖属性进行注入(`@Autowired`)
- `initializeBean`：回到一些形如`initMethod`、`InitializingBean`等方法

从对`单例Bean`的初始化可以看出，循环依赖主要发生在**第二步（populateBean）**，也就是field属性注入的处理。

![image-20210701181234130](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025513.png)



缓存添加过程主要发生在创建Bean也就是doCreateBean()过程中。doCreateBean()中在实例化完后，属性填充前， 有这样一句代码

即：添加三级缓存:存放Bean 的原始工厂 `ObjectFactory`: **将创建对象的步骤封装到ObjectFactory中 交给自定义的Scope来选择是否需要创建对象来灵活的实现scope。**

```java
addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
```

`getEarlyBeanReference()`:获取早期暴露对象—>对对象进行AOP代理

即：保证自己被循环依赖的时候，即使被别的Bean @Autowire进去的也是代理对象~~~~  AOP自动代理创建器此方法里会创建的代理对象

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (SmartInstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().smartInstantiationAware) {
            exposedObject = bp.getEarlyBeanReference(exposedObject, beanName);
        }
    }
    return exposedObject;
}
```

其添加三级缓存源码如下:

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
	Assert.notNull(singletonFactory, "Singleton factory must not be null");
	synchronized (this.singletonObjects) {
		if (!this.singletonObjects.containsKey(beanName)) {
			this.singletonFactories.put(beanName, singletonFactory);
			this.earlySingletonObjects.remove(beanName);
			this.registeredSingletons.add(beanName);
		}
	}
}
```

#### 2. 三级缓存作用

第一级缓存存的是对外暴露的对象，也就是我们应用需要用到的

第二级缓存的作用是为了处理循环依赖的对象创建问题，里面存的是半成品对象或半成品对象的代理对象

第三级缓存的作用处理存在 AOP + 循环依赖的对象创建问题，能将代理对象提前创建

#### 3. Spring 为什么要引入第三级缓存

严格来讲，第三级缓存并非缺它不可，因为可以提前创建代理对象

提前创建代理对象只是会节省那么一丢丢内存空间，并不会带来性能上的提升，但是会破环 Spring 的设计原则

Spring 的设计原则是尽可能保证普通对象创建完成之后，再生成其 AOP 代理（尽可能延迟代理对象的生成）

所以 Spring 用了第三级缓存，既维持了设计原则，又处理了循环依赖；牺牲那么一丢丢内存空间是愿意接受的



### 4. Spring 循环引用创建过程

![image-20210701150204090](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025514.png)

 依旧以上面`A`、`B`类使用属性`field`注入循环依赖的例子为例，对整个流程做文字步骤总结如下：

1. 使用`context.getBean(A.class)`，旨在获取容器内的单例A(若A不存在，就会走A这个Bean的创建流程)，显然初次获取A是不存在的，因此走**A的创建之路~**
2. `实例化`A（注意此处仅仅是实例化），并将它放进`缓存`（此时A已经实例化完成，已经可以被引用了）**实例化A：-> 放入三级缓存**
3. `初始化`A：`@Autowired`依赖注入B（此时需要去容器内获取B）
4. 为了完成依赖注入B，会通过`getBean(B)`去容器内找B。但此时B在容器内不存在，就走向**B的创建之路~**
5. `实例化`B，并将其放入缓存。（此时B也能够被引用了）**实例化B：-> 放入三级缓存**
6. `初始化`B，`@Autowired`依赖注入A（此时需要去容器内获取A）**三级缓存删除A：放入二级缓存**
7. `此处重要`：初始化B时会调用`getBean(A)`去容器内找到A，上面我们已经说过了此时候因为A已经实例化完成了并且放进了缓存里，所以这个时候去看缓存里是已经存在A的引用了的，所以`getBean(A)`能够正常返回
8. **B初始化成功**（此时已经注入A成功了，已成功持有A的引用了），return（注意此处return相当于是返回最上面的`getBean(B)`这句代码，回到了初始化A的流程中~）。
9. 因为B实例已经成功返回了，因此最终**A也初始化成功**

### 5. bean的生命流程

![image-20210701180844101](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025515.png)



