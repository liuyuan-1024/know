# Listener


```plain
Listener接口是JavaWeb三大件(Filter Listener Servlet)之一, 主要用于监听事件.
监听器使用的是观察者模式
```



## JavaWeb的八大监听器


1.  ServletContextListener: 监听ServletContext对象的创建和销毁. 
2.  HttpSessionListener: 监听HttpSession对象的创建和销毁过程. 
3.  ServletRequsetListener: 监听ServletRequest对象的创建和销毁.  
1 2 3 是一组, 有创建和销毁方法: 
    - xxxInitialized(xxEvent event)
    - xxxDestroyed(xxEvent event).
    - xxx代表被监听对象
4.  ServletContextAttributeListener: 监听ServletContext对象的属性的 添加, 移除, 替换. 
5.  HttpSessionAttributeListener: 监听HttpSesssion对象的属性的 添加, 移除, 替换. 
6.  ServeltRequestAttributeListener: 监听ServletRequset对象的属性的 添加, 移除, 替换.  
4 5 6 是一组, 有 添加, 移除, 替换 获取属性名 获取属性值 获取相应的作用域: 
    - attributeAdded(ServletContextAttributeEvent\HttpSessionBindingEvent\ServeltRequestAttributeEvent event)
    - attributeRemoved(ServletContextAttributeEvent\HttpSessionBindingEvent\ServeltRequestAttributeEvent event)
    - attributeReplaced(ServletContextAttributeEvent\HttpSessionBindingEvent\ServeltRequestAttributeEvent event)
    - getName()
    - getValue()
    - getxxx()
7.  HttpSessionBindingListener: 监听某个对象在Session作用域中的创建和移除. 
    - valueBound(HttpSessionBindingEvent event): 该类实例被放入Session域中时调用
    - valueUnbound(HttpSessionBindingEvent event): 该类实例从Session域中移除时调用
    - getName(): 获取当前事件涉及的属性名
    - getValue(): 获取当前事件涉及的属性值
    - getSession(): 获取触发事件的HttpSession对象
8.  HttpSessionActivationListener: 监听某个对象在Session域中的序列化(钝化)和反序列化(活化) 
    - sessionWillPassivate(HttpSessionEvent event): 该类实例与Session一起钝化到硬盘时调用.
    - sessionDidActivate(HttpSessionEvent event): 该类实例与Session一起活化到内存时调用.

