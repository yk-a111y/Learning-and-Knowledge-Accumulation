#### defineProperty 和 Proxy的区别
1. 性能方面：defineProperty是将对象中每一个属性的拦截，当对象层级过深，深度遍历耗时。Proxy是创建整个对象的代理，无需遍历每一个属性。
2. 功能方面：defineProperty无法实现对属性删除等操作的拦截，proxy可以。