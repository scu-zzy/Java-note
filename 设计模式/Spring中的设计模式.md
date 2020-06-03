# 简单工厂

spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得bean对象

# 单例

Spring下默认的bean均为singleton


# 代理

在 AOP 中，其实就是动态代理的过程。比如 Spring 中，我们自己不定义代理类，但是 Spring 会帮我们动态来定义代理，然后把我们定义在 @Before、@After、@Around 中的代码逻辑动态添加到代理中。
