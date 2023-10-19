# JavaSE-java.nio

**java.nio** (New I/O) is a package introduced in Java to provide support for high-performance I/O operations. 

It includes buffers, channels, selectors, and other classes to handle non-blocking I/O. Let me give you a brief overview with some samples.

## 1. Buffers

Buffers are objects that hold data, facilitating the transfer of data between a channel and an application. 

Here's an example using a ByteBuffer:

```java
import java.nio.ByteBuffer;

public class BufferExample {
    public static void main(String[] args) {
        // Creating a ByteBuffer with a capacity of 10
        ByteBuffer buffer = ByteBuffer.allocate(10);

        // Adding data to the buffer
        buffer.put((byte) 1);
        buffer.put((byte) 2);
        buffer.put((byte) 3);

        // Flipping the buffer to prepare for reading
        buffer.flip();

        // Reading data from the buffer
        while (buffer.hasRemaining()) {
            System.out.println(buffer.get());
        }
    }
}
```

## 2. Channels

Channels are conduits for I/O operations. They can be associated with files, sockets, etc. 

Here's a simple example using FileChannel:

```java
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class ChannelExample {
    public static void main(String[] args) throws Exception {
        // Opening a RandomAccessFile and obtaining its FileChannel
        RandomAccessFile file = new RandomAccessFile("example.txt", "rw");
        FileChannel channel = file.getChannel();

        // Creating a ByteBuffer with data
        ByteBuffer buffer = ByteBuffer.allocate(10);
        buffer.put("Hello".getBytes());
        buffer.flip();

        // Writing data to the channel
        channel.write(buffer);

        // Closing the channel and file
        channel.close();
        file.close();
    }
}
```

## 3. Selectors
Selectors allow multiplexed I/O, which means handling multiple channels in a single thread. Here's a basic example using a Selector:

```java
import java.nio.channels.SocketChannel;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.Selector;
import java.util.Set;

public class SelectorExample {
    public static void main(String[] args) throws Exception {
        // Opening a Selector
        Selector selector = Selector.open();

        // Opening channels
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.configureBlocking(false);

        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.configureBlocking(false);

        // Registering channels with the Selector
        serverSocketChannel.register(selector, serverSocketChannel.validOps());
        socketChannel.register(selector, socketChannel.validOps());

        // Performing a select operation
        int readyChannels = selector.select();

        // Getting the set of selected keys
        Set<java.nio.channels.SelectionKey> selectedKeys = selector.selectedKeys();

        // Closing channels and selector
        serverSocketChannel.close();
        socketChannel.close();
        selector.close();
    }
}
```

These examples provide a basic understanding of java.nio concepts. 

The package is extensive, and there are more features and classes to explore based on specific use cases.
