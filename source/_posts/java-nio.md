---
title: 【java】NIO 小结
date: 2020-02-11 17:15:45
tags: [java,nio]
categories: java
---

java NIO 学习后的小结

### 1. NIO 总述

nio 为 Non-blocking io，即不阻塞io操作，java在为并发提供的 io 操作类，主要有三个核心类，分别为：

* Channel 操作数据通道
* Buffer 缓冲数据区域
* Selector 用于管理 channel

### 2. BIO 与 NIO 的主要区别

#### 2.1 面向操作
BIO 是面向流操作，NIO 是面向缓冲操作。BIO 每次从流读写一个或多个字节，直至所有字节被读写完成，该过程数据没有被缓存到其它地方，它不能前后移动流中的数据。

NIO 将数据先缓冲到稍后处理的区域，需要时可以在缓冲区前后移动，具备处理过程中的灵活性。

#### 2.2 阻塞与非阻塞
Java IO 流失阻塞的，意味着，当线程调用 read（）或 write（）时，该线程被阻塞，直到数据完全读取或者写入，期间线程无法进行其它处理。

NIO 的非阻塞模式，可以让线程请求写入一些数据到某通道，但不需要等到操作完成，这个现场同时可以去做其他事情。线程通常将非阻塞IO空闲时间用于其他通道上执行IO操作，所以一个线程可以管理多个输入、输出通道。

#### 2.3 选择器
NIO 的选择器允许一个单独线程监视多个输入通道，可以注册多个通道使用一个选择器，然后监控可以处理的输入通道进行操作。

### 3. NIO 中的 channel

#### 3.1 FileChannel
FileChannel 可以通过 `RandomAccessFile.getChannel()` 或 `InputStream,OutputStream .getChannel()` 获取，示例代码如下

```java
private void channelCopy() {
    Instant begin = Instant.now();
    try {

        RandomAccessFile source = new RandomAccessFile("./res/threeWithoutPunctuation", "r");
        RandomAccessFile target = new RandomAccessFile("./res/copyFileNio", "rw");

        ByteBuffer buffer = ByteBuffer.allocate(1024*8);
        FileChannel sourceChannel = source.getChannel();
        FileChannel targetChannel = target.getChannel();
        while (sourceChannel.read(buffer) != -1) {
            buffer.flip();
            while (buffer.hasRemaining()) {
                targetChannel.write(buffer);
            }
            buffer.clear();
        }
        sourceChannel.close();
        targetChannel.close();

    } catch (IOException e) {
        e.printStackTrace();
    }
    System.out.println("[channelCopy] Done >> " + (Duration.between(begin, Instant.now()).toMillis()) + " ms");
}
``` 

测试在小文件复制速度可能不如流操作，但在大文件拷贝速度比流复制快，测试拷贝1.03G文件，channel 耗时 1.08s，而 stream 需要 11.31s。

#### 3.2 DatagramChannel
DatagramChannel 广播包的操作，区别不大，示例代码如下：

服务端

```java
private SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault());

private void startServer() {
    try {
        DatagramChannel datagramChannel = DatagramChannel.open();
        datagramChannel.configureBlocking(true);
        datagramChannel.socket().bind(new InetSocketAddress(8000));

        System.out.println("启动服务端");
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        SocketAddress socketAddress;
        while (true) {
            //超过buffer大小部分将被丢弃
            if ((socketAddress = datagramChannel.receive(buffer)) != null) {
                buffer.flip();
                System.out.println(Charset.forName("UTF-8").decode(buffer));
                buffer.clear();
                datagramChannel.send(Charset.forName("UTF-8").encode("服务端已收到[" + format.format(new Date())), socketAddress);
            }
        }

    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

客户端

```java
public DatagramNioClient() {
    try {
        DatagramChannel datagramChannel = DatagramChannel.open();
        datagramChannel.socket().bind(new InetSocketAddress(8001));

        new Thread(() -> {
            try {
                ByteBuffer buffer = ByteBuffer.allocate(1024);
                while (true) {
                    if (datagramChannel.receive(buffer) != null) {
                        buffer.flip();
                        System.out.print("收到消息：");
                        System.out.println(Charset.forName("UTF-8").decode(buffer));
                        buffer.clear();
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();

        System.out.println("启动客户端");
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNextLine()) {
            send(datagramChannel, scanner.nextLine());
        }

    } catch (IOException e) {
        e.printStackTrace();
    }
}

private void send(DatagramChannel datagramChannel, String msg) {
    try {
        datagramChannel.send(Charset.forName("UTF-8").encode(msg), new InetSocketAddress("127.0.0.1", 8000));
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
	
需注意，如果接收数据超过设定容器的大小，超过部分会丢弃。比如对方发送了128K数据，而我容器只有48K大小，那么我只接收到48K数据，而剩余部分直接被丢弃。

对于数据的读取可以传入 ByteBuffer[] 数据，将按照顺序进行填充，对于一些固定大小数据头的数据包，使用非常方便，缺点容量一旦确定不可修改，弹性差，示例：
```java
private void start(){
	try{
	    DatagramChannel datagramChannel = DatagramChannel.open();
	    datagramChannel.connect(new InetSocketAddress("127.0.0.1",8001));
	    ByteBuffer headBuffer = ByteBuffer.allocate(48);
	    ByteBuffer bodyBuffer = ByteBuffer.allocate(128);
	    datagramChannel.read(new ByteBuffer[]{headBuffer,bodyBuffer});
	} catch (IOException e) {
        e.printStackTrace();
    }
}
```
这里的 `connect` 并非真正的建立连接，而是锁定 channel 通道，让该通道只能通过特定的地址收发数据。 

#### 3.3 SocketChannel,ServerSocketChannel

这两个 SocketChannel 是 socket 的并发版本，通常我们通过 `ServerSocket` 来监听端口，一旦有客户端连接，就创建线程进行通信，示例如下：

```java
private void server() throws IOException {
    ServerSocket serverSocket = new ServerSocket(8000);
    Socket socket;
    while ((socket = serverSocket.accept()) != null) {
        final Socket ss = socket;
        new Thread(() -> {
            try {
                InputStream in = ss.getInputStream();
                OutputStream out = ss.getOutputStream();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

每一次连接都需要创建线程进行通信，所以服务端线程数与客户端数量呈1:1关系，线程的创建需要消耗服务器资源，而服务器资源有限，在并发高且传输数据小的环境，这种方式无法满足要求。

SocketChannel 能够很好的解决高并发下的资源问题，通过 Selector 注册后，在非阻塞模式下仅使用单线程可以管理多个通道并实现数据传输，服务端示例代码如下：
```java
private void server() throws IOException {

	//建立服务端监听，设置为非阻塞模式
    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
    serverSocketChannel.configureBlocking(false);
    serverSocketChannel.bind(new InetSocketAddress(8000));

	//将 channel 注册到 selector
    Selector selector = Selector.open();
    serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

    while (true) {
		//检查是否有可处理通道
        int num = selector.select();
        if (num == 0) continue;//jdk 中空转有一定几率造成 cpu 100%，属于系统bug，需谨慎

        Set<SelectionKey> set = selector.selectedKeys();
        Iterator<SelectionKey> it = set.iterator();
        while (it.hasNext()) {
            SelectionKey key = it.next();

            it.remove();

            if (key.isAcceptable()) {//把通道注册为可读

                ServerSocketChannel channel = (ServerSocketChannel) key.channel();

                SocketChannel acceptChannel = channel.accept();
                acceptChannel.configureBlocking(false);
                acceptChannel.register(selector,SelectionKey.OP_READ);

            } else if (key.isReadable()) {//处理已连接 channel 数据
                SocketChannel channel = (SocketChannel) key.channel();
                ByteBuffer buffer = ByteBuffer.allocate(1024);
                channel.read(buffer);
                System.out.println(Charset.forName("UTF-8").decode(buffer));
				//do something
				channel.write(Charset.forName("UTF-8").encode("回复"));
                channel.close();
            }
        }
    }
}
```

从上面可以看出，SocketChannel 在并发量大处理的优越性。

如果是需要管理成千上万个连接，并且这些连接每次只是发送少量的数据，如聊天服务器这类需求，实现NIO服务器可能是一个优势，但如果是少量连接使用，一次发送大量数据，还是典型的IO服务器实现更符合要求。

当然 SocketChannel 还是存在缺点，比如注释中写到可能出现cpu占用100%的bug（说已修复但仍有小概率出现），api使用比较复杂，但对于小型的服务处理，NIO服务器仍然是一个高效可用的实现。