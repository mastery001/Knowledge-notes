# 源码
`AbstractAppicationContext#refresh`:核心点，Spring加载的全过程

1. `prepareRefresh`：前置准备；主要是加载servlet相关的配置
2. `obtainFreshBeanFactory`：创建并获取BeanFactory，并从配置文件中读取BeanDefinitions，默认实现为`DefaultListableBeanFactory`，有以下几个可配置参数
    - `setAllowBeanDefinitionOverriding`：允许同名bean被覆盖，默认为true
    - `allowCircularReferences`：允许计算循环依赖，默认为true
3. `prepareBeanFactory`：为BeanFactory注册环境变量、关联hook函数或忽略一些接口进行自动注入
4. `postProcessBeanFactory`：钩子函数，子类可以对BeanFactory再进行个性化处理
5. `invokeBeanFactoryPostProcessors`：BeanFactory的后置处理器，主要先提前解析一些注解及配置，例如Autowire等，用于处理BeanFactoryPostProcessor接口
6. `registerBeanPostProcessors`：用于处理BeanPostProcessor接口，允许在bean实例化前后对其进行修改
7. `initMessageSource`：注册MessageSource接口，用于国际化处理
8. `initApplicationEventMulticaster`：注册ApplicationEventMulticaster接口，用于事件广播
9. `onRefresh`：让自己注册一些自己所需的接口或做一些前置初始化的动作
10. `registerListeners`：注册监听器
11. `finishBeanFactoryInitialization`：初始化所有非延迟加载的bean，核心就是BeanFactory的getBean方法
12. `finishRefresh`：完成bean注册，并publish ContextRefreshed事件
13. `resetCommonCaches`：finally块执行；清除相关缓存，反射、注解、解析器、classloader等