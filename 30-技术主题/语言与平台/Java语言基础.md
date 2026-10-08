## 多线程/高并发
+ 并发：同一时刻，多个任务交替执行
+ 并行：同一时刻，多个任务同时进行



+ 进程：运行中的程序，有一个 main 线程。
+ 线程：由进程创建，是进程的实体；一个进程可以有多个线程。一个线程可以开启其他线程。



+ 用户线程：工作线程，任务执行完成后终止，或通知终止
+ 守护线程：一般是为工作线程服务的；当所有用户线程结束，守护线程自动结束。  
GC 是最常见的守护线程。

### 线程基础
#### 线程的生命周期
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/gif/43050341/1758335698881-2381a1dd-e925-4d76-b5cd-5779a3473018.gif)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/43050341/1758335826600-c862567d-4568-49ab-ab27-b538c6bfe40b.png)

#### 线程同步
在多线程编程中，一些敏感数据不允许被多个线程同时访问，此时就需要使用同步机制（互斥锁），保证在任何同一时刻，最多有一个线程操作敏感数据，以保证数据完整性。同步会减低程序执行效率。

+ 非静态同步方法：互斥锁是 this 或其他对象（要求是同一个对象）
+ 静态同步方法：互斥锁是当前类本身
+ 同步代码块：在非静态方法中，互斥锁是 this 或其他对象（要求是同一个对象）；在静态方法中，互斥锁是当前类本身（当前类.class）

**死锁：**多个线程占用了对方的锁资源，不肯相让，导致多个线程无法解锁。

**释放锁：**

1. 同步方法、同步代码块执行结束。
2. break、return
3. 出现未处理的 Error 或 Exception，导致同步方法异常结束。
4. 执行了线程对象.wait()，暂停该线程并释放锁。

**不释放锁：**

1. Thread.sleep()、Thread.yield()：暂停执行但不释放锁
2. 线程对象.suspend()：挂起该线程但不释放锁

提示：suspend()、resume() 被废弃了。

```java
// 1. 同步代码块：持有对象的锁，才能操作同步代码块
synchronized(this / 其他对象 / 当前类.class) {
	// 需要同步的代码
}

// 2. 同步方法
public synchronized void func(){
	// 需要同步的代码
}


```

#### 线程的使用
每一个线程类对象就是一个线程。线程类有下面两种实现方式：

1. 实现 Runable 接口，重写 run()。  
更适合多个线程共享一个线程类的资源，且避免了单继承的限制。（建议使用）
2. 继承 Thread 类，重写 run()。（Thread 也是实现了 Runable 接口）

```java
// 实现 Runable 接口，重写 run()
public class ThreadLearning implements Runnable {
	// 控制线程终止
	private boolean loop = true
	
    @Override
    public void run() { // 实现业务逻辑
        
    }

	// 设置 loop 的值
	public void setLoop(boolean loop) {
		this.loop = loop;
	}
}
// 开启子线程
ThreadLearning tl = new ThreadLearning()
Thread thread = new Thread(tl)
thread.start() // 启动子线程
tl.setLoop(false) // 通知线程终止
```

```java
// 继承 Thread 类，重写 run()
public class ThreadLearning extends Thread {
    @Override
    public void run() { // 实现业务逻辑

    }
}

// 开启多线程。threadLearning.start() 会额外开启一个线程来执行 ThreadLearning 的 run()
// 直接执行 threadLearning.run() 不会额外开启一个线程，而是 main 线程执行 ThreadLearning 的 run()
ThreadLearning tl = new ThreadLearning()
tl.start() // 启动线程
```

```java
ThreadLearning tl = new ThreadLearning()
tl.setName("自定义线程名称")
tl.getName()
tl.setPriority(int 线程优先级)
tl.getPriority()
// 中断线程：并不是终止线程。主要是中断线程的休眠
tl.interrupt()
// 线程休眠 xxx 毫秒
tl.sleep(1000)
// 线程插队，插队成功就先执行插队线程的业务
tl.join()
// 将线程设置为守护线程
tl.setDaemon(true)

// 获取当前线程
Thread.currentThread()
// 线程礼让：当前线程让出cpu，让其他线程执行，但礼让不一定成功，保持礼让的时间也不固定
Thread.yield()
```

#### Jconsole 监控线程
```shell
// 终端中输入 jconsole 直接回车
jconsole
```

## 网络编程
java.net 包下提供了一系列的类和接口以供使用，完成网络通信。

### 网络相关概念
**ip 地址：**唯一标识网络中的每一台主机

IPV4：点分十进制，4 段 4 字节；0~2^8-1；127.0.0.1（本机地址）

IPV6：冒分十六进制，8 段 16 字节；0~2^16-1；  
		  2002:0db8:85a3:0000:0000:8a2e:0370:7334   
		（连续的全零组可以用 :: 代替，但只能使用一次）

**域名 www.xxx.xx**：将 ip 地址映射成域名，方便记忆。HTTP 协议规定了如何映射。

**端口号**：用于标识主机上特定的程序或服务；16 位数字；0~2^16-1；0~1024 已被使用；

**ipconfig：**查看主机的 ip 地址

**netstat -an | more：** 分页显示当前主机的网络情况，包括协议、本地地址（服务器）、外部地址（客户端）、状态。一个外部程序连接到该端口，就会显示一条信息。

### TCP 和 UDP
**TCP：传输控制协议**

1. 建立 TCP 连接，形成数据传输通道
2. 传输前，采用可靠的“三次握手”认证。  
三次握手：1. 主机 A ->“B，你能接收到吗？”-> 主机 B  
		  2. 主机 B ->“A，我能接收到，你能接收到吗？”-> 主机 A  
		  3. 主机 A ->“B，我也能接收到，下面可以进行大量数据的传输了”-> 主机 B

**UDP：用户数据协议**

1. 将数据封装成数据包，不需要建立连接，不可靠但速度快
2. 每一个数据包的大小限制在 64KB 以内，不适合传输大量数据

### Socket
+ 通信的两端都要有 Socket，Socket 是主机间通信的端点
+ 网络通信就是 Socket 间的通信
+ 读写数据：socket.getOutputStream()向通信管道中写入数据、socket.getInputStream()从通信管道中读取数据

### 基于 Socket 的 TCP 网络通信编程
```java
public class Server {
    public static void main(String[] args) throws IOException {
        new Server().server();
    }

    public void server() throws IOException {
        // 1. 服务器监听9999端口，等待客户端连接；没有客户端连接9999端口，服务器会阻塞，等待客户端连接；当客户端连接成功后，服务端会返回Socket对象，服务器继续运行。
        ServerSocket serverSocket = new ServerSocket(9999);
        System.out.println("等待客户端连接");
        Socket clientSocket = serverSocket.accept();
        // 2. 获取 OutputStream 对象，发送数据给客户端
        new FileInputStream("");

        OutputStream outputStream = clientSocket.getOutputStream();
        outputStream.write("hello, client".getBytes());
        outputStream.flush();
        clientSocket.shutdownOutput();
        // 3. 获取 InputStream 对象，接收客户端的数据
        InputStream inputStream = clientSocket.getInputStream();
        byte[] buf = new byte[1024];
        int length = inputStream.read(buf);
        System.out.println("接收de客户端数据：" + new String(buf, 0, length));
        // 4.关闭流对象、客户端Socket对象、ServerSocket对象
        inputStream.close();
        outputStream.close();
        clientSocket.close();
        // 5. 关闭服务器
        serverSocket.close();
        System.out.println("服务端关闭");
    }
}
```

```java
public class Client {
    public static void main(String[] args) throws IOException {
        new Client().client();
    }

    public void client() throws IOException {
        // 1. 连接服务器，连接成功则返回Socket对象
        Socket clientSocket = new Socket("127.0.0.1", 9999);
        System.out.println("连接服务器成功");
        // 2. 获取 OutputStream 对象，发送数据给服务器
        OutputStream outputStream = clientSocket.getOutputStream();
        outputStream.write("hello, server".getBytes());
        outputStream.flush();
        clientSocket.shutdownOutput();
        // 3. 获取 InputStream 对象，接收服务器的数据
        InputStream inputStream = clientSocket.getInputStream();
        byte[] buf = new byte[1024];
        int length = inputStream.read(buf);
        System.out.println("接收de服务器数据：" + new String(buf, 0, length));
        // 4. 关闭流对象和Socket对象
        outputStream.close();
        inputStream.close();
        clientSocket.close();
        System.out.println("客户端退出");
    }
}
```

