# ThreadLocal<T>组件源码解析


+ ThreadLocal: 本地线程, set()在当前线程中存储数据, get()在当前线程中取出数据



## set()源码解析


```java
    public void set(T value) {
        Thread t = Thread.currentThread();  //获取当前线程
        ThreadLocalMap map = getMap(t);     //通过getMap()获取当前线程的ThreadLocalMap容器
        if (map != null) {
            map.set(this, value);           //key: 当前的ThreadLocal  value: 传进来的参数
        } else {
            createMap(t, value);            //为当前线程创建一个ThreadLocalMap容器, 并添加键值对
        }                                   //(默认情况下, ThreadLocalMap是没有初始化的)
    }
```



## get()源码解析


```java
    public T get() {
        Thread t = Thread.currentThread();                  //获取当前线程
        ThreadLocalMap map = getMap(t);                     //通过getMap()获取当前线程的ThreadLocalMap容器
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);    //获取当前ThreadaLocal的map中的entry
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;                      //获取entry的value值并返回
                return result;
            }
        }
        return setInitialValue();                           //初始化ThreadLocalMap容器
    }
```

