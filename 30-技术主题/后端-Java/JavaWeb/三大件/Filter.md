# Filter


```plain
Filter接口是JavaWeb三大件(Filter Listener Servlet)之一, 主要用于过滤和拦截请求.
Filter中有三个方法:
1. 初始化: init(FilterConfig filterConfig)
2. 放行请求: doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
3. 销毁: void destroy()
```



## Filter中的doFilter方法


```java
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) {
        //放行前的操作
        
        //放行请求
        filterChain.doFilter(servletRequest, servletResponse);

        //放行后的操作, 请求放行后, 在返回途中会经过Filter
    }
```

