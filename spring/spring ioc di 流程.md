spring ioc di 流程

1. 解析xml中配置，将配置构建成dom树。
2. 将dom树进行解析，构建beanDefinition
3. 在构建beanDefinition过程中如果有bean依赖的情况，会先调用getBean进行构建或者在缓存中直接获取。
4. 在构建bean的过程中会调用postProcessor

spring aop

在ioc构建过程中创建wrapper。会判断是否实现接口。如果实现接口会创建jdkAOPproxy，否则会创建cglibAopProxy。

jdkproxy 根据Proxy.newProxyInstantce构建代理类。将对象包裹起来。调用inoke方法从而调用被代理的对象的方法。

cglib是asm字节码增强技术，构建新的字节码类。首先jvm根据字节码一次生成到处运行的思想。所以代码并不重要。asm通过对字节码增强来实现对对象进行代理的操作。



