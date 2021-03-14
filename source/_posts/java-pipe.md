---
title: 【java】Pipe
date: 2021-03-14 18:32:17
tags: [java,nio]
categories: java
---
java Pipe 管道流

### 1. Pipe 管道

对于数据流的处理，一般情况下我们在同一个线程中进行。如果遇到异步处理场景，一边进行数据写入，另一线程进行数据读取，那么 Pipe 管道可以很好解决这个问题。

先看 Pipe 的原理图示：

![](http://ifeve.com/wp-content/uploads/2013/06/pipe.bmp)

在同一管道中，Sink 作为头进行数据写入，而 Source 端进行数据读取，实现了数据的流通，就像是水管，一头流入，一头流出。因为 Pipe 具备了 sink 和 source，所以在不同线程中，可以通过同一个 pipe 实现一端数据写入，一端读取，从而实现了不同线程间的数据流通信。

### 2. 使用

先打开一个 Pipe，为了模拟不同线程通信，先启动线程，读取本地文件，再写入管道，另一个线程读取，并将接收的数据打印出来。

```
//开启一个通道
Pipe pipe = Pipe.open();
//线程：模拟数据写入
CompletableFuture.runAsync(() -> {
    Pipe.SinkChannel sinkChannel = pipe.sink();

    try {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        FileChannel fileChannel = new FileInputStream("./res/normal.txt").getChannel();
        //读取文件中数据，写入管道
        while (fileChannel.read(buffer) != -1) {
            buffer.flip();
            sinkChannel.write(buffer);
            buffer.clear();

            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("exit system");
        fileChannel.close();
        sinkChannel.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
});

ByteBuffer buffer = ByteBuffer.allocate(1024);
//拿到 source 端，读取数据
Pipe.SourceChannel source = pipe.source();
while (source.read(buffer) != -1) {
    buffer.flip();
    String data = new String(buffer.array(), StandardCharsets.UTF_8);
    System.out.println(data);
    buffer.clear();
    if ("stop".equals(data)) {
        System.out.println("load stop,exit!");
        break;
    }
}
source.close();
```

在数据流处理角度上看，和通常使用的 InputStream OutputStream，并没有区别。

### 3. 小结

Pipe 在开发中使用较少，写这篇的主要原因在于一天看到一篇关于 zero-copy 的文章，里面除了利用 NIO Channel 进行举例，还使用了 Pipe，让我误会 Pipe 能够进一步加速文件数据的复制操作，在了解之后发现那个博主纯粹是为了使用而使用。   
Pipe 就是一个单纯连接输入端和输出端的工具，仅此而已。

