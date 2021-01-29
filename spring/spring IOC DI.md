spring IOC

“IoC容器的初始化包括BeanDefinition的Resouce定位、载入和注册这三个基本的过程”



DI 

在getBean的过程进行的。 在AbstractBeanFactory的doGetBean方法中实现。会循环Bean所依赖的对象并递归getBean方法创建。

