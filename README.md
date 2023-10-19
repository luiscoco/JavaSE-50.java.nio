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

Let's explore a few more examples with different aspects of java.nio.

## 4. Memory-Mapped Files

Memory-mapped files allow you to map a region of a file directly into memory. Changes made to the buffer affect the file, and vice versa.

```java
import java.io.RandomAccessFile;
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;

public class MemoryMappedFileExample {
    public static void main(String[] args) throws Exception {
        RandomAccessFile file = new RandomAccessFile("mappedFile.txt", "rw");
        FileChannel channel = file.getChannel();

        // Mapping the file into memory
        MappedByteBuffer buffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 1024);

        // Writing data to the buffer
        buffer.put("Hello, Memory-Mapped File!".getBytes());

        // Flushing changes to the file
        buffer.force();

        // Closing the channel and file
        channel.close();
        file.close();
    }
}
```

## 5. File Locking

File locking can be used to prevent multiple processes from concurrently modifying a file.

```java
import java.io.RandomAccessFile;
import java.nio.channels.FileChannel;
import java.nio.channels.FileLock;

public class FileLockExample {
    public static void main(String[] args) throws Exception {
        RandomAccessFile file = new RandomAccessFile("lockedFile.txt", "rw");
        FileChannel channel = file.getChannel();

        // Acquiring an exclusive lock on the file
        FileLock lock = channel.lock();

        // Performing operations on the locked file

        // Releasing the lock
        lock.release();

        // Closing the channel and file
        channel.close();
        file.close();
    }
}
```

## 6. Asynchronous File I/O

Java NIO provides asynchronous I/O operations for improved performance.

```java
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousFileChannel;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;
import java.util.concurrent.Future;

public class AsynchronousFileIOExample {
    public static void main(String[] args) throws Exception {
        Path path = Path.of("asyncFile.txt");

        // Opening an asynchronous file channel
        AsynchronousFileChannel channel = AsynchronousFileChannel.open(
                path, StandardOpenOption.READ, StandardOpenOption.WRITE);

        // Reading data asynchronously
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        Future<Integer> readResult = channel.read(buffer, 0);

        // Performing other tasks while reading is in progress

        // Waiting for the read operation to complete
        readResult.get();

        // Writing data asynchronously
        buffer.flip(); // Prepare buffer for writing
        Future<Integer> writeResult = channel.write(buffer, 0);

        // Performing other tasks while writing is in progress

        // Waiting for the write operation to complete
        writeResult.get();

        // Closing the channel
        channel.close();
    }
}
```

# More advanced topics about java.nio

Let's dive into some more advanced topics related to java.nio.

## Memory-Mapped Buffers

Memory-mapped buffers provide a way to map a region of a file directly into memory. 

This allows you to perform I/O operations directly on the mapped memory, providing potential performance benefits. 

Here's a more detailed example:

```java
import java.io.RandomAccessFile;
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;

public class AdvancedMemoryMappedBuffer {
    public static void main(String[] args) throws Exception {
        RandomAccessFile file = new RandomAccessFile("advancedMappedFile.txt", "rw");
        FileChannel channel = file.getChannel();

        // Mapping only a portion of the file into memory
        MappedByteBuffer buffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 1024);

        // Reading and writing data using the buffer
        buffer.putInt(123);
        buffer.put("Hello, Memory-Mapped Buffer!".getBytes());

        // Flush changes to the file
        buffer.force();

        // Unmap the buffer (optional, as it will be automatically unmapped when the program exits)
        ((sun.nio.ch.DirectBuffer) buffer).cleaner().clean();

        // Closing the channel and file
        channel.close();
        file.close();
    }
}
```

## Scatter-Gather I/O

Scatter-Gather I/O allows you to read or write to multiple buffers in a single I/O operation. This is useful when dealing with complex data structures. Here's an example:

```java
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class ScatterGatherIO {
    public static void main(String[] args) throws Exception {
        RandomAccessFile file = new RandomAccessFile("scatterGatherFile.txt", "rw");
        FileChannel channel = file.getChannel();

        // Creating multiple buffers for scatter operation
        ByteBuffer buffer1 = ByteBuffer.allocate(10);
        ByteBuffer buffer2 = ByteBuffer.allocate(20);

        ByteBuffer[] buffers = { buffer1, buffer2 };

        // Scatter read into multiple buffers
        channel.read(buffers);

        // Do something with the data in individual buffers

        // Creating multiple buffers for gather operation
        ByteBuffer buffer3 = ByteBuffer.allocate(10);
        ByteBuffer buffer4 = ByteBuffer.allocate(20);

        ByteBuffer[] gatherBuffers = { buffer3, buffer4 };

        // Do something to fill data into individual buffers

        // Gather write from multiple buffers
        channel.write(gatherBuffers);

        // Closing the channel and file
        channel.close();
        file.close();
    }
}
```

## Non-blocking I/O with Selectors

Selectors allow a single thread to manage multiple channels, which is useful for handling I/O operations without blocking the entire application. Here's a simplified example:

```java
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SelectionKey;
import java.util.Iterator;
import java.util.Set;

public class AdvancedSelectorExample {
    public static void main(String[] args) throws Exception {
        Selector selector = Selector.open();

        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.socket().bind(null);
        serverSocketChannel.configureBlocking(false);

        // Register the server socket channel with the selector for accepting connections
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        while (true) {
            int readyChannels = selector.select();

            if (readyChannels == 0) {
                continue;
            }

            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

            while (keyIterator.hasNext()) {
                SelectionKey key = keyIterator.next();

                if (key.isAcceptable()) {
                    // Accept the connection
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    socketChannel.configureBlocking(false);

                    // Register the new socket channel for reading
                    socketChannel.register(selector, SelectionKey.OP_READ);
                } else if (key.isReadable()) {
                    // Handle read operation
                    // ...
                }

                keyIterator.remove();
            }
        }
    }
}
```

These advanced topics provide a deeper understanding of java.nio and its capabilities. 
