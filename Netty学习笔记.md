

[TOC]

---

















# NIO

## 三大组件

### Channel 和 Buffer

channel 有一点类似于 stream，它就是读写数据的**双向通道**，可以从 channel 将数据读入 buffer，也可以将 buffer 的数据写入 channel，而之前的 stream 要么是输入，要么是输出，channel 比 stream 更为底层



常见的 Channel 有

* FileChannel
* DatagramChannel
* SocketChannel
* ServerSocketChannel



buffer 则用来缓冲读写数据，常见的 buffer 有

* ByteBuffer
  * MappedByteBuffer
  * DirectByteBuffer
  * HeapByteBuffer
* ShortBuffer
* IntBuffer
* LongBuffer
* FloatBuffer
* DoubleBuffer
* CharBuffer





### Selector

选择器

selector 的作用就是配合一个线程来管理多个 channel，获取这些 channel 上发生的事件，这些 channel 工作在非阻塞模式下，不会让线程吊死在一个 channel 上。适合连接数特别多，但流量低的场景（low traffic）



![image-20230304213852830](img/Netty学习笔记/image-20230304213852830.png)



调用 selector 的 select() 会阻塞直到 channel 发生了读写就绪事件，这些事件发生，select 方法就会返回这些事件交给 thread 来处理











## ByteBuffer

### 使用流程

1. 向 buffer 写入数据，例如调用 channel.read(buffer)
2. 调用 flip() 切换至**读模式**
3. 从 buffer 读取数据，例如调用 buffer.get()
4. 调用 clear() 或 compact() 切换至**写模式**
5. 重复 1~4 步骤



Buffer 是**非线程安全的**





### 入门

有一普通文本文件 data.txt，内容为

```sh
1234567890abcdef12345678

```



现在需要从文件里读取数据，要求：使用 FileChannel 来读取文件内容



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

/**
 * Project name(项目名称)：Netty_ByteBuffer
 * Package(包名): mao.t1
 * Class(类名): FileChannelTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/5
 * Time(创建时间)： 21:08
 * Version(版本): 1.0
 * Description(描述)： ByteBuffer入门，ByteBuffer一次读10个
 */

public class FileChannelTest
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(FileChannelTest.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args)
    {
        //RandomAccessFile对象
        try (RandomAccessFile randomAccessFile = new RandomAccessFile("data.txt", "rw"))
        {
            //得到FileChannel
            FileChannel fileChannel = randomAccessFile.getChannel();
            //ByteBuffer，大小为10
            ByteBuffer byteBuffer = ByteBuffer.allocate(10);
            do
            {
                //写入，并记录长度
                int len = fileChannel.read(byteBuffer);
                log.debug("读到字节数：{}", len);
                //如果长度为-1，就是后面没有了
                if (len == -1)
                {
                    break;
                }
                //切换到读模式，开始打印
                byteBuffer.flip();
                //判断后面是否还有，如果有，打印
                while (byteBuffer.hasRemaining())
                {
                    log.debug("{}", (char) byteBuffer.get());
                }
                //ByteBuffer读到最后面了
                //切换到写模式
                byteBuffer.clear();
            }
            while (true);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}

```



运行结果：

```sh
2023-03-05  21:28:08.337  [main] DEBUG mao.t1.FileChannelTest:  读到字节数：10
2023-03-05  21:28:08.338  [main] DEBUG mao.t1.FileChannelTest:  1
2023-03-05  21:28:08.338  [main] DEBUG mao.t1.FileChannelTest:  2
2023-03-05  21:28:08.338  [main] DEBUG mao.t1.FileChannelTest:  3
2023-03-05  21:28:08.338  [main] DEBUG mao.t1.FileChannelTest:  4
2023-03-05  21:28:08.338  [main] DEBUG mao.t1.FileChannelTest:  5
2023-03-05  21:28:08.338  [main] DEBUG mao.t1.FileChannelTest:  6
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  7
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  8
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  9
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  0
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  读到字节数：10
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  a
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  b
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  c
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  d
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  e
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  f
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  1
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  2
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  3
2023-03-05  21:28:08.339  [main] DEBUG mao.t1.FileChannelTest:  4
2023-03-05  21:28:08.340  [main] DEBUG mao.t1.FileChannelTest:  读到字节数：6
2023-03-05  21:28:08.340  [main] DEBUG mao.t1.FileChannelTest:  5
2023-03-05  21:28:08.340  [main] DEBUG mao.t1.FileChannelTest:  6
2023-03-05  21:28:08.340  [main] DEBUG mao.t1.FileChannelTest:  7
2023-03-05  21:28:08.340  [main] DEBUG mao.t1.FileChannelTest:  8
2023-03-05  21:28:08.340  [main] DEBUG mao.t1.FileChannelTest:  
2023-03-05  21:28:08.340  [main] DEBUG mao.t1.FileChannelTest:  

2023-03-05  21:28:08.340  [main] DEBUG mao.t1.FileChannelTest:  读到字节数：-1
```







### ByteBuffer 结构

ByteBuffer 有以下重要属性

* capacity
* position
* limit



一开始

![image-20230305213101095](img/Netty学习笔记/image-20230305213101095.png)



写模式下，position 是写入位置，limit 等于容量

写入4字节后



![image-20230305213156908](img/Netty学习笔记/image-20230305213156908.png)





flip 动作发生后，position 切换为读取位置，limit 切换为读取限制

![image-20230305213255468](img/Netty学习笔记/image-20230305213255468.png)





读取 4 个字节后，状态



![image-20230305213308513](img/Netty学习笔记/image-20230305213308513.png)





clear 动作发生后，状态



![image-20230305213322899](img/Netty学习笔记/image-20230305213322899.png)





compact 方法，是把未读完的部分向前压缩，然后切换至写模式



![image-20230305213354895](img/Netty学习笔记/image-20230305213354895.png)









### 工具类

需要两个依赖

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>

<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>20.0</version>
</dependency>
```







```java
package mao.utils;


import io.netty.util.internal.StringUtil;

import java.nio.ByteBuffer;

import static io.netty.util.internal.MathUtil.isOutOfBounds;
import static io.netty.util.internal.StringUtil.NEWLINE;

/**
 * Project name(项目名称)：Netty_ByteBuffer
 * Package(包名): mao.utils
 * Class(类名): ByteBufferUtil
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/5
 * Time(创建时间)： 21:34
 * Version(版本): 1.0
 * Description(描述)： 工具类
 */

public class ByteBufferUtil
{
    private static final char[] BYTE2CHAR = new char[256];
    private static final char[] HEXDUMP_TABLE = new char[256 * 4];
    private static final String[] HEXPADDING = new String[16];
    private static final String[] HEXDUMP_ROWPREFIXES = new String[65536 >>> 4];
    private static final String[] BYTE2HEX = new String[256];
    private static final String[] BYTEPADDING = new String[16];

    static
    {
        final char[] DIGITS = "0123456789abcdef".toCharArray();
        for (int i = 0; i < 256; i++)
        {
            HEXDUMP_TABLE[i << 1] = DIGITS[i >>> 4 & 0x0F];
            HEXDUMP_TABLE[(i << 1) + 1] = DIGITS[i & 0x0F];
        }

        int i;

        // Generate the lookup table for hex dump paddings
        for (i = 0; i < HEXPADDING.length; i++)
        {
            int padding = HEXPADDING.length - i;
            StringBuilder buf = new StringBuilder(padding * 3);
            for (int j = 0; j < padding; j++)
            {
                buf.append("   ");
            }
            HEXPADDING[i] = buf.toString();
        }

        // Generate the lookup table for the start-offset header in each row (up to 64KiB).
        for (i = 0; i < HEXDUMP_ROWPREFIXES.length; i++)
        {
            StringBuilder buf = new StringBuilder(12);
            buf.append(NEWLINE);
            buf.append(Long.toHexString(i << 4 & 0xFFFFFFFFL | 0x100000000L));
            buf.setCharAt(buf.length() - 9, '|');
            buf.append('|');
            HEXDUMP_ROWPREFIXES[i] = buf.toString();
        }

        // Generate the lookup table for byte-to-hex-dump conversion
        for (i = 0; i < BYTE2HEX.length; i++)
        {
            BYTE2HEX[i] = ' ' + StringUtil.byteToHexStringPadded(i);
        }

        // Generate the lookup table for byte dump paddings
        for (i = 0; i < BYTEPADDING.length; i++)
        {
            int padding = BYTEPADDING.length - i;
            StringBuilder buf = new StringBuilder(padding);
            for (int j = 0; j < padding; j++)
            {
                buf.append(' ');
            }
            BYTEPADDING[i] = buf.toString();
        }

        // Generate the lookup table for byte-to-char conversion
        for (i = 0; i < BYTE2CHAR.length; i++)
        {
            if (i <= 0x1f || i >= 0x7f)
            {
                BYTE2CHAR[i] = '.';
            }
            else
            {
                BYTE2CHAR[i] = (char) i;
            }
        }
    }

    /**
     * 打印所有内容
     *
     * @param buffer 缓冲
     */
    public static void debugAll(ByteBuffer buffer)
    {
        int oldlimit = buffer.limit();
        buffer.limit(buffer.capacity());
        StringBuilder origin = new StringBuilder(256);
        appendPrettyHexDump(origin, buffer, 0, buffer.capacity());
        System.out.println("+--------+-------------------- all ------------------------+----------------+");
        System.out.printf("position: [%d], limit: [%d]\n", buffer.position(), oldlimit);
        System.out.println(origin);
        buffer.limit(oldlimit);
    }

    /**
     * 打印可读取内容
     *
     * @param buffer 缓冲
     */
    public static void debugRead(ByteBuffer buffer)
    {
        StringBuilder builder = new StringBuilder(256);
        appendPrettyHexDump(builder, buffer, buffer.position(), buffer.limit() - buffer.position());
        System.out.println("+--------+-------------------- read -----------------------+----------------+");
        System.out.printf("position: [%d], limit: [%d]\n", buffer.position(), buffer.limit());
        System.out.println(builder);
    }

    private static void appendPrettyHexDump(StringBuilder dump, ByteBuffer buf, int offset, int length)
    {
        if (isOutOfBounds(offset, length, buf.capacity()))
        {
            throw new IndexOutOfBoundsException(
                    "expected: " + "0 <= offset(" + offset + ") <= offset + length(" + length
                            + ") <= " + "buf.capacity(" + buf.capacity() + ')');
        }
        if (length == 0)
        {
            return;
        }
        dump.append("         +-------------------------------------------------+").append(NEWLINE).append("         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |").append(NEWLINE).append("+--------+-------------------------------------------------+----------------+");

        final int startIndex = offset;
        final int fullRows = length >>> 4;
        final int remainder = length & 0xF;

        // Dump the rows which have 16 bytes.
        for (int row = 0; row < fullRows; row++)
        {
            int rowStartIndex = (row << 4) + startIndex;

            // Per-row prefix.
            appendHexDumpRowPrefix(dump, row, rowStartIndex);

            // Hex dump
            int rowEndIndex = rowStartIndex + 16;
            for (int j = rowStartIndex; j < rowEndIndex; j++)
            {
                dump.append(BYTE2HEX[getUnsignedByte(buf, j)]);
            }
            dump.append(" |");

            // ASCII dump
            for (int j = rowStartIndex; j < rowEndIndex; j++)
            {
                dump.append(BYTE2CHAR[getUnsignedByte(buf, j)]);
            }
            dump.append('|');
        }

        // Dump the last row which has less than 16 bytes.
        if (remainder != 0)
        {
            int rowStartIndex = (fullRows << 4) + startIndex;
            appendHexDumpRowPrefix(dump, fullRows, rowStartIndex);

            // Hex dump
            int rowEndIndex = rowStartIndex + remainder;
            for (int j = rowStartIndex; j < rowEndIndex; j++)
            {
                dump.append(BYTE2HEX[getUnsignedByte(buf, j)]);
            }
            dump.append(HEXPADDING[remainder]);
            dump.append(" |");

            // Ascii dump
            for (int j = rowStartIndex; j < rowEndIndex; j++)
            {
                dump.append(BYTE2CHAR[getUnsignedByte(buf, j)]);
            }
            dump.append(BYTEPADDING[remainder]);
            dump.append('|');
        }

        dump.append(NEWLINE +
                "+--------+-------------------------------------------------+----------------+");
    }

    /**
     * 将十六进制转储行前缀
     *
     * @param dump          转储
     * @param row           行
     * @param rowStartIndex 行开始指数
     */
    private static void appendHexDumpRowPrefix(StringBuilder dump, int row, int rowStartIndex)
    {
        if (row < HEXDUMP_ROWPREFIXES.length)
        {
            dump.append(HEXDUMP_ROWPREFIXES[row]);
        }
        else
        {
            dump.append(NEWLINE);
            dump.append(Long.toHexString(rowStartIndex & 0xFFFFFFFFL | 0x100000000L));
            dump.setCharAt(dump.length() - 9, '|');
            dump.append('|');
        }
    }

    /**
     * 得到无符号字节
     *
     * @param buffer 缓冲
     * @param index  指数
     * @return short
     */
    public static short getUnsignedByte(ByteBuffer buffer, int index)
    {
        return (short) (buffer.get(index) & 0xFF);
    }
}
```









### ByteBuffer常见方法

#### 分配空间

可以使用 allocate 方法为 ByteBuffer 分配空间，其它 buffer 类也有该方法

```java
//ByteBuffer分配空间，大小为10
ByteBuffer byteBuffer = ByteBuffer.allocate(10);
```





#### 向ByteBuffer写入数据

有两种办法

* 调用 channel 的 read 方法
* 调用 buffer 自己的 put 方法



```java
package mao.t2;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.nio.ByteBuffer;

/**
 * Project name(项目名称)：Netty_ByteBuffer
 * Package(包名): mao.t2
 * Class(类名): ByteBufferPutTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/5
 * Time(创建时间)： 21:50
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class ByteBufferPutTest
{
    private static final Logger log = LoggerFactory.getLogger(ByteBufferPutTest.class);

    public static void main(String[] args)
    {
        ByteBuffer byteBuffer = ByteBuffer.allocate(10);
        //调用put方法
        byteBuffer.put(new byte[]{'1', '2', '3', '4'});
        //打印
        ByteBufferUtil.debugAll(byteBuffer);

        //调用put方法
        byteBuffer.put(new byte[]{'5', '6'});
        //打印
        ByteBufferUtil.debugAll(byteBuffer);
    }
}

```



运行结果：

```sh
+--------+-------------------- all ------------------------+----------------+
position: [4], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 00 00 00 00 00 00                   |1234......      |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [6], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 00 00 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
```







#### 从ByteBuffer读取数据

同样有两种办法

* 调用 channel 的 write 方法
* 调用 buffer 自己的 get 方法



```java
int writeBytes = channel.write(byteBuffer);
```

或者

```java
byte b = byteBuffer.get();
```



get 方法会让 position 读指针向后走，如果想重复读取数据

* 可以调用 rewind 方法将 position 重新置为 0
* 或者调用 get(int i) 方法获取索引 i 的内容，它不会移动读指针



```java
package mao.t2;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.nio.ByteBuffer;

/**
 * Project name(项目名称)：Netty_ByteBuffer
 * Package(包名): mao.t2
 * Class(类名): ByteBufferGetTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/5
 * Time(创建时间)： 21:59
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class ByteBufferGetTest
{
    private static final Logger log = LoggerFactory.getLogger(ByteBufferPutTest.class);

    public static void main(String[] args)
    {
        ByteBuffer byteBuffer = ByteBuffer.allocate(10);
        //调用put方法
        byteBuffer.put(new byte[]{'1', '2', '3', '4'});

        //调用put方法
        byteBuffer.put(new byte[]{'5', '6'});

        //打印
        ByteBufferUtil.debugAll(byteBuffer);

        //按索引位置读取
        byte b = byteBuffer.get(2);
        System.out.println((char) b);
        //打印
        ByteBufferUtil.debugAll(byteBuffer);
        //按索引位置读取
        b = byteBuffer.get(4);
        System.out.println((char) b);
        //打印
        ByteBufferUtil.debugAll(byteBuffer);

        //调用get方法读取
        b = byteBuffer.get();
        System.out.println((char) b);
        //打印
        ByteBufferUtil.debugAll(byteBuffer);

        //调用get方法读取
        b = byteBuffer.get();
        System.out.println((char) b);
        //打印
        ByteBufferUtil.debugAll(byteBuffer);

        //调用rewind方法，将 position 重新置为 0
        byteBuffer.rewind();
        //打印
        ByteBufferUtil.debugAll(byteBuffer);

        //调用get方法读取
        b = byteBuffer.get();
        System.out.println((char) b);
        //打印
        ByteBufferUtil.debugAll(byteBuffer);
    }
}
```



运行结果：

```sh
+--------+-------------------- all ------------------------+----------------+
position: [6], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 00 00 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
3
+--------+-------------------- all ------------------------+----------------+
position: [6], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 00 00 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
5
+--------+-------------------- all ------------------------+----------------+
position: [6], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 00 00 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
 
+--------+-------------------- all ------------------------+----------------+
position: [7], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 00 00 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
 
+--------+-------------------- all ------------------------+----------------+
position: [8], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 00 00 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 00 00 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
1
+--------+-------------------- all ------------------------+----------------+
position: [1], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 00 00 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
```







#### mark 和 reset

mark 是在读取时，做一个标记，即使 position 改变，只要调用 reset 就能回到 mark 的位置



```java
package mao.t2;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.nio.ByteBuffer;

/**
 * Project name(项目名称)：Netty_ByteBuffer
 * Package(包名): mao.t2
 * Class(类名): ByteBufferMarkAndResetTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/5
 * Time(创建时间)： 22:04
 * Version(版本): 1.0
 * Description(描述)： mark 和 reset
 */

public class ByteBufferMarkAndResetTest
{
    private static final Logger log = LoggerFactory.getLogger(ByteBufferPutTest.class);

    public static void main(String[] args)
    {
        ByteBuffer byteBuffer = ByteBuffer.allocate(10);
        //调用put方法
        byteBuffer.put(new byte[]{'1', '2', '3', '4'});
        //调用put方法
        byteBuffer.put(new byte[]{'5', '6'});

        //打印
        ByteBufferUtil.debugAll(byteBuffer);

        //调用mark方法
        byteBuffer.mark();

        //调用两次get方法
        byteBuffer.get();
        byteBuffer.get();

        //打印
        ByteBufferUtil.debugAll(byteBuffer);

        //调用reset方法
        byteBuffer.reset();

        //打印
        ByteBufferUtil.debugAll(byteBuffer);

    }
}
```



运行结果：

```sh
+--------+-------------------- all ------------------------+----------------+
position: [6], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 00 00 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [8], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 00 00 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [6], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 00 00 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
```





**rewind 和 flip 都会清除 mark 位置**







#### 字符串与ByteBuffer互转

```java
package mao.t2;

import mao.utils.ByteBufferUtil;

import java.nio.ByteBuffer;
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuffer
 * Package(包名): mao.t2
 * Class(类名): StringToByteBufferTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/5
 * Time(创建时间)： 22:13
 * Version(版本): 1.0
 * Description(描述)： 字符串转ByteBuffer
 */

public class StringToByteBufferTest
{
    public static void main(String[] args)
    {
        String s = "hello\0你好";

        //第1种
        ByteBuffer byteBuffer1 = StandardCharsets.UTF_8.encode(s);
        //第2种
        ByteBuffer byteBuffer2 = Charset.forName("utf-8").encode(s);

        ByteBufferUtil.debugAll(byteBuffer1);
        ByteBufferUtil.debugAll(byteBuffer2);
    }
}
```



运行结果：

```sh
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [12]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 e4 bd a0 e5 a5 bd 00 00 00 00 |hello...........|
|00000010| 00                                              |.               |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [12]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 e4 bd a0 e5 a5 bd 00 00 00 00 |hello...........|
|00000010| 00                                              |.               |
+--------+-------------------------------------------------+----------------+
```





**ByteBuffer转String**

```java
package mao.t2;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuffer
 * Package(包名): mao.t2
 * Class(类名): ByteBufferToStringTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/5
 * Time(创建时间)： 22:22
 * Version(版本): 1.0
 * Description(描述)： ByteBuffer转String
 */

public class ByteBufferToStringTest
{
    private static final Logger log = LoggerFactory.getLogger(ByteBufferToStringTest.class);

    public static void main(String[] args)
    {
        String s = "1234 hello 你好";

        //第1种
        ByteBuffer byteBuffer1 = StandardCharsets.UTF_8.encode(s);
        //第2种
        ByteBuffer byteBuffer2 = Charset.forName("utf-8").encode(s);

        ByteBufferUtil.debugAll(byteBuffer1);
        ByteBufferUtil.debugAll(byteBuffer2);

        CharBuffer charBuffer1 = StandardCharsets.UTF_8.decode(byteBuffer1);
        CharBuffer charBuffer2 = StandardCharsets.UTF_8.decode(byteBuffer2);

        String s1 = charBuffer1.toString();
        String s2 = charBuffer2.toString();

        log.info(s1);
        log.info(s2);
    }
}
```



运行结果：

```sh
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [17]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 20 68 65 6c 6c 6f 20 e4 bd a0 e5 a5 |1234 hello .....|
|00000010| bd 00 00 00 00 00 00 00 00 00 00 00 00          |.............   |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [17]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 20 68 65 6c 6c 6f 20 e4 bd a0 e5 a5 |1234 hello .....|
|00000010| bd 00 00 00 00 00 00 00 00 00 00 00 00          |.............   |
+--------+-------------------------------------------------+----------------+
2023-03-05  22:26:02.809  [main] INFO  mao.t2.ByteBufferToStringTest:  1234 hello 你好
2023-03-05  22:26:02.810  [main] INFO  mao.t2.ByteBufferToStringTest:  1234 hello 你好
```







### Scattering Reads

现在有以下文件data2.txt，文件内容为：

```sh
onetwothreefour
```



现在要将这4个单词读按顺序读到4个ByteBuffer里，已知每个单词的长度



```java
package mao.t3;

import mao.utils.ByteBufferUtil;

import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

/**
 * Project name(项目名称)：Netty_ByteBuffer
 * Package(包名): mao.t3
 * Class(类名): ScatteringReadsTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/5
 * Time(创建时间)： 22:39
 * Version(版本): 1.0
 * Description(描述)： Scattering Reads
 */

public class ScatteringReadsTest
{
    public static void main(String[] args)
    {
        try (RandomAccessFile randomAccessFile = new RandomAccessFile("data2.txt", "rw"))
        {
            FileChannel fileChannel = randomAccessFile.getChannel();

            //构建4个ByteBuffer
            ByteBuffer byteBuffer1 = ByteBuffer.allocate(3);
            ByteBuffer byteBuffer2 = ByteBuffer.allocate(3);
            ByteBuffer byteBuffer3 = ByteBuffer.allocate(5);
            ByteBuffer byteBuffer4 = ByteBuffer.allocate(4);

            //ByteBuffer数组
            ByteBuffer[] byteBuffers = new ByteBuffer[]{byteBuffer1, byteBuffer2, byteBuffer3, byteBuffer4};

            //读取，并写入到4个ByteBuffer中
            fileChannel.read(byteBuffers);

            //打印
            ByteBufferUtil.debugAll(byteBuffer1);
            ByteBufferUtil.debugAll(byteBuffer2);
            ByteBufferUtil.debugAll(byteBuffer3);
            ByteBufferUtil.debugAll(byteBuffer4);

        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```



运行结果：

```sh
+--------+-------------------- all ------------------------+----------------+
position: [3], limit: [3]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 6f 6e 65                                        |one             |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [3], limit: [3]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 74 77 6f                                        |two             |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [5], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 74 68 72 65 65                                  |three           |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [4], limit: [4]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 66 6f 75 72                                     |four            |
+--------+-------------------------------------------------+----------------+
```







### Gathering Writes

需求：将多个 ByteBuffer 的数据填充至 channel



```java
package mao.t3;

import mao.utils.ByteBufferUtil;

import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuffer
 * Package(包名): mao.t3
 * Class(类名): GatheringWritesTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/5
 * Time(创建时间)： 22:47
 * Version(版本): 1.0
 * Description(描述)： Gathering Writes
 */

public class GatheringWritesTest
{
    public static void main(String[] args)
    {
        try (RandomAccessFile randomAccessFile = new RandomAccessFile("data4.txt", "rw"))
        {
            FileChannel fileChannel = randomAccessFile.getChannel();

            ByteBuffer byteBuffer1 = ByteBuffer.allocate(4);
            ByteBuffer byteBuffer2 = ByteBuffer.allocate(3);

            byteBuffer1.put("five".getBytes(StandardCharsets.UTF_8));
            byteBuffer2.put("six".getBytes(StandardCharsets.UTF_8));

            ByteBuffer[] byteBuffers = new ByteBuffer[]{byteBuffer1, byteBuffer2};

            ByteBufferUtil.debugAll(byteBuffer1);
            ByteBufferUtil.debugAll(byteBuffer2);

            //切换到读模式
            byteBuffer1.flip();
            byteBuffer2.flip();

            ByteBufferUtil.debugAll(byteBuffer1);
            ByteBufferUtil.debugAll(byteBuffer2);

            //写入
            fileChannel.write(byteBuffers);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```



运行结果：

```sh
+--------+-------------------- all ------------------------+----------------+
position: [4], limit: [4]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 66 69 76 65                                     |five            |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [3], limit: [3]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 73 69 78                                        |six             |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [4]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 66 69 76 65                                     |five            |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [3]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 73 69 78                                        |six             |
+--------+-------------------------------------------------+----------------+
```



文件内容：

```sh
fivesix
```









### 黏包和半包

#### 说明

网络上有多条数据发送给服务端，数据之间使用 \n 进行分隔
但由于某种原因这些数据在接收时，被进行了重新组合，例如原始数据有3条为

* Hello,world\n
* I'm zhangsan\n
* How are you?\n

变成了下面的两个 byteBuffer (黏包，半包)

* Hello,world\nI'm zhangsan\nHo
* w are you?\n



现在要求你编写程序，将错乱的数据恢复成原始的按 \n 分隔的数据



#### 解决

```java
package mao.t4;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.nio.ByteBuffer;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuffer
 * Package(包名): mao.t4
 * Class(类名): StickyAndHalfPackedTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/8
 * Time(创建时间)： 21:41
 * Version(版本): 1.0
 * Description(描述)： 黏包和半包
 */

public class StickyAndHalfPackedTest
{

    private static final Logger log = LoggerFactory.getLogger(StickyAndHalfPackedTest.class);

    public static void main(String[] args)
    {
        ByteBuffer byteBuffer = ByteBuffer.allocate(64);
        byteBuffer.put("Hello,world\nI'm zhangsan\nHo".getBytes(StandardCharsets.UTF_8));
        split(byteBuffer);
        byteBuffer.put("w are you?\nhaha!\n".getBytes(StandardCharsets.UTF_8));
        split(byteBuffer);
    }


    /**
     * 分割
     *
     * @param byteBuffer 字节缓冲区
     */
    private static void split(ByteBuffer byteBuffer)
    {
        //切换到读模式
        byteBuffer.flip();
        //得到limit
        int limit = byteBuffer.limit();
        for (int i = 0; i < limit; i++)
        {
            //判断是否遇到了换换行符
            if (byteBuffer.get(i) == '\n')
            {
                //换行符
                log.debug(String.valueOf(i));
                //开辟一个一行长度的ByteBuffer
                ByteBuffer target = ByteBuffer.allocate(i + 1 - byteBuffer.position());
                //设置limit
                byteBuffer.limit(i + 1);
                //从byteBuffer读，向target写
                target.put(byteBuffer);
                //打印
                ByteBufferUtil.debugAll(target);
                byteBuffer.limit(limit);
            }
        }
        //切换到写模式，未读完的部分继续
        byteBuffer.compact();
    }
}
```



运行结果：

```sh
+--------+-------------------- all ------------------------+----------------+
position: [12], limit: [12]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 48 65 6c 6c 6f 2c 77 6f 72 6c 64 0a             |Hello,world.    |
+--------+-------------------------------------------------+----------------+
2023-03-08  21:58:18.919  [main] DEBUG mao.t4.StickyAndHalfPackedTest:  24
+--------+-------------------- all ------------------------+----------------+
position: [13], limit: [13]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 49 27 6d 20 7a 68 61 6e 67 73 61 6e 0a          |I'm zhangsan.   |
+--------+-------------------------------------------------+----------------+
2023-03-08  21:58:18.919  [main] DEBUG mao.t4.StickyAndHalfPackedTest:  12
+--------+-------------------- all ------------------------+----------------+
position: [13], limit: [13]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 48 6f 77 20 61 72 65 20 79 6f 75 3f 0a          |How are you?.   |
+--------+-------------------------------------------------+----------------+
2023-03-08  21:58:18.920  [main] DEBUG mao.t4.StickyAndHalfPackedTest:  18
+--------+-------------------- all ------------------------+----------------+
position: [6], limit: [6]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 61 68 61 21 0a                               |haha!.          |
+--------+-------------------------------------------------+----------------+
```













## 文件编程

### FileChannel

FileChannel 只能工作在阻塞模式下



#### 获取

不能直接打开 FileChannel，必须通过 FileInputStream、FileOutputStream 或者 RandomAccessFile 来获取 FileChannel，它们都有 getChannel 方法

* 通过 FileInputStream 获取的 channel 只能读
* 通过 FileOutputStream 获取的 channel 只能写
* 通过 RandomAccessFile 是否能读写根据构造 RandomAccessFile 时的读写模式决定





```java
package mao.t1;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t1
 * Class(类名): FileChannelGetTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/8
 * Time(创建时间)： 22:10
 * Version(版本): 1.0
 * Description(描述)： 得到FileChannel
 */

public class FileChannelGetTest
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(FileChannelGetTest.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args)
    {
        //第一种方法，通过 FileInputStream 获取
        try (FileInputStream fileInputStream = new FileInputStream("test.txt"))
        {
            FileChannel fileChannel = fileInputStream.getChannel();
            log.debug(fileChannel.toString());
            //尝试读
            ByteBuffer byteBuffer = ByteBuffer.allocate(10);
            fileChannel.read(byteBuffer);
            byteBuffer.flip();
            byte b = byteBuffer.get(2);
            log.debug(String.valueOf((char) b));
            //打印
            ByteBufferUtil.debugAll(byteBuffer);

            //尝试写
            byteBuffer.clear();
            byteBuffer.put("abc".getBytes(StandardCharsets.UTF_8));
            //打印
            ByteBufferUtil.debugAll(byteBuffer);
            try
            {
                fileChannel.write(byteBuffer);
            }
            catch (Exception e)
            {
                log.error("写入失败：", e);
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }




        //第二种方法，通过 FileOutputStream 获取
        try (FileOutputStream fileOutputStream = new FileOutputStream("test.txt"))
        {
            FileChannel fileChannel = fileOutputStream.getChannel();
            log.debug(fileChannel.toString());
            //尝试读
            ByteBuffer byteBuffer = ByteBuffer.allocate(10);
            try
            {
                fileChannel.read(byteBuffer);
                byteBuffer.flip();
                byte b = byteBuffer.get(2);
                log.debug(String.valueOf((char) b));
                //打印
                ByteBufferUtil.debugAll(byteBuffer);
            }
            catch (Exception e)
            {
                log.error("读失败：", e);
            }

            //尝试写
            byteBuffer.clear();
            byteBuffer.put("abc".getBytes(StandardCharsets.UTF_8));
            //打印
            ByteBufferUtil.debugAll(byteBuffer);
            try
            {
                //切换读模式
                byteBuffer.flip();
                fileChannel.write(byteBuffer);
            }
            catch (Exception e)
            {
                log.error("写入失败：", e);
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }



       //第三种方法，通过 FileInputStream 获取
        try (RandomAccessFile randomAccessFile = new RandomAccessFile("test.txt","rw"))
        {
            FileChannel fileChannel = randomAccessFile.getChannel();
            log.debug(fileChannel.toString());
            //尝试读
            ByteBuffer byteBuffer = ByteBuffer.allocate(10);
            try
            {
                fileChannel.read(byteBuffer);
                byteBuffer.flip();
                byte b = byteBuffer.get(2);
                log.debug(String.valueOf((char) b));
                //打印
                ByteBufferUtil.debugAll(byteBuffer);
            }
            catch (Exception e)
            {
                log.error("读失败：", e);
            }

            //尝试写
            byteBuffer.clear();
            byteBuffer.put("abcdefg".getBytes(StandardCharsets.UTF_8));
            //打印
            ByteBufferUtil.debugAll(byteBuffer);
            try
            {
                //切换读模式
                byteBuffer.flip();
                fileChannel.write(byteBuffer);
            }
            catch (Exception e)
            {
                log.error("写入失败：", e);
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```



运行结果：

```sh
2023-03-08  22:36:45.992  [main] DEBUG mao.t1.FileChannelGetTest:  sun.nio.ch.FileChannelImpl@22ef9844
2023-03-08  22:36:45.993  [main] DEBUG mao.t1.FileChannelGetTest:  3
2023-03-08  22:36:45.995  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [8]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 0d 0a 00 00                   |123456....      |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [3], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 61 62 63 34 35 36 0d 0a 00 00                   |abc456....      |
+--------+-------------------------------------------------+----------------+
2023-03-08  22:36:45.999  [main] ERROR mao.t1.FileChannelGetTest:  写入失败：
java.nio.channels.NonWritableChannelException: null
	at sun.nio.ch.FileChannelImpl.write(FileChannelImpl.java:273) ~[?:?]
	at mao.t1.FileChannelGetTest.main(FileChannelGetTest.java:62) [classes/:?]
2023-03-08  22:36:46.007  [main] DEBUG mao.t1.FileChannelGetTest:  sun.nio.ch.FileChannelImpl@73e9cf30
2023-03-08  22:36:46.007  [main] ERROR mao.t1.FileChannelGetTest:  读失败：
java.nio.channels.NonReadableChannelException: null
	at sun.nio.ch.FileChannelImpl.read(FileChannelImpl.java:217) ~[?:?]
	at mao.t1.FileChannelGetTest.main(FileChannelGetTest.java:86) [classes/:?]
+--------+-------------------- all ------------------------+----------------+
position: [3], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 61 62 63 00 00 00 00 00 00 00                   |abc.......      |
+--------+-------------------------------------------------+----------------+
2023-03-08  22:36:46.007  [main] DEBUG mao.t1.FileChannelGetTest:  sun.nio.ch.FileChannelImpl@242b836
2023-03-08  22:36:46.008  [main] DEBUG mao.t1.FileChannelGetTest:  c
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [3]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 61 62 63 00 00 00 00 00 00 00                   |abc.......      |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [7], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 61 62 63 64 65 66 67 00 00 00                   |abcdefg...      |
+--------+-------------------------------------------------+----------------+
```







#### 读取

会从 channel 读取数据填充 ByteBuffer，返回值表示读到了多少字节，-1 表示到达了文件的末尾



```java
package mao.t1;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t1
 * Class(类名): FileChannelReadTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/8
 * Time(创建时间)： 22:39
 * Version(版本): 1.0
 * Description(描述)： FileChannel读
 */

public class FileChannelReadTest
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(FileChannelReadTest.class);


    public static void main(String[] args)
    {
        try (RandomAccessFile randomAccessFile = new RandomAccessFile("test.txt", "rw"))
        {
            FileChannel fileChannel = randomAccessFile.getChannel();
            ByteBuffer byteBuffer = ByteBuffer.allocate(16);
            //读
            int length = fileChannel.read(byteBuffer);
            log.debug("长度：" + length);
            ByteBufferUtil.debugAll(byteBuffer);
            length = fileChannel.read(byteBuffer);
            log.debug("长度：" + length);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```



运行结果：

```sh
2023-03-10  22:51:57.481  [main] DEBUG mao.t1.FileChannelReadTest:  长度：10
2023-03-10  22:51:57.484  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [10], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 61 62 63 61 62 63 64 65 66 67 00 00 00 00 00 00 |abcabcdefg......|
+--------+-------------------------------------------------+----------------+
2023-03-10  22:51:57.490  [main] DEBUG mao.t1.FileChannelReadTest:  长度：-1
```







#### 写入

```java
package mao.t1;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t1
 * Class(类名): FileChannelWriteTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/10
 * Time(创建时间)： 22:54
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class FileChannelWriteTest
{

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(FileChannelWriteTest.class);

    public static void main(String[] args)
    {
        try (RandomAccessFile randomAccessFile = new RandomAccessFile("test.txt", "rw"))
        {
            FileChannel fileChannel = randomAccessFile.getChannel();
            ByteBuffer byteBuffer = ByteBuffer.allocate(16);
            byteBuffer.put("hello.".getBytes(StandardCharsets.UTF_8));
            ByteBufferUtil.debugAll(byteBuffer);
            //切换读模式
            byteBuffer.flip();
            ByteBufferUtil.debugAll(byteBuffer);
            //写
            if (byteBuffer.hasRemaining())
            {
                fileChannel.write(byteBuffer);
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```



运行结果：

```sh
+--------+-------------------- all ------------------------+----------------+
position: [6], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 2e 00 00 00 00 00 00 00 00 00 00 |hello...........|
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [6]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 2e 00 00 00 00 00 00 00 00 00 00 |hello...........|
+--------+-------------------------------------------------+----------------+
```



文件内容：

```sh
hello.defg
```





#### 关闭

channel 必须关闭，不过调用了 FileInputStream、FileOutputStream 或者 RandomAccessFile 的 close 方法会间接地调用 channel 的 close 方法





#### 位置

获取当前位置

```sh
channel.position();
```



设置当前位置

```sh
channel.position(newPos);
```





设置当前位置时，如果设置为文件的末尾

* 这时读取会返回 -1 
* 这时写入，会追加内容，但要注意如果 position 超过了文件末尾，再写入时在新内容和原末尾之间会有空洞（00）



```java
package mao.t1;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t1
 * Class(类名): FileChannelPositionTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/10
 * Time(创建时间)： 23:01
 * Version(版本): 1.0
 * Description(描述)： 位置操作
 */

public class FileChannelPositionTest
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(FileChannelPositionTest.class);

    public static void main(String[] args)
    {
        try (RandomAccessFile randomAccessFile = new RandomAccessFile("test.txt", "rw"))
        {
            FileChannel fileChannel = randomAccessFile.getChannel();
            ByteBuffer byteBuffer = ByteBuffer.allocate(16);
            System.out.println("当前位置：" + fileChannel.position());
            byteBuffer.put("123456789456".getBytes(StandardCharsets.UTF_8));
            ByteBufferUtil.debugAll(byteBuffer);
            byteBuffer.flip();
            fileChannel.write(byteBuffer);
            System.out.println("当前位置：" + fileChannel.position());

            byteBuffer = ByteBuffer.allocate(16);
            byteBuffer.put("abc".getBytes(StandardCharsets.UTF_8));
            ByteBufferUtil.debugAll(byteBuffer);
            byteBuffer.flip();
            fileChannel.write(byteBuffer);
            System.out.println("当前位置：" + fileChannel.position());

            //设置位置
            fileChannel.position(20);

            byteBuffer = ByteBuffer.allocate(16);
            byteBuffer.put("def".getBytes(StandardCharsets.UTF_8));
            ByteBufferUtil.debugAll(byteBuffer);
            byteBuffer.flip();
            fileChannel.write(byteBuffer);
            System.out.println("当前位置：" + fileChannel.position());

            //设置位置
            fileChannel.position(3);

            byteBuffer = ByteBuffer.allocate(16);
            byteBuffer.put("-".getBytes(StandardCharsets.UTF_8));
            ByteBufferUtil.debugAll(byteBuffer);
            byteBuffer.flip();
            fileChannel.write(byteBuffer);
            System.out.println("当前位置：" + fileChannel.position());
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```



运行结果：

```sh
当前位置：0
2023-03-10  23:11:23.139  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [12], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 34 35 36 00 00 00 00 |123456789456....|
+--------+-------------------------------------------------+----------------+
当前位置：12
+--------+-------------------- all ------------------------+----------------+
position: [3], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 61 62 63 00 00 00 00 00 00 00 00 00 00 00 00 00 |abc.............|
+--------+-------------------------------------------------+----------------+
当前位置：15
+--------+-------------------- all ------------------------+----------------+
position: [3], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 64 65 66 00 00 00 00 00 00 00 00 00 00 00 00 00 |def.............|
+--------+-------------------------------------------------+----------------+
当前位置：23
+--------+-------------------- all ------------------------+----------------+
position: [1], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2d 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |-...............|
+--------+-------------------------------------------------+----------------+
当前位置：4
```



文件内容：

```sh
123-56789456abc     def
```



![image-20230310231309092](img/Netty学习笔记/image-20230310231309092.png)







#### 大小

```java
package mao.t1;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t1
 * Class(类名): FileChannelSizeTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/10
 * Time(创建时间)： 23:13
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class FileChannelSizeTest
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(FileChannelSizeTest.class);

    public static void main(String[] args)
    {
        try (RandomAccessFile randomAccessFile = new RandomAccessFile("test.txt", "rw"))
        {
            FileChannel fileChannel = randomAccessFile.getChannel();
            ByteBuffer byteBuffer = ByteBuffer.allocate(16);

            log.debug("文件大小：" + fileChannel.size());

            byteBuffer.put("def".getBytes(StandardCharsets.UTF_8));
            ByteBufferUtil.debugAll(byteBuffer);
            byteBuffer.flip();
            fileChannel.write(byteBuffer);
            log.debug("文件大小：" + fileChannel.size());


            //设置位置
            fileChannel.position(30);

            byteBuffer = ByteBuffer.allocate(16);
            byteBuffer.put("-".getBytes(StandardCharsets.UTF_8));
            ByteBufferUtil.debugAll(byteBuffer);
            byteBuffer.flip();
            fileChannel.write(byteBuffer);
            log.debug("文件大小：" + fileChannel.size());
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```



运行结果：

```sh
2023-03-10  23:16:55.236  [main] DEBUG mao.t1.FileChannelSizeTest:  文件大小：0
2023-03-10  23:16:55.239  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [3], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 64 65 66 00 00 00 00 00 00 00 00 00 00 00 00 00 |def.............|
+--------+-------------------------------------------------+----------------+
2023-03-10  23:16:55.243  [main] DEBUG mao.t1.FileChannelSizeTest:  文件大小：3
+--------+-------------------- all ------------------------+----------------+
position: [1], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2d 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |-...............|
+--------+-------------------------------------------------+----------------+
2023-03-10  23:16:55.243  [main] DEBUG mao.t1.FileChannelSizeTest:  文件大小：31
```







#### 强制写入

操作系统出于性能的考虑，会将数据缓存，不是立刻写入磁盘。可以调用 force(true)  方法将文件内容和元数据（文件的权限等信息）立刻写入磁盘







### 两个 Channel 传输数据



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.nio.channels.FileChannel;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t2
 * Class(类名): FileChannelTransferToTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/10
 * Time(创建时间)： 23:25
 * Version(版本): 1.0
 * Description(描述)： 文件拷贝测试
 */

public class FileChannelTransferToTest
{
    private static final Logger log = LoggerFactory.getLogger(FileChannelTransferToTest.class);

    public static void main(String[] args)
    {
        String inputFilePath = "./input.txt";
        String outputFilePath = "./output.txt";

        //准备输入文件
        writeInputFile(inputFilePath);

        try (FileChannel fileInputChannel = new FileInputStream(inputFilePath).getChannel();
             FileChannel fileOutputChannel = new FileOutputStream(outputFilePath).getChannel())
        {
            log.info("开始拷贝");
            long start = System.currentTimeMillis();
            //拷贝
            fileInputChannel.transferTo(0, fileInputChannel.size(), fileOutputChannel);
            log.info("拷贝完成，耗时" + (System.currentTimeMillis() - start) + "毫秒");
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }


    /**
     * 写输入文件
     *
     * @param inputFilePath 输入文件路径
     */
    private static void writeInputFile(String inputFilePath)
    {
        long start = System.currentTimeMillis();
        try (FileOutputStream fileOutputStream = new FileOutputStream(inputFilePath))
        {
            byte[] bytes = "1234567890".getBytes(StandardCharsets.UTF_8);
            for (int i = 0; i < 1000000; i++)
            {
                fileOutputStream.write(bytes);
            }
            log.info("文件准备完成，耗时" + (System.currentTimeMillis() - start) + "毫秒");
        }
        catch (Exception e)
        {
            log.error("失败：", e);
        }
    }
}
```



运行结果：

```sh
2023-03-10  23:41:02.050  [main] INFO  mao.t2.FileChannelTransferToTest:  文件准备完成，耗时1301毫秒
2023-03-10  23:41:02.064  [main] INFO  mao.t2.FileChannelTransferToTest:  开始拷贝
2023-03-10  23:41:02.069  [main] INFO  mao.t2.FileChannelTransferToTest:  拷贝完成，耗时5毫秒
```



**注意：一次不能拷贝超过2个G的大小**，如果文件超过2G，可以考虑分多次拷贝







### Path

jdk7 引入了 Path 和 Paths 类

* Path 用来表示文件路径
* Paths 是工具类，用来获取 Path 实例



```java
package mao.t3;

import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t3
 * Class(类名): PathTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/11
 * Time(创建时间)： 21:28
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class PathTest
{
    public static void main(String[] args)
    {
        //得到path
        Path path = Paths.get("./test.txt");
        System.out.println(path);
        Path path2 = Paths.get("./", "test.txt");
        System.out.println(path2);
        //文件系统
        System.out.println(path.getFileSystem());
        //比较
        System.out.println(path.compareTo(path2));
        //判断文件名
        System.out.println(path.endsWith("test.txt"));
        System.out.println(path.endsWith("est.txt"));
        System.out.println(path.endsWith("txt"));

        System.out.println(path.startsWith("./"));
        System.out.println(path.startsWith("D:\\"));

        //得到文件名
        System.out.println(path.getFileName());

        //路径中的元素数，如果此路径仅表示根组件，则为 0
        System.out.println(path.getNameCount());

        //得到上级
        System.out.println(path.getParent());

        //将此路径的根组件作为 Path 对象返回，如果此路径没有根组件，则返回 null。
        System.out.println(path.getRoot());

        //说明此路径是否为绝对路径。
        //绝对路径是完整的，因为它不需要与其他路径信息组合即可找到文件。
        System.out.println(path.isAbsolute());

        //转绝对路径
        System.out.println(path = path.toAbsolutePath());

        //说明此路径是否为绝对路径。
        //绝对路径是完整的，因为它不需要与其他路径信息组合即可找到文件。
        System.out.println(path.isAbsolute());

        //正常化路径
        System.out.println(path.normalize());
    }
}
```



运行结果：

```sh
.\test.txt
.\test.txt
sun.nio.fs.WindowsFileSystem@2d98a335
0
true
false
false
true
false
test.txt
2
.
null
false
D:\程序\大四下期\Netty_File_Programming\.\test.txt
true
D:\程序\大四下期\Netty_File_Programming\test.txt
```







### Files

#### 检查文件是否存在

```java
Files.exists(path)
```



```java
package mao.t4;


import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t4
 * Class(类名): FilesTest1
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/11
 * Time(创建时间)： 21:48
 * Version(版本): 1.0
 * Description(描述)： 检查文件是否存在
 */

public class FilesTest1
{
    public static void main(String[] args)
    {
        Path path1 = Paths.get("./test.txt");
        Path path2 = Paths.get("./test2.txt");
        //检查文件是否存在
        System.out.println(Files.exists(path1));
        System.out.println(Files.exists(path2));
    }
}
```



运行结果：

```sh
true
false
```





#### 创建一级目录

```java
Files.createDirectory(path);
```



```java
package mao.t4;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t4
 * Class(类名): FilesTest2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/11
 * Time(创建时间)： 21:51
 * Version(版本): 1.0
 * Description(描述)： 创建一级目录
 */

public class FilesTest2
{
    public static void main(String[] args) throws IOException
    {
        Path path = Paths.get("./abc");
        //创建一级目录
        Path directory = Files.createDirectory(path);
        System.out.println(directory);
    }
}
```



运行结果：

```sh
.\abc
```



![image-20230311215453524](img/Netty学习笔记/image-20230311215453524.png)



* 如果目录已存在，会抛异常 FileAlreadyExistsException
* 不能一次创建多级目录，否则会抛异常 NoSuchFileException



```java
package mao.t4;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t4
 * Class(类名): FilesTest2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/11
 * Time(创建时间)： 21:51
 * Version(版本): 1.0
 * Description(描述)： 创建一级目录
 */

public class FilesTest2
{
    public static void main(String[] args) throws IOException
    {
        Path path = Paths.get("./abc");
        //创建一级目录
        Path directory = Files.createDirectory(path);
        System.out.println(directory);

        try
        {
            //如果目录已存在，会抛异常 FileAlreadyExistsException
            path = Paths.get("./abc");
            //创建一级目录
            directory = Files.createDirectory(path);
            System.out.println(directory);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }

        try
        {
            //不能一次创建多级目录，否则会抛异常NoSuchFileException
            path = Paths.get("./abc/de/f/gh");
            //创建一级目录
            directory = Files.createDirectory(path);
            System.out.println(directory);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```



运行结果：

```sh
.\abc
java.nio.file.FileAlreadyExistsException: .\abc
	at java.base/sun.nio.fs.WindowsException.translateToIOException(WindowsException.java:87)
	at java.base/sun.nio.fs.WindowsException.rethrowAsIOException(WindowsException.java:103)
	at java.base/sun.nio.fs.WindowsException.rethrowAsIOException(WindowsException.java:108)
	at java.base/sun.nio.fs.WindowsFileSystemProvider.createDirectory(WindowsFileSystemProvider.java:519)
	at java.base/java.nio.file.Files.createDirectory(Files.java:694)
	at mao.t4.FilesTest2.main(FilesTest2.java:35)
java.nio.file.NoSuchFileException: .\abc\de\f\gh
	at java.base/sun.nio.fs.WindowsException.translateToIOException(WindowsException.java:85)
	at java.base/sun.nio.fs.WindowsException.rethrowAsIOException(WindowsException.java:103)
	at java.base/sun.nio.fs.WindowsException.rethrowAsIOException(WindowsException.java:108)
	at java.base/sun.nio.fs.WindowsFileSystemProvider.createDirectory(WindowsFileSystemProvider.java:519)
	at java.base/java.nio.file.Files.createDirectory(Files.java:694)
	at mao.t4.FilesTest2.main(FilesTest2.java:48)
```







#### 创建多级目录

```java
Files.createDirectories(path);
```



```java
package mao.t4;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t4
 * Class(类名): FilesTest3
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/11
 * Time(创建时间)： 21:58
 * Version(版本): 1.0
 * Description(描述)： 创建多级目录
 */

public class FilesTest3
{
    public static void main(String[] args) throws IOException
    {
        Path path = Paths.get("./abc/d/e/f/g/h");
        //创建多级目录
        Path directories = Files.createDirectories(path);
        System.out.println(directories);

        //再次创建
        path = Paths.get("./abc/d/e/f/g/h");
        directories = Files.createDirectories(path);
        System.out.println(directories);
    }
}
```



运行结果：

```sh
D:\程序\大四下期\Netty_File_Programming\.\abc\d\e\f\g\h
.\abc\d\e\f\g\h
```



![image-20230311220100652](img/Netty学习笔记/image-20230311220100652.png)







#### 拷贝文件

```sh
Files.copy(source, target);
```



如果文件已存在，会抛异常 FileAlreadyExistsException



如果希望用 source 覆盖掉 target，需要用 StandardCopyOption 来控制

```sh
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
```





```java
package mao.t4;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t4
 * Class(类名): FilesTest4
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/11
 * Time(创建时间)： 22:02
 * Version(版本): 1.0
 * Description(描述)： 拷贝文件
 */

public class FilesTest4
{
    public static void main(String[] args) throws InterruptedException
    {
        Path path1 = Paths.get("./test.txt");
        Path path2 = Paths.get("./test2.txt");

        try
        {
            //拷贝文件
            Path path = Files.copy(path1, path2);
            System.out.println(path);
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }

        try
        {
            //再次尝试拷贝文件，如果文件已存在，会抛异常 FileAlreadyExistsException
            Files.copy(path1, path2);
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }

        Thread.sleep(100);

        try
        {
            //拷贝文件
            Path path = Files.copy(path1, path2, StandardCopyOption.REPLACE_EXISTING);
            System.out.println("文件覆盖成功");
            System.out.println(path);
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
    }
}

```



运行结果：

```sh
.\test2.txt
java.nio.file.FileAlreadyExistsException: .\test2.txt
	at java.base/sun.nio.fs.WindowsFileCopy.copy(WindowsFileCopy.java:123)
	at java.base/sun.nio.fs.WindowsFileSystemProvider.copy(WindowsFileSystemProvider.java:284)
	at java.base/java.nio.file.Files.copy(Files.java:1299)
	at mao.t4.FilesTest4.main(FilesTest4.java:43)
文件覆盖成功
.\test2.txt
```







#### 移动文件

```java
Files.move(path, path1);
```

StandardCopyOption.ATOMIC_MOVE 保证文件移动的原子性



```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t4
 * Class(类名): FilesTest6
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/12
 * Time(创建时间)： 14:28
 * Version(版本): 1.0
 * Description(描述)： 移动文件
 */

public class FilesTest6
{
    private static final Logger log = LoggerFactory.getLogger(FilesTest6.class);

    public static void main(String[] args) throws IOException
    {
        Path path = Paths.get("test.txt");
        Path path1 = Paths.get("test3.txt");
        log.info(path.toString() + " to " + path1.toString());
        Path path2 = Files.move(path, path1);
        log.info(path2.toString());
    }
}
```



运行结果：

```sh
2023-03-12  14:31:35.995  [main] INFO  mao.t4.FilesTest6:  test.txt to test3.txt
2023-03-12  14:31:35.997  [main] INFO  mao.t4.FilesTest6:  test3.txt
```







#### 删除文件

```java
Files.delete(target);
```



如果文件不存在，会抛异常 NoSuchFileException



```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t4
 * Class(类名): FilesTest7
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/12
 * Time(创建时间)： 14:34
 * Version(版本): 1.0
 * Description(描述)： 删除文件
 */

public class FilesTest7
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(FilesTest7.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args)
    {
        Path path = Paths.get("./test3.txt");
        try
        {
            Files.delete(path);
            log.info("第一次删除成功");
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }

        try
        {
            //如果文件不存在，会抛异常 NoSuchFileException
            Files.delete(path);
            log.info("第二次删除成功");
        }
        catch (Exception e)
        {
            log.info("第二次删除失败");
            e.printStackTrace();
        }
    }
}
```



运行结果：

```sh
2023-03-12  14:37:43.127  [main] INFO  mao.t4.FilesTest7:  第一次删除成功
2023-03-12  14:37:43.129  [main] INFO  mao.t4.FilesTest7:  第二次删除失败
java.nio.file.NoSuchFileException: .\test3.txt
	at java.base/sun.nio.fs.WindowsException.translateToIOException(WindowsException.java:85)
	at java.base/sun.nio.fs.WindowsException.rethrowAsIOException(WindowsException.java:103)
	at java.base/sun.nio.fs.WindowsException.rethrowAsIOException(WindowsException.java:108)
	at java.base/sun.nio.fs.WindowsFileSystemProvider.implDelete(WindowsFileSystemProvider.java:275)
	at java.base/sun.nio.fs.AbstractFileSystemProvider.delete(AbstractFileSystemProvider.java:105)
	at java.base/java.nio.file.Files.delete(Files.java:1146)
	at mao.t4.FilesTest7.main(FilesTest7.java:51)
```







#### 遍历目录文件

```java
Files.walkFileTree(path, new SimpleFileVisitor<Path>()
```



```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t4
 * Class(类名): FilesTest8
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/12
 * Time(创建时间)： 14:39
 * Version(版本): 1.0
 * Description(描述)： 遍历目录文件
 */

public class FilesTest8
{

    private static final Logger log = LoggerFactory.getLogger(FilesTest8.class);

    public static void main(String[] args) throws IOException
    {
        Path path = Paths.get("./");
        path = path.toAbsolutePath();
        path = path.normalize();
        log.info("遍历的目录：" + path);
        Files.walkFileTree(path, new SimpleFileVisitor<Path>()
        {
            /**
             * 访问目录之前的回调方法
             *
             * @param dir   dir
             * @param attrs attrs
             * @return {@link FileVisitResult}
             * @throws IOException ioexception
             */
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException
            {
                if (dir.endsWith(".git"))
                {
                    log.info("去除.git目录：" + dir);
                    return FileVisitResult.SKIP_SUBTREE;
                }
                log.info("-------进入目录：" + dir);
                return FileVisitResult.CONTINUE;
            }

            /**
             * 访问文件
             *
             * @param file  文件
             * @param attrs attrs
             * @return {@link FileVisitResult}
             * @throws IOException ioexception
             */
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException
            {
                log.info(file.toString());
                return FileVisitResult.CONTINUE;
            }

            /**
             * 访问文件失败
             *
             * @param file 文件
             * @param exc  exc
             * @return {@link FileVisitResult}
             * @throws IOException ioexception
             */
            @Override
            public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException
            {
                log.warn("访问文件失败:" + file);
                return FileVisitResult.CONTINUE;
            }

            /**
             * 访问目录之后的回调方法
             *
             * @param dir dir
             * @param exc exc
             * @return {@link FileVisitResult}
             * @throws IOException ioexception
             */
            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException
            {
                log.info("-------退出目录：" + dir);
                return FileVisitResult.CONTINUE;
            }
        });
    }
}
```



运行结果：

```sh
2023-03-12  14:54:23.163  [main] INFO  mao.t4.FilesTest8:  遍历的目录：D:\程序\大四下期\Netty_File_Programming
2023-03-12  14:54:23.174  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming
2023-03-12  14:54:23.174  [main] INFO  mao.t4.FilesTest8:  去除.git目录：D:\程序\大四下期\Netty_File_Programming\.git
2023-03-12  14:54:23.174  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\.idea
2023-03-12  14:54:23.175  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\.idea\.gitignore
2023-03-12  14:54:23.175  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\.idea\compiler.xml
2023-03-12  14:54:23.175  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\.idea\dictionaries
2023-03-12  14:54:23.175  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\.idea\encodings.xml
2023-03-12  14:54:23.175  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\.idea\git_toolbox_prj.xml
2023-03-12  14:54:23.175  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\.idea\intellij-javadocs-4.0.1.xml
2023-03-12  14:54:23.175  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\.idea\jarRepositories.xml
2023-03-12  14:54:23.175  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\.idea\misc.xml
2023-03-12  14:54:23.175  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\.idea\runConfigurations.xml
2023-03-12  14:54:23.175  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\.idea\vcs.xml
2023-03-12  14:54:23.175  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\.idea\workspace.xml
2023-03-12  14:54:23.176  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\.idea
2023-03-12  14:54:23.176  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\abc
2023-03-12  14:54:23.176  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\abc\d
2023-03-12  14:54:23.176  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\abc\d\e
2023-03-12  14:54:23.176  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\abc\d\e\f
2023-03-12  14:54:23.176  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\abc\d\e\f\g
2023-03-12  14:54:23.177  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\abc\d\e\f\g\h
2023-03-12  14:54:23.177  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\abc\d\e\f\g\h
2023-03-12  14:54:23.177  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\abc\d\e\f\g
2023-03-12  14:54:23.177  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\abc\d\e\f
2023-03-12  14:54:23.177  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\abc\d\e
2023-03-12  14:54:23.177  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\abc\d
2023-03-12  14:54:23.177  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\abc
2023-03-12  14:54:23.177  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\pom.xml
2023-03-12  14:54:23.177  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src\main
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src\main\java
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1\FileChannelGetTest.java
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1\FileChannelPositionTest.java
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1\FileChannelReadTest.java
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1\FileChannelSizeTest.java
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1\FileChannelWriteTest.java
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t2
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t2\FileChannelTransferToTest.java
2023-03-12  14:54:23.178  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t2
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t3
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t3\PathTest.java
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t3
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest1.java
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest2.java
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest3.java
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest4.java
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest5.java
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest6.java
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest7.java
2023-03-12  14:54:23.180  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest8.java
2023-03-12  14:54:23.181  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4
2023-03-12  14:54:23.181  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\utils
2023-03-12  14:54:23.181  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\utils\ByteBufferUtil.java
2023-03-12  14:54:23.181  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\utils
2023-03-12  14:54:23.181  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src\main\java\mao
2023-03-12  14:54:23.181  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src\main\java
2023-03-12  14:54:23.181  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src\main\resources
2023-03-12  14:54:23.181  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\src\main\resources\log4j2.xml
2023-03-12  14:54:23.181  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src\main\resources
2023-03-12  14:54:23.182  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src\main
2023-03-12  14:54:23.182  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src\test
2023-03-12  14:54:23.182  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\src\test\java
2023-03-12  14:54:23.182  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src\test\java
2023-03-12  14:54:23.182  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src\test
2023-03-12  14:54:23.182  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\src
2023-03-12  14:54:23.183  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\target
2023-03-12  14:54:23.183  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\target\classes
2023-03-12  14:54:23.183  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\log4j2.xml
2023-03-12  14:54:23.183  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao
2023-03-12  14:54:23.183  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t1
2023-03-12  14:54:23.183  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t1\FileChannelGetTest.class
2023-03-12  14:54:23.183  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t1\FileChannelPositionTest.class
2023-03-12  14:54:23.183  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t1\FileChannelReadTest.class
2023-03-12  14:54:23.183  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t1\FileChannelSizeTest.class
2023-03-12  14:54:23.183  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t1\FileChannelWriteTest.class
2023-03-12  14:54:23.184  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t1
2023-03-12  14:54:23.184  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t2
2023-03-12  14:54:23.184  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t2\FileChannelTransferToTest.class
2023-03-12  14:54:23.184  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t2
2023-03-12  14:54:23.184  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t3
2023-03-12  14:54:23.184  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t3\PathTest.class
2023-03-12  14:54:23.184  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t3
2023-03-12  14:54:23.185  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t4
2023-03-12  14:54:23.185  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t4\FilesTest1.class
2023-03-12  14:54:23.185  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t4\FilesTest2.class
2023-03-12  14:54:23.185  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t4\FilesTest3.class
2023-03-12  14:54:23.185  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t4\FilesTest4.class
2023-03-12  14:54:23.185  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t4\FilesTest5.class
2023-03-12  14:54:23.185  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t4\FilesTest6.class
2023-03-12  14:54:23.185  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t4\FilesTest7.class
2023-03-12  14:54:23.185  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t4\FilesTest8$1.class
2023-03-12  14:54:23.185  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t4\FilesTest8.class
2023-03-12  14:54:23.185  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao\t4
2023-03-12  14:54:23.186  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao\utils
2023-03-12  14:54:23.186  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\target\classes\mao\utils\ByteBufferUtil.class
2023-03-12  14:54:23.186  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao\utils
2023-03-12  14:54:23.186  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\target\classes\mao
2023-03-12  14:54:23.186  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\target\classes
2023-03-12  14:54:23.186  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\target\generated-sources
2023-03-12  14:54:23.186  [main] INFO  mao.t4.FilesTest8:  -------进入目录：D:\程序\大四下期\Netty_File_Programming\target\generated-sources\annotations
2023-03-12  14:54:23.186  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\target\generated-sources\annotations
2023-03-12  14:54:23.186  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\target\generated-sources
2023-03-12  14:54:23.186  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming\target
2023-03-12  14:54:23.187  [main] INFO  mao.t4.FilesTest8:  D:\程序\大四下期\Netty_File_Programming\test2.txt
2023-03-12  14:54:23.187  [main] INFO  mao.t4.FilesTest8:  -------退出目录：D:\程序\大四下期\Netty_File_Programming
```







#### 统计某一类型文件的数目



```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t4
 * Class(类名): FilesTest9
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/12
 * Time(创建时间)： 14:59
 * Version(版本): 1.0
 * Description(描述)： 统计某一类型文件的数目
 */

public class FilesTest9
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(FilesTest9.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException
    {
        Path path = Paths.get("./");
        path = path.toAbsolutePath();
        path = path.normalize();
        log.info("统计.java文件的数量");
        AtomicInteger atomicInteger = new AtomicInteger();

        Files.walkFileTree(path, new FileVisitor<Path>()
        {
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException
            {
                return FileVisitResult.CONTINUE;
            }

            /**
             * 访问文件
             *
             * @param file  文件
             * @param attrs attrs
             * @return {@link FileVisitResult}
             * @throws IOException ioexception
             */
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException
            {
                if (file.toFile().getName().endsWith(".java"))
                {
                    log.debug(file.toString());
                    atomicInteger.incrementAndGet();
                }
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException
            {
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException
            {
                return FileVisitResult.CONTINUE;
            }
        });

        log.info("总数量：" + atomicInteger.get());
    }
}
```



运行结果：

```sh
2023-03-12  15:03:41.242  [main] INFO  mao.t4.FilesTest9:  统计.java文件的数量
2023-03-12  15:03:41.262  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1\FileChannelGetTest.java
2023-03-12  15:03:41.262  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1\FileChannelPositionTest.java
2023-03-12  15:03:41.262  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1\FileChannelReadTest.java
2023-03-12  15:03:41.262  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1\FileChannelSizeTest.java
2023-03-12  15:03:41.262  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t1\FileChannelWriteTest.java
2023-03-12  15:03:41.262  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t2\FileChannelTransferToTest.java
2023-03-12  15:03:41.262  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t3\PathTest.java
2023-03-12  15:03:41.262  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest1.java
2023-03-12  15:03:41.262  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest2.java
2023-03-12  15:03:41.263  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest3.java
2023-03-12  15:03:41.263  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest4.java
2023-03-12  15:03:41.263  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest5.java
2023-03-12  15:03:41.263  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest6.java
2023-03-12  15:03:41.263  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest7.java
2023-03-12  15:03:41.263  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest8.java
2023-03-12  15:03:41.263  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\t4\FilesTest9.java
2023-03-12  15:03:41.263  [main] DEBUG mao.t4.FilesTest9:  D:\程序\大四下期\Netty_File_Programming\src\main\java\mao\utils\ByteBufferUtil.java
2023-03-12  15:03:41.264  [main] INFO  mao.t4.FilesTest9:  总数量：17
```







#### 删除多级目录

先删除目录中的文件，再删除目录

```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;

/**
 * Project name(项目名称)：Netty_File_Programming
 * Package(包名): mao.t4
 * Class(类名): FilesTest10
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/12
 * Time(创建时间)： 15:11
 * Version(版本): 1.0
 * Description(描述)： 删除多级目录
 */

public class FilesTest10
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(FilesTest10.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException
    {
        Path path = Paths.get("./0/");
        path = path.toAbsolutePath();
        path = path.normalize();
        log.info("删除目录：" + path);
        Files.walkFileTree(path, new FileVisitor<Path>()
        {
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException
            {
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException
            {
                log.debug("删除文件：" + file);
                Files.delete(file);
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException
            {
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException
            {
                log.debug("删除目录：" + dir);
                Files.delete(dir);
                return FileVisitResult.CONTINUE;
            }
        });
    }
}
```



运行结果：

```sh
2023-03-12  15:15:09.996  [main] INFO  mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0
2023-03-12  15:15:10.007  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\0\0\0
2023-03-12  15:15:10.007  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\0\0\1
2023-03-12  15:15:10.008  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\0\0
2023-03-12  15:15:10.008  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\0\1\0
2023-03-12  15:15:10.008  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\0\1\1
2023-03-12  15:15:10.008  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\0\1
2023-03-12  15:15:10.008  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\0
2023-03-12  15:15:10.008  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\1\0\0
2023-03-12  15:15:10.008  [main] DEBUG mao.t4.FilesTest10:  删除文件：D:\程序\大四下期\Netty_File_Programming\0\0\1\0\1\test2.txt
2023-03-12  15:15:10.010  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\1\0\1
2023-03-12  15:15:10.010  [main] DEBUG mao.t4.FilesTest10:  删除文件：D:\程序\大四下期\Netty_File_Programming\0\0\1\0\test2.txt
2023-03-12  15:15:10.010  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\1\0
2023-03-12  15:15:10.010  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\1\1\0
2023-03-12  15:15:10.010  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\1\1\1
2023-03-12  15:15:10.010  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\1\1
2023-03-12  15:15:10.010  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0\1
2023-03-12  15:15:10.010  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\0
2023-03-12  15:15:10.011  [main] DEBUG mao.t4.FilesTest10:  删除文件：D:\程序\大四下期\Netty_File_Programming\0\1\0\0\0\test2.txt
2023-03-12  15:15:10.011  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\0\0\0
2023-03-12  15:15:10.011  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\0\0\1
2023-03-12  15:15:10.011  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\0\0
2023-03-12  15:15:10.012  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\0\1\0
2023-03-12  15:15:10.012  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\0\1\1
2023-03-12  15:15:10.012  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\0\1
2023-03-12  15:15:10.012  [main] DEBUG mao.t4.FilesTest10:  删除文件：D:\程序\大四下期\Netty_File_Programming\0\1\0\test2.txt
2023-03-12  15:15:10.013  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\0
2023-03-12  15:15:10.013  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\1\0\0
2023-03-12  15:15:10.013  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\1\0\1
2023-03-12  15:15:10.013  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\1\0
2023-03-12  15:15:10.014  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\1\1\0
2023-03-12  15:15:10.014  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\1\1\1
2023-03-12  15:15:10.014  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\1\1
2023-03-12  15:15:10.014  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1\1
2023-03-12  15:15:10.014  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0\1
2023-03-12  15:15:10.015  [main] DEBUG mao.t4.FilesTest10:  删除文件：D:\程序\大四下期\Netty_File_Programming\0\test2.txt
2023-03-12  15:15:10.015  [main] DEBUG mao.t4.FilesTest10:  删除目录：D:\程序\大四下期\Netty_File_Programming\0
```













## 网络编程

### 非阻塞和阻塞

#### 阻塞

* 阻塞模式下，相关方法都会导致线程暂停
  * ServerSocketChannel.accept 会在没有连接建立时让线程暂停
  * SocketChannel.read 会在没有数据可读时让线程暂停
  * 阻塞的表现其实就是线程暂停了，暂停期间不会占用 cpu，但线程相当于闲置
* 单线程下，阻塞方法之间相互影响，几乎不能正常工作，需要多线程支持
* 但多线程下，有新的问题，体现在以下方面
  * 32 位 jvm 一个线程 320k，64 位 jvm 一个线程 1024k，如果连接数过多，必然导致 OOM，并且线程太多，反而会因为频繁上下文切换导致性能降低
  * 可以采用线程池技术来减少线程数和线程上下文切换，但治标不治本，如果有很多连接建立，但长时间 inactive，会阻塞线程池中所有线程，因此不适合长连接，只适合短连接



服务器端

```java
package mao.t1;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.ArrayList;
import java.util.List;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t1
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/13
 * Time(创建时间)： 19:28
 * Version(版本): 1.0
 * Description(描述)： 阻塞模式 - 服务器端
 */

public class Server
{

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Server.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException
    {
        ByteBuffer byteBuffer = ByteBuffer.allocate(16);
        //创建服务器
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //绑定
        serverSocketChannel.bind(new InetSocketAddress(8080));
        //连接集合
        List<SocketChannel> socketChannelList = new ArrayList<>();

        while (true)
        {
            log.debug("等待客户端连接...");
            SocketChannel socketChannel = serverSocketChannel.accept();
            log.debug("客户端已连接：" + socketChannel);
            //放入连接集合
            socketChannelList.add(socketChannel);

            //遍历连接集合
            for (SocketChannel channel : socketChannelList)
            {
                log.debug("等待读：" + channel);
                int read = channel.read(byteBuffer);
                byteBuffer.flip();
                ByteBufferUtil.debugAll(byteBuffer);
                byteBuffer.clear();
                log.debug("读取成功：" + channel);
            }
        }
    }
}
```





客户端

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t1
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/13
 * Time(创建时间)： 19:29
 * Version(版本): 1.0
 * Description(描述)： 阻塞模式 - 客户端
 */

public class Client
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Client.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException
    {
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.connect(new InetSocketAddress("127.0.0.1", 8080));
        Scanner input = new Scanner(System.in);
        input.nextLine();
        socketChannel.write(ByteBuffer.wrap("hello".getBytes(StandardCharsets.UTF_8)));
        input.nextLine();
        socketChannel.close();
    }
}
```



运行结果：

```sh
2023-03-13  19:55:29.913  [main] DEBUG mao.t1.Server:  等待客户端连接...
2023-03-13  19:55:34.369  [main] DEBUG mao.t1.Server:  客户端已连接：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52287]
2023-03-13  19:55:34.369  [main] DEBUG mao.t1.Server:  等待读：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52287]
2023-03-13  19:55:47.916  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
+--------+-------------------------------------------------+----------------+
2023-03-13  19:55:47.924  [main] DEBUG mao.t1.Server:  读取成功：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52287]
2023-03-13  19:55:47.924  [main] DEBUG mao.t1.Server:  等待客户端连接...
```



如果在等待客户端读的过程中有另一个客户端建立连接，那么必须要等到第一个客户端发送数据后才能建立连接



```sh
2023-03-13  19:56:50.020  [main] DEBUG mao.t1.Server:  等待客户端连接...
2023-03-13  19:56:55.019  [main] DEBUG mao.t1.Server:  客户端已连接：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52309]
2023-03-13  19:56:55.019  [main] DEBUG mao.t1.Server:  等待读：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52309]
2023-03-13  19:57:14.891  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
+--------+-------------------------------------------------+----------------+
2023-03-13  19:57:14.895  [main] DEBUG mao.t1.Server:  读取成功：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52309]
2023-03-13  19:57:14.895  [main] DEBUG mao.t1.Server:  等待客户端连接...
2023-03-13  19:57:14.895  [main] DEBUG mao.t1.Server:  客户端已连接：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52314]
2023-03-13  19:57:14.895  [main] DEBUG mao.t1.Server:  等待读：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52309]
```







#### 非阻塞

* 非阻塞模式下，相关方法都会不会让线程暂停
  * 在 ServerSocketChannel.accept 在没有连接建立时，会返回 null，继续运行
  * SocketChannel.read 在没有数据可读时，会返回 0，但线程不必阻塞，可以去执行其它 SocketChannel 的 read 或是去执行 ServerSocketChannel.accept 
  * 写数据时，线程只是等待数据写入 Channel 即可，无需等 Channel 通过网络把数据发送出去
* 但非阻塞模式下，即使没有连接建立，和可读数据，线程仍然在不断运行，白白浪费了 cpu
* 数据复制过程中，线程实际还是阻塞的（AIO 改进的地方）





服务器端

```java
package mao.t2;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.ArrayList;
import java.util.List;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t2
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/13
 * Time(创建时间)： 20:08
 * Version(版本): 1.0
 * Description(描述)： 非阻塞模式 - 服务器端
 */

public class Server
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(mao.t1.Server.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException, InterruptedException
    {
        ByteBuffer byteBuffer = ByteBuffer.allocate(16);
        //创建服务器
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //设置成非阻塞模式
        serverSocketChannel.configureBlocking(false);
        //绑定
        serverSocketChannel.bind(new InetSocketAddress(8080));
        //连接集合
        List<SocketChannel> socketChannelList = new ArrayList<>();


        while (true)
        {
            //log.debug("等待客户端连接...");
            SocketChannel socketChannel = serverSocketChannel.accept();
            if (socketChannel != null)
            {
                log.debug("客户端已连接：" + socketChannel);
                //设置成非阻塞模式
                socketChannel.configureBlocking(false);
                //放入连接集合
                socketChannelList.add(socketChannel);
            }

            //遍历连接集合
            for (SocketChannel channel : socketChannelList)
            {
                int read = channel.read(byteBuffer);
                if (read > 0)
                {
                    log.debug("等待读：" + channel);
                    byteBuffer.flip();
                    ByteBufferUtil.debugAll(byteBuffer);
                    byteBuffer.clear();
                    log.debug("读取成功：" + channel);
                }
            }
        }
    }
}
```



运行结果：

```sh
2023-03-13  20:21:36.165  [main] DEBUG mao.t1.Server:  客户端已连接：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52663]
2023-03-13  20:22:00.145  [main] DEBUG mao.t1.Server:  客户端已连接：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52674]
2023-03-13  20:22:15.488  [main] DEBUG mao.t1.Server:  等待读：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52663]
2023-03-13  20:22:15.490  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
+--------+-------------------------------------------------+----------------+
2023-03-13  20:22:15.494  [main] DEBUG mao.t1.Server:  读取成功：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52663]
2023-03-13  20:22:28.888  [main] DEBUG mao.t1.Server:  客户端已连接：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52683]
2023-03-13  20:22:35.359  [main] DEBUG mao.t1.Server:  等待读：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52683]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
+--------+-------------------------------------------------+----------------+
2023-03-13  20:22:35.359  [main] DEBUG mao.t1.Server:  读取成功：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52683]
2023-03-13  20:22:47.549  [main] DEBUG mao.t1.Server:  等待读：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52674]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
+--------+-------------------------------------------------+----------------+
2023-03-13  20:22:47.549  [main] DEBUG mao.t1.Server:  读取成功：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:52674]
```







#### 多路复用

单线程可以配合 Selector 完成对多个 Channel 可读写事件的监控，这称之为多路复用

* 多路复用仅针对网络 IO、普通文件 IO 没法利用多路复用
* 如果不用 Selector 的非阻塞模式，线程大部分时间都在做无用功，而 Selector 能够保证
  * 有可连接事件时才去连接
  * 有可读事件才去读取
  * 有可写事件才去写入
    * 限于网络传输能力，Channel 未必时时可写，一旦 Channel 可写，会触发 Selector 的可写事件









### Selector

![image-20230313203813297](img/Netty学习笔记/image-20230313203813297.png)





好处

* 一个线程配合 selector 就可以监控多个 channel 的事件，事件发生线程才去处理。避免非阻塞模式下所做无用功
* 让这个线程能够被充分利用
* 节约了线程的数量
* 减少了线程上下文切换





#### 创建

```java
Selector selector = Selector.open();
```





#### 绑定 Channel 事件

也称之为注册事件

```java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, 绑定事件);
```



* channel 必须工作在非阻塞模式
* FileChannel 没有非阻塞模式，因此不能配合 selector 一起使用
* 绑定的事件类型可以有
  * connect - 客户端连接成功时触发
  * accept - 服务器端成功接受连接时触发
  * read - 数据可读入时触发，有因为接收能力弱，数据暂不能读入的情况
  * write - 数据可写出时触发，有因为发送能力弱，数据暂不能写出的情况





#### 监听 Channel 事件

可以通过下面三种方法来监听是否有事件发生，方法的返回值代表有多少 channel 发生了事件



方法1，阻塞直到绑定事件发生

```java
int count = selector.select();
```





方法2，阻塞直到绑定事件发生，或是超时（时间单位为 ms）

```java
int count = selector.select(long timeout);
```





方法3，不会阻塞，也就是不管有没有事件，立刻返回，自己根据返回值检查是否有事件

```sh
int count = selector.selectNow();
```







**select 何时不阻塞?**

* 事件发生时
  * 客户端发起连接请求，会触发 accept 事件
  * 客户端发送数据过来，客户端正常、异常关闭时，都会触发 read 事件，另外如果发送的数据大于 buffer 缓冲区，会触发多次读取事件
  * channel 可写，会触发 write 事件
  * 在 linux 下 nio bug 发生时
* 调用 selector.wakeup()
* 调用 selector.close()
* selector 所在线程 interrupt







#### 处理 accept 事件



```java
package mao.t3;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Set;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t3
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/13
 * Time(创建时间)： 20:50
 * Version(版本): 1.0
 * Description(描述)： select - 服务端
 */

public class Server
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(mao.t3.Server.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException, InterruptedException
    {
        ByteBuffer byteBuffer = ByteBuffer.allocate(16);
        //创建服务器
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //设置成非阻塞模式
        serverSocketChannel.configureBlocking(false);
        //绑定
        serverSocketChannel.bind(new InetSocketAddress(8080));

        //Selector
        Selector selector = Selector.open();
        //注册，事件为OP_WRITE
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    serverSocketChannel.close();
                }
                catch (IOException e)
                {
                    e.printStackTrace();
                }
                try
                {
                    selector.close();
                }
                catch (IOException e)
                {
                    e.printStackTrace();
                }

            }
        }));

        while (true)
        {
            int count = selector.select();
            log.debug("事件总数：" + count);

            //获取所有事件
            Set<SelectionKey> selectionKeys = selector.selectedKeys();

            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext())
            {
                //获得SelectionKey
                SelectionKey selectionKey = iterator.next();
                //判断事件类型

                //连接服务器
                if (selectionKey.isAcceptable())
                {
                    ServerSocketChannel ssc = (ServerSocketChannel) selectionKey.channel();
                    //处理连接事件
                    SocketChannel socketChannel = ssc.accept();
                    log.debug("连接事件：" + socketChannel);
                }

                // 处理完毕，必须将事件移除
                iterator.remove();
            }
        }
    }
}
```



运行结果：

```sh
2023-03-13  21:21:39.835  [main] DEBUG mao.t3.Server:  事件总数：1
2023-03-13  21:21:39.837  [main] DEBUG mao.t3.Server:  连接事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:62918]
2023-03-13  21:21:41.854  [main] DEBUG mao.t3.Server:  事件总数：1
2023-03-13  21:21:41.854  [main] DEBUG mao.t3.Server:  连接事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:62922]
2023-03-13  21:21:47.043  [main] DEBUG mao.t3.Server:  事件总数：1
2023-03-13  21:21:47.044  [main] DEBUG mao.t3.Server:  连接事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:62927]
```







#### 处理 read 事件

```java
package mao.t3;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Set;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t3
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/13
 * Time(创建时间)： 20:50
 * Version(版本): 1.0
 * Description(描述)： select - 服务端
 */

public class Server
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(mao.t3.Server.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException, InterruptedException
    {
        ByteBuffer byteBuffer = ByteBuffer.allocate(16);
        //创建服务器
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //设置成非阻塞模式
        serverSocketChannel.configureBlocking(false);
        //绑定
        serverSocketChannel.bind(new InetSocketAddress(8080));

        //Selector
        Selector selector = Selector.open();
        //注册，事件为OP_WRITE
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    serverSocketChannel.close();
                }
                catch (IOException e)
                {
                    e.printStackTrace();
                }
                try
                {
                    selector.close();
                }
                catch (IOException e)
                {
                    e.printStackTrace();
                }

            }
        }));

        while (true)
        {
            int count = selector.select();
            log.debug("事件总数：" + count);

            //获取所有事件
            Set<SelectionKey> selectionKeys = selector.selectedKeys();

            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext())
            {
                //获得SelectionKey
                SelectionKey selectionKey = iterator.next();
                //判断事件类型

                //连接服务器
                if (selectionKey.isAcceptable())
                {
                    ServerSocketChannel ssc = (ServerSocketChannel) selectionKey.channel();
                    //处理连接事件
                    SocketChannel socketChannel = ssc.accept();
                    log.debug("连接事件：" + socketChannel);
                    //非阻塞
                    socketChannel.configureBlocking(false);
                    //注册，事件为OP_WRITE
                    socketChannel.register(selector, SelectionKey.OP_READ);
                    log.debug("连接已注册到selector");
                }

                //读事件
                else if (selectionKey.isReadable())
                {
                    SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
                    //处理读事件
                    log.debug("读事件：" + socketChannel);
                    int read = socketChannel.read(byteBuffer);
                    if (read == -1)
                    {
                        selectionKey.cancel();
                        socketChannel.close();
                    }
                    else
                    {
                        byteBuffer.flip();
                        ByteBufferUtil.debugAll(byteBuffer);
                        byteBuffer.clear();
                    }

                }

                // 处理完毕，必须将事件移除
                iterator.remove();
            }
        }
    }
}
```



运行结果：

```sh
2023-03-13  21:40:39.869  [main] DEBUG mao.t3.Server:  事件总数：1
2023-03-13  21:40:39.871  [main] DEBUG mao.t3.Server:  连接事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:63466]
2023-03-13  21:40:39.871  [main] DEBUG mao.t3.Server:  连接已注册到selector
2023-03-13  21:40:56.924  [main] DEBUG mao.t3.Server:  事件总数：1
2023-03-13  21:40:56.925  [main] DEBUG mao.t3.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:63466]
2023-03-13  21:40:56.929  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
+--------+-------------------------------------------------+----------------+
2023-03-13  21:41:11.388  [main] DEBUG mao.t3.Server:  事件总数：1
2023-03-13  21:41:11.388  [main] DEBUG mao.t3.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:63466]
2023-03-13  21:41:23.993  [main] DEBUG mao.t3.Server:  事件总数：1
2023-03-13  21:41:23.994  [main] DEBUG mao.t3.Server:  连接事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:63475]
2023-03-13  21:41:23.994  [main] DEBUG mao.t3.Server:  连接已注册到selector
2023-03-13  21:41:36.106  [main] DEBUG mao.t3.Server:  事件总数：1
2023-03-13  21:41:36.106  [main] DEBUG mao.t3.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:63475]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
+--------+-------------------------------------------------+----------------+
2023-03-13  21:41:46.074  [main] DEBUG mao.t3.Server:  事件总数：1
2023-03-13  21:41:46.074  [main] DEBUG mao.t3.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:63475]
```







**为何要 iter.remove()?**

因为 select 在事件发生后，就会将相关的 key 放入 selectedKeys 集合，但不会在处理完后从 selectedKeys 集合中移除，需要我们自己编码删除。例如

* 第一次触发了 ssckey 上的 accept 事件，没有移除 ssckey 
* 第二次触发了 sckey 上的 read 事件，但这时 selectedKeys 中还有上次的 ssckey ，在处理时因为没有真正的 serverSocket 连上了，就会导致空指针异常



**cancel 的作用**

cancel 会取消注册在 selector 上的 channel，并从 keys 集合中删除 key 后续不会再监听事件







#### 处理消息的边界

如果客户端一次发送的消息过长，则会出现以下情况

```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t4
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/13
 * Time(创建时间)： 21:55
 * Version(版本): 1.0
 * Description(描述)： 处理消息的边界
 */

public class Client
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(mao.t4.Client.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException
    {
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.connect(new InetSocketAddress("127.0.0.1", 8080));
        Scanner input = new Scanner(System.in);
        input.nextLine();
        socketChannel.write(ByteBuffer.wrap("hellohellohellohellohellohello\nhellohellohellohellohello".getBytes(StandardCharsets.UTF_8)));
        input.nextLine();
        socketChannel.write(ByteBuffer.wrap("88888888888888888888888889".getBytes(StandardCharsets.UTF_8)));
        input.nextLine();
        socketChannel.close();
    }
}

```



服务端运行结果：

```sh
2023-03-14  13:15:41.805  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:15:41.807  [main] DEBUG mao.t4.Server:  连接事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58367]
2023-03-14  13:15:41.807  [main] DEBUG mao.t4.Server:  连接已注册到selector
2023-03-14  13:15:45.156  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:15:45.157  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58367]
2023-03-14  13:15:45.161  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 68 65 6c 6c 6f 68 65 6c 6c 6f 68 |hellohellohelloh|
+--------+-------------------------------------------------+----------------+
2023-03-14  13:15:45.166  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:15:45.166  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58367]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 65 6c 6c 6f 68 65 6c 6c 6f 68 65 6c 6c 6f 0a 68 |ellohellohello.h|
+--------+-------------------------------------------------+----------------+
2023-03-14  13:15:45.166  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:15:45.166  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58367]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 65 6c 6c 6f 68 65 6c 6c 6f 68 65 6c 6c 6f 68 65 |ellohellohellohe|
+--------+-------------------------------------------------+----------------+
2023-03-14  13:15:45.166  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:15:45.166  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58367]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [8]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 6c 6c 6f 68 65 6c 6c 6f 6f 68 65 6c 6c 6f 68 65 |llohelloohellohe|
+--------+-------------------------------------------------+----------------+
2023-03-14  13:15:46.576  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:15:46.576  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58367]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 38 38 38 38 38 38 38 38 38 38 38 38 38 38 38 38 |8888888888888888|
+--------+-------------------------------------------------+----------------+
2023-03-14  13:15:46.576  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:15:46.577  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58367]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 38 38 38 38 38 38 38 38 38 39 38 38 38 38 38 38 |8888888889888888|
+--------+-------------------------------------------------+----------------+
```





![image-20230314131256897](img/Netty学习笔记/image-20230314131256897.png)



**解决思路：**

* 一种思路是固定消息长度，数据包大小一样，服务器按预定长度读取，缺点是浪费带宽
* 另一种思路是按分隔符拆分，缺点是效率低
* TLV 格式，即 Type 类型、Length 长度、Value 数据，类型和长度已知的情况下，就可以方便获取消息大小，分配合适的 buffer，缺点是 buffer 需要提前分配，如果内容过大，则影响 server 吞吐量
  * Http 1.1 是 TLV 格式
  * Http 2.0 是 LTV 格式





服务端：

```java
package mao.t4;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t4
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/13
 * Time(创建时间)： 21:54
 * Version(版本): 1.0
 * Description(描述)： 处理消息的边界
 */

public class Server
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(mao.t4.Server.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException, InterruptedException
    {
        ByteBuffer byteBuffer = ByteBuffer.allocate(16);
        //创建服务器
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //设置成非阻塞模式
        serverSocketChannel.configureBlocking(false);
        //绑定
        serverSocketChannel.bind(new InetSocketAddress(8080));

        //Selector
        Selector selector = Selector.open();
        //注册，事件为OP_WRITE
        SelectionKey selectionKey1 = serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        selectionKey1.interestOps(SelectionKey.OP_ACCEPT);
        log.debug("SelectionKey:" + selectionKey1);


        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    serverSocketChannel.close();
                }
                catch (IOException e)
                {
                    e.printStackTrace();
                }
                try
                {
                    selector.close();
                }
                catch (IOException e)
                {
                    e.printStackTrace();
                }

            }
        }));

        while (true)
        {
            int count = selector.select();
            log.debug("事件总数：" + count);

            //获取所有事件
            Set<SelectionKey> selectionKeys = selector.selectedKeys();

            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext())
            {
                //获得SelectionKey
                SelectionKey selectionKey = iterator.next();
                //判断事件类型

                //连接服务器
                if (selectionKey.isAcceptable())
                {
                    ServerSocketChannel ssc = (ServerSocketChannel) selectionKey.channel();
                    //处理连接事件
                    SocketChannel socketChannel = ssc.accept();
                    log.debug("连接事件：" + socketChannel);
                    //非阻塞
                    socketChannel.configureBlocking(false);
                    //注册，事件为OP_WRITE
                    socketChannel.register(selector, SelectionKey.OP_READ);
                    log.debug("连接已注册到selector");
                }

                //读事件
                else if (selectionKey.isReadable())
                {
                    try
                    {
                        SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
                        // 获取 selectionKey 上关联的附件
                        ByteBuffer buffer = (ByteBuffer) selectionKey.attachment();
                        if (buffer == null)
                        {
                            buffer = byteBuffer;
                        }
                        //处理读事件
                        log.debug("读事件：" + socketChannel);
                        int read = socketChannel.read(buffer);
                        if (read == -1)
                        {
                            selectionKey.cancel();
                            //socketChannel.close();
                        }
                        else
                        {
                            split(buffer);
                            if (buffer.position() == buffer.limit())
                            {
                                //需要扩容
                                ByteBuffer newByteBuffer = ByteBuffer.allocate(buffer.capacity() * 2);
                                buffer.flip();
                                newByteBuffer.put(buffer);
                                selectionKey.attach(newByteBuffer);
                            }

                        }
                    }
                    catch (Exception e)
                    {
                        e.printStackTrace();
                        selectionKey.cancel();
                    }
                }

                // 处理完毕，必须将事件移除
                iterator.remove();
            }
        }
    }

    private static void split(ByteBuffer source)
    {
        //切换到读模式
        source.flip();
        for (int i = 0; i < source.limit(); i++)
        {
            //找到一条完整消息
            if (source.get(i) == '\n')
            {
                int length = i + 1 - source.position();
                // 把这条完整消息存入新的 ByteBuffer
                ByteBuffer target = ByteBuffer.allocate(length);
                // 从 source 读，向 target 写
                for (int j = 0; j < length; j++)
                {
                    target.put(source.get());
                }
                ByteBufferUtil.debugAll(target);
            }
        }
        //切换到写模式，没读完的部分继续
        source.compact();
    }
}
```





客户端：

```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t4
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/13
 * Time(创建时间)： 21:55
 * Version(版本): 1.0
 * Description(描述)： 处理消息的边界
 */

public class Client
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(mao.t4.Client.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException
    {
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.connect(new InetSocketAddress("127.0.0.1", 8080));
        Scanner input = new Scanner(System.in);
        input.nextLine();
        socketChannel.write(ByteBuffer.wrap("hellohellohellohellohellohello\nhellohellohellohellohello".getBytes(StandardCharsets.UTF_8)));
        input.nextLine();
        socketChannel.write(ByteBuffer.wrap("88888888888888888888888889\n".getBytes(StandardCharsets.UTF_8)));
        input.nextLine();
        socketChannel.write(ByteBuffer.wrap("1234\n0000000\n".getBytes(StandardCharsets.UTF_8)));
        input.nextLine();
        socketChannel.close();
    }
}
```



服务端运行结果：

```sh
2023-03-14  13:39:12.533  [main] DEBUG mao.t4.Server:  SelectionKey:channel=sun.nio.ch.ServerSocketChannelImpl[/[0:0:0:0:0:0:0:0]:8080], selector=sun.nio.ch.WindowsSelectorImpl@563f38c4, interestOps=16, readyOps=0
2023-03-14  13:39:15.063  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:39:15.063  [main] DEBUG mao.t4.Server:  连接事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58911]
2023-03-14  13:39:15.063  [main] DEBUG mao.t4.Server:  连接已注册到selector
2023-03-14  13:39:21.462  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:39:21.462  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58911]
2023-03-14  13:39:21.463  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:39:21.463  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58911]
2023-03-14  13:39:21.467  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [31], limit: [31]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 68 65 6c 6c 6f 68 65 6c 6c 6f 68 |hellohellohelloh|
|00000010| 65 6c 6c 6f 68 65 6c 6c 6f 68 65 6c 6c 6f 0a    |ellohellohello. |
+--------+-------------------------------------------------+----------------+
2023-03-14  13:39:21.473  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:39:21.473  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58911]
2023-03-14  13:39:25.573  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:39:25.573  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58911]
2023-03-14  13:39:25.573  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:39:25.574  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58911]
+--------+-------------------- all ------------------------+----------------+
position: [52], limit: [52]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 68 65 6c 6c 6f 68 65 6c 6c 6f 68 |hellohellohelloh|
|00000010| 65 6c 6c 6f 68 65 6c 6c 6f 38 38 38 38 38 38 38 |ellohello8888888|
|00000020| 38 38 38 38 38 38 38 38 38 38 38 38 38 38 38 38 |8888888888888888|
|00000030| 38 38 39 0a                                     |889.            |
+--------+-------------------------------------------------+----------------+
2023-03-14  13:39:30.963  [main] DEBUG mao.t4.Server:  事件总数：1
2023-03-14  13:39:30.963  [main] DEBUG mao.t4.Server:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58911]
+--------+-------------------- all ------------------------+----------------+
position: [5], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 0a                                  |1234.           |
+--------+-------------------------------------------------+----------------+
+--------+-------------------- all ------------------------+----------------+
position: [8], limit: [8]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 30 30 30 30 30 30 30 0a                         |0000000.        |
+--------+-------------------------------------------------+----------------+
```









#### ByteBuffer 大小分配

* 每个 channel 都需要记录可能被切分的消息，因为 ByteBuffer 不能被多个 channel 共同使用，因此需要为每个 channel 维护一个独立的 ByteBuffer
* ByteBuffer 不能太大，比如一个 ByteBuffer 1Mb 的话，要支持百万连接就要 1Tb 内存，因此需要设计大小可变的 ByteBuffer
  * 一种思路是首先分配一个较小的 buffer，例如 4k，如果发现数据不够，再分配 8k 的 buffer，将 4k buffer 内容拷贝至 8k buffer，优点是消息连续容易处理，缺点是数据拷贝耗费性能
  * 另一种思路是用多个数组组成 buffer，一个数组不够，把多出来的内容写入新的数组，与前面的区别是消息存储不连续解析复杂，优点是避免了拷贝引起的性能损耗









#### 处理 write 事件

* 非阻塞模式下，无法保证把 buffer 中所有数据都写入 channel，因此需要追踪 write 方法的返回值（代表实际写入字节数）
* 用 selector 监听所有 channel 的可写事件，每个 channel 都需要一个 key 来跟踪 buffer，但这样又会导致占用内存过多，就有两阶段策略
  * 当消息处理器第一次写入消息时，才将 channel 注册到 selector 上
  * selector 检查 channel 上的可写事件，如果所有的数据写完了，就取消 channel 的注册
  * 如果不取消，会每次可写均会触发 write 事件





服务端：

```java
package mao.t5;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.nio.charset.Charset;
import java.util.Iterator;
import java.util.Set;
import java.util.concurrent.atomic.AtomicLong;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t5
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/14
 * Time(创建时间)： 13:46
 * Version(版本): 1.0
 * Description(描述)： 处理 write 事件
 */

public class Server
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(mao.t5.Server.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException, InterruptedException
    {
        AtomicLong atomicLong = new AtomicLong(0);
        ByteBuffer byteBuffer = ByteBuffer.allocate(16);
        //创建服务器
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //设置成非阻塞模式
        serverSocketChannel.configureBlocking(false);
        //绑定
        serverSocketChannel.bind(new InetSocketAddress(8080));

        //Selector
        Selector selector = Selector.open();
        //注册，事件为OP_WRITE
        SelectionKey selectionKey1 = serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        selectionKey1.interestOps(SelectionKey.OP_ACCEPT);
        log.debug("SelectionKey:" + selectionKey1);


        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    serverSocketChannel.close();
                }
                catch (IOException e)
                {
                    e.printStackTrace();
                }
                try
                {
                    selector.close();
                }
                catch (IOException e)
                {
                    e.printStackTrace();
                }

            }
        }));

        while (true)
        {
            int count = selector.select();
            log.debug("事件总数：" + count);

            //获取所有事件
            Set<SelectionKey> selectionKeys = selector.selectedKeys();

            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext())
            {
                //获得SelectionKey
                SelectionKey selectionKey = iterator.next();
                //判断事件类型

                //连接服务器
                if (selectionKey.isAcceptable())
                {
                    ServerSocketChannel ssc = (ServerSocketChannel) selectionKey.channel();
                    //处理连接事件
                    SocketChannel socketChannel = ssc.accept();
                    log.debug("连接事件：" + socketChannel);
                    //非阻塞
                    socketChannel.configureBlocking(false);
                    //注册，事件为OP_WRITE
                    SelectionKey selectionKey2 = socketChannel.register(selector, SelectionKey.OP_READ);
                    log.debug("连接已注册到selector");
                    StringBuilder stringBuilder = new StringBuilder();
                    for (int i = 0; i < 10000000; i++)
                    {
                        stringBuilder.append("0");
                    }
                    ByteBuffer buffer = Charset.defaultCharset().encode(stringBuilder.toString());
                    //写
                    int write = socketChannel.write(buffer);
                    log.debug("写入的字节数：" + write);
                    atomicLong.set(write);
                    //判断是否写完
                    if (buffer.hasRemaining())
                    {
                        //没有一次写完
                        //再关注写事件
                        selectionKey2.interestOps(SelectionKey.OP_WRITE);
                        //加入到附件
                        selectionKey2.attach(buffer);
                    }
                }

                //读事件
                else if (selectionKey.isReadable())
                {
                    try
                    {
                        SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
                        // 获取 selectionKey 上关联的附件
                        ByteBuffer buffer = (ByteBuffer) selectionKey.attachment();
                        if (buffer == null)
                        {
                            buffer = byteBuffer;
                        }
                        //处理读事件
                        log.debug("读事件：" + socketChannel);
                        int read = socketChannel.read(buffer);
                        if (read == -1)
                        {
                            selectionKey.cancel();
                            //socketChannel.close();
                        }
                        else
                        {
                            split(buffer);
                            if (buffer.position() == buffer.limit())
                            {
                                //需要扩容
                                ByteBuffer newByteBuffer = ByteBuffer.allocate(buffer.capacity() * 2);
                                buffer.flip();
                                newByteBuffer.put(buffer);
                                selectionKey.attach(newByteBuffer);
                            }

                        }
                    }
                    catch (Exception e)
                    {
                        e.printStackTrace();
                        selectionKey.cancel();
                    }
                }

                //写事件
                else if (selectionKey.isWritable())
                {
                    try
                    {
                        //取附件
                        ByteBuffer buffer = (ByteBuffer) selectionKey.attachment();
                        //SocketChannel
                        SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
                        //继续写
                        log.debug("写事件：" + socketChannel);
                        int write = socketChannel.write(buffer);
                        log.debug("写入的字节数：" + write);
                        atomicLong.getAndAdd(write);
                        //判断是否读完
                        if (!buffer.hasRemaining())
                        {
                            //写完了
                            //取消关注写事件
                            selectionKey.interestOps(SelectionKey.OP_READ);
                            //加入到附件
                            selectionKey.attach(null);
                            log.debug("写完成，总字节数：" + atomicLong.get());
                        }
                    }
                    catch (Exception e)
                    {
                        e.printStackTrace();
                        selectionKey.cancel();
                    }
                }

                // 处理完毕，必须将事件移除
                iterator.remove();
            }
        }
    }

    private static void split(ByteBuffer source)
    {
        //切换到读模式
        source.flip();
        for (int i = 0; i < source.limit(); i++)
        {
            //找到一条完整消息
            if (source.get(i) == '\n')
            {
                int length = i + 1 - source.position();
                // 把这条完整消息存入新的 ByteBuffer
                ByteBuffer target = ByteBuffer.allocate(length);
                // 从 source 读，向 target 写
                for (int j = 0; j < length; j++)
                {
                    target.put(source.get());
                }
                ByteBufferUtil.debugAll(target);
            }
        }
        //切换到写模式，没读完的部分继续
        source.compact();
    }
}

```





客户端：

```java
package mao.t5;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.nio.charset.StandardCharsets;
import java.util.Iterator;
import java.util.Scanner;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t5
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/14
 * Time(创建时间)： 13:47
 * Version(版本): 1.0
 * Description(描述)： 处理 write 事件
 */

public class Client
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(mao.t5.Client.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException
    {
        Selector selector = Selector.open();
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.configureBlocking(false);
        socketChannel.register(selector, SelectionKey.OP_CONNECT | SelectionKey.OP_READ);
        socketChannel.connect(new InetSocketAddress("localhost", 8080));
        int count = 0;
        while (true)
        {
            selector.select();
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext())
            {
                SelectionKey selectionKey = iterator.next();
                if (selectionKey.isConnectable())
                {
                    log.debug(String.valueOf(socketChannel.finishConnect()));
                }
                //可读
                else if (selectionKey.isReadable())
                {
                    ByteBuffer buffer = ByteBuffer.allocate(1024 * 1024);
                    count += socketChannel.read(buffer);
                    //ByteBufferUtil.debugAll(buffer);
                    buffer.clear();
                    log.info("已读完：" + count + "字节");
                }
                iterator.remove();
            }
        }
    }
}
```



服务端运行结果：

```sh
2023-03-14  14:42:56.197  [main] DEBUG mao.t5.Server:  SelectionKey:channel=sun.nio.ch.ServerSocketChannelImpl[/[0:0:0:0:0:0:0:0]:8080], selector=sun.nio.ch.WindowsSelectorImpl@563f38c4, interestOps=16, readyOps=0
2023-03-14  14:42:59.202  [main] DEBUG mao.t5.Server:  事件总数：1
2023-03-14  14:42:59.203  [main] DEBUG mao.t5.Server:  连接事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58802]
2023-03-14  14:42:59.203  [main] DEBUG mao.t5.Server:  连接已注册到selector
2023-03-14  14:42:59.286  [main] DEBUG mao.t5.Server:  写入的字节数：3669988
2023-03-14  14:42:59.289  [main] DEBUG mao.t5.Server:  事件总数：1
2023-03-14  14:42:59.289  [main] DEBUG mao.t5.Server:  写事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58802]
2023-03-14  14:42:59.291  [main] DEBUG mao.t5.Server:  写入的字节数：2490349
2023-03-14  14:42:59.298  [main] DEBUG mao.t5.Server:  事件总数：1
2023-03-14  14:42:59.298  [main] DEBUG mao.t5.Server:  写事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58802]
2023-03-14  14:42:59.300  [main] DEBUG mao.t5.Server:  写入的字节数：3669988
2023-03-14  14:42:59.303  [main] DEBUG mao.t5.Server:  事件总数：1
2023-03-14  14:42:59.303  [main] DEBUG mao.t5.Server:  写事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:58802]
2023-03-14  14:42:59.304  [main] DEBUG mao.t5.Server:  写入的字节数：169675
2023-03-14  14:42:59.305  [main] DEBUG mao.t5.Server:  写完成，总字节数：10000000
```



客户端运行结果：

```sh
2023-03-14  14:42:59.204  [main] DEBUG mao.t5.Client:  true
2023-03-14  14:42:59.285  [main] INFO  mao.t5.Client:  已读完：131071字节
2023-03-14  14:42:59.285  [main] INFO  mao.t5.Client:  已读完：262142字节
2023-03-14  14:42:59.286  [main] INFO  mao.t5.Client:  已读完：393213字节
2023-03-14  14:42:59.286  [main] INFO  mao.t5.Client:  已读完：524284字节
2023-03-14  14:42:59.286  [main] INFO  mao.t5.Client:  已读完：655355字节
2023-03-14  14:42:59.287  [main] INFO  mao.t5.Client:  已读完：786426字节
2023-03-14  14:42:59.287  [main] INFO  mao.t5.Client:  已读完：917497字节
2023-03-14  14:42:59.287  [main] INFO  mao.t5.Client:  已读完：1048568字节
2023-03-14  14:42:59.288  [main] INFO  mao.t5.Client:  已读完：1179639字节
2023-03-14  14:42:59.289  [main] INFO  mao.t5.Client:  已读完：1310710字节
2023-03-14  14:42:59.289  [main] INFO  mao.t5.Client:  已读完：1441781字节
2023-03-14  14:42:59.290  [main] INFO  mao.t5.Client:  已读完：1572852字节
2023-03-14  14:42:59.290  [main] INFO  mao.t5.Client:  已读完：1703923字节
2023-03-14  14:42:59.291  [main] INFO  mao.t5.Client:  已读完：1834994字节
2023-03-14  14:42:59.292  [main] INFO  mao.t5.Client:  已读完：1966065字节
2023-03-14  14:42:59.292  [main] INFO  mao.t5.Client:  已读完：2097136字节
2023-03-14  14:42:59.296  [main] INFO  mao.t5.Client:  已读完：2228207字节
2023-03-14  14:42:59.296  [main] INFO  mao.t5.Client:  已读完：2359278字节
2023-03-14  14:42:59.296  [main] INFO  mao.t5.Client:  已读完：2490349字节
2023-03-14  14:42:59.297  [main] INFO  mao.t5.Client:  已读完：2621420字节
2023-03-14  14:42:59.297  [main] INFO  mao.t5.Client:  已读完：2752491字节
2023-03-14  14:42:59.297  [main] INFO  mao.t5.Client:  已读完：2883562字节
2023-03-14  14:42:59.297  [main] INFO  mao.t5.Client:  已读完：3014633字节
2023-03-14  14:42:59.297  [main] INFO  mao.t5.Client:  已读完：3145704字节
2023-03-14  14:42:59.297  [main] INFO  mao.t5.Client:  已读完：3276775字节
2023-03-14  14:42:59.298  [main] INFO  mao.t5.Client:  已读完：3407846字节
2023-03-14  14:42:59.298  [main] INFO  mao.t5.Client:  已读完：3538917字节
2023-03-14  14:42:59.299  [main] INFO  mao.t5.Client:  已读完：3669988字节
2023-03-14  14:42:59.299  [main] INFO  mao.t5.Client:  已读完：3801059字节
2023-03-14  14:42:59.299  [main] INFO  mao.t5.Client:  已读完：3932130字节
2023-03-14  14:42:59.299  [main] INFO  mao.t5.Client:  已读完：4063201字节
2023-03-14  14:42:59.300  [main] INFO  mao.t5.Client:  已读完：4194272字节
2023-03-14  14:42:59.300  [main] INFO  mao.t5.Client:  已读完：4325343字节
2023-03-14  14:42:59.300  [main] INFO  mao.t5.Client:  已读完：4456414字节
2023-03-14  14:42:59.301  [main] INFO  mao.t5.Client:  已读完：4587485字节
2023-03-14  14:42:59.301  [main] INFO  mao.t5.Client:  已读完：4718556字节
2023-03-14  14:42:59.301  [main] INFO  mao.t5.Client:  已读完：4849627字节
2023-03-14  14:42:59.301  [main] INFO  mao.t5.Client:  已读完：4980698字节
2023-03-14  14:42:59.302  [main] INFO  mao.t5.Client:  已读完：5111769字节
2023-03-14  14:42:59.302  [main] INFO  mao.t5.Client:  已读完：5242840字节
2023-03-14  14:42:59.302  [main] INFO  mao.t5.Client:  已读完：5373911字节
2023-03-14  14:42:59.304  [main] INFO  mao.t5.Client:  已读完：5504982字节
2023-03-14  14:42:59.304  [main] INFO  mao.t5.Client:  已读完：5636053字节
2023-03-14  14:42:59.304  [main] INFO  mao.t5.Client:  已读完：5767124字节
2023-03-14  14:42:59.304  [main] INFO  mao.t5.Client:  已读完：5898195字节
2023-03-14  14:42:59.304  [main] INFO  mao.t5.Client:  已读完：6029266字节
2023-03-14  14:42:59.305  [main] INFO  mao.t5.Client:  已读完：6160337字节
2023-03-14  14:42:59.306  [main] INFO  mao.t5.Client:  已读完：6291408字节
2023-03-14  14:42:59.306  [main] INFO  mao.t5.Client:  已读完：6422479字节
2023-03-14  14:42:59.307  [main] INFO  mao.t5.Client:  已读完：6553550字节
2023-03-14  14:42:59.308  [main] INFO  mao.t5.Client:  已读完：6684621字节
2023-03-14  14:42:59.308  [main] INFO  mao.t5.Client:  已读完：6815692字节
2023-03-14  14:42:59.309  [main] INFO  mao.t5.Client:  已读完：6946763字节
2023-03-14  14:42:59.310  [main] INFO  mao.t5.Client:  已读完：7077834字节
2023-03-14  14:42:59.311  [main] INFO  mao.t5.Client:  已读完：7208905字节
2023-03-14  14:42:59.311  [main] INFO  mao.t5.Client:  已读完：7339976字节
2023-03-14  14:42:59.312  [main] INFO  mao.t5.Client:  已读完：7471047字节
2023-03-14  14:42:59.313  [main] INFO  mao.t5.Client:  已读完：7602118字节
2023-03-14  14:42:59.313  [main] INFO  mao.t5.Client:  已读完：7733189字节
2023-03-14  14:42:59.314  [main] INFO  mao.t5.Client:  已读完：7864260字节
2023-03-14  14:42:59.314  [main] INFO  mao.t5.Client:  已读完：7995331字节
2023-03-14  14:42:59.315  [main] INFO  mao.t5.Client:  已读完：8126402字节
2023-03-14  14:42:59.316  [main] INFO  mao.t5.Client:  已读完：8257473字节
2023-03-14  14:42:59.316  [main] INFO  mao.t5.Client:  已读完：8388544字节
2023-03-14  14:42:59.317  [main] INFO  mao.t5.Client:  已读完：8519615字节
2023-03-14  14:42:59.317  [main] INFO  mao.t5.Client:  已读完：8650686字节
2023-03-14  14:42:59.318  [main] INFO  mao.t5.Client:  已读完：8781757字节
2023-03-14  14:42:59.318  [main] INFO  mao.t5.Client:  已读完：8912828字节
2023-03-14  14:42:59.319  [main] INFO  mao.t5.Client:  已读完：9043899字节
2023-03-14  14:42:59.319  [main] INFO  mao.t5.Client:  已读完：9174970字节
2023-03-14  14:42:59.320  [main] INFO  mao.t5.Client:  已读完：9306041字节
2023-03-14  14:42:59.320  [main] INFO  mao.t5.Client:  已读完：9437112字节
2023-03-14  14:42:59.321  [main] INFO  mao.t5.Client:  已读完：9568183字节
2023-03-14  14:42:59.321  [main] INFO  mao.t5.Client:  已读完：9699254字节
2023-03-14  14:42:59.322  [main] INFO  mao.t5.Client:  已读完：9830325字节
2023-03-14  14:42:59.322  [main] INFO  mao.t5.Client:  已读完：9961396字节
2023-03-14  14:42:59.323  [main] INFO  mao.t5.Client:  已读完：10000000字节
```









### 多线程优化

现在都是多核 cpu，设计时要充分考虑别让 cpu 的力量被白白浪费

* 单线程配一个选择器，专门处理 accept 事件
* 创建 cpu 核心数的线程，每个线程配一个选择器，轮流处理 read 事件



```java
package mao.t6;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.concurrent.atomic.AtomicLong;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t6
 * Class(类名): AcceptHandler
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/14
 * Time(创建时间)： 22:23
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class AcceptHandler implements Runnable
{

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(AcceptHandler.class);

    /**
     * 选择器
     */
    private Selector selector;

    /**
     * 工人处理程序
     */
    private WorkerHandler[] workerHandlers;

    /**
     * 是否已经注册
     */
    private volatile boolean isRegister = false;

    /**
     * 原子Long，cas方式累加
     */
    private final AtomicLong atomicLong = new AtomicLong(0);

    /**
     * 注册
     *
     * @param port 端口号
     * @throws IOException ioexception
     */
    public void register(int port) throws IOException
    {
        //判断是否已经注册过
        if (!isRegister)
        {
            //还没有注册过
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
            //绑定端口
            serverSocketChannel.bind(new InetSocketAddress(port));
            //非阻塞
            serverSocketChannel.configureBlocking(false);
            //创建Selector
            selector = Selector.open();
            //注册到Selector
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
            //创建WorkerHandler
            workerHandlers = initWorkerHandlers();
            log.debug("服务启动");
            new Thread(this, "Accept").start();
            isRegister = true;
        }
    }

    public WorkerHandler[] initWorkerHandlers()
    {
        int processors = Runtime.getRuntime().availableProcessors();
        log.debug("线程数量：" + processors);
        WorkerHandler[] workerHandlers = new WorkerHandler[processors];
        for (int i = 0; i < processors; i++)
        {
            log.debug("初始化WorkerHandler" + i);
            workerHandlers[i] = new WorkerHandler(i);
        }
        return workerHandlers;
    }

    @Override
    public void run()
    {
        while (true)
        {
            try
            {
                int select = selector.select();
                //iterator
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                while (iterator.hasNext())
                {
                    //得到selectionKey
                    SelectionKey selectionKey = iterator.next();

                    //连接事件
                    if (selectionKey.isAcceptable())
                    {
                        try
                        {
                            //得到ServerSocketChannel
                            ServerSocketChannel serverSocketChannel = (ServerSocketChannel) selectionKey.channel();
                            log.debug("注册事件：" + serverSocketChannel);
                            //得到SocketChannel
                            SocketChannel socketChannel = serverSocketChannel.accept();
                            //非阻塞
                            socketChannel.configureBlocking(false);
                            //轮询负载均衡注册到每个workerHandler
                            workerHandlers[Math.toIntExact((atomicLong.getAndIncrement() %
                                    workerHandlers.length))].register(socketChannel);
                        }
                        catch (Exception e)
                        {
                            selectionKey.cancel();
                        }
                    }
                    //移除
                    iterator.remove();
                }
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
    }
}
```





```java
package mao.t6;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.concurrent.ConcurrentLinkedQueue;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t6
 * Class(类名): WorkerHandler
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/14
 * Time(创建时间)： 22:25
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class WorkerHandler implements Runnable
{

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(WorkerHandler.class);

    /**
     * 索引
     */
    private final int index;

    /**
     * 选择器
     */
    private Selector selector;

    /**
     * 线程安全的任务队列
     */
    private final ConcurrentLinkedQueue<Runnable> tasks = new ConcurrentLinkedQueue<>();

    /**
     * 是否已经注册
     */
    private volatile boolean isRegister = false;

    /**
     * 构造方法
     *
     * @param index 索引
     */
    public WorkerHandler(int index)
    {
        this.index = index;
    }


    /**
     * 注册
     *
     * @param socketChannel 套接字通道
     */
    public void register(SocketChannel socketChannel) throws IOException
    {
        //判断是否已经注册过
        if (!isRegister)
        {
            //创建selector
            selector = Selector.open();
            //开启线程
            new Thread(this, "Worker-" + index).start();
            log.debug("启动工作线程：Worker-" + index + " ,监听读事件");
            isRegister = true;
        }
        //添加一个任务到队列
        tasks.add(() ->
        {
            try
            {
                //注册
                SelectionKey selectionKey = socketChannel.register(selector, SelectionKey.OP_READ);
                selector.selectNow();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        });
        //唤醒阻塞的selector
        selector.wakeup();
    }

    @Override
    public void run()
    {
        while (true)
        {
            try
            {
                selector.select();
                //取得任务，非阻塞取
                Runnable runnable = tasks.poll();
                //判断是否为空
                if (runnable != null)
                {
                    //直接调用run方法，不启动新线程
                    runnable.run();
                }
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                while (iterator.hasNext())
                {
                    SelectionKey selectionKey = iterator.next();
                    //读事件
                    if (selectionKey.isReadable())
                    {
                        try
                        {
                            SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
                            log.debug("读事件：" + socketChannel);
                            ByteBuffer buffer = ByteBuffer.allocate(32);
                            int read = socketChannel.read(buffer);
                            if (read == -1)
                            {
                                selectionKey.cancel();
                                socketChannel.close();
                            }
                            else
                            {
                                //切换到读模式
                                buffer.flip();
                                ByteBufferUtil.debugAll(buffer);
                            }
                        }
                        catch (Exception e)
                        {
                            e.printStackTrace();
                            selectionKey.cancel();
                        }

                    }
                    //移除
                    iterator.remove();
                }
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
    }
}
```





```java
package mao.t6;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t6
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/14
 * Time(创建时间)： 22:22
 * Version(版本): 1.0
 * Description(描述)： 多线程优化
 */

public class Server
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Server.class);

    public static void main(String[] args) throws IOException
    {
        AcceptHandler acceptHandler = new AcceptHandler();
        acceptHandler.register(8080);
    }
}
```





```java
package mao.t6;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t6
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/14
 * Time(创建时间)： 23:14
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Client
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(mao.t6.Client.class);

    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args) throws IOException
    {
        while (true)
        {
            try
            {
                SocketChannel socketChannel = SocketChannel.open();
                socketChannel.connect(new InetSocketAddress("127.0.0.1", 8080));
                Scanner input = new Scanner(System.in);
                input.nextLine();
                socketChannel.write(ByteBuffer.wrap("hello".getBytes(StandardCharsets.UTF_8)));
                input.nextLine();
                socketChannel.write(ByteBuffer.wrap("world".getBytes(StandardCharsets.UTF_8)));
                input.nextLine();
                socketChannel.close();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
    }
}
```





服务端运行结果：

```sh
2023-03-15  13:32:41.190  [main] DEBUG mao.t6.AcceptHandler:  线程数量：32
2023-03-15  13:32:41.191  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler0
2023-03-15  13:32:41.192  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler1
2023-03-15  13:32:41.192  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler2
2023-03-15  13:32:41.192  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler3
2023-03-15  13:32:41.192  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler4
2023-03-15  13:32:41.192  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler5
2023-03-15  13:32:41.192  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler6
2023-03-15  13:32:41.192  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler7
2023-03-15  13:32:41.192  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler8
2023-03-15  13:32:41.192  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler9
2023-03-15  13:32:41.192  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler10
2023-03-15  13:32:41.192  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler11
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler12
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler13
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler14
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler15
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler16
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler17
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler18
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler19
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler20
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler21
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler22
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler23
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler24
2023-03-15  13:32:41.193  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler25
2023-03-15  13:32:41.194  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler26
2023-03-15  13:32:41.194  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler27
2023-03-15  13:32:41.194  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler28
2023-03-15  13:32:41.194  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler29
2023-03-15  13:32:41.194  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler30
2023-03-15  13:32:41.194  [main] DEBUG mao.t6.AcceptHandler:  初始化WorkerHandler31
2023-03-15  13:32:41.194  [main] DEBUG mao.t6.AcceptHandler:  服务启动
2023-03-15  13:32:47.588  [Accept] DEBUG mao.t6.AcceptHandler:  注册事件：sun.nio.ch.ServerSocketChannelImpl[/[0:0:0:0:0:0:0:0]:8080]
2023-03-15  13:32:47.590  [Accept] DEBUG mao.t6.WorkerHandler:  启动工作线程：Worker-0 ,监听读事件
2023-03-15  13:33:13.363  [Worker-0] DEBUG mao.t6.WorkerHandler:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:51773]
2023-03-15  13:33:13.367  [Worker-0] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [10]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 77 6f 72 6c 64 00 00 00 00 00 00 |helloworld......|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-15  13:33:27.341  [Worker-0] DEBUG mao.t6.WorkerHandler:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:51773]
2023-03-15  13:33:27.341  [Accept] DEBUG mao.t6.AcceptHandler:  注册事件：sun.nio.ch.ServerSocketChannelImpl[/[0:0:0:0:0:0:0:0]:8080]
2023-03-15  13:33:27.343  [Accept] DEBUG mao.t6.WorkerHandler:  启动工作线程：Worker-1 ,监听读事件
2023-03-15  13:33:44.701  [Worker-1] DEBUG mao.t6.WorkerHandler:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:51784]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-15  13:33:51.335  [Worker-1] DEBUG mao.t6.WorkerHandler:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:51784]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 77 6f 72 6c 64 00 00 00 00 00 00 00 00 00 00 00 |world...........|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-15  13:34:09.064  [Worker-1] DEBUG mao.t6.WorkerHandler:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:51784]
2023-03-15  13:34:09.064  [Accept] DEBUG mao.t6.AcceptHandler:  注册事件：sun.nio.ch.ServerSocketChannelImpl[/[0:0:0:0:0:0:0:0]:8080]
2023-03-15  13:34:09.066  [Accept] DEBUG mao.t6.WorkerHandler:  启动工作线程：Worker-2 ,监听读事件
2023-03-15  13:34:17.957  [Worker-2] DEBUG mao.t6.WorkerHandler:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:51796]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-15  13:34:29.808  [Worker-2] DEBUG mao.t6.WorkerHandler:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:51796]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 77 6f 72 6c 64 00 00 00 00 00 00 00 00 00 00 00 |world...........|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-15  13:34:30.388  [Worker-2] DEBUG mao.t6.WorkerHandler:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:51796]
2023-03-15  13:34:30.389  [Accept] DEBUG mao.t6.AcceptHandler:  注册事件：sun.nio.ch.ServerSocketChannelImpl[/[0:0:0:0:0:0:0:0]:8080]
2023-03-15  13:34:30.391  [Accept] DEBUG mao.t6.WorkerHandler:  启动工作线程：Worker-3 ,监听读事件
2023-03-15  13:34:56.666  [Worker-3] DEBUG mao.t6.WorkerHandler:  读事件：java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:51799]
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
```









### UDP

* UDP 是无连接的，client 发送数据不会管 server 是否开启
* server 这边的 receive 方法会将接收到的数据存入 byte buffer，但如果数据报文超过 buffer 大小，多出来的数据会被默默抛弃



服务端

```java
package mao.t7;

import mao.utils.ByteBufferUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.DatagramChannel;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t7
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/15
 * Time(创建时间)： 13:40
 * Version(版本): 1.0
 * Description(描述)： UDP
 */

public class Server
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Server.class);

    public static void main(String[] args)
    {
        try (DatagramChannel datagramChannel = DatagramChannel.open())
        {
            //绑定端口
            datagramChannel.bind(new InetSocketAddress(8080));
            log.debug("服务已启动");
            ByteBuffer buffer = ByteBuffer.allocate(16);
            //等待接收数据
            datagramChannel.receive(buffer);
            buffer.flip();
            ByteBufferUtil.debugAll(buffer);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```



客户端

```java
package mao.t7;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.DatagramChannel;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_Net_Programming
 * Package(包名): mao.t7
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/15
 * Time(创建时间)： 13:41
 * Version(版本): 1.0
 * Description(描述)： UDP
 */

public class Client
{
    private static final Logger log = LoggerFactory.getLogger(Client.class);

    public static void main(String[] args)
    {
        try (DatagramChannel datagramChannel = DatagramChannel.open())
        {
            ByteBuffer buffer = StandardCharsets.UTF_8.encode("hello");
            datagramChannel.send(buffer, new InetSocketAddress("127.0.0.1", 8080));
            log.debug("消息已发送");
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```



服务端运行结果：

```sh
2023-03-15  13:51:03.425  [main] DEBUG mao.t7.Server:  服务已启动
2023-03-15  13:51:06.478  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [5]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
+--------+-------------------------------------------------+----------------+
```











## NIO和BIO

### stream和channel

* stream 不会自动缓冲数据，channel 会利用系统提供的发送缓冲区、接收缓冲区（更为底层）
* stream 仅支持阻塞 API，channel 同时支持阻塞、非阻塞 API，网络 channel 可配合 selector 实现多路复用
* 二者均为全双工，即读写可以同时进行





### IO 模型

同步阻塞、同步非阻塞、同步多路复用、异步阻塞（没有此情况）、异步非阻塞

* 同步：线程自己去获取结果（一个线程）
* 异步：线程自己不去获取结果，而是由其它线程送结果（至少两个线程）



当调用一次 channel.read 或 stream.read 后，会切换至操作系统内核态来完成真正数据读取，而读取又分为两个阶段，分别为：

* 等待数据阶段
* 复制数据阶段



![image-20230316204824197](img/Netty学习笔记/image-20230316204824197.png)





阻塞 IO



![image-20230316204848439](img/Netty学习笔记/image-20230316204848439.png)





非阻塞  IO



![image-20230316204924508](img/Netty学习笔记/image-20230316204924508.png)





多路复用



![image-20230316205011601](img/Netty学习笔记/image-20230316205011601.png)



异步 IO

![image-20230316205212259](img/Netty学习笔记/image-20230316205212259.png)







### 零拷贝

#### 传统 IO 问题

传统的 IO 将一个文件通过 socket 写出内部工作流程是这样的：

![image-20230316205342226](img/Netty学习笔记/image-20230316205342226.png)





1. java 本身并不具备 IO 读写能力，因此 read 方法调用后，要从 java 程序的**用户态**切换至**内核态**，去调用操作系统（Kernel）的读能力，将数据读入**内核缓冲区**。这期间用户线程阻塞，操作系统使用 DMA（Direct Memory Access）来实现文件读，其间也不会使用 cpu
2. 从**内核态**切换回**用户态**，将数据从**内核缓冲区**读入**用户缓冲区**（即 byte[] buf），这期间 cpu 会参与拷贝，无法利用 DMA
3. 调用 write 方法，这时将数据从**用户缓冲区**（byte[] buf）写入 **socket 缓冲区**，cpu 会参与拷贝
4. 接下来要向网卡写数据，这项能力 java 又不具备，因此又得从**用户态**切换至**内核态**，调用操作系统的写能力，使用 DMA 将 **socket 缓冲区**的数据写入网卡，不会使用 cpu





可以看到中间环节较多，java 的 IO 实际不是物理设备级别的读写，而是缓存的复制，底层的真正读写是操作系统来完成的

* 用户态与内核态的切换发生了 3 次，这个操作比较重量级
* 数据拷贝了共 4 次





#### NIO 优化

通过 DirectByteBuf 

* ByteBuffer.allocate(10)  HeapByteBuffer 使用的还是 java 内存
* ByteBuffer.allocateDirect(10)  DirectByteBuffer 使用的是操作系统内存



java 可以使用 DirectByteBuf 将堆外内存映射到 jvm 内存中来直接访问使用

* 这块内存不受 jvm 垃圾回收的影响，因此内存地址固定，有助于 IO 读写
* java 中的 DirectByteBuf 对象仅维护了此内存的虚引用，内存回收分成两步
  * DirectByteBuf 对象被垃圾回收，将虚引用加入引用队列
  * 通过专门线程访问引用队列，根据虚引用释放堆外内存
* 减少了一次数据拷贝，用户态与内核态的切换次数没有减少



![image-20230316205817198](img/Netty学习笔记/image-20230316205817198.png)





进一步优化（底层采用了 linux 2.1 后提供的 sendFile 方法），java 中对应着两个 channel 调用 transferTo/transferFrom 方法拷贝数据



1. java 调用 transferTo 方法后，要从 java 程序的**用户态**切换至**内核态**，使用 DMA将数据读入**内核缓冲区**，不会使用 cpu
2. 数据从**内核缓冲区**传输到 **socket 缓冲区**，cpu 会参与拷贝
3. 最后使用 DMA 将 **socket 缓冲区**的数据写入网卡，不会使用 cpu



![image-20230316205924360](img/Netty学习笔记/image-20230316205924360.png)





进一步优化（linux 2.4）

1. java 调用 transferTo 方法后，要从 java 程序的**用户态**切换至**内核态**，使用 DMA将数据读入**内核缓冲区**，不会使用 cpu
2. 只会将一些 offset 和 length 信息拷入 **socket 缓冲区**，几乎无消耗
3. 使用 DMA 将 **内核缓冲区**的数据写入网卡，不会使用 cpu



整个过程仅只发生了一次用户态与内核态的切换，数据拷贝了 2 次。所谓的【零拷贝】，并不是真正无拷贝，而是在不会拷贝重复数据到 jvm 内存中，零拷贝的优点有：

* 更少的用户态与内核态的切换
* 不利用 cpu 计算，减少 cpu 缓存伪共享
* 零拷贝适合小文件传输













# AIO

AIO 用来解决数据复制阶段的阻塞问题

* 同步意味着，在进行读写操作时，线程需要等待结果，还是相当于闲置
* 异步意味着，在进行读写操作时，线程不必等待结果，而是将来由操作系统来通过回调方式由另外的线程来获得结果



异步模型需要底层操作系统（Kernel）提供支持

* Windows 系统通过 IOCP 实现了真正的异步 IO
* Linux 系统异步 IO 在 2.6 版本引入，但其底层实现还是用多路复用模拟了异步 IO，性能没有优势





## 文件 AIO

默认文件 AIO 使用的线程都是守护线程

```java
package mao.t1;

import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import mao.utils.ByteBufferUtil;

import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousFileChannel;
import java.nio.channels.CompletionHandler;
import java.nio.file.Paths;
import java.nio.file.StandardOpenOption;

/**
 * Project name(项目名称)：Netty_AIO
 * Package(包名): mao.t1
 * Class(类名): FileAIO
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/16
 * Time(创建时间)： 21:24
 * Version(版本): 1.0
 * Description(描述)： 文件AIO
 */

@Slf4j
public class FileAIO
{
    @SneakyThrows
    public static void main(String[] args)
    {
        try
        {
            log.debug("开始读取...");
            AsynchronousFileChannel asynchronousFileChannel =
                    AsynchronousFileChannel.open(Paths.get("test.txt"), StandardOpenOption.READ);
            ByteBuffer buffer = ByteBuffer.allocate(16);
            asynchronousFileChannel.read(buffer, 0, null, new CompletionHandler<Integer, ByteBuffer>()
            {
                @Override
                public void completed(Integer result, ByteBuffer attachment)
                {
                    log.debug("读取完成：" + result);
                    buffer.flip();
                    ByteBufferUtil.debugAll(buffer);
                }

                @Override
                public void failed(Throwable exc, ByteBuffer attachment)
                {
                    log.warn("读取失败：", exc);
                }
            });
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }

        Thread.sleep(1000);
    }
}
```



运行结果：

```sh
2023-03-16  21:36:19.104  [main] DEBUG mao.t1.FileAIO:  开始读取...
2023-03-16  21:36:19.114  [Thread-33] DEBUG mao.t1.FileAIO:  读取完成：8
2023-03-16  21:36:19.116  [Thread-33] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [0], limit: [8]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 0d 0a 00 00 00 00 00 00 00 00 |123456..........|
+--------+-------------------------------------------------+----------------+
```







## 网络 AIO

ReadHandler

```java
package mao.t2;

import lombok.extern.slf4j.Slf4j;

import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;
import java.nio.charset.Charset;

/**
 * Project name(项目名称)：Netty_AIO
 * Package(包名): mao.t2
 * Class(类名): ReadHandler
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/16
 * Time(创建时间)： 21:40
 * Version(版本): 1.0
 * Description(描述)： 读事件处理器
 */

@Slf4j
public class ReadHandler implements CompletionHandler<Integer, ByteBuffer>
{

    /**
     * 异步套接字通道
     */
    private final AsynchronousSocketChannel asynchronousSocketChannel;

    /**
     * 读事件处理器构造方法
     *
     * @param asynchronousSocketChannel 异步套接字通道
     */
    public ReadHandler(AsynchronousSocketChannel asynchronousSocketChannel)
    {
        this.asynchronousSocketChannel = asynchronousSocketChannel;
    }

    @Override
    public void completed(Integer result, ByteBuffer attachment)
    {
        try
        {
            if (result == -1)
            {
                //已读完
                log.debug("读事件处理完成，关闭通道：" + asynchronousSocketChannel);
                asynchronousSocketChannel.close();
            }
            log.debug("读事件：" + asynchronousSocketChannel);
            attachment.flip();
            CharBuffer charBuffer = Charset.defaultCharset().decode(attachment);
            log.debug(charBuffer.toString());
            attachment.clear();
            //处理完第一个 read 时，需要再次调用 read 方法来处理下一个 read 事件
            asynchronousSocketChannel.read(attachment, attachment, this);
        }
        catch (Exception e)
        {
            log.warn("读取时出现异常：", e);
        }
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment)
    {
        log.error("读取失败：", exc);
        try
        {
            asynchronousSocketChannel.close();
        }
        catch (IOException ignored)
        {
        }
    }
}
```





WriteHandler

```java
package mao.t2;

import lombok.extern.slf4j.Slf4j;

import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;
import java.nio.charset.Charset;

/**
 * Project name(项目名称)：Netty_AIO
 * Package(包名): mao.t2
 * Class(类名): WriteHandler
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/16
 * Time(创建时间)： 21:40
 * Version(版本): 1.0
 * Description(描述)： 写事件处理器
 */

@Slf4j
public class WriteHandler implements CompletionHandler<Integer, ByteBuffer>
{

    /**
     * 异步套接字通道
     */
    private final AsynchronousSocketChannel asynchronousSocketChannel;

    /**
     * 写事件处理器构造方法
     *
     * @param asynchronousSocketChannel 异步套接字通道
     */
    public WriteHandler(AsynchronousSocketChannel asynchronousSocketChannel)
    {
        this.asynchronousSocketChannel = asynchronousSocketChannel;
    }


    @Override
    public void completed(Integer result, ByteBuffer attachment)
    {
        log.debug("写事件");
        if (attachment.hasRemaining())
        {
            //如果有剩余内容,继续写
            asynchronousSocketChannel.write(attachment);
        }
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment)
    {
        log.error("写错误：", exc);
    }
}
```





AcceptHandler

```java
package mao.t2;

import lombok.extern.slf4j.Slf4j;

import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousServerSocketChannel;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;
import java.nio.charset.Charset;

/**
 * Project name(项目名称)：Netty_AIO
 * Package(包名): mao.t2
 * Class(类名): AcceptHandler
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/16
 * Time(创建时间)： 21:40
 * Version(版本): 1.0
 * Description(描述)： 接收事件处理器
 */

@Slf4j
public class AcceptHandler implements CompletionHandler<AsynchronousSocketChannel, Object>
{
    /**
     * 异步服务器套接字通道
     */
    private final AsynchronousServerSocketChannel asynchronousServerSocketChannel;

    /**
     * 接收事件处理器构造方法
     *
     * @param asynchronousServerSocketChannel 异步服务器套接字通道
     */
    public AcceptHandler(AsynchronousServerSocketChannel asynchronousServerSocketChannel)
    {
        this.asynchronousServerSocketChannel = asynchronousServerSocketChannel;
    }


    @Override
    public void completed(AsynchronousSocketChannel result, Object attachment)
    {
        try
        {
            log.debug("接收事件：" + asynchronousServerSocketChannel.toString());
            ByteBuffer buffer = ByteBuffer.allocate(16);
            //读事件
            result.read(buffer, buffer, new ReadHandler(result));
            //写事件
            result.write(Charset.defaultCharset().encode("hello"),
                    ByteBuffer.allocate(16), new WriteHandler(result));
            //处理完第一个 accept事件时，需要再次调用 accept 方法来处理下一个 accept 事件
            asynchronousServerSocketChannel.accept(null, this);
        }
        catch (Exception e)
        {
            log.warn("处理接收事件时出现异常：" + e);
        }
    }

    @Override
    public void failed(Throwable exc, Object attachment)
    {
        log.error("接收异常：" + exc);
    }
}
```





AioServer

```java
package mao.t2;

import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.net.InetSocketAddress;
import java.nio.channels.AsynchronousServerSocketChannel;
import java.util.concurrent.locks.LockSupport;

/**
 * Project name(项目名称)：Netty_AIO
 * Package(包名): mao.t2
 * Class(类名): AioServer
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/16
 * Time(创建时间)： 21:39
 * Version(版本): 1.0
 * Description(描述)： 无
 */

@Slf4j
public class AioServer
{
    @SneakyThrows
    public static void main(String[] args)
    {
        AsynchronousServerSocketChannel asynchronousServerSocketChannel = AsynchronousServerSocketChannel.open();
        asynchronousServerSocketChannel.bind(new InetSocketAddress(8080));
        asynchronousServerSocketChannel.accept(null, new AcceptHandler(asynchronousServerSocketChannel));
        log.debug("注册服务");

        //因为是异步，所以要阻塞
        LockSupport.park();
    }
}
```





Client

```java
package mao.t2;

import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import mao.utils.ByteBufferUtil;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.nio.charset.Charset;
import java.util.Iterator;

/**
 * Project name(项目名称)：Netty_AIO
 * Package(包名): mao.t2
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/16
 * Time(创建时间)： 22:05
 * Version(版本): 1.0
 * Description(描述)： 客户端
 */

@Slf4j
public class Client
{
    @SneakyThrows
    public static void main(String[] args)
    {
        Selector selector = Selector.open();
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.configureBlocking(false);
        socketChannel.register(selector, SelectionKey.OP_CONNECT | SelectionKey.OP_READ);
        socketChannel.connect(new InetSocketAddress("localhost", 8080));
        new Thread(new Runnable()
        {
            @SneakyThrows
            @Override
            public void run()
            {
                while (true)
                {
                    log.debug("客户端写数据");
                    socketChannel.write(Charset.defaultCharset().encode("hello server!"));
                    Thread.sleep(2000);
                }
            }
        }).start();

        int count = 0;
        while (true)
        {
            selector.select();
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext())
            {
                SelectionKey selectionKey = iterator.next();
                if (selectionKey.isConnectable())
                {
                    log.debug(String.valueOf(socketChannel.finishConnect()));
                }
                //可读
                else if (selectionKey.isReadable())
                {
                    log.debug("客户端读事件");
                    ByteBuffer buffer = ByteBuffer.allocate(16);
                    count += socketChannel.read(buffer);
                    ByteBufferUtil.debugAll(buffer);
                    buffer.clear();
                }
                iterator.remove();
            }
        }
    }
}
```





服务端运行结果：

```sh
2023-03-16  22:17:33.929  [main] DEBUG mao.t2.AioServer:  注册服务
2023-03-16  22:17:38.058  [Thread-34] DEBUG mao.t2.AcceptHandler:  接收事件：sun.nio.ch.WindowsAsynchronousServerSocketChannelImpl[/0:0:0:0:0:0:0:0:8080]
2023-03-16  22:17:38.060  [Thread-33] DEBUG mao.t2.WriteHandler:  写事件
2023-03-16  22:17:38.061  [Thread-32] DEBUG mao.t2.ReadHandler:  读事件：sun.nio.ch.WindowsAsynchronousSocketChannelImpl[connected local=/127.0.0.1:8080 remote=/127.0.0.1:49998]
2023-03-16  22:17:38.061  [Thread-32] DEBUG mao.t2.ReadHandler:  hello server!
2023-03-16  22:17:40.074  [Thread-32] DEBUG mao.t2.ReadHandler:  读事件：sun.nio.ch.WindowsAsynchronousSocketChannelImpl[connected local=/127.0.0.1:8080 remote=/127.0.0.1:49998]
2023-03-16  22:17:40.074  [Thread-32] DEBUG mao.t2.ReadHandler:  hello server!
2023-03-16  22:17:42.089  [Thread-32] DEBUG mao.t2.ReadHandler:  读事件：sun.nio.ch.WindowsAsynchronousSocketChannelImpl[connected local=/127.0.0.1:8080 remote=/127.0.0.1:49998]
2023-03-16  22:17:42.089  [Thread-32] DEBUG mao.t2.ReadHandler:  hello server!
2023-03-16  22:17:44.093  [Thread-32] DEBUG mao.t2.ReadHandler:  读事件：sun.nio.ch.WindowsAsynchronousSocketChannelImpl[connected local=/127.0.0.1:8080 remote=/127.0.0.1:49998]
2023-03-16  22:17:44.093  [Thread-32] DEBUG mao.t2.ReadHandler:  hello server!
2023-03-16  22:17:46.106  [Thread-32] DEBUG mao.t2.ReadHandler:  读事件：sun.nio.ch.WindowsAsynchronousSocketChannelImpl[connected local=/127.0.0.1:8080 remote=/127.0.0.1:49998]
2023-03-16  22:17:46.106  [Thread-32] DEBUG mao.t2.ReadHandler:  hello server!
```





客户端运行结果：

```sh
2023-03-16  22:17:38.059  [Thread-1] DEBUG mao.t2.Client:  客户端写数据
2023-03-16  22:17:38.059  [main] DEBUG mao.t2.Client:  true
2023-03-16  22:17:38.061  [main] DEBUG mao.t2.Client:  客户端读事件
2023-03-16  22:17:38.064  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
+--------+-------------------- all ------------------------+----------------+
position: [16], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
+--------+-------------------------------------------------+----------------+
2023-03-16  22:17:38.069  [main] DEBUG mao.t2.Client:  客户端读事件
+--------+-------------------- all ------------------------+----------------+
position: [5], limit: [16]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-16  22:17:40.074  [Thread-1] DEBUG mao.t2.Client:  客户端写数据
2023-03-16  22:17:42.089  [Thread-1] DEBUG mao.t2.Client:  客户端写数据
2023-03-16  22:17:44.093  [Thread-1] DEBUG mao.t2.Client:  客户端写数据
2023-03-16  22:17:46.106  [Thread-1] DEBUG mao.t2.Client:  客户端写数据
```



















# Netty概述

## Netty 是什么？

Netty 是一个异步的、基于事件驱动的网络应用框架，用于快速开发可维护、高性能的网络服务器和客户端



## Netty 的地位

Netty 在 Java 网络应用框架中的地位就好比：Spring 框架在 JavaEE 开发中的地位

以下的框架都使用了 Netty，因为它们有网络通信需求！

* Cassandra - nosql 数据库
* Spark - 大数据分布式计算框架
* Hadoop - 大数据分布式存储框架
* RocketMQ - ali 开源的消息队列
* ElasticSearch - 搜索引擎
* gRPC - rpc 框架
* Dubbo - rpc 框架
* Spring 5.x - flux api 完全抛弃了 tomcat ，使用 netty 作为服务器端
* Zookeeper - 分布式协调框架







## Netty 的优势

* 解决 TCP 传输问题，如粘包、半包
* epoll 空轮询导致 CPU 100%
* 对 API 进行增强，使之更易用，如 FastThreadLocal => ThreadLocal，ByteBuf => ByteBuffer
* Netty 的开发迭代更迅速，API 更简洁、文档更优秀
* 久经考验









# Netty入门

## 目标

开发一个简单的服务器端和客户端

* 客户端向服务器端发送 hello, world
* 服务器仅接收，不返回





## maven依赖

```xml
<!--netty-->
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
```





## 服务器端

```java
package mao.t1;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_hello_world
 * Package(包名): mao.t1
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/17
 * Time(创建时间)： 13:55
 * Version(版本): 1.0
 * Description(描述)： 服务端
 */

@Slf4j
public class Server
{
    public static void main(String[] args)
    {
        new ServerBootstrap()
                //创建 NioEventLoopGroup，可以简单理解为 `线程池 + Selector`
                .group(new NioEventLoopGroup())
                //选择服务 Socket 实现类，其中 NioServerSocketChannel 表示基于 NIO 的服务器端实现
                .channel(NioServerSocketChannel.class)
                //为啥方法叫 childHandler，是接下来添加的处理器都是给 SocketChannel 用的，而不是给 ServerSocketChannel。
                // ChannelInitializer 处理器（仅执行一次），
                // 它的作用是待客户端 SocketChannel 建立连接后，执行 initChannel 以便添加更多的处理器
                .childHandler(new ChannelInitializer<NioSocketChannel>()
                {
                    /**
                     * 初始化通道
                     *
                     * @param nioSocketChannel nio套接字通道
                     * @throws Exception 异常
                     */
                    @Override
                    protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception
                    {
                        //SocketChannel 的处理器，解码 ByteBuf => String
                        nioSocketChannel.pipeline().addLast(new StringDecoder())
                                //SocketChannel 的业务处理器，使用上一个处理器的处理结果
                                .addLast(new SimpleChannelInboundHandler<String>()
                                {
                                    /**
                                     * 通道读
                                     *
                                     * @param channelHandlerContext 通道处理程序上下文
                                     * @param s                     字符串
                                     * @throws Exception 异常
                                     */
                                    @Override
                                    protected void channelRead0(ChannelHandlerContext channelHandlerContext, String s)
                                            throws Exception
                                    {
                                        log.debug(channelHandlerContext.toString());
                                        log.debug(s);
                                    }
                                });
                    }
                    //ServerSocketChannel 绑定的监听端口
                }).bind(8080);
    }
}
```





## 客户端

```java
package mao.t1;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.net.InetSocketAddress;

/**
 * Project name(项目名称)：Netty_hello_world
 * Package(包名): mao.t1
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/17
 * Time(创建时间)： 14:18
 * Version(版本): 1.0
 * Description(描述)： 客户端
 */

@Slf4j
public class Client
{
    @SneakyThrows
    public static void main(String[] args)
    {
        new Bootstrap()
                //创建 NioEventLoopGroup，同 Server
                .group(new NioEventLoopGroup())
                //选择客户 Socket 实现类，NioSocketChannel 表示基于 NIO 的客户端实现
                .channel(NioSocketChannel.class)
                //添加 SocketChannel 的处理器，ChannelInitializer 处理器（仅执行一次），
                // 它的作用是待客户端 SocketChannel 建立连接后，执行 initChannel 以便添加更多的处理器
                .handler(new ChannelInitializer<Channel>()
                {
                    /**
                     * 初始化通道
                     *
                     * @param channel 通道
                     * @throws Exception 异常
                     */
                    @Override
                    protected void initChannel(Channel channel) throws Exception
                    {
                        //消息会经过通道 handler 处理，这里是将 String => ByteBuf 发出
                        channel.pipeline().addLast(new StringEncoder());
                    }
                })
                //指定要连接的服务器和端口
                .connect(new InetSocketAddress(8080))
                //Netty 中很多方法都是异步的，如 connect，这时需要使用 sync 方法等待 connect 建立连接完毕
                .sync()
                //获取 channel 对象，它即为通道抽象，可以进行数据读写操作
                .channel()
                //写入消息并清空缓冲区
                .writeAndFlush("hello world.");
    }
}
```





服务端运行结果：

```sh
2023-03-17  14:01:23.644  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
2023-03-17  14:01:23.650  [main] DEBUG io.netty.channel.MultithreadEventLoopGroup:  -Dio.netty.eventLoopThreads: 64
2023-03-17  14:01:23.661  [main] DEBUG io.netty.util.internal.InternalThreadLocalMap:  -Dio.netty.threadLocalMap.stringBuilder.initialSize: 1024
2023-03-17  14:01:23.661  [main] DEBUG io.netty.util.internal.InternalThreadLocalMap:  -Dio.netty.threadLocalMap.stringBuilder.maxSize: 4096
2023-03-17  14:01:23.666  [main] DEBUG io.netty.channel.nio.NioEventLoop:  -Dio.netty.noKeySetOptimization: false
2023-03-17  14:01:23.666  [main] DEBUG io.netty.channel.nio.NioEventLoop:  -Dio.netty.selectorAutoRebuildThreshold: 512
2023-03-17  14:01:23.672  [main] DEBUG io.netty.util.internal.PlatformDependent:  Platform: Windows
2023-03-17  14:01:23.674  [main] DEBUG io.netty.util.internal.PlatformDependent0:  -Dio.netty.noUnsafe: false
2023-03-17  14:01:23.674  [main] DEBUG io.netty.util.internal.PlatformDependent0:  Java version: 16
2023-03-17  14:01:23.675  [main] DEBUG io.netty.util.internal.PlatformDependent0:  sun.misc.Unsafe.theUnsafe: available
2023-03-17  14:01:23.675  [main] DEBUG io.netty.util.internal.PlatformDependent0:  sun.misc.Unsafe.copyMemory: available
2023-03-17  14:01:23.676  [main] DEBUG io.netty.util.internal.PlatformDependent0:  java.nio.Buffer.address: available
2023-03-17  14:01:23.676  [main] DEBUG io.netty.util.internal.PlatformDependent0:  direct buffer constructor: unavailable
java.lang.UnsupportedOperationException: Reflective setAccessible(true) disabled

2023-03-17  14:01:23.684  [main] DEBUG io.netty.util.internal.PlatformDependent0:  java.nio.Bits.unaligned: available, true
2023-03-17  14:01:23.684  [main] DEBUG io.netty.util.internal.PlatformDependent0:  jdk.internal.misc.Unsafe.allocateUninitializedArray(int): unavailable
java.lang.IllegalAccessException: class io.netty.util.internal.PlatformDependent0$6 cannot access class jdk.internal.misc.Unsafe (in module java.base) because module java.base does not export jdk.internal.misc to unnamed module @70a9f84e

2023-03-17  14:01:23.685  [main] DEBUG io.netty.util.internal.PlatformDependent0:  java.nio.DirectByteBuffer.<init>(long, int): unavailable
2023-03-17  14:01:23.685  [main] DEBUG io.netty.util.internal.PlatformDependent:  sun.misc.Unsafe: available
2023-03-17  14:01:23.685  [main] DEBUG io.netty.util.internal.PlatformDependent:  maxDirectMemory: 8522825728 bytes (maybe)
2023-03-17  14:01:23.686  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.tmpdir: C:\Users\mao\AppData\Local\Temp (java.io.tmpdir)
2023-03-17  14:01:23.686  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.bitMode: 64 (sun.arch.data.model)
2023-03-17  14:01:23.686  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.maxDirectMemory: -1 bytes
2023-03-17  14:01:23.687  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.uninitializedArrayAllocationThreshold: -1
2023-03-17  14:01:23.688  [main] DEBUG io.netty.util.internal.CleanerJava9:  java.nio.ByteBuffer.cleaner(): available
2023-03-17  14:01:23.688  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.noPreferDirect: false
2023-03-17  14:01:23.691  [main] DEBUG io.netty.util.internal.PlatformDependent:  org.jctools-core.MpscChunkedArrayQueue: available
2023-03-17  14:01:23.806  [main] DEBUG io.netty.channel.DefaultChannelId:  -Dio.netty.processId: 9908 (auto-detected)
2023-03-17  14:01:23.808  [main] DEBUG io.netty.util.NetUtil:  -Djava.net.preferIPv4Stack: false
2023-03-17  14:01:23.808  [main] DEBUG io.netty.util.NetUtil:  -Djava.net.preferIPv6Addresses: false
2023-03-17  14:01:23.816  [main] DEBUG io.netty.util.NetUtil:  Loopback interface: lo (Software Loopback Interface 1, 127.0.0.1)
2023-03-17  14:01:23.816  [main] DEBUG io.netty.util.NetUtil:  Failed to get SOMAXCONN from sysctl and file \proc\sys\net\core\somaxconn. Default: 200
2023-03-17  14:01:23.872  [main] DEBUG io.netty.channel.DefaultChannelId:  -Dio.netty.machineId: 70:d8:23:ff:fe:4a:12:94 (auto-detected)
2023-03-17  14:01:23.878  [main] DEBUG io.netty.util.ResourceLeakDetector:  -Dio.netty.leakDetection.level: simple
2023-03-17  14:01:23.878  [main] DEBUG io.netty.util.ResourceLeakDetector:  -Dio.netty.leakDetection.targetRecords: 4
2023-03-17  14:01:23.895  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.numHeapArenas: 64
2023-03-17  14:01:23.895  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.numDirectArenas: 64
2023-03-17  14:01:23.896  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.pageSize: 8192
2023-03-17  14:01:23.896  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.maxOrder: 11
2023-03-17  14:01:23.896  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.chunkSize: 16777216
2023-03-17  14:01:23.896  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.tinyCacheSize: 512
2023-03-17  14:01:23.896  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.smallCacheSize: 256
2023-03-17  14:01:23.896  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.normalCacheSize: 64
2023-03-17  14:01:23.896  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.maxCachedBufferCapacity: 32768
2023-03-17  14:01:23.896  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.cacheTrimInterval: 8192
2023-03-17  14:01:23.896  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.cacheTrimIntervalMillis: 0
2023-03-17  14:01:23.896  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.useCacheForAllThreads: true
2023-03-17  14:01:23.897  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.maxCachedByteBuffersPerChunk: 1023
2023-03-17  14:01:23.904  [main] DEBUG io.netty.buffer.ByteBufUtil:  -Dio.netty.allocator.type: pooled
2023-03-17  14:01:23.904  [main] DEBUG io.netty.buffer.ByteBufUtil:  -Dio.netty.threadLocalDirectBufferSize: 0
2023-03-17  14:01:23.904  [main] DEBUG io.netty.buffer.ByteBufUtil:  -Dio.netty.maxThreadLocalCharBufferSize: 16384
2023-03-17  14:25:15.366  [nioEventLoopGroup-2-2] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-17  14:25:15.367  [nioEventLoopGroup-2-2] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-17  14:25:15.367  [nioEventLoopGroup-2-2] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-17  14:25:15.367  [nioEventLoopGroup-2-2] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-17  14:25:15.370  [nioEventLoopGroup-2-2] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-17  14:25:15.370  [nioEventLoopGroup-2-2] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-17  14:25:15.371  [nioEventLoopGroup-2-2] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@2283b228
2023-03-17  14:25:15.381  [nioEventLoopGroup-2-2] DEBUG mao.t1.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xef4fe496, L:/127.0.0.1:8080 - R:/127.0.0.1:54853])
2023-03-17  14:25:15.381  [nioEventLoopGroup-2-2] DEBUG mao.t1.Server:  hello world.
```



客户端运行结果：

```sh
2023-03-17  14:25:15.088  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
2023-03-17  14:25:15.099  [main] DEBUG io.netty.channel.MultithreadEventLoopGroup:  -Dio.netty.eventLoopThreads: 64
2023-03-17  14:25:15.110  [main] DEBUG io.netty.util.internal.InternalThreadLocalMap:  -Dio.netty.threadLocalMap.stringBuilder.initialSize: 1024
2023-03-17  14:25:15.110  [main] DEBUG io.netty.util.internal.InternalThreadLocalMap:  -Dio.netty.threadLocalMap.stringBuilder.maxSize: 4096
2023-03-17  14:25:15.115  [main] DEBUG io.netty.channel.nio.NioEventLoop:  -Dio.netty.noKeySetOptimization: false
2023-03-17  14:25:15.115  [main] DEBUG io.netty.channel.nio.NioEventLoop:  -Dio.netty.selectorAutoRebuildThreshold: 512
2023-03-17  14:25:15.127  [main] DEBUG io.netty.util.internal.PlatformDependent:  Platform: Windows
2023-03-17  14:25:15.128  [main] DEBUG io.netty.util.internal.PlatformDependent0:  -Dio.netty.noUnsafe: false
2023-03-17  14:25:15.129  [main] DEBUG io.netty.util.internal.PlatformDependent0:  Java version: 16
2023-03-17  14:25:15.130  [main] DEBUG io.netty.util.internal.PlatformDependent0:  sun.misc.Unsafe.theUnsafe: available
2023-03-17  14:25:15.131  [main] DEBUG io.netty.util.internal.PlatformDependent0:  sun.misc.Unsafe.copyMemory: available
2023-03-17  14:25:15.131  [main] DEBUG io.netty.util.internal.PlatformDependent0:  java.nio.Buffer.address: available
2023-03-17  14:25:15.131  [main] DEBUG io.netty.util.internal.PlatformDependent0:  direct buffer constructor: unavailable
java.lang.UnsupportedOperationException: Reflective setAccessible(true) disabled
2023-03-17  14:25:15.140  [main] DEBUG io.netty.util.internal.PlatformDependent0:  java.nio.Bits.unaligned: available, true
2023-03-17  14:25:15.140  [main] DEBUG io.netty.util.internal.PlatformDependent0:  jdk.internal.misc.Unsafe.allocateUninitializedArray(int): unavailable
java.lang.IllegalAccessException: class io.netty.util.internal.PlatformDependent0$6 cannot access class jdk.internal.misc.Unsafe (in module java.base) because module java.base does not export jdk.internal.misc to unnamed module @70a9f84e
2023-03-17  14:25:15.141  [main] DEBUG io.netty.util.internal.PlatformDependent0:  java.nio.DirectByteBuffer.<init>(long, int): unavailable
2023-03-17  14:25:15.141  [main] DEBUG io.netty.util.internal.PlatformDependent:  sun.misc.Unsafe: available
2023-03-17  14:25:15.141  [main] DEBUG io.netty.util.internal.PlatformDependent:  maxDirectMemory: 8522825728 bytes (maybe)
2023-03-17  14:25:15.141  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.tmpdir: C:\Users\mao\AppData\Local\Temp (java.io.tmpdir)
2023-03-17  14:25:15.142  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.bitMode: 64 (sun.arch.data.model)
2023-03-17  14:25:15.142  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.maxDirectMemory: -1 bytes
2023-03-17  14:25:15.142  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.uninitializedArrayAllocationThreshold: -1
2023-03-17  14:25:15.143  [main] DEBUG io.netty.util.internal.CleanerJava9:  java.nio.ByteBuffer.cleaner(): available
2023-03-17  14:25:15.143  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.noPreferDirect: false
2023-03-17  14:25:15.146  [main] DEBUG io.netty.util.internal.PlatformDependent:  org.jctools-core.MpscChunkedArrayQueue: available
2023-03-17  14:25:15.259  [main] DEBUG io.netty.channel.DefaultChannelId:  -Dio.netty.processId: 2604 (auto-detected)
2023-03-17  14:25:15.261  [main] DEBUG io.netty.util.NetUtil:  -Djava.net.preferIPv4Stack: false
2023-03-17  14:25:15.261  [main] DEBUG io.netty.util.NetUtil:  -Djava.net.preferIPv6Addresses: false
2023-03-17  14:25:15.269  [main] DEBUG io.netty.util.NetUtil:  Loopback interface: lo (Software Loopback Interface 1, 127.0.0.1)
2023-03-17  14:25:15.270  [main] DEBUG io.netty.util.NetUtil:  Failed to get SOMAXCONN from sysctl and file \proc\sys\net\core\somaxconn. Default: 200
2023-03-17  14:25:15.309  [main] DEBUG io.netty.channel.DefaultChannelId:  -Dio.netty.machineId: 70:d8:23:ff:fe:4a:12:94 (auto-detected)
2023-03-17  14:25:15.315  [main] DEBUG io.netty.util.ResourceLeakDetector:  -Dio.netty.leakDetection.level: simple
2023-03-17  14:25:15.316  [main] DEBUG io.netty.util.ResourceLeakDetector:  -Dio.netty.leakDetection.targetRecords: 4
2023-03-17  14:25:15.332  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.numHeapArenas: 64
2023-03-17  14:25:15.332  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.numDirectArenas: 64
2023-03-17  14:25:15.332  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.pageSize: 8192
2023-03-17  14:25:15.332  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.maxOrder: 11
2023-03-17  14:25:15.332  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.chunkSize: 16777216
2023-03-17  14:25:15.332  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.tinyCacheSize: 512
2023-03-17  14:25:15.332  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.smallCacheSize: 256
2023-03-17  14:25:15.332  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.normalCacheSize: 64
2023-03-17  14:25:15.333  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.maxCachedBufferCapacity: 32768
2023-03-17  14:25:15.333  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.cacheTrimInterval: 8192
2023-03-17  14:25:15.333  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.cacheTrimIntervalMillis: 0
2023-03-17  14:25:15.333  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.useCacheForAllThreads: true
2023-03-17  14:25:15.333  [main] DEBUG io.netty.buffer.PooledByteBufAllocator:  -Dio.netty.allocator.maxCachedByteBuffersPerChunk: 1023
2023-03-17  14:25:15.338  [main] DEBUG io.netty.buffer.ByteBufUtil:  -Dio.netty.allocator.type: pooled
2023-03-17  14:25:15.338  [main] DEBUG io.netty.buffer.ByteBufUtil:  -Dio.netty.threadLocalDirectBufferSize: 0
2023-03-17  14:25:15.338  [main] DEBUG io.netty.buffer.ByteBufUtil:  -Dio.netty.maxThreadLocalCharBufferSize: 16384
2023-03-17  14:25:15.351  [main] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-17  14:25:15.352  [main] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-17  14:25:15.352  [main] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-17  14:25:15.352  [main] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-17  14:25:15.358  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-17  14:25:15.358  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-17  14:25:15.359  [nioEventLoopGroup-2-1] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@43140ebb
```









## 总结

* 把 channel 理解为数据的通道
* 把 msg 理解为流动的数据，最开始输入是 ByteBuf，但经过 pipeline 的加工，会变成其它类型对象，最后输出又变成 ByteBuf
* 把 handler 理解为数据的处理工序
  * 工序有多道，合在一起就是 pipeline，pipeline 负责发布事件（读、读取完成...）传播给每个 handler， handler 对自己感兴趣的事件进行处理（重写了相应事件处理方法）
  * handler 分 Inbound 和 Outbound 两类
* 把 eventLoop 理解为处理数据的工人
  * 工人可以管理多个 channel 的 io 操作，并且一旦工人负责了某个 channel，就要负责到底（绑定）
  * 工人既可以执行 io 操作，也可以进行任务处理，每位工人有任务队列，队列里可以堆放多个 channel 的待处理任务，任务分为普通任务、定时任务
  * 工人按照 pipeline 顺序，依次按照 handler 的规划（代码）处理数据，可以为每道工序指定不同的工人











# Netty组件

## EventLoop

**事件循环对象**

EventLoop 本质是一个单线程执行器（同时维护了一个 Selector），里面有 run 方法处理 Channel 上源源不断的 io 事件。

它的继承关系比较复杂

* 一条线是继承自 j.u.c.ScheduledExecutorService 因此包含了线程池中所有的方法
* 另一条线是继承自 netty 自己的 OrderedEventExecutor，
  * 提供了 boolean inEventLoop(Thread thread) 方法判断一个线程是否属于此 EventLoop
  * 提供了 parent 方法来看看自己属于哪个 EventLoopGroup



![image-20230317235126857](img/Netty学习笔记/image-20230317235126857.png)



![image-20230317235136276](img/Netty学习笔记/image-20230317235136276.png)



![image-20230317235242949](img/Netty学习笔记/image-20230317235242949.png)



![image-20230317235349870](img/Netty学习笔记/image-20230317235349870.png)



![image-20230317235359042](img/Netty学习笔记/image-20230317235359042.png)



![image-20230317235406745](img/Netty学习笔记/image-20230317235406745.png)



![image-20230317235424490](img/Netty学习笔记/image-20230317235424490.png)





**事件循环组**

EventLoopGroup 是一组 EventLoop，Channel 一般会调用 EventLoopGroup 的 register 方法来绑定其中一个 EventLoop，后续这个 Channel 上的 io 事件都由此 EventLoop 来处理（保证了 io 事件处理时的线程安全）

* 继承自 netty 自己的 EventExecutorGroup
  * 实现了 Iterable 接口提供遍历 EventLoop 的能力
  * 另有 next 方法获取集合中下一个 EventLoop



![image-20230317235800820](img/Netty学习笔记/image-20230317235800820.png)



![image-20230317235820395](img/Netty学习笔记/image-20230317235820395.png)





使用示例：

```java
package mao.t1;

import io.netty.channel.DefaultEventLoopGroup;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t1
 * Class(类名): EventLoopGroupTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/17
 * Time(创建时间)： 23:59
 * Version(版本): 1.0
 * Description(描述)： EventLoopGroup测试
 */

@Slf4j
public class EventLoopGroupTest
{
    @SneakyThrows
    public static void main(String[] args)
    {
        //内部创建了三个EventLoop,每个EventLoop维护一个线程
        DefaultEventLoopGroup defaultEventLoopGroup = new DefaultEventLoopGroup(3);
        log.debug(defaultEventLoopGroup.next().toString());
        log.debug(defaultEventLoopGroup.next().toString());
        log.debug(defaultEventLoopGroup.next().toString());
        //第4个和第一个地址一样
        log.debug(defaultEventLoopGroup.next().toString());
        defaultEventLoopGroup.submit(() -> log.debug("1"));
        Thread.sleep(100);
        defaultEventLoopGroup.submit(() -> log.debug("2"));
        Thread.sleep(100);
        defaultEventLoopGroup.submit(() -> log.debug("3"));
        Thread.sleep(100);
        //循环
        defaultEventLoopGroup.submit(() -> log.debug("4"));
        Thread.sleep(100);
        defaultEventLoopGroup.submit(() -> log.debug("5"));
        Thread.sleep(100);
        //和直接调用submit方法一样
        defaultEventLoopGroup.next().submit(() -> log.debug("6"));
    }
}
```



运行结果：

```sh
2023-03-18  00:05:25.636  [main] DEBUG io.netty.util.internal.logging.InternalLoggerFactory:  Using SLF4J as the default logging framework
2023-03-18  00:05:25.639  [main] DEBUG io.netty.channel.MultithreadEventLoopGroup:  -Dio.netty.eventLoopThreads: 64
2023-03-18  00:05:25.644  [main] DEBUG io.netty.util.internal.InternalThreadLocalMap:  -Dio.netty.threadLocalMap.stringBuilder.initialSize: 1024
2023-03-18  00:05:25.644  [main] DEBUG io.netty.util.internal.InternalThreadLocalMap:  -Dio.netty.threadLocalMap.stringBuilder.maxSize: 4096
2023-03-18  00:05:25.647  [main] DEBUG mao.t1.EventLoopGroupTest:  io.netty.channel.DefaultEventLoop@1ca3b418
2023-03-18  00:05:25.647  [main] DEBUG mao.t1.EventLoopGroupTest:  io.netty.channel.DefaultEventLoop@58cbafc2
2023-03-18  00:05:25.648  [main] DEBUG mao.t1.EventLoopGroupTest:  io.netty.channel.DefaultEventLoop@2034b64c
2023-03-18  00:05:25.648  [main] DEBUG mao.t1.EventLoopGroupTest:  io.netty.channel.DefaultEventLoop@1ca3b418
2023-03-18  00:05:25.649  [defaultEventLoopGroup-2-1] DEBUG mao.t1.EventLoopGroupTest:  1
2023-03-18  00:05:25.756  [defaultEventLoopGroup-2-2] DEBUG mao.t1.EventLoopGroupTest:  2
2023-03-18  00:05:25.867  [defaultEventLoopGroup-2-3] DEBUG mao.t1.EventLoopGroupTest:  3
2023-03-18  00:05:25.979  [defaultEventLoopGroup-2-1] DEBUG mao.t1.EventLoopGroupTest:  4
2023-03-18  00:05:26.088  [defaultEventLoopGroup-2-2] DEBUG mao.t1.EventLoopGroupTest:  5
2023-03-18  00:05:26.197  [defaultEventLoopGroup-2-3] DEBUG mao.t1.EventLoopGroupTest:  6
```



![image-20230318000712126](img/Netty学习笔记/image-20230318000712126.png)







也可以使用 for 循环:

```java
package mao.t1;

import io.netty.channel.DefaultEventLoopGroup;
import io.netty.util.concurrent.EventExecutor;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t1
 * Class(类名): EventLoopGroupTest2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/18
 * Time(创建时间)： 0:08
 * Version(版本): 1.0
 * Description(描述)： EventLoopGroup测试
 */

@Slf4j
public class EventLoopGroupTest2
{
    public static void main(String[] args)
    {
        //内部创建了三个EventLoop,每个EventLoop维护一个线程
        DefaultEventLoopGroup defaultEventLoopGroup = new DefaultEventLoopGroup(3);
        for (EventExecutor eventExecutor : defaultEventLoopGroup)
        {
            log.debug(eventExecutor.toString());
        }
        //再次调用
        for (EventExecutor eventExecutor : defaultEventLoopGroup)
        {
            log.debug(eventExecutor.toString());
        }
    }
}
```



运行结果：

```sh
2023-03-18  00:09:15.981  [main] DEBUG mao.t1.EventLoopGroupTest2:  io.netty.channel.DefaultEventLoop@1ca3b418
2023-03-18  00:09:15.981  [main] DEBUG mao.t1.EventLoopGroupTest2:  io.netty.channel.DefaultEventLoop@58cbafc2
2023-03-18  00:09:15.981  [main] DEBUG mao.t1.EventLoopGroupTest2:  io.netty.channel.DefaultEventLoop@2034b64c
2023-03-18  00:09:15.981  [main] DEBUG mao.t1.EventLoopGroupTest2:  io.netty.channel.DefaultEventLoop@1ca3b418
2023-03-18  00:09:15.981  [main] DEBUG mao.t1.EventLoopGroupTest2:  io.netty.channel.DefaultEventLoop@58cbafc2
2023-03-18  00:09:15.981  [main] DEBUG mao.t1.EventLoopGroupTest2:  io.netty.channel.DefaultEventLoop@2034b64c
```





**关闭：**

`shutdownGracefully` 方法。该方法会首先切换 `EventLoopGroup` 到关闭状态从而拒绝新的任务的加入，然后在任务队列的任务都处理完成后，停止线程的运行。从而确保整体应用是在正常有序的状态下退出的

```java
defaultEventLoopGroup.shutdownGracefully();
```







### NioEventLoop处理io事件

服务端

```java
package mao.t2;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t2
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/18
 * Time(创建时间)： 13:57
 * Version(版本): 1.0
 * Description(描述)： NioEventLoop处理io事件
 */

@Slf4j
public class Server
{
    @SneakyThrows
    public static void main(String[] args)
    {
        new ServerBootstrap()
                //第一个参数是处理接收事件的EventLoop，线程数量为1个，第二个参数为处理读写事件的EventLoop，线程数量为3个
                .group(new NioEventLoopGroup(1), new NioEventLoopGroup(3))
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new StringDecoder())
                                .addLast(new SimpleChannelInboundHandler<String>()
                                {
                                    @Override
                                    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception
                                    {
                                        log.debug(ctx.toString());
                                        log.debug(msg);
                                    }
                                });
                    }
                })
                .bind(8080)
                .sync();

    }
}
```



客户端

```java
package mao.t2;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.net.InetSocketAddress;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t2
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/18
 * Time(创建时间)： 14:07
 * Version(版本): 1.0
 * Description(描述)： NioEventLoop处理io事件
 */

@Slf4j
public class Client
{
    @SneakyThrows
    public static void main(String[] args)
    {

        for (int i = 0; i < 5; i++)
        {
            Channel channel = new Bootstrap()
                    .group(new NioEventLoopGroup(1))
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<NioSocketChannel>()
                    {
                        @Override
                        protected void initChannel(NioSocketChannel ch) throws Exception
                        {
                            ch.pipeline().addLast(new StringEncoder())
                                    .addLast(new LoggingHandler(LogLevel.DEBUG));
                        }
                    })
                    .connect(new InetSocketAddress(8080)).sync().channel();

            channel.writeAndFlush("客户端" + (i + 1) + " -1");
            Thread.sleep(100);
            channel.writeAndFlush("客户端" + (i + 1) + " -2");
            Thread.sleep(100);
            channel.writeAndFlush("客户端" + (i + 1) + " -3");
            Thread.sleep(100);
            channel.writeAndFlush("客户端" + (i + 1) + " -4");
            Thread.sleep(100);
            channel.writeAndFlush("客户端" + (i + 1) + " -5");
            Thread.sleep(100);
            channel.close();
        }

    }
}
```



服务端运行结果：

```sh
2023-03-18  14:18:43.716  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x69e92d02, L:/127.0.0.1:8080 - R:/127.0.0.1:64288])
2023-03-18  14:18:43.716  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端1 -1
2023-03-18  14:18:43.815  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x69e92d02, L:/127.0.0.1:8080 - R:/127.0.0.1:64288])
2023-03-18  14:18:43.815  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端1 -2
2023-03-18  14:18:43.928  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x69e92d02, L:/127.0.0.1:8080 - R:/127.0.0.1:64288])
2023-03-18  14:18:43.928  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端1 -3
2023-03-18  14:18:44.033  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x69e92d02, L:/127.0.0.1:8080 - R:/127.0.0.1:64288])
2023-03-18  14:18:44.033  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端1 -4
2023-03-18  14:18:44.144  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x69e92d02, L:/127.0.0.1:8080 - R:/127.0.0.1:64288])
2023-03-18  14:18:44.144  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端1 -5
2023-03-18  14:18:44.259  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xce929642, L:/127.0.0.1:8080 - R:/127.0.0.1:64289])
2023-03-18  14:18:44.259  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端2 -1
2023-03-18  14:18:44.361  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xce929642, L:/127.0.0.1:8080 - R:/127.0.0.1:64289])
2023-03-18  14:18:44.362  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端2 -2
2023-03-18  14:18:44.471  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xce929642, L:/127.0.0.1:8080 - R:/127.0.0.1:64289])
2023-03-18  14:18:44.471  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端2 -3
2023-03-18  14:18:44.581  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xce929642, L:/127.0.0.1:8080 - R:/127.0.0.1:64289])
2023-03-18  14:18:44.581  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端2 -4
2023-03-18  14:18:44.690  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xce929642, L:/127.0.0.1:8080 - R:/127.0.0.1:64289])
2023-03-18  14:18:44.690  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端2 -5
2023-03-18  14:18:44.804  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xc76a17e1, L:/127.0.0.1:8080 - R:/127.0.0.1:64290])
2023-03-18  14:18:44.805  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  客户端3 -1
2023-03-18  14:18:44.908  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xc76a17e1, L:/127.0.0.1:8080 - R:/127.0.0.1:64290])
2023-03-18  14:18:44.908  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  客户端3 -2
2023-03-18  14:18:45.015  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xc76a17e1, L:/127.0.0.1:8080 - R:/127.0.0.1:64290])
2023-03-18  14:18:45.015  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  客户端3 -3
2023-03-18  14:18:45.126  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xc76a17e1, L:/127.0.0.1:8080 - R:/127.0.0.1:64290])
2023-03-18  14:18:45.126  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  客户端3 -4
2023-03-18  14:18:45.234  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xc76a17e1, L:/127.0.0.1:8080 - R:/127.0.0.1:64290])
2023-03-18  14:18:45.234  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  客户端3 -5
2023-03-18  14:18:45.352  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xdc86e43f, L:/127.0.0.1:8080 - R:/127.0.0.1:64291])
2023-03-18  14:18:45.353  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端4 -1
2023-03-18  14:18:45.451  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xdc86e43f, L:/127.0.0.1:8080 - R:/127.0.0.1:64291])
2023-03-18  14:18:45.452  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端4 -2
2023-03-18  14:18:45.562  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xdc86e43f, L:/127.0.0.1:8080 - R:/127.0.0.1:64291])
2023-03-18  14:18:45.562  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端4 -3
2023-03-18  14:18:45.670  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xdc86e43f, L:/127.0.0.1:8080 - R:/127.0.0.1:64291])
2023-03-18  14:18:45.670  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端4 -4
2023-03-18  14:18:45.779  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xdc86e43f, L:/127.0.0.1:8080 - R:/127.0.0.1:64291])
2023-03-18  14:18:45.779  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端4 -5
2023-03-18  14:18:45.886  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x77c2d44c, L:/127.0.0.1:8080 - R:/127.0.0.1:64292])
2023-03-18  14:18:45.886  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端5 -1
2023-03-18  14:18:45.997  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x77c2d44c, L:/127.0.0.1:8080 - R:/127.0.0.1:64292])
2023-03-18  14:18:45.997  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端5 -2
2023-03-18  14:18:46.105  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x77c2d44c, L:/127.0.0.1:8080 - R:/127.0.0.1:64292])
2023-03-18  14:18:46.105  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端5 -3
2023-03-18  14:18:46.214  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x77c2d44c, L:/127.0.0.1:8080 - R:/127.0.0.1:64292])
2023-03-18  14:18:46.214  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端5 -4
2023-03-18  14:18:46.323  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x77c2d44c, L:/127.0.0.1:8080 - R:/127.0.0.1:64292])
2023-03-18  14:18:46.323  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端5 -5
```





可以看到三个工人轮流处理 channel，但工人与 channel 之间进行了绑定



![image-20230318142153209](img/Netty学习笔记/image-20230318142153209.png)





在添加一个个人，由3个个人变成4个个人：

```sh
2023-03-18  14:25:18.883  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xc25920ba, L:/127.0.0.1:8080 - R:/127.0.0.1:50228])
2023-03-18  14:25:18.883  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端1 -1
2023-03-18  14:25:18.965  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xc25920ba, L:/127.0.0.1:8080 - R:/127.0.0.1:50228])
2023-03-18  14:25:18.965  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端1 -2
2023-03-18  14:25:19.088  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xc25920ba, L:/127.0.0.1:8080 - R:/127.0.0.1:50228])
2023-03-18  14:25:19.088  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端1 -3
2023-03-18  14:25:19.199  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xc25920ba, L:/127.0.0.1:8080 - R:/127.0.0.1:50228])
2023-03-18  14:25:19.199  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端1 -4
2023-03-18  14:25:19.310  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xc25920ba, L:/127.0.0.1:8080 - R:/127.0.0.1:50228])
2023-03-18  14:25:19.311  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端1 -5
2023-03-18  14:25:19.426  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x10b46bbc, L:/127.0.0.1:8080 - R:/127.0.0.1:50229])
2023-03-18  14:25:19.427  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端2 -1
2023-03-18  14:25:19.528  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x10b46bbc, L:/127.0.0.1:8080 - R:/127.0.0.1:50229])
2023-03-18  14:25:19.529  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端2 -2
2023-03-18  14:25:19.637  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x10b46bbc, L:/127.0.0.1:8080 - R:/127.0.0.1:50229])
2023-03-18  14:25:19.638  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端2 -3
2023-03-18  14:25:19.746  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x10b46bbc, L:/127.0.0.1:8080 - R:/127.0.0.1:50229])
2023-03-18  14:25:19.746  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端2 -4
2023-03-18  14:25:19.854  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x10b46bbc, L:/127.0.0.1:8080 - R:/127.0.0.1:50229])
2023-03-18  14:25:19.854  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  客户端2 -5
2023-03-18  14:25:19.974  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x6bcf6c90, L:/127.0.0.1:8080 - R:/127.0.0.1:50231])
2023-03-18  14:25:19.974  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  客户端3 -1
2023-03-18  14:25:20.071  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x6bcf6c90, L:/127.0.0.1:8080 - R:/127.0.0.1:50231])
2023-03-18  14:25:20.071  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  客户端3 -2
2023-03-18  14:25:20.181  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x6bcf6c90, L:/127.0.0.1:8080 - R:/127.0.0.1:50231])
2023-03-18  14:25:20.182  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  客户端3 -3
2023-03-18  14:25:20.289  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x6bcf6c90, L:/127.0.0.1:8080 - R:/127.0.0.1:50231])
2023-03-18  14:25:20.289  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  客户端3 -4
2023-03-18  14:25:20.399  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x6bcf6c90, L:/127.0.0.1:8080 - R:/127.0.0.1:50231])
2023-03-18  14:25:20.399  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  客户端3 -5
2023-03-18  14:25:20.515  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x28537042, L:/127.0.0.1:8080 - R:/127.0.0.1:50232])
2023-03-18  14:25:20.515  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  客户端4 -1
2023-03-18  14:25:20.617  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x28537042, L:/127.0.0.1:8080 - R:/127.0.0.1:50232])
2023-03-18  14:25:20.618  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  客户端4 -2
2023-03-18  14:25:20.726  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x28537042, L:/127.0.0.1:8080 - R:/127.0.0.1:50232])
2023-03-18  14:25:20.726  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  客户端4 -3
2023-03-18  14:25:20.835  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x28537042, L:/127.0.0.1:8080 - R:/127.0.0.1:50232])
2023-03-18  14:25:20.836  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  客户端4 -4
2023-03-18  14:25:20.943  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x28537042, L:/127.0.0.1:8080 - R:/127.0.0.1:50232])
2023-03-18  14:25:20.943  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  客户端4 -5
2023-03-18  14:25:21.062  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x65f7825d, L:/127.0.0.1:8080 - R:/127.0.0.1:50233])
2023-03-18  14:25:21.062  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端5 -1
2023-03-18  14:25:21.162  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x65f7825d, L:/127.0.0.1:8080 - R:/127.0.0.1:50233])
2023-03-18  14:25:21.163  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端5 -2
2023-03-18  14:25:21.272  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x65f7825d, L:/127.0.0.1:8080 - R:/127.0.0.1:50233])
2023-03-18  14:25:21.272  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端5 -3
2023-03-18  14:25:21.379  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x65f7825d, L:/127.0.0.1:8080 - R:/127.0.0.1:50233])
2023-03-18  14:25:21.379  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端5 -4
2023-03-18  14:25:21.489  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x65f7825d, L:/127.0.0.1:8080 - R:/127.0.0.1:50233])
2023-03-18  14:25:21.489  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  客户端5 -5
```











### handler执行中更换工人

思路：

* 如果两个 handler 绑定的是同一个线程，那么就直接调用
* 否则，把要调用的代码封装为一个任务对象，由下一个 handler 的线程来调用



服务端

```java
package mao.t3;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.EventLoop;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.util.concurrent.EventExecutor;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t3
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/18
 * Time(创建时间)： 21:13
 * Version(版本): 1.0
 * Description(描述)： handler执行中更换工人
 */

@Slf4j
public class Server
{
    @SneakyThrows
    public static void main(String[] args)
    {
        NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup(3);
        new ServerBootstrap()
                //第一个参数是处理接收事件的EventLoop，线程数量为1个，第二个参数为处理读写事件的EventLoop，线程数量为3个
                .group(new NioEventLoopGroup(1), nioEventLoopGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new StringDecoder())
                                .addLast(new SimpleChannelInboundHandler<String>()
                                {
                                    @Override
                                    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception
                                    {
                                        log.debug("当前处理的工人：" + Thread.currentThread().getName());
                                        EventLoop next = nioEventLoopGroup.next();
                                        //下一个 handler 的事件循环是否与当前的事件循环是同一个线程
                                        if (next.inEventLoop())
                                        {
                                            log.debug("更换工人，是同一个工人");
                                            log.debug(ctx.toString());
                                            log.debug(msg);
                                        }
                                        else
                                        {
                                            log.debug("更换工人，不是同一个工人");
                                            next.execute(new Runnable()
                                            {
                                                @Override
                                                public void run()
                                                {
                                                    log.debug("现在处理的工人：" + Thread.currentThread().getName());
                                                    log.debug(ctx.toString());
                                                    log.debug(msg);
                                                }
                                            });
                                        }
                                    }
                                });
                    }
                })
                .bind(8080)
                .sync();

    }
}

```



客户端

```java
package mao.t3;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.SneakyThrows;

import java.net.InetSocketAddress;
import java.util.Scanner;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t3
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/18
 * Time(创建时间)： 21:15
 * Version(版本): 1.0
 * Description(描述)： handler执行中更换工人
 */

public class Client
{
    @SneakyThrows
    public static void main(String[] args)
    {
        while (true)
        {
            Channel channel = new Bootstrap()
                    .group(new NioEventLoopGroup(1))
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<NioSocketChannel>()
                    {
                        @Override
                        protected void initChannel(NioSocketChannel ch) throws Exception
                        {
                            ch.pipeline().addLast(new StringEncoder())
                                    .addLast(new LoggingHandler(LogLevel.DEBUG));
                        }
                    })
                    .connect(new InetSocketAddress(8080)).sync().channel();

            channel.writeAndFlush("hello");
            Thread.sleep(100);
            Scanner input = new Scanner(System.in);
            input.nextLine();
            channel.close();
        }
    }


}
```



服务端运行结果：

```sh
2023-03-18  21:30:47.807  [nioEventLoopGroup-2-1] DEBUG mao.t3.Server:  当前处理的工人：nioEventLoopGroup-2-1
2023-03-18  21:30:47.807  [nioEventLoopGroup-2-1] DEBUG mao.t3.Server:  更换工人，不是同一个工人
2023-03-18  21:30:47.808  [nioEventLoopGroup-2-2] DEBUG mao.t3.Server:  现在处理的工人：nioEventLoopGroup-2-2
2023-03-18  21:30:47.811  [nioEventLoopGroup-2-2] DEBUG mao.t3.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0x280521ce, L:/127.0.0.1:8080 - R:/127.0.0.1:53483])
2023-03-18  21:30:47.811  [nioEventLoopGroup-2-2] DEBUG mao.t3.Server:  hello
2023-03-18  21:31:06.647  [nioEventLoopGroup-2-3] DEBUG mao.t3.Server:  当前处理的工人：nioEventLoopGroup-2-3
2023-03-18  21:31:06.647  [nioEventLoopGroup-2-3] DEBUG mao.t3.Server:  更换工人，不是同一个工人
2023-03-18  21:31:06.647  [nioEventLoopGroup-2-1] DEBUG mao.t3.Server:  现在处理的工人：nioEventLoopGroup-2-1
2023-03-18  21:31:06.647  [nioEventLoopGroup-2-1] DEBUG mao.t3.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xac85c1c5, L:/127.0.0.1:8080 - R:/127.0.0.1:53486])
2023-03-18  21:31:06.647  [nioEventLoopGroup-2-1] DEBUG mao.t3.Server:  hello
2023-03-18  21:31:14.819  [nioEventLoopGroup-2-2] DEBUG mao.t3.Server:  当前处理的工人：nioEventLoopGroup-2-2
2023-03-18  21:31:14.819  [nioEventLoopGroup-2-2] DEBUG mao.t3.Server:  更换工人，不是同一个工人
2023-03-18  21:31:14.819  [nioEventLoopGroup-2-3] DEBUG mao.t3.Server:  现在处理的工人：nioEventLoopGroup-2-3
2023-03-18  21:31:14.819  [nioEventLoopGroup-2-3] DEBUG mao.t3.Server:  ChannelHandlerContext(Server$1$1#0, [id: 0xefff5a5b, L:/127.0.0.1:8080 - R:/127.0.0.1:53488])
2023-03-18  21:31:14.819  [nioEventLoopGroup-2-3] DEBUG mao.t3.Server:  hello
```







### NioEventLoop处理普通任务

```java
package mao.t4;

import io.netty.channel.nio.NioEventLoopGroup;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t4
 * Class(类名): NioEventLoopTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/18
 * Time(创建时间)： 21:33
 * Version(版本): 1.0
 * Description(描述)： NioEventLoop处理普通任务
 */

@Slf4j
public class NioEventLoopTest
{
    @SneakyThrows
    public static void main(String[] args)
    {

        NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup(3);
        log.debug("启动");
        for (int i = 0; i < 10; i++)
        {
            nioEventLoopGroup.execute(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug("当前线程：" + Thread.currentThread().getName());
                }
            });
            Thread.sleep(100);
        }
    }
}
```



运行结果：

```sh
2023-03-18  21:37:17.780  [main] DEBUG mao.t4.NioEventLoopTest:  启动
2023-03-18  21:37:17.782  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest:  当前线程：nioEventLoopGroup-2-1
2023-03-18  21:37:17.884  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest:  当前线程：nioEventLoopGroup-2-2
2023-03-18  21:37:17.993  [nioEventLoopGroup-2-3] DEBUG mao.t4.NioEventLoopTest:  当前线程：nioEventLoopGroup-2-3
2023-03-18  21:37:18.104  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest:  当前线程：nioEventLoopGroup-2-1
2023-03-18  21:37:18.215  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest:  当前线程：nioEventLoopGroup-2-2
2023-03-18  21:37:18.322  [nioEventLoopGroup-2-3] DEBUG mao.t4.NioEventLoopTest:  当前线程：nioEventLoopGroup-2-3
2023-03-18  21:37:18.432  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest:  当前线程：nioEventLoopGroup-2-1
2023-03-18  21:37:18.542  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest:  当前线程：nioEventLoopGroup-2-2
2023-03-18  21:37:18.654  [nioEventLoopGroup-2-3] DEBUG mao.t4.NioEventLoopTest:  当前线程：nioEventLoopGroup-2-3
2023-03-18  21:37:18.763  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest:  当前线程：nioEventLoopGroup-2-1
```







### NioEventLoop处理定时任务

```java
package mao.t4;

import io.netty.channel.nio.NioEventLoopGroup;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t4
 * Class(类名): NioEventLoopTest2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/18
 * Time(创建时间)： 21:41
 * Version(版本): 1.0
 * Description(描述)： NioEventLoop处理定时任务
 */

@Slf4j
public class NioEventLoopTest2
{

    public static void main(String[] args)
    {
        AtomicInteger count1 = new AtomicInteger(0);
        AtomicInteger count2 = new AtomicInteger(0);
        NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup(2);
        nioEventLoopGroup.scheduleAtFixedRate(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("执行定时任务1，第" + count1.incrementAndGet() + "次");
            }
        }, 0, 1000, TimeUnit.MILLISECONDS);

        nioEventLoopGroup.scheduleAtFixedRate(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("执行定时任务2，第" + count2.incrementAndGet() + "次");
            }
        }, 0, 600, TimeUnit.MILLISECONDS);
    }
}
```



运行结果：

```sh
2023-03-18  21:50:07.046  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务1，第1次
2023-03-18  21:50:07.055  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第1次
2023-03-18  21:50:07.670  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第2次
2023-03-18  21:50:08.059  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务1，第2次
2023-03-18  21:50:08.260  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第3次
2023-03-18  21:50:08.856  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第4次
2023-03-18  21:50:09.056  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务1，第3次
2023-03-18  21:50:09.464  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第5次
2023-03-18  21:50:10.054  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务1，第4次
2023-03-18  21:50:10.070  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第6次
2023-03-18  21:50:10.661  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第7次
2023-03-18  21:50:11.051  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务1，第5次
2023-03-18  21:50:11.269  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第8次
2023-03-18  21:50:11.861  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第9次
2023-03-18  21:50:12.045  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务1，第6次
2023-03-18  21:50:12.465  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第10次
2023-03-18  21:50:13.055  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务1，第7次
2023-03-18  21:50:13.055  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第11次
2023-03-18  21:50:13.660  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第12次
2023-03-18  21:50:14.048  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务1，第8次
2023-03-18  21:50:14.264  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第13次
2023-03-18  21:50:14.869  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第14次
2023-03-18  21:50:15.056  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务1，第9次
2023-03-18  21:50:15.460  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第15次
2023-03-18  21:50:16.050  [nioEventLoopGroup-2-1] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务1，第10次
2023-03-18  21:50:16.066  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第16次
2023-03-18  21:50:16.669  [nioEventLoopGroup-2-2] DEBUG mao.t4.NioEventLoopTest2:  执行定时任务2，第17次
```











## Channel

channel 的主要作用

* close() 可以用来关闭 channel
* closeFuture() 用来处理 channel 的关闭
  * sync 方法作用是同步等待 channel 关闭
  * 而 addListener 方法是异步等待 channel 关闭
* pipeline() 方法添加处理器
* write() 方法将数据写入
* writeAndFlush() 方法将数据写入并刷出





### ChannelFuture

获得ChannelFuture：

```java
package mao.t5;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.net.InetSocketAddress;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t5
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/18
 * Time(创建时间)： 21:54
 * Version(版本): 1.0
 * Description(描述)： ChannelFuture
 */

@Slf4j
public class Client
{
    @SneakyThrows
    public static void main(String[] args)
    {
        //获得channelFuture对象
        ChannelFuture channelFuture = new Bootstrap()
                .channel(NioSocketChannel.class)
                .group(new NioEventLoopGroup())
                .handler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new StringEncoder());
                    }
                })
                .connect(new InetSocketAddress(8080));

        log.debug("------------");
        log.debug(channelFuture.toString());
        channelFuture.sync();
        channelFuture.channel().writeAndFlush("hello");
    }
}
```



**connect 方法是异步的，意味着不等连接建立，方法执行就返回了。因此 channelFuture 对象中不能立刻获得到正确的 Channel 对象**



除了用 sync 方法可以让异步操作同步以外，还可以使用回调的方式：

```java
package mao.t5;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.net.InetSocketAddress;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t5
 * Class(类名): Client2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/18
 * Time(创建时间)： 22:03
 * Version(版本): 1.0
 * Description(描述)： ChannelFuture
 */

@Slf4j
public class Client2
{
    @SneakyThrows
    public static void main(String[] args)
    {
        //获得channelFuture对象
        ChannelFuture channelFuture = new Bootstrap()
                .channel(NioSocketChannel.class)
                .group(new NioEventLoopGroup())
                .handler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new StringEncoder());
                    }
                })
                .connect(new InetSocketAddress(8080));

        log.debug("------------");
        log.debug(channelFuture.toString());
        //channelFuture.sync();
        channelFuture.addListener(new ChannelFutureListener()
        {
            /**
             * 操作完成
             *
             * @param future ChannelFuture
             * @throws Exception 异常
             */
            @Override
            public void operationComplete(ChannelFuture future) throws Exception
            {
                log.debug("连接完成");
                channelFuture.channel().writeAndFlush("hello");
            }
        });

    }
}
```







### CloseFuture

需求：输入字符串q退出，输入其他字符串发送数据，要求在调用channel.close()之后处理一些关闭相关的操作

```java
package mao.t5;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.net.InetSocketAddress;
import java.util.Scanner;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t5
 * Class(类名): Client3
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/18
 * Time(创建时间)： 22:09
 * Version(版本): 1.0
 * Description(描述)： CloseFuture
 */

@Slf4j
public class Client3
{
    @SneakyThrows
    public static void main(String[] args)
    {
        NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
        //获得channelFuture对象
        ChannelFuture channelFuture = new Bootstrap()
                .channel(NioSocketChannel.class)
                .group(nioEventLoopGroup)
                .handler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new StringEncoder());
                    }
                })
                .connect(new InetSocketAddress(8080));

        log.debug("------------");
        log.debug(channelFuture.toString());
        channelFuture.sync();
        Channel channel = channelFuture.channel();

        Scanner input = new Scanner(System.in);

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    String s = input.next();
                    if ("q".equals(s))
                    {
                        log.info("即将退出");
                        //调用close方法是异步操作，不能在close方法之后写关闭的业务逻辑
                        channel.close();
                    }
                    else
                    {
                        log.debug("即将发送的字符串：" + s);
                        channel.writeAndFlush(s);
                    }
                }
            }
        }, "input").start();

        //获得closeFuture
        ChannelFuture closeFuture = channel.closeFuture();
        closeFuture.addListener(new ChannelFutureListener()
        {
            /**
             * 操作完成(关闭)
             *
             * @param future ChannelFuture
             * @throws Exception 异常
             */
            @Override
            public void operationComplete(ChannelFuture future) throws Exception
            {
                log.debug("处理关闭之后的操作");
                nioEventLoopGroup.shutdownGracefully();
                log.info("关闭完成");
            }
        });

    }
}
```



运行结果：

```sh
2023-03-18  22:16:39.614  [main] DEBUG mao.t5.Client3:  ------------
2023-03-18  22:16:39.614  [main] DEBUG mao.t5.Client3:  AbstractBootstrap$PendingRegistrationPromise@60856961(incomplete)
123
2023-03-18  22:16:48.753  [input] DEBUG mao.t5.Client3:  即将发送的字符串：123
2023-03-18  22:16:48.756  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-18  22:16:48.756  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-18  22:16:48.756  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-18  22:16:48.756  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-18  22:16:48.763  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-18  22:16:48.764  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-18  22:16:48.764  [nioEventLoopGroup-2-1] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@71d15640
58775
2023-03-18  22:17:00.772  [input] DEBUG mao.t5.Client3:  即将发送的字符串：58775
wafag
2023-03-18  22:17:11.096  [input] DEBUG mao.t5.Client3:  即将发送的字符串：wafag
q
2023-03-18  22:17:17.811  [input] INFO  mao.t5.Client3:  即将退出
2023-03-18  22:17:17.812  [nioEventLoopGroup-2-1] DEBUG mao.t5.Client3:  处理关闭之后的操作
2023-03-18  22:17:17.818  [nioEventLoopGroup-2-1] INFO  mao.t5.Client3:  关闭完成
2023-03-18  22:17:20.063  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.PoolThreadCache:  Freed 1 thread-local buffer(s) from thread: nioEventLoopGroup-2-1
```









## 异步提升的是什么

为什么不在一个线程中去执行建立连接、去执行关闭 channel，那样不是也可以吗？非要用这么复杂的异步方式：比如一个线程发起建立连接，另一个线程去真正建立连接



可以参考**CPU的指令重排**



思考下面的场景，4 个医生给人看病，每个病人花费 20 分钟，而且医生看病的过程中是以病人为单位的，一个病人看完了，才能看下一个病人。假设病人源源不断地来，可以计算一下 4 个医生一天工作 8 小时，处理的病人总数是：`4 * 8 * 3 = 96`

![image-20230318222325096](img/Netty学习笔记/image-20230318222325096.png)





经研究发现，看病可以细分为四个步骤，经拆分后每个步骤需要 5 分钟

![image-20230318222340087](img/Netty学习笔记/image-20230318222340087.png)



因此可以做如下优化，只有一开始，医生 2、3、4 分别要等待 5、10、15 分钟才能执行工作，但只要后续病人源源不断地来，他们就能够满负荷工作，并且处理病人的能力提高到了 `4 * 8 * 12` 效率几乎是原来的四倍



![image-20230318222429099](img/Netty学习笔记/image-20230318222429099.png)





* 单线程没法异步提高效率，必须配合多线程、多核 cpu 才能发挥异步的优势
* 异步并没有缩短响应时间，反而有所增加
* 合理进行任务拆分，也是利用异步的关键







## Future和Promise

在异步处理时，经常用到这两个接口

netty 中的 Future 与 jdk 中的 Future 同名，但是是两个接口，netty 的 Future 继承自 jdk 的 Future，而 Promise 又对 netty Future 进行了扩展

* jdk Future 只能同步等待任务结束（或成功、或失败）才能得到结果
* netty Future 可以同步等待任务结束得到结果，也可以异步方式得到结果，但都是要等任务结束
* netty Promise 不仅有 netty Future 的功能，而且脱离了任务独立存在，只作为两个线程间传递结果的容器



|  功能/名称   |           jdk Future           |                         netty Future                         |   Promise    |
| :----------: | :----------------------------: | :----------------------------------------------------------: | :----------: |
|    cancel    |            取消任务            |                              -                               |      -       |
|  isCanceled  |          任务是否取消          |                              -                               |      -       |
|    isDone    | 任务是否完成，不能区分成功失败 |                              -                               |      -       |
|     get      |     获取任务结果，阻塞等待     |                              -                               |      -       |
|    getNow    |               -                |        获取任务结果，非阻塞，还未产生结果时返回 null         |      -       |
|    await     |               -                | 等待任务结束，如果任务失败，不会抛异常，而是通过 isSuccess 判断 |      -       |
|     sync     |               -                |             等待任务结束，如果任务失败，抛出异常             |      -       |
|  isSuccess   |               -                |                       判断任务是否成功                       |      -       |
|    cause     |               -                |         获取失败信息，非阻塞，如果没有失败，返回null         |      -       |
| addLinstener |               -                |                    添加回调，异步接收结果                    |      -       |
|  setSuccess  |               -                |                              -                               | 设置成功结果 |
|  setFailure  |               -                |                              -                               | 设置失败结果 |



![image-20230319222247557](img/Netty学习笔记/image-20230319222247557.png)



![image-20230319222310007](img/Netty学习笔记/image-20230319222310007.png)





### Future

**同步处理任务**

```java
package mao.t6;

import io.netty.channel.DefaultEventLoopGroup;
import io.netty.util.concurrent.Future;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.Callable;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t6
 * Class(类名): FutureTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/19
 * Time(创建时间)： 22:24
 * Version(版本): 1.0
 * Description(描述)： Future测试，同步处理任务
 */

@Slf4j
public class FutureTest
{
    @SneakyThrows
    public static void main(String[] args)
    {
        DefaultEventLoopGroup defaultEventLoopGroup = new DefaultEventLoopGroup(3);
        Future<Integer> future = defaultEventLoopGroup.submit(new Callable<Integer>()
        {
            @Override
            public Integer call() throws Exception
            {
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                log.debug("执行成功");
                return 10;
            }
        });

        log.debug(future.toString());
        //现在读取，无法读取
        Integer integer = future.getNow();
        log.debug(String.valueOf(integer));
        log.debug("开始同步等待结果");
        future.sync();
        integer = future.getNow();
        log.debug(integer.toString());
    }
}
```



运行结果：

```sh
2023-03-19  22:33:31.171  [main] DEBUG mao.t6.FutureTest:  PromiseTask@3a7442c7(incomplete, task: mao.t6.FutureTest$1@4be29ed9)
2023-03-19  22:33:31.172  [main] DEBUG mao.t6.FutureTest:  null
2023-03-19  22:33:31.172  [main] DEBUG mao.t6.FutureTest:  开始同步等待结果
2023-03-19  22:33:32.182  [defaultEventLoopGroup-2-1] DEBUG mao.t6.FutureTest:  执行成功
2023-03-19  22:33:32.182  [main] DEBUG mao.t6.FutureTest:  10
```





**异步处理任务**

```java
package mao.t6;

import io.netty.channel.DefaultEventLoopGroup;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.GenericFutureListener;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.Callable;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t6
 * Class(类名): FutureTest2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/19
 * Time(创建时间)： 22:35
 * Version(版本): 1.0
 * Description(描述)： Future测试，异步处理任务
 */

@Slf4j
public class FutureTest2
{
    public static void main(String[] args)
    {
        DefaultEventLoopGroup defaultEventLoopGroup = new DefaultEventLoopGroup(3);
        Future<Integer> future = defaultEventLoopGroup.submit(new Callable<Integer>()
        {
            @Override
            public Integer call() throws Exception
            {
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                log.debug("执行成功");
                return 20;
            }
        });

        log.debug(future.toString());
        //现在读取，无法读取
        Integer integer = future.getNow();
        log.debug(String.valueOf(integer));
        log.debug("开始异步等待结果");
        future.addListener(new GenericFutureListener<Future<? super Integer>>()
        {
            /**
             * 操作完成
             *
             * @param future Future
             * @throws Exception 异常
             */
            @Override
            public void operationComplete(Future<? super Integer> future) throws Exception
            {
                log.debug("等待完成，结果：" + future.get());
            }
        });
    }
}
```



运行结果：

```sh
2023-03-19  22:37:45.795  [main] DEBUG mao.t6.FutureTest2:  PromiseTask@3a7442c7(incomplete, task: mao.t6.FutureTest2$1@4be29ed9)
2023-03-19  22:37:45.796  [main] DEBUG mao.t6.FutureTest2:  null
2023-03-19  22:37:45.796  [main] DEBUG mao.t6.FutureTest2:  开始异步等待结果
2023-03-19  22:37:46.810  [defaultEventLoopGroup-2-1] DEBUG mao.t6.FutureTest2:  执行成功
2023-03-19  22:37:46.811  [defaultEventLoopGroup-2-1] DEBUG mao.t6.FutureTest2:  等待完成，结果：20
```







### Promise

**同步处理任务成功**

```java
package mao.t6;

import io.netty.channel.DefaultEventLoop;
import io.netty.channel.DefaultEventLoopGroup;
import io.netty.util.concurrent.DefaultPromise;
import io.netty.util.concurrent.Promise;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.Callable;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t6
 * Class(类名): PromiseTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/19
 * Time(创建时间)： 22:39
 * Version(版本): 1.0
 * Description(描述)： Promise测试 ，同步处理任务成功
 */

@Slf4j
public class PromiseTest
{
    @SneakyThrows
    public static void main(String[] args)
    {
        DefaultEventLoop defaultEventLoop = new DefaultEventLoop();
        Promise<Integer> promise = new DefaultPromise<>(defaultEventLoop);
        defaultEventLoop.execute(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    Thread.sleep(1000);
                }
                catch (Exception e)
                {
                    e.printStackTrace();
                }
                log.debug("执行完成");
                promise.setSuccess(30);
            }
        });

        log.debug(promise.toString());
        log.debug("读取数据：" + promise.getNow());
        log.debug("同步等待结果");
        log.debug("结果：" + promise.get());

    }
}
```



运行结果：

```sh
2023-03-19  22:44:26.183  [main] DEBUG mao.t6.PromiseTest:  DefaultPromise@15bb5034(incomplete)
2023-03-19  22:44:26.183  [main] DEBUG mao.t6.PromiseTest:  读取数据：null
2023-03-19  22:44:26.183  [main] DEBUG mao.t6.PromiseTest:  同步等待结果
2023-03-19  22:44:27.191  [defaultEventLoop-1-1] DEBUG mao.t6.PromiseTest:  执行完成
2023-03-19  22:44:27.192  [main] DEBUG mao.t6.PromiseTest:  结果：30
```





**异步处理任务成功**

```java
package mao.t6;

import io.netty.channel.DefaultEventLoop;
import io.netty.util.concurrent.DefaultPromise;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.GenericFutureListener;
import io.netty.util.concurrent.Promise;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t6
 * Class(类名): PromiseTest2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/19
 * Time(创建时间)： 22:45
 * Version(版本): 1.0
 * Description(描述)： Promise测试 ，异步处理任务成功
 */

@Slf4j
public class PromiseTest2
{
    @SneakyThrows
    public static void main(String[] args)
    {
        DefaultEventLoop defaultEventLoop = new DefaultEventLoop();
        Promise<Integer> promise = new DefaultPromise<>(defaultEventLoop);
        defaultEventLoop.execute(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    Thread.sleep(1000);
                }
                catch (Exception e)
                {
                    e.printStackTrace();
                }
                log.debug("执行完成");
                promise.setSuccess(40);
            }
        });

        log.debug(promise.toString());
        log.debug("读取数据：" + promise.getNow());
        log.debug("同步等待结果");
        promise.addListener(new GenericFutureListener<Future<? super Integer>>()
        {
            @Override
            public void operationComplete(Future<? super Integer> future) throws Exception
            {
                log.debug("结果：" + promise.get());
            }
        });
    }
}
```



运行结果：

```sh
2023-03-19  22:46:58.927  [main] DEBUG mao.t6.PromiseTest2:  DefaultPromise@2eae8e6e(incomplete)
2023-03-19  22:46:58.927  [main] DEBUG mao.t6.PromiseTest2:  读取数据：null
2023-03-19  22:46:58.927  [main] DEBUG mao.t6.PromiseTest2:  同步等待结果
2023-03-19  22:46:59.942  [defaultEventLoop-1-1] DEBUG mao.t6.PromiseTest2:  执行完成
2023-03-19  22:46:59.942  [defaultEventLoop-1-1] DEBUG mao.t6.PromiseTest2:  结果：40
```





**同步处理任务失败 get**

sync() 也会出现异常，只是 get 会再用 ExecutionException 包一层异常

```java
package mao.t6;

import io.netty.channel.DefaultEventLoop;
import io.netty.util.concurrent.DefaultPromise;
import io.netty.util.concurrent.Promise;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t6
 * Class(类名): PromiseTest3
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/19
 * Time(创建时间)： 22:54
 * Version(版本): 1.0
 * Description(描述)： 同步处理任务失败 get
 */

@Slf4j
public class PromiseTest3
{
    @SneakyThrows
    public static void main(String[] args)
    {
        DefaultEventLoop defaultEventLoop = new DefaultEventLoop();
        Promise<Integer> promise = new DefaultPromise<>(defaultEventLoop);
        defaultEventLoop.execute(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    Thread.sleep(1000);
                    throw new RuntimeException("执行错误");
                    //log.debug("执行完成");
                    //promise.setSuccess(30);
                }
                catch (Exception e)
                {
                    log.debug("执行错误");
                    promise.setFailure(e);
                }
            }
        });

        log.debug(promise.toString());
        log.debug("读取数据：" + promise.getNow());
        log.debug("同步等待结果");
        log.debug("结果：" + promise.get());

    }
}

```



运行结果：

```sh
2023-03-19  22:58:02.197  [main] DEBUG mao.t6.PromiseTest3:  DefaultPromise@15bb5034(incomplete)
2023-03-19  22:58:02.198  [main] DEBUG mao.t6.PromiseTest3:  读取数据：null
2023-03-19  22:58:02.198  [main] DEBUG mao.t6.PromiseTest3:  同步等待结果
2023-03-19  22:58:03.203  [defaultEventLoop-1-1] DEBUG mao.t6.PromiseTest3:  执行错误
Exception in thread "main" java.util.concurrent.ExecutionException: java.lang.RuntimeException: 执行错误
	at io.netty.util.concurrent.AbstractFuture.get(AbstractFuture.java:41)
	at mao.t6.PromiseTest3.main(PromiseTest3.java:53)
Caused by: java.lang.RuntimeException: 执行错误
	at mao.t6.PromiseTest3$1.run(PromiseTest3.java:38)
	at io.netty.channel.DefaultEventLoop.run(DefaultEventLoop.java:54)
	at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:918)
	at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
	at java.base/java.lang.Thread.run(Thread.java:831)
```





**同步处理任务失败 sync**

```java
package mao.t6;

import io.netty.channel.DefaultEventLoop;
import io.netty.util.concurrent.DefaultPromise;
import io.netty.util.concurrent.Promise;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t6
 * Class(类名): PromiseTest4
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/19
 * Time(创建时间)： 22:59
 * Version(版本): 1.0
 * Description(描述)： 同步处理任务失败 sync
 */

@Slf4j
public class PromiseTest4
{
    @SneakyThrows
    public static void main(String[] args)
    {
        DefaultEventLoop defaultEventLoop = new DefaultEventLoop();
        Promise<Integer> promise = new DefaultPromise<>(defaultEventLoop);
        defaultEventLoop.execute(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    Thread.sleep(1000);
                    throw new RuntimeException("执行错误");
                    //log.debug("执行完成");
                    //promise.setSuccess(30);
                }
                catch (Exception e)
                {
                    log.debug("执行错误");
                    promise.setFailure(e);
                }
            }
        });

        log.debug(promise.toString());
        log.debug("读取数据：" + promise.getNow());
        log.debug("同步等待结果");
        promise.sync();
        log.debug("结果：" + promise.getNow());
    }
}
```



运行结果：

```sh
2023-03-19  23:01:02.550  [main] DEBUG mao.t6.PromiseTest4:  DefaultPromise@15bb5034(incomplete)
2023-03-19  23:01:02.552  [main] DEBUG mao.t6.PromiseTest4:  读取数据：null
2023-03-19  23:01:02.552  [main] DEBUG mao.t6.PromiseTest4:  同步等待结果
2023-03-19  23:01:03.554  [defaultEventLoop-1-1] DEBUG mao.t6.PromiseTest4:  执行错误
2023-03-19  23:01:03.562  [main] DEBUG io.netty.util.internal.PlatformDependent:  Platform: Windows
2023-03-19  23:01:03.564  [main] DEBUG io.netty.util.internal.PlatformDependent0:  -Dio.netty.noUnsafe: false
2023-03-19  23:01:03.565  [main] DEBUG io.netty.util.internal.PlatformDependent0:  Java version: 16
2023-03-19  23:01:03.566  [main] DEBUG io.netty.util.internal.PlatformDependent0:  sun.misc.Unsafe.theUnsafe: available
2023-03-19  23:01:03.566  [main] DEBUG io.netty.util.internal.PlatformDependent0:  sun.misc.Unsafe.copyMemory: available
2023-03-19  23:01:03.567  [main] DEBUG io.netty.util.internal.PlatformDependent0:  java.nio.Buffer.address: available
2023-03-19  23:01:03.568  [main] DEBUG io.netty.util.internal.PlatformDependent0:  direct buffer constructor: unavailable
java.lang.UnsupportedOperationException: Reflective setAccessible(true) disabled
	at io.netty.util.internal.ReflectionUtil.trySetAccessible(ReflectionUtil.java:31) ~[netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.internal.PlatformDependent0$4.run(PlatformDependent0.java:224) ~[netty-all-4.1.39.Final.jar:4.1.39.Final]
	at java.security.AccessController.doPrivileged(AccessController.java:312) ~[?:?]
	at io.netty.util.internal.PlatformDependent0.<clinit>(PlatformDependent0.java:218) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.internal.PlatformDependent.isAndroid(PlatformDependent.java:272) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.internal.PlatformDependent.<clinit>(PlatformDependent.java:92) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.concurrent.DefaultPromise.rethrowIfFailed(DefaultPromise.java:573) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.concurrent.DefaultPromise.sync(DefaultPromise.java:327) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at mao.t6.PromiseTest4.main(PromiseTest4.java:53) [classes/:?]
2023-03-19  23:01:03.587  [main] DEBUG io.netty.util.internal.PlatformDependent0:  java.nio.Bits.unaligned: available, true
2023-03-19  23:01:03.588  [main] DEBUG io.netty.util.internal.PlatformDependent0:  jdk.internal.misc.Unsafe.allocateUninitializedArray(int): unavailable
java.lang.IllegalAccessException: class io.netty.util.internal.PlatformDependent0$6 cannot access class jdk.internal.misc.Unsafe (in module java.base) because module java.base does not export jdk.internal.misc to unnamed module @1188e820
	at jdk.internal.reflect.Reflection.newIllegalAccessException(Reflection.java:385) ~[?:?]
	at java.lang.reflect.AccessibleObject.checkAccess(AccessibleObject.java:687) ~[?:?]
	at java.lang.reflect.Method.invoke(Method.java:559) ~[?:?]
	at io.netty.util.internal.PlatformDependent0$6.run(PlatformDependent0.java:334) ~[netty-all-4.1.39.Final.jar:4.1.39.Final]
	at java.security.AccessController.doPrivileged(AccessController.java:312) ~[?:?]
	at io.netty.util.internal.PlatformDependent0.<clinit>(PlatformDependent0.java:325) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.internal.PlatformDependent.isAndroid(PlatformDependent.java:272) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.internal.PlatformDependent.<clinit>(PlatformDependent.java:92) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.concurrent.DefaultPromise.rethrowIfFailed(DefaultPromise.java:573) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.concurrent.DefaultPromise.sync(DefaultPromise.java:327) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at mao.t6.PromiseTest4.main(PromiseTest4.java:53) [classes/:?]
2023-03-19  23:01:03.589  [main] DEBUG io.netty.util.internal.PlatformDependent0:  java.nio.DirectByteBuffer.<init>(long, int): unavailable
2023-03-19  23:01:03.589  [main] DEBUG io.netty.util.internal.PlatformDependent:  sun.misc.Unsafe: available
2023-03-19  23:01:03.589  [main] DEBUG io.netty.util.internal.PlatformDependent:  maxDirectMemory: 8522825728 bytes (maybe)
2023-03-19  23:01:03.590  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.tmpdir: C:\Users\mao\AppData\Local\Temp (java.io.tmpdir)
2023-03-19  23:01:03.590  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.bitMode: 64 (sun.arch.data.model)
2023-03-19  23:01:03.590  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.maxDirectMemory: -1 bytes
2023-03-19  23:01:03.592  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.uninitializedArrayAllocationThreshold: -1
2023-03-19  23:01:03.592  [main] DEBUG io.netty.util.internal.CleanerJava9:  java.nio.ByteBuffer.cleaner(): available
2023-03-19  23:01:03.593  [main] DEBUG io.netty.util.internal.PlatformDependent:  -Dio.netty.noPreferDirect: false
Exception in thread "main" java.lang.RuntimeException: 执行错误
	at mao.t6.PromiseTest4$1.run(PromiseTest4.java:38)
	at io.netty.channel.DefaultEventLoop.run(DefaultEventLoop.java:54)
	at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:918)
	at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
	at java.base/java.lang.Thread.run(Thread.java:831)
```







**同步处理任务失败 await**

与 sync 和 get 区别在于，不会抛异常

```java
package mao.t6;

import io.netty.channel.DefaultEventLoop;
import io.netty.util.concurrent.DefaultPromise;
import io.netty.util.concurrent.Promise;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t6
 * Class(类名): PromiseTest5
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/19
 * Time(创建时间)： 23:03
 * Version(版本): 1.0
 * Description(描述)： 同步处理任务失败 await
 */

@Slf4j
public class PromiseTest5
{
    @SneakyThrows
    public static void main(String[] args)
    {
        DefaultEventLoop defaultEventLoop = new DefaultEventLoop();
        Promise<Integer> promise = new DefaultPromise<>(defaultEventLoop);
        defaultEventLoop.execute(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    Thread.sleep(1000);

                    if (System.currentTimeMillis() % 2 == 0)
                    {
                        throw new RuntimeException("执行错误");
                    }
                    log.debug("执行完成");
                    promise.setSuccess(100);
                }
                catch (Exception e)
                {
                    log.debug("执行错误");
                    promise.setFailure(e);
                }
            }
        });

        log.debug(promise.toString());
        log.debug("读取数据：" + promise.getNow());
        log.debug("同步等待结果");
        promise.await();
        boolean success = promise.isSuccess();
        if (success)
        {
            log.debug("结果：" + promise.getNow());
        }
        else
        {
            Throwable throwable = promise.cause();
            log.warn("处理失败", throwable);
        }

    }
}
```



运行结果：

```sh
2023-03-19  23:08:47.960  [main] DEBUG mao.t6.PromiseTest5:  DefaultPromise@15bb5034(incomplete)
2023-03-19  23:08:47.962  [main] DEBUG mao.t6.PromiseTest5:  读取数据：null
2023-03-19  23:08:47.962  [main] DEBUG mao.t6.PromiseTest5:  同步等待结果
2023-03-19  23:08:48.967  [defaultEventLoop-1-1] DEBUG mao.t6.PromiseTest5:  执行完成
2023-03-19  23:08:48.969  [main] DEBUG mao.t6.PromiseTest5:  结果：30
```

```sh
2023-03-19  23:09:25.030  [main] DEBUG mao.t6.PromiseTest5:  DefaultPromise@15bb5034(incomplete)
2023-03-19  23:09:25.032  [main] DEBUG mao.t6.PromiseTest5:  读取数据：null
2023-03-19  23:09:25.032  [main] DEBUG mao.t6.PromiseTest5:  同步等待结果
2023-03-19  23:09:26.044  [defaultEventLoop-1-1] DEBUG mao.t6.PromiseTest5:  执行错误
2023-03-19  23:09:26.045  [main] WARN  mao.t6.PromiseTest5:  处理失败
java.lang.RuntimeException: 执行错误
	at mao.t6.PromiseTest5$1.run(PromiseTest5.java:41) ~[classes/:?]
	at io.netty.channel.DefaultEventLoop.run(DefaultEventLoop.java:54) ~[netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:918) ~[netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) ~[netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30) ~[netty-all-4.1.39.Final.jar:4.1.39.Final]
	at java.lang.Thread.run(Thread.java:831) ~[?:?]
```





**异步处理任务失败**

```java
package mao.t6;

import io.netty.channel.DefaultEventLoop;
import io.netty.util.concurrent.DefaultPromise;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.GenericFutureListener;
import io.netty.util.concurrent.Promise;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t6
 * Class(类名): PromiseTest6
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/19
 * Time(创建时间)： 23:10
 * Version(版本): 1.0
 * Description(描述)： 异步处理任务失败
 */

@Slf4j
public class PromiseTest6
{
    @SneakyThrows
    public static void main(String[] args)
    {
        DefaultEventLoop defaultEventLoop = new DefaultEventLoop();
        Promise<Integer> promise = new DefaultPromise<>(defaultEventLoop);
        defaultEventLoop.execute(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    Thread.sleep(1000);

                    if (System.currentTimeMillis() % 2 == 0)
                    {
                        throw new RuntimeException("执行错误");
                    }
                    log.debug("执行完成");
                    promise.setSuccess(110);
                }
                catch (Exception e)
                {
                    log.debug("执行错误");
                    promise.setFailure(e);
                }
            }
        });

        log.debug(promise.toString());
        log.debug("读取数据：" + promise.getNow());
        log.debug("异步等待结果");
        promise.addListener(new GenericFutureListener<Future<? super Integer>>()
        {
            @Override
            public void operationComplete(Future<? super Integer> future) throws Exception
            {
                boolean success = promise.isSuccess();
                if (success)
                {
                    log.debug("结果：" + promise.getNow());
                }
                else
                {
                    Throwable throwable = promise.cause();
                    log.warn("处理失败", throwable);
                }
            }
        });

    }
}
```



运行结果：

```sh
2023-03-19  23:13:01.522  [main] DEBUG mao.t6.PromiseTest6:  DefaultPromise@2eae8e6e(incomplete)
2023-03-19  23:13:01.522  [main] DEBUG mao.t6.PromiseTest6:  读取数据：null
2023-03-19  23:13:01.522  [main] DEBUG mao.t6.PromiseTest6:  异步等待结果
2023-03-19  23:13:02.523  [defaultEventLoop-1-1] DEBUG mao.t6.PromiseTest6:  执行完成
2023-03-19  23:13:02.523  [defaultEventLoop-1-1] DEBUG mao.t6.PromiseTest6:  结果：110
```

```sh
2023-03-19  23:13:19.069  [main] DEBUG mao.t6.PromiseTest6:  DefaultPromise@2eae8e6e(incomplete)
2023-03-19  23:13:19.069  [main] DEBUG mao.t6.PromiseTest6:  读取数据：null
2023-03-19  23:13:19.069  [main] DEBUG mao.t6.PromiseTest6:  异步等待结果
2023-03-19  23:13:20.084  [defaultEventLoop-1-1] DEBUG mao.t6.PromiseTest6:  执行错误
2023-03-19  23:13:20.085  [defaultEventLoop-1-1] WARN  mao.t6.PromiseTest6:  处理失败
java.lang.RuntimeException: 执行错误
	at mao.t6.PromiseTest6$1.run(PromiseTest6.java:43) [classes/:?]
	at io.netty.channel.DefaultEventLoop.run(DefaultEventLoop.java:54) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:918) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30) [netty-all-4.1.39.Final.jar:4.1.39.Final]
	at java.lang.Thread.run(Thread.java:831) [?:?]
```









## Handler和Pipeline

ChannelHandler 用来处理 Channel 上的各种事件，分为入站、出站两种。所有 ChannelHandler 被连成一串，就是 Pipeline

* 入站处理器通常是 ChannelInboundHandlerAdapter 的子类，主要用来读取客户端数据，写回结果
* 出站处理器通常是 ChannelOutboundHandlerAdapter 的子类，主要对写回结果进行加工



每个 Channel 是一个产品的加工车间，Pipeline 是车间中的流水线，ChannelHandler 就是流水线上的各道工序，而后面要讲的 ByteBuf 是原材料，经过很多工序的加工：先经过一道道入站工序，再经过一道道出站工序最终变成产品



```java
package mao.t7;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t7
 * Class(类名): HandlerTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/20
 * Time(创建时间)： 19:45
 * Version(版本): 1.0
 * Description(描述)： Handler
 */

@Slf4j
public class HandlerTest
{
    public static void main(String[] args)
    {
        new ServerBootstrap()
                .group(new NioEventLoopGroup(), new NioEventLoopGroup(3))
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                                {
                                    @Override
                                    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
                                    {
                                        log.debug("入栈处理器：1");
                                        super.channelRead(ctx, msg);
                                    }
                                })
                                .addLast(new ChannelInboundHandlerAdapter()
                                {
                                    @Override
                                    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
                                    {
                                        log.debug("入栈处理器：2");
                                        super.channelRead(ctx, msg);
                                    }
                                })
                                .addLast(new ChannelInboundHandlerAdapter()
                                {
                                    @Override
                                    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
                                    {
                                        log.debug("入栈处理器：3");
                                        super.channelRead(ctx, msg);
                                    }
                                })
                                .addLast(new ChannelInboundHandlerAdapter()
                                {
                                    @Override
                                    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
                                    {
                                        log.debug("入栈处理器：4");
                                        ctx.channel().write("hello");
                                    }
                                })
                                .addLast(new ChannelOutboundHandlerAdapter()
                                {
                                    @Override
                                    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise)
                                            throws Exception
                                    {
                                        log.debug("出栈处理器：5");
                                        super.write(ctx, msg, promise);
                                    }
                                })
                                .addLast(new ChannelOutboundHandlerAdapter()
                                {
                                    @Override
                                    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise)
                                            throws Exception
                                    {
                                        log.debug("出栈处理器：6");
                                        super.write(ctx, msg, promise);
                                    }
                                })
                                .addLast(new ChannelOutboundHandlerAdapter()
                                {
                                    @Override
                                    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise)
                                            throws Exception
                                    {
                                        log.debug("出栈处理器：7");
                                        super.write(ctx, msg, promise);
                                    }
                                })
                                .addLast(new ChannelOutboundHandlerAdapter()
                                {
                                    @Override
                                    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise)
                                            throws Exception
                                    {
                                        log.debug("出栈处理器：8");
                                        super.write(ctx, msg, promise);
                                    }
                                });
                    }
                })
                .bind(8080);
    }
}
```



客户端

```java
package mao.t7;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.GenericFutureListener;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.net.InetSocketAddress;

/**
 * Project name(项目名称)：Netty_Component
 * Package(包名): mao.t7
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/20
 * Time(创建时间)： 20:50
 * Version(版本): 1.0
 * Description(描述)： 无
 */

@Slf4j
public class Client
{
    @SneakyThrows
    public static void main(String[] args)
    {
        Channel channel = new Bootstrap()
                .group(new NioEventLoopGroup())
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<Channel>()
                {
                    @Override
                    protected void initChannel(Channel ch) throws Exception
                    {
                        ch.pipeline().addLast(new StringEncoder());
                    }
                })
                .connect(new InetSocketAddress(8080)).sync().channel();
        channel.writeAndFlush("hello");
    }
}
```



运行结果：

```sh
2023-03-20  21:02:01.307  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：1
2023-03-20  21:02:01.307  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：2
2023-03-20  21:02:01.307  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：3
2023-03-20  21:02:01.307  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：4
2023-03-20  21:02:01.307  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  出栈处理器：8
2023-03-20  21:02:01.307  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  出栈处理器：7
2023-03-20  21:02:01.307  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  出栈处理器：6
2023-03-20  21:02:01.307  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  出栈处理器：5
```





可以看到，ChannelInboundHandlerAdapter 是按照 addLast 的顺序执行的，而 ChannelOutboundHandlerAdapter 是按照 addLast 的逆序执行的。ChannelPipeline 的实现是一个 ChannelHandlerContext（包装了 ChannelHandler） 组成的双向链表



![image-20230320210227348](img/Netty学习笔记/image-20230320210227348.png)





如果注释掉处理器1的super.channelRead(ctx, msg)代码

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
{
    log.debug("入栈处理器：1");
    //super.channelRead(ctx, msg);
}
```



那么只会打印1

```sh
2023-03-20  21:04:37.663  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：1
```



如果只注释掉处理器3的super.channelRead(ctx, msg)代码

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
{
    log.debug("入栈处理器：3");
    //super.channelRead(ctx, msg);
}
```



那么会打印1、2和3

```sh
2023-03-20  21:05:28.347  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：1
2023-03-20  21:05:28.348  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：2
2023-03-20  21:05:28.348  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：3
```



处理器4的ctx.channel().write(msg) 会 **从尾部开始触发** 后续出站处理器的执行



如果注释掉处理器6的super.write(ctx, msg, promise)代码

```java
@Override
public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise)
        throws Exception
{
    log.debug("出栈处理器：6");
    //super.write(ctx, msg, promise);
}
```



不会打印5

```sh
2023-03-20  21:08:11.394  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：1
2023-03-20  21:08:11.394  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：2
2023-03-20  21:08:11.394  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：3
2023-03-20  21:08:11.395  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：4
2023-03-20  21:08:11.395  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  出栈处理器：8
2023-03-20  21:08:11.395  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  出栈处理器：7
2023-03-20  21:08:11.395  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  出栈处理器：6
```





如果处理器4更改成以下代码

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
{
    log.debug("入栈处理器：4");
    //ctx.channel().write("hello");
    ctx.write("hello");
}
```



```sh
2023-03-20  21:10:35.279  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：1
2023-03-20  21:10:35.279  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：2
2023-03-20  21:10:35.279  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：3
2023-03-20  21:10:35.280  [nioEventLoopGroup-3-1] DEBUG mao.t7.HandlerTest:  入栈处理器：4
```



原因：4 处的 ctx.channel().write(msg) 如果改为 ctx.write(msg) 仅会打印 1 2 3  4，因为节点4之前没有其它出站处理器了





**ctx.channel().write(msg) 和 ctx.write(msg) 的区别**

* 都是触发出站处理器的执行
* ctx.channel().write(msg) 从尾部开始查找出站处理器
* ctx.write(msg) 是从当前节点找上一个出站处理器





![image-20230320211329291](img/Netty学习笔记/image-20230320211329291.png)













# ByteBuf

ByteBuf是对字节数据的封装



## ByteBuf创建

```java
package mao.t1;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import lombok.extern.slf4j.Slf4j;

import static io.netty.buffer.ByteBufUtil.appendPrettyHexDump;
import static io.netty.util.internal.StringUtil.NEWLINE;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t1
 * Class(类名): ByteBufTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/20
 * Time(创建时间)： 21:26
 * Version(版本): 1.0
 * Description(描述)： ByteBuf创建
 */

@Slf4j
public class ByteBufTest
{
    public static void main(String[] args)
    {
        //创建一个ByteBuf
        ByteBuf byteBuf = ByteBufAllocator.DEFAULT.buffer();
        log.info(byteBuf.toString());
        //创建一个ByteBuf，初始容量为20（cap）
        byteBuf = ByteBufAllocator.DEFAULT.buffer(20);
        log.info(byteBuf.toString());
    }
}
```



运行结果：

```sh
2023-03-20  21:30:27.447  [main] INFO  mao.t1.ByteBufTest:  PooledUnsafeDirectByteBuf(ridx: 0, widx: 0, cap: 256)
2023-03-20  21:30:27.447  [main] INFO  mao.t1.ByteBufTest:  PooledUnsafeDirectByteBuf(ridx: 0, widx: 0, cap: 20)
```







## 直接内存和堆内存

可以使用下面的代码来创建池化基于堆的 ByteBuf

```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.heapBuffer();
```



也可以使用下面的代码来创建池化基于直接内存的 ByteBuf

```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.directBuffer();
```

* 直接内存创建和销毁的代价昂贵，但读写性能高（少一次内存复制），适合配合池化功能一起用
* 直接内存对 GC 压力小，因为这部分内存不受 JVM 垃圾回收的管理，但也要注意及时主动释放



```java
package mao.t2;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t2
 * Class(类名): ByteBufTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/20
 * Time(创建时间)： 21:38
 * Version(版本): 1.0
 * Description(描述)： 直接内存和堆内存
 */

@Slf4j
public class ByteBufTest
{
    public static void main(String[] args)
    {
        //基于堆的 ByteBuf
        ByteBuf byteBuf = ByteBufAllocator.DEFAULT.heapBuffer();
        log.info(byteBuf.toString());
        //释放
        boolean release = byteBuf.release();
        log.debug(String.valueOf(release));
        //基于直接内存的 ByteBuf
        byteBuf = ByteBufAllocator.DEFAULT.directBuffer();
        log.info(byteBuf.toString());
        //释放
        release = byteBuf.release();
        log.debug(String.valueOf(release));
        //默认ByteBuf
        byteBuf = ByteBufAllocator.DEFAULT.buffer();
        log.info(byteBuf.toString());
        //释放
        release = byteBuf.release();
        log.debug(String.valueOf(release));

        //基于堆的 ByteBuf，容量为32
        byteBuf = ByteBufAllocator.DEFAULT.heapBuffer(32);
        log.info(byteBuf.toString());
        //释放
        release = byteBuf.release();
        log.debug(String.valueOf(release));
        //基于直接内存的 ByteBuf，容量为64
        byteBuf = ByteBufAllocator.DEFAULT.directBuffer(64);
        log.info(byteBuf.toString());
        //释放
        release = byteBuf.release();
        log.debug(String.valueOf(release));
    }
}
```



运行结果：

```sh
2023-03-20  21:45:41.028  [main] INFO  mao.t2.ByteBufTest:  PooledUnsafeHeapByteBuf(ridx: 0, widx: 0, cap: 256)
2023-03-20  21:45:41.028  [main] DEBUG mao.t2.ByteBufTest:  true
2023-03-20  21:45:41.032  [main] INFO  mao.t2.ByteBufTest:  PooledUnsafeDirectByteBuf(ridx: 0, widx: 0, cap: 256)
2023-03-20  21:45:41.032  [main] DEBUG mao.t2.ByteBufTest:  true
2023-03-20  21:45:41.032  [main] INFO  mao.t2.ByteBufTest:  PooledUnsafeDirectByteBuf(ridx: 0, widx: 0, cap: 256)
2023-03-20  21:45:41.032  [main] DEBUG mao.t2.ByteBufTest:  true
2023-03-20  21:45:41.032  [main] INFO  mao.t2.ByteBufTest:  PooledUnsafeHeapByteBuf(ridx: 0, widx: 0, cap: 32)
2023-03-20  21:45:41.032  [main] DEBUG mao.t2.ByteBufTest:  true
2023-03-20  21:45:41.032  [main] INFO  mao.t2.ByteBufTest:  PooledUnsafeDirectByteBuf(ridx: 0, widx: 0, cap: 64)
2023-03-20  21:45:41.032  [main] DEBUG mao.t2.ByteBufTest:  true
```









## 池化和非池化

池化的最大意义在于可以重用 ByteBuf，优点有

* 没有池化，则每次都得创建新的 ByteBuf 实例，这个操作对直接内存代价昂贵，就算是堆内存，也会增加 GC 压力
* 有了池化，则可以重用池中 ByteBuf 实例，并且采用了与 jemalloc 类似的内存分配算法提升分配效率
* 高并发时，池化功能更节约内存，减少内存溢出的可能



池化功能是否开启，可以通过下面的系统环境变量来设置

```java
-Dio.netty.allocator.type={unpooled|pooled}
```

* 4.1 以后，非 Android 平台默认启用池化实现，Android 平台启用非池化实现
* 4.1 之前，池化功能还不成熟，默认是非池化实现







## 组成

ByteBuf 由四部分组成

![image-20230321215942964](img/Netty学习笔记/image-20230321215942964.png)



最开始读写指针都在 0 位置



## 写入





|                           方法签名                           |          含义          |                    备注                     |
| :----------------------------------------------------------: | :--------------------: | :-----------------------------------------: |
|                 writeBoolean(boolean value)                  |    写入 boolean 值     |      用一字节 01\|00 代表 true\|false       |
|                     writeByte(int value)                     |      写入 byte 值      |                                             |
|                    writeShort(int value)                     |     写入 short 值      |                                             |
|                     writeInt(int value)                      |      写入 int 值       |  Big Endian，即 0x250，写入后 00 00 02 50   |
|                    writeIntLE(int value)                     |      写入 int 值       | Little Endian，即 0x250，写入后 50 02 00 00 |
|                    writeLong(long value)                     |      写入 long 值      |                                             |
|                     writeChar(int value)                     |      写入 char 值      |                                             |
|                   writeFloat(float value)                    |     写入 float 值      |                                             |
|                  writeDouble(double value)                   |     写入 double 值     |                                             |
|                   writeBytes(ByteBuf src)                    | 写入 netty 的 ByteBuf  |                                             |
|                    writeBytes(byte[] src)                    |      写入 byte[]       |                                             |
|                  writeBytes(ByteBuffer src)                  | 写入 nio 的 ByteBuffer |                                             |
| int writeCharSequence(CharSequence sequence, Charset charset) |       写入字符串       |                                             |



* 这些方法的未指明返回值的，其返回值都是 ByteBuf，意味着可以链式调用
* 网络传输，默认习惯是 Big Endian



```java
package mao.t3;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import lombok.extern.slf4j.Slf4j;
import mao.utils.ByteBufUtils;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t3
 * Class(类名): ByteBufTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/22
 * Time(创建时间)： 14:23
 * Version(版本): 1.0
 * Description(描述)： 写入
 */

@Slf4j
public class ByteBufTest
{
    public static void main(String[] args)
    {
        //创建一个16字节的ByteBuf
        ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(16);
        //写入数据
        buffer.writeBoolean(false);
        //打印
        ByteBufUtils.debug(buffer);
        //写入数据
        buffer.writeBoolean(true);
        //打印
        ByteBufUtils.debug(buffer);

        System.out.println("\n\n");

        //创建一个16字节的ByteBuf
        buffer = ByteBufAllocator.DEFAULT.buffer(16);
        //写入数据
        buffer.writeByte(0x22);
        //打印
        ByteBufUtils.debug(buffer);
        //写入数据
        buffer.writeByte(66);
        //打印
        ByteBufUtils.debug(buffer);

        System.out.println("\n\n");

        //创建一个16字节的ByteBuf
        buffer = ByteBufAllocator.DEFAULT.buffer(16);
        //写入数据
        buffer.writeShort(0x6943);
        //打印
        ByteBufUtils.debug(buffer);

        System.out.println("\n\n");

        //创建一个16字节的ByteBuf
        buffer = ByteBufAllocator.DEFAULT.buffer(16);
        //写入数据
        buffer.writeInt(0x6946);
        //打印
        ByteBufUtils.debug(buffer);
        //写入数据，大端模式
        buffer.writeIntLE(0x4522);
        //打印
        ByteBufUtils.debug(buffer);

        System.out.println("\n\n");

        //创建一个16字节的ByteBuf
        buffer = ByteBufAllocator.DEFAULT.buffer(16);
        //写入数据
        buffer.writeLong(0x7547);
        //打印
        ByteBufUtils.debug(buffer);
        //写入数据，大端模式
        buffer.writeLongLE(0x379856);
        //打印
        ByteBufUtils.debug(buffer);

        System.out.println("\n\n");

        //创建一个16字节的ByteBuf
        buffer = ByteBufAllocator.DEFAULT.buffer(16);
        //写入数据
        buffer.writeChar('5');
        //打印
        ByteBufUtils.debug(buffer);

        System.out.println("\n\n");

        //创建一个16字节的ByteBuf
        buffer = ByteBufAllocator.DEFAULT.buffer(16);
        //写入数据
        buffer.writeFloat(66.3f);
        //打印
        ByteBufUtils.debug(buffer);
        //写入数据
        buffer.writeFloat(57.14f);
        //打印
        ByteBufUtils.debug(buffer);

        System.out.println("\n\n");

        //创建一个16字节的ByteBuf
        buffer = ByteBufAllocator.DEFAULT.buffer(16);
        //写入数据
        buffer.writeBytes(new byte[]{65, 66, 67, 68, 69, 70});
        //打印
        ByteBufUtils.debug(buffer);

        System.out.println("\n\n");

        //创建一个16字节的ByteBuf
        buffer = ByteBufAllocator.DEFAULT.buffer(16);
        //写入数据
        buffer.writeCharSequence("hello,world", StandardCharsets.UTF_8);
        //打印
        ByteBufUtils.debug(buffer);
    }
}
```



运行结果：

```sh
read index:0 write index:1 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00                                              |.               |
+--------+-------------------------------------------------+----------------+
read index:0 write index:2 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01                                           |..              |
+--------+-------------------------------------------------+----------------+



read index:0 write index:1 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 22                                              |"               |
+--------+-------------------------------------------------+----------------+
read index:0 write index:2 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 22 42                                           |"B              |
+--------+-------------------------------------------------+----------------+



read index:0 write index:2 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 69 43                                           |iC              |
+--------+-------------------------------------------------+----------------+



read index:0 write index:4 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 00 69 46                                     |..iF            |
+--------+-------------------------------------------------+----------------+
read index:0 write index:8 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 00 69 46 22 45 00 00                         |..iF"E..        |
+--------+-------------------------------------------------+----------------+



read index:0 write index:8 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 00 00 00 00 00 75 47                         |......uG        |
+--------+-------------------------------------------------+----------------+
read index:0 write index:16 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 00 00 00 00 00 75 47 56 98 37 00 00 00 00 00 |......uGV.7.....|
+--------+-------------------------------------------------+----------------+



read index:0 write index:2 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 35                                           |.5              |
+--------+-------------------------------------------------+----------------+



read index:0 write index:4 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 42 84 99 9a                                     |B...            |
+--------+-------------------------------------------------+----------------+
read index:0 write index:8 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 42 84 99 9a 42 64 8f 5c                         |B...Bd.\        |
+--------+-------------------------------------------------+----------------+



read index:0 write index:6 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 41 42 43 44 45 46                               |ABCDEF          |
+--------+-------------------------------------------------+----------------+



read index:0 write index:11 capacity:64
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 2c 77 6f 72 6c 64                |hello,world     |
+--------+-------------------------------------------------+----------------+
```





还有一类方法是 set 开头的一系列方法，也可以写入数据，但不会改变写指针位置



```java
package mao.t3;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import mao.utils.ByteBufUtils;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t3
 * Class(类名): ByteBufTest2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/22
 * Time(创建时间)： 14:42
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class ByteBufTest2
{
    public static void main(String[] args)
    {
        //创建一个16字节的ByteBuf
        ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(16);
        buffer.writeCharSequence("12345", StandardCharsets.UTF_8);
        //打印
        ByteBufUtils.debug(buffer);
        //设置数据
        buffer.setByte(0, 0x45);
        //打印
        ByteBufUtils.debug(buffer);
        //设置数据
        buffer.setByte(2, 0x47);
        //打印
        ByteBufUtils.debug(buffer);
    }
}
```



运行结果：

```sh
read index:0 write index:5 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35                                  |12345           |
+--------+-------------------------------------------------+----------------+
read index:0 write index:5 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 45 32 33 34 35                                  |E2345           |
+--------+-------------------------------------------------+----------------+
read index:0 write index:5 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 45 32 47 34 35                                  |E2G45           |
+--------+-------------------------------------------------+----------------+
```









## 扩容

再写入一个数时，如果容量不够了，这时会引发扩容

```java
package mao.t4;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import lombok.extern.slf4j.Slf4j;
import mao.utils.ByteBufUtils;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t4
 * Class(类名): ByteBufTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/22
 * Time(创建时间)： 14:50
 * Version(版本): 1.0
 * Description(描述)： 扩容测试
 */

@Slf4j
public class ByteBufTest
{
    public static void main(String[] args)
    {
        ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(16);
        buffer.writeBytes("123456789012345".getBytes(StandardCharsets.UTF_8));
        ByteBufUtils.debug(buffer);
        //再次写入，发现空间不够，触发扩容
        buffer.writeInt(0x3369);
        ByteBufUtils.debug(buffer);
    }
}
```



运行结果：

```sh
read index:0 write index:15 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35    |123456789012345 |
+--------+-------------------------------------------------+----------------+
read index:0 write index:19 capacity:64
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 00 |123456789012345.|
|00000010| 00 33 69                                        |.3i             |
+--------+-------------------------------------------------+----------------+
```





扩容规则是

* 如何写入后数据大小未超过 512，则选择下一个 16 的整数倍，例如写入后大小为 12 ，则扩容后 capacity 是 16
* 如果写入后数据大小超过 512，则选择下一个 2^n，例如写入后大小为 513，则扩容后 capacity 是 2^10=1024（2^9=512 已经不够了）
* 扩容不能超过 max capacity 会报错







## 读取

例如读了 4 次，每次一个字节

读过的内容，就属于废弃部分了，再读只能读那些尚未读取的部分



```java
package mao.t5;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import lombok.extern.slf4j.Slf4j;
import mao.utils.ByteBufUtils;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t5
 * Class(类名): ByteBufTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/22
 * Time(创建时间)： 15:00
 * Version(版本): 1.0
 * Description(描述)： 读取
 */

@Slf4j
public class ByteBufTest
{
    public static void main(String[] args)
    {
        ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(16);
        buffer.writeBytes("1234567890123456".getBytes(StandardCharsets.UTF_8));
        ByteBufUtils.debug(buffer);
        //读取
        log.debug(String.valueOf(buffer.readByte()));
        log.debug(String.valueOf(buffer.readByte()));
        log.debug(String.valueOf(buffer.readByte()));
        log.debug(String.valueOf(buffer.readByte()));
        log.debug(String.valueOf(buffer.readByte()));
        ByteBufUtils.debug(buffer);
        log.debug(String.valueOf(buffer.readByte()));
        ByteBufUtils.debug(buffer);
    }
}
```



运行结果：

```sh
read index:0 write index:16 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36 |1234567890123456|
+--------+-------------------------------------------------+----------------+
2023-03-22  15:02:10.122  [main] DEBUG mao.t5.ByteBufTest:  49
2023-03-22  15:02:10.122  [main] DEBUG mao.t5.ByteBufTest:  50
2023-03-22  15:02:10.122  [main] DEBUG mao.t5.ByteBufTest:  51
2023-03-22  15:02:10.122  [main] DEBUG mao.t5.ByteBufTest:  52
2023-03-22  15:02:10.122  [main] DEBUG mao.t5.ByteBufTest:  53
read index:5 write index:16 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 36 37 38 39 30 31 32 33 34 35 36                |67890123456     |
+--------+-------------------------------------------------+----------------+
2023-03-22  15:02:10.122  [main] DEBUG mao.t5.ByteBufTest:  54
read index:6 write index:16 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 37 38 39 30 31 32 33 34 35 36                   |7890123456      |
+--------+-------------------------------------------------+----------------+
```





如果需要重复读取 int 整数 50，可以在 read 前先做个标记 mark，这时要重复读取的话，重置到标记位置 reset



```java
package mao.t5;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import lombok.extern.slf4j.Slf4j;
import mao.utils.ByteBufUtils;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t5
 * Class(类名): ByteBufTest2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/22
 * Time(创建时间)： 15:03
 * Version(版本): 1.0
 * Description(描述)： 读取某一个位置的数值，mark和reset
 */

@Slf4j
public class ByteBufTest2
{
    public static void main(String[] args)
    {
        ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(16);
        buffer.writeBytes("1234567890123456".getBytes(StandardCharsets.UTF_8));
        ByteBufUtils.debug(buffer);
        //读取
        log.debug(String.valueOf(buffer.readByte()));
        //标记
        buffer.markReaderIndex();
        log.debug(String.valueOf(buffer.readByte()));
        log.debug(String.valueOf(buffer.readByte()));
        ByteBufUtils.debug(buffer);
        //重置
        buffer.resetReaderIndex();
        //再次读
        log.debug(String.valueOf(buffer.readByte()));
        ByteBufUtils.debug(buffer);
        //重置
        buffer.resetReaderIndex();
        //再次读
        log.debug(String.valueOf(buffer.readByte()));
        //重置
        buffer.resetReaderIndex();
        //再次读
        log.debug(String.valueOf(buffer.readByte()));
    }
}

```



运行结果：

```sh
read index:0 write index:16 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36 |1234567890123456|
+--------+-------------------------------------------------+----------------+
2023-03-22  15:06:51.115  [main] DEBUG mao.t5.ByteBufTest2:  49
2023-03-22  15:06:51.115  [main] DEBUG mao.t5.ByteBufTest2:  50
2023-03-22  15:06:51.115  [main] DEBUG mao.t5.ByteBufTest2:  51
read index:3 write index:16 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 34 35 36 37 38 39 30 31 32 33 34 35 36          |4567890123456   |
+--------+-------------------------------------------------+----------------+
2023-03-22  15:06:51.116  [main] DEBUG mao.t5.ByteBufTest2:  50
read index:2 write index:16 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 33 34 35 36 37 38 39 30 31 32 33 34 35 36       |34567890123456  |
+--------+-------------------------------------------------+----------------+
2023-03-22  15:06:51.116  [main] DEBUG mao.t5.ByteBufTest2:  50
2023-03-22  15:06:51.116  [main] DEBUG mao.t5.ByteBufTest2:  50
```





还有种办法是采用 get 开头的一系列方法，这些方法不会改变 read index



```java
package mao.t5;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import lombok.extern.slf4j.Slf4j;
import mao.utils.ByteBufUtils;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t5
 * Class(类名): ByteBufTest3
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/22
 * Time(创建时间)： 15:07
 * Version(版本): 1.0
 * Description(描述)： 读取某一个位置的数值，get方法
 */

@Slf4j
public class ByteBufTest3
{
    public static void main(String[] args)
    {
        ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(16);
        buffer.writeBytes("1234567890123456".getBytes(StandardCharsets.UTF_8));
        ByteBufUtils.debug(buffer);
        //读取
        log.debug(String.valueOf(buffer.getByte(1)));
        log.debug(String.valueOf(buffer.getByte(1)));
        log.debug(String.valueOf(buffer.getByte(1)));
        log.debug(String.valueOf(buffer.getByte(6)));
        log.debug(String.valueOf(buffer.getByte(0)));
    }
}
```



运行结果：

```sh
read index:0 write index:16 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36 |1234567890123456|
+--------+-------------------------------------------------+----------------+
2023-03-22  15:09:51.775  [main] DEBUG mao.t5.ByteBufTest3:  50
2023-03-22  15:09:51.775  [main] DEBUG mao.t5.ByteBufTest3:  50
2023-03-22  15:09:51.775  [main] DEBUG mao.t5.ByteBufTest3:  50
2023-03-22  15:09:51.775  [main] DEBUG mao.t5.ByteBufTest3:  55
2023-03-22  15:09:51.775  [main] DEBUG mao.t5.ByteBufTest3:  49
```







## retain和release

由于 Netty 中有堆外内存的 ByteBuf 实现，堆外内存最好是手动来释放，而不是等 GC 垃圾回收。

* UnpooledHeapByteBuf 使用的是 JVM 内存，只需等 GC 回收内存即可
* UnpooledDirectByteBuf 使用的就是直接内存了，需要特殊的方法来回收内存
* PooledByteBuf 和它的子类使用了池化机制，需要更复杂的规则来回收内存



Netty 这里采用了引用计数法来控制回收内存，每个 ByteBuf 都实现了 ReferenceCounted 接口

* 每个 ByteBuf 对象的初始计数为 1
* 调用 release 方法计数减 1，如果计数为 0，ByteBuf 内存被回收
* 调用 retain 方法计数加 1，表示调用者没用完之前，其它 handler 即使调用了 release 也不会造成回收
* 当计数为 0 时，底层内存会被回收，这时即使 ByteBuf 对象还在，其各个方法均无法正常使用



谁来负责 release 呢？

因为 pipeline 的存在，一般需要将 ByteBuf 传递给下一个 ChannelHandler，如果在 finally 中 release 了，就失去了传递性，当然，如果在这个 ChannelHandler 内这个 ByteBuf 已完成了它的使命，那么便无须再传递

基本规则是，**谁是最后使用者，谁负责 release**，详细分析如下：

* 起点，对于 NIO 实现来讲，在 io.netty.channel.nio.AbstractNioByteChannel.NioByteUnsafe#read 方法中首次创建 ByteBuf 放入 pipeline（line 163 pipeline.fireChannelRead(byteBuf)）
* 入站 ByteBuf 处理原则
  * 对原始 ByteBuf 不做处理，调用 ctx.fireChannelRead(msg) 向后传递，这时无须 release
  * 将原始 ByteBuf 转换为其它类型的 Java 对象，这时 ByteBuf 就没用了，必须 release
  * 如果不调用 ctx.fireChannelRead(msg) 向后传递，那么也必须 release
  * 注意各种异常，如果 ByteBuf 没有成功传递到下一个 ChannelHandler，必须 release
  * 假设消息一直向后传，那么 TailContext 会负责释放未处理消息（原始的 ByteBuf）
* 出站 ByteBuf 处理原则
  * 出站消息最终都会转为 ByteBuf 输出，一直向前传，由 HeadContext flush 后 release
* 异常处理原则
  * 有时候不清楚 ByteBuf 被引用了多少次，但又必须彻底释放，可以循环调用 release 直到返回 true







## slice

**零拷贝**的体现之一，对原始 ByteBuf 进行切片成多个 ByteBuf，切片后的 ByteBuf 并没有发生内存复制，还是使用原始 ByteBuf 的内存，切片后的 ByteBuf 维护独立的 read，write 指针



![image-20230322152344153](img/Netty学习笔记/image-20230322152344153.png)





原始 ByteBuf 进行一些初始操作

```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(16);
buffer.writeBytes(new byte[]{1, 2, 3, 4});
log.debug(String.valueOf(buffer.readByte()));
ByteBufUtils.debug(buffer);
```



```sh
2023-03-22  15:32:08.032  [main] DEBUG mao.t6.ByteBufTest:  1
read index:1 write index:4 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 02 03 04                                        |...             |
+--------+-------------------------------------------------+----------------+
```



这时调用 slice 进行切片，无参 slice 是从原始 ByteBuf 的 read index 到 write index 之间的内容进行切片，切片后的 max capacity 被固定为这个区间的大小，因此不能追加 write

```java
//这时调用 slice 进行切片，
// 无参 slice 是从原始 ByteBuf 的 read index 到 write index 之间的内容进行切片，
// 切片后的 max capacity 被固定为这个区间的大小，因此不能追加 write
ByteBuf buffer2 = buffer.slice();
ByteBufUtils.debug(buffer2);
```



```sh
read index:0 write index:3 capacity:3
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 02 03 04                                        |...             |
+--------+-------------------------------------------------+----------------+
```





如果原始 ByteBuf 再次读操作（又读了一个字节）

```java
//如果原始 ByteBuf 再次读操作
log.debug(String.valueOf(buffer.readByte()));
ByteBufUtils.debug(buffer);
```



```sh
read index:2 write index:4 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 03 04                                           |..              |
+--------+-------------------------------------------------+----------------+
```



这时的slice后的ByteBuf不受影响，因为它有独立的读写指针

```java
//这时的slice后的ByteBuf不受影响，因为它有独立的读写指针
ByteBufUtils.debug(buffer2);
```



```sh
read index:0 write index:3 capacity:3
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 02 03 04                                        |...             |
+--------+-------------------------------------------------+----------------+
```





如果slice后的ByteBuf的内容发生了更改

```java
//如果slice后的ByteBuf的内容发生了更改
buffer2.setByte(1, 68);
ByteBufUtils.debug(buffer2);
```



```sh
read index:0 write index:3 capacity:3
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 02 44 04                                        |.D.             |
+--------+-------------------------------------------------+----------------+
```





这时，原始 ByteBuf 也会受影响，因为底层都是同一块内存

```java
//这时，原始ByteBuf也会受影响，因为底层都是同一块内存
ByteBufUtils.debug(buffer);
```



```sh
read index:2 write index:4 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 44 04                                           |D.              |
+--------+-------------------------------------------------+----------------+
```









## duplicate

**零拷贝**的体现之一，就好比截取了原始 ByteBuf 所有内容，并且没有 max capacity 的限制，也是与原始 ByteBuf 使用同一块底层内存，只是读写指针是独立的



![image-20230322154303290](img/Netty学习笔记/image-20230322154303290.png)





```java
package mao.t7;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import lombok.extern.slf4j.Slf4j;
import mao.utils.ByteBufUtils;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t7
 * Class(类名): ByteBufTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/22
 * Time(创建时间)： 15:43
 * Version(版本): 1.0
 * Description(描述)： duplicate
 */

@Slf4j
public class ByteBufTest
{
    public static void main(String[] args)
    {
        ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(16);
        buffer.writeBytes("123456".getBytes(StandardCharsets.UTF_8));
        ByteBufUtils.debug(buffer);
        //读取
        buffer.readByte();

        //duplicate
        ByteBuf duplicate = buffer.duplicate();
        ByteBufUtils.debug(duplicate);

        //写
        duplicate.setByte(2, 65);
        ByteBufUtils.debug(duplicate);
        ByteBufUtils.debug(buffer);
    }
}
```



运行结果：

```sh
read index:0 write index:6 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36                               |123456          |
+--------+-------------------------------------------------+----------------+
read index:1 write index:6 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 32 33 34 35 36                                  |23456           |
+--------+-------------------------------------------------+----------------+
read index:1 write index:6 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 32 41 34 35 36                                  |2A456           |
+--------+-------------------------------------------------+----------------+
read index:1 write index:6 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 32 41 34 35 36                                  |2A456           |
+--------+-------------------------------------------------+----------------+
```







## copy

会将底层内存数据进行深拷贝，因此无论读写，都与原始 ByteBuf 无关



```java
package mao.t8;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import lombok.extern.slf4j.Slf4j;
import mao.utils.ByteBufUtils;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t8
 * Class(类名): ByteBufTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/22
 * Time(创建时间)： 15:48
 * Version(版本): 1.0
 * Description(描述)： copy:会将底层内存数据进行深拷贝，因此无论读写，都与原始 ByteBuf 无关
 */

@Slf4j
public class ByteBufTest
{
    public static void main(String[] args)
    {
        ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(16);
        buffer.writeBytes("123456".getBytes(StandardCharsets.UTF_8));
        ByteBufUtils.debug(buffer);

        //深拷贝
        ByteBuf copy = buffer.copy();
        ByteBufUtils.debug(copy);

        //读取
        log.debug(String.valueOf(copy.readByte()));
        ByteBufUtils.debug(buffer);
        ByteBufUtils.debug(copy);

        //写入
        buffer.setByte(4, 66);
        ByteBufUtils.debug(buffer);
        ByteBufUtils.debug(copy);
    }
}
```



运行结果：

```sh
read index:0 write index:6 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36                               |123456          |
+--------+-------------------------------------------------+----------------+
read index:0 write index:6 capacity:6
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36                               |123456          |
+--------+-------------------------------------------------+----------------+
2023-03-22  15:51:35.446  [main] DEBUG mao.t8.ByteBufTest:  49
read index:0 write index:6 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36                               |123456          |
+--------+-------------------------------------------------+----------------+
read index:1 write index:6 capacity:6
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 32 33 34 35 36                                  |23456           |
+--------+-------------------------------------------------+----------------+
read index:0 write index:6 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 42 36                               |1234B6          |
+--------+-------------------------------------------------+----------------+
read index:1 write index:6 capacity:6
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 32 33 34 35 36                                  |23456           |
+--------+-------------------------------------------------+----------------+
```







## CompositeByteBuf

**零拷贝**的体现之一，可以将多个 ByteBuf 合并为一个逻辑上的 ByteBuf，避免拷贝



```java
package mao.t9;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import io.netty.buffer.CompositeByteBuf;
import lombok.extern.slf4j.Slf4j;
import mao.utils.ByteBufUtils;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t9
 * Class(类名): ByteBufTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/22
 * Time(创建时间)： 22:31
 * Version(版本): 1.0
 * Description(描述)： CompositeByteBuf：将多个 ByteBuf 合并为一个逻辑上的 ByteBuf
 */

@Slf4j
public class ByteBufTest
{
    public static void main(String[] args)
    {
        ByteBuf buffer1 = ByteBufAllocator.DEFAULT.buffer(10);
        ByteBuf buffer2 = ByteBufAllocator.DEFAULT.buffer(10);
        buffer1.writeBytes("12345".getBytes(StandardCharsets.UTF_8));
        buffer2.writeBytes("67890".getBytes(StandardCharsets.UTF_8));
        ByteBufUtils.debug(buffer1);
        ByteBufUtils.debug(buffer2);
        //方法一：性能不高，进行了数据的内存复制操作
        ByteBuf buffer3 = ByteBufAllocator.DEFAULT.buffer(buffer1.readableBytes()
                + buffer2.readableBytes());
        buffer3.writeBytes(buffer1);
        buffer3.writeBytes(buffer2);
        ByteBufUtils.debug(buffer3);

        //方法二：CompositeByteBuf
        CompositeByteBuf compositeByteBuf =
                ByteBufAllocator.DEFAULT.compositeBuffer()
                        // true 表示增加新的 ByteBuf 自动递增 write index, 否则 write index 会始终为 0
                        .addComponents(true,buffer1, buffer2);
        log.info(compositeByteBuf.toString());


    }
}
```



运行结果：

```sh
read index:0 write index:5 capacity:10
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35                                  |12345           |
+--------+-------------------------------------------------+----------------+
read index:0 write index:5 capacity:10
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 36 37 38 39 30                                  |67890           |
+--------+-------------------------------------------------+----------------+
read index:0 write index:10 capacity:10
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30                   |1234567890      |
+--------+-------------------------------------------------+----------------+
2023-03-22  22:47:22.579  [main] INFO  mao.t9.ByteBufTest:  CompositeByteBuf(ridx: 0, widx: 0, cap: 0, components=2)
```



CompositeByteBuf 是一个组合的 ByteBuf，它内部维护了一个 Component 数组，每个 Component 管理一个 ByteBuf，记录了这个 ByteBuf 相对于整体偏移量等信息，代表着整体中某一段的数据。

* 优点，对外是一个虚拟视图，组合这些 ByteBuf 不会产生内存复制
* 缺点，复杂了很多，多次操作会带来性能的损耗







## Unpooled

Unpooled 是一个工具类，类如其名，提供了非池化的 ByteBuf 创建、组合、复制等操作



```java
package mao.t10;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import io.netty.buffer.Unpooled;
import lombok.extern.slf4j.Slf4j;
import mao.utils.ByteBufUtils;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_ByteBuf
 * Package(包名): mao.t10
 * Class(类名): ByteBufTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/22
 * Time(创建时间)： 23:07
 * Version(版本): 1.0
 * Description(描述)： Unpooled 是一个工具类，类如其名，提供了非池化的 ByteBuf 创建、组合、复制等操作
 */

@Slf4j
public class ByteBufTest
{
    public static void main(String[] args)
    {
        ByteBuf buffer1 = ByteBufAllocator.DEFAULT.buffer(10);
        ByteBuf buffer2 = ByteBufAllocator.DEFAULT.buffer(10);
        buffer1.writeBytes("12345".getBytes(StandardCharsets.UTF_8));
        buffer2.writeBytes("67890".getBytes(StandardCharsets.UTF_8));
        ByteBufUtils.debug(buffer1);
        ByteBufUtils.debug(buffer2);
        //当包装ByteBuf个数超过一个时, 底层使用了CompositeByteBuf
        ByteBuf buffer = Unpooled.wrappedBuffer(buffer1, buffer2);
        ByteBufUtils.debug(buffer);
        buffer1.setByte(3, 68);
        ByteBufUtils.debug(buffer);
    }
}
```



运行结果：

```sh
read index:0 write index:5 capacity:10
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35                                  |12345           |
+--------+-------------------------------------------------+----------------+
read index:0 write index:5 capacity:10
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 36 37 38 39 30                                  |67890           |
+--------+-------------------------------------------------+----------------+
read index:0 write index:10 capacity:10
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30                   |1234567890      |
+--------+-------------------------------------------------+----------------+
read index:0 write index:10 capacity:10
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 44 35 36 37 38 39 30                   |123D567890      |
+--------+-------------------------------------------------+----------------+
```







## ByteBuf 优势

* 池化 - 可以重用池中 ByteBuf 实例，更节约内存，减少内存溢出的可能
* 读写指针分离，不需要像 ByteBuffer 一样切换读写模式
* 可以自动扩容
* 支持链式调用，使用更流畅
* 很多地方体现零拷贝，例如 slice、duplicate、CompositeByteBuf











# 双向通信

## 需求

客户端向服务器端发送一段字符串，服务器端立马响应 hello! +发送的字符串



## 服务端

```java
package mao;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_双向通信
 * Package(包名): mao
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/23
 * Time(创建时间)： 13:51
 * Version(版本): 1.0
 * Description(描述)： 服务端
 */

@Slf4j
public class Server
{
    @SneakyThrows
    public static void main(String[] args)
    {
        new ServerBootstrap()
                .group(new NioEventLoopGroup(), new NioEventLoopGroup(Runtime.getRuntime().availableProcessors()))
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                        {
                            /**
                             * 连接建立
                             *
                             * @param ctx ctx
                             * @throws Exception 异常
                             */
                            @Override
                            public void channelActive(ChannelHandlerContext ctx) throws Exception
                            {
                                log.info("连接建立：" + ctx.channel().toString());
                                super.channelActive(ctx);
                            }

                            /**
                             * 通道读
                             *
                             * @param ctx ctx
                             * @param msg 消息
                             * @throws Exception 异常
                             */
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
                            {
                                ByteBuf buffer = (ByteBuf) msg;
                                String s = buffer.toString(StandardCharsets.UTF_8);
                                log.debug(s);
                                String respMsg = "hello! " + s;
                                //buffer.release();
                                //建议使用ctx.alloc()创建ByteBuf
                                buffer = ctx.alloc().buffer();
                                buffer.writeCharSequence(respMsg, StandardCharsets.UTF_8);
                                ctx.writeAndFlush(buffer);
                                super.channelRead(ctx, msg);
                            }

                            @Override
                            public void channelInactive(ChannelHandlerContext ctx) throws Exception
                            {
                                log.info("连接关闭：" + ctx.channel().toString());
                                super.channelInactive(ctx);
                            }
                        });
                    }
                })
                .bind(8080)
                .sync();
        log.info("服务启动成功");
    }
}

```





## 客户端

```java
package mao;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.GenericFutureListener;
import lombok.extern.slf4j.Slf4j;

import java.net.InetSocketAddress;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

/**
 * Project name(项目名称)：Netty_双向通信
 * Package(包名): mao
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/23
 * Time(创建时间)： 14:02
 * Version(版本): 1.0
 * Description(描述)： 客户端
 */

@Slf4j
public class Client
{
    public static void main(String[] args)
    {
        NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
        ChannelFuture channelFuture = new Bootstrap()
                .group(nioEventLoopGroup)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                        {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
                            {
                                ByteBuf byteBuf = (ByteBuf) msg;
                                String respMsg = byteBuf.toString(StandardCharsets.UTF_8);
                                log.info("接收到来自服务端的响应信息：" + respMsg);
                                byteBuf.release();
                            }
                        });
                    }
                })
                .connect(new InetSocketAddress("127.0.0.1", 8080));
        Channel channel = channelFuture.channel();

        Scanner input = new Scanner(System.in);

        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    //System.out.print("请输入：");
                    String next = input.next();
                    if ("q".equals(next))
                    {
                        channel.close();
                    }
                    ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(next.length());
                    buffer.writeCharSequence(next, StandardCharsets.UTF_8);
                    channel.writeAndFlush(buffer);
                    //buffer.release();
                }
            }
        }, "input");
        thread.setDaemon(true);

        channelFuture.addListener(new GenericFutureListener<Future<? super Void>>()
        {
            /**
             * 操作完成(客户端启动完成)
             *
             * @param future future
             * @throws Exception 异常
             */
            @Override
            public void operationComplete(Future<? super Void> future) throws Exception
            {
                boolean success = future.isSuccess();
                if (!success)
                {
                    Throwable throwable = future.cause();
                    log.error("错误：" + throwable.getMessage());
                }
                else
                {
                    log.debug("服务器连接成功");
                    thread.start();
                }
            }
        });

        channel.closeFuture().addListener(new GenericFutureListener<Future<? super Void>>()
        {
            /**
             * 操作完成(关闭客户端)
             *
             * @param future Future
             * @throws Exception 异常
             */
            @Override
            public void operationComplete(Future<? super Void> future) throws Exception
            {
                log.info("关闭客户端");
                nioEventLoopGroup.shutdownGracefully();
            }
        });
    }
}

```





## 运行

启动一个服务端和3个客户端

随即发送数据

服务端

```sh
2023-03-23  14:41:24.717  [main] INFO  mao.Server:  服务启动成功
2023-03-23  14:41:29.905  [nioEventLoopGroup-3-1] INFO  mao.Server:  连接建立：[id: 0x42de88c6, L:/127.0.0.1:8080 - R:/127.0.0.1:64294]
2023-03-23  14:41:32.988  [nioEventLoopGroup-3-2] INFO  mao.Server:  连接建立：[id: 0x0d7f76af, L:/127.0.0.1:8080 - R:/127.0.0.1:64298]
2023-03-23  14:41:35.105  [nioEventLoopGroup-3-3] INFO  mao.Server:  连接建立：[id: 0x7393b555, L:/127.0.0.1:8080 - R:/127.0.0.1:64302]
2023-03-23  14:41:42.038  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-23  14:41:42.039  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-23  14:41:42.039  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-23  14:41:42.039  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-23  14:41:42.042  [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-23  14:41:42.042  [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-23  14:41:42.042  [nioEventLoopGroup-3-1] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@2f39d977
2023-03-23  14:41:42.047  [nioEventLoopGroup-3-1] DEBUG mao.Server:  45326246
2023-03-23  14:41:42.048  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 8, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-23  14:41:42.048  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x42de88c6, L:/127.0.0.1:8080 - R:/127.0.0.1:64294].
2023-03-23  14:41:46.462  [nioEventLoopGroup-3-1] DEBUG mao.Server:  34
2023-03-23  14:41:46.462  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 2, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-23  14:41:46.463  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x42de88c6, L:/127.0.0.1:8080 - R:/127.0.0.1:64294].
2023-03-23  14:41:51.026  [nioEventLoopGroup-3-2] DEBUG mao.Server:  rtet
2023-03-23  14:41:51.026  [nioEventLoopGroup-3-2] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 4, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-23  14:41:51.027  [nioEventLoopGroup-3-2] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x0d7f76af, L:/127.0.0.1:8080 - R:/127.0.0.1:64298].
2023-03-23  14:41:59.197  [nioEventLoopGroup-3-3] DEBUG mao.Server:  你好
2023-03-23  14:41:59.198  [nioEventLoopGroup-3-3] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 6, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-23  14:41:59.198  [nioEventLoopGroup-3-3] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x7393b555, L:/127.0.0.1:8080 - R:/127.0.0.1:64302].
2023-03-23  14:42:06.732  [nioEventLoopGroup-3-3] DEBUG mao.Server:  sfaf
2023-03-23  14:42:06.732  [nioEventLoopGroup-3-3] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 4, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-23  14:42:06.732  [nioEventLoopGroup-3-3] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x7393b555, L:/127.0.0.1:8080 - R:/127.0.0.1:64302].
2023-03-23  14:42:10.048  [nioEventLoopGroup-3-1] DEBUG mao.Server:  asf
2023-03-23  14:42:10.049  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 3, cap: 512) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-23  14:42:10.049  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x42de88c6, L:/127.0.0.1:8080 - R:/127.0.0.1:64294].
2023-03-23  14:42:14.680  [nioEventLoopGroup-3-2] DEBUG mao.Server:  asfgs
2023-03-23  14:42:14.681  [nioEventLoopGroup-3-2] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 5, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-23  14:42:14.681  [nioEventLoopGroup-3-2] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x0d7f76af, L:/127.0.0.1:8080 - R:/127.0.0.1:64298].
2023-03-23  14:42:31.074  [nioEventLoopGroup-3-2] DEBUG mao.Server:  再见
2023-03-23  14:42:31.074  [nioEventLoopGroup-3-2] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 6, cap: 512) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-23  14:42:31.074  [nioEventLoopGroup-3-2] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x0d7f76af, L:/127.0.0.1:8080 - R:/127.0.0.1:64298].
2023-03-23  14:42:38.723  [nioEventLoopGroup-3-2] INFO  mao.Server:  连接关闭：[id: 0x0d7f76af, L:/127.0.0.1:8080 ! R:/127.0.0.1:64298]
2023-03-23  14:42:42.986  [nioEventLoopGroup-3-1] INFO  mao.Server:  连接关闭：[id: 0x42de88c6, L:/127.0.0.1:8080 ! R:/127.0.0.1:64294]
2023-03-23  14:42:47.697  [nioEventLoopGroup-3-3] INFO  mao.Server:  连接关闭：[id: 0x7393b555, L:/127.0.0.1:8080 ! R:/127.0.0.1:64302]
```



客户端1

```sh
2023-03-23  14:41:29.898  [nioEventLoopGroup-2-1] DEBUG mao.Client:  服务器连接成功
45326246
2023-03-23  14:41:42.025  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-23  14:41:42.025  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-23  14:41:42.025  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-23  14:41:42.025  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-23  14:41:42.029  [input] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-23  14:41:42.029  [input] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-23  14:41:42.030  [input] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@5e3df20
2023-03-23  14:41:42.051  [nioEventLoopGroup-2-1] INFO  mao.Client:  接收到来自服务端的响应信息：hello! 45326246
34
2023-03-23  14:41:46.463  [nioEventLoopGroup-2-1] INFO  mao.Client:  接收到来自服务端的响应信息：hello! 34
asf
2023-03-23  14:42:10.049  [nioEventLoopGroup-2-1] INFO  mao.Client:  接收到来自服务端的响应信息：hello! asf
q
2023-03-23  14:42:42.986  [nioEventLoopGroup-2-1] INFO  mao.Client:  关闭客户端
2023-03-23  14:42:45.218  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.PoolThreadCache:  Freed 2 thread-local buffer(s) from thread: nioEventLoopGroup-2-1
```

客户端2

```sh
2023-03-23  14:41:32.989  [nioEventLoopGroup-2-1] DEBUG mao.Client:  服务器连接成功
rtet
2023-03-23  14:41:51.014  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-23  14:41:51.014  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-23  14:41:51.014  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-23  14:41:51.014  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-23  14:41:51.017  [input] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-23  14:41:51.017  [input] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-23  14:41:51.017  [input] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@5e3df20
2023-03-23  14:41:51.031  [nioEventLoopGroup-2-1] INFO  mao.Client:  接收到来自服务端的响应信息：hello! rtet
asfgs
2023-03-23  14:42:14.681  [nioEventLoopGroup-2-1] INFO  mao.Client:  接收到来自服务端的响应信息：hello! asfgs
再见
2023-03-23  14:42:31.074  [nioEventLoopGroup-2-1] INFO  mao.Client:  接收到来自服务端的响应信息：hello! 再见
q
2023-03-23  14:42:38.722  [nioEventLoopGroup-2-1] INFO  mao.Client:  关闭客户端
2023-03-23  14:42:40.962  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.PoolThreadCache:  Freed 2 thread-local buffer(s) from thread: nioEventLoopGroup-2-1
```

客户端3

```sh
2023-03-23  14:41:35.106  [nioEventLoopGroup-2-1] DEBUG mao.Client:  服务器连接成功
你好
2023-03-23  14:41:59.182  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-23  14:41:59.182  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-23  14:41:59.182  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-23  14:41:59.183  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-23  14:41:59.187  [input] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-23  14:41:59.187  [input] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-23  14:41:59.188  [input] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@5e3df20
2023-03-23  14:41:59.202  [nioEventLoopGroup-2-1] INFO  mao.Client:  接收到来自服务端的响应信息：hello! 你好
sfaf
2023-03-23  14:42:06.732  [nioEventLoopGroup-2-1] INFO  mao.Client:  接收到来自服务端的响应信息：hello! sfaf
q
2023-03-23  14:42:47.697  [nioEventLoopGroup-2-1] INFO  mao.Client:  关闭客户端
2023-03-23  14:42:49.944  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.PoolThreadCache:  Freed 1 thread-local buffer(s) from thread: nioEventLoopGroup-2-1
```















# 粘包与半包

## 粘包现象

### 服务端

```java
package mao.t1;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_黏包与半包现象
 * Package(包名): mao.t1
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/23
 * Time(创建时间)： 22:10
 * Version(版本): 1.0
 * Description(描述)： 服务端，演示粘包现象
 */

@Slf4j
public class Server
{
    public static void main(String[] args)
    {
        new Server().start();
    }

    private void start()
    {
        NioEventLoopGroup boss = new NioEventLoopGroup(1);
        NioEventLoopGroup worker = new NioEventLoopGroup();
        try
        {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.channel(NioServerSocketChannel.class);
            serverBootstrap.group(boss, worker);
            serverBootstrap.childHandler(new ChannelInitializer<SocketChannel>()
            {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception
                {
                    ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                    ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                    {
                        @Override
                        public void channelActive(ChannelHandlerContext ctx) throws Exception
                        {
                            log.debug("connected {}", ctx.channel());
                            super.channelActive(ctx);
                        }

                        @Override
                        public void channelInactive(ChannelHandlerContext ctx) throws Exception
                        {
                            log.debug("disconnect {}", ctx.channel());
                            super.channelInactive(ctx);
                        }
                    });
                }
            });
            ChannelFuture channelFuture = serverBootstrap.bind(8080);
            log.debug("{} binding...", channelFuture.channel());
            channelFuture.sync();
            log.debug("{} bound...", channelFuture.channel());
            channelFuture.channel().closeFuture().sync();
        }
        catch (InterruptedException e)
        {
            log.error("server error", e);
        }
        finally
        {
            boss.shutdownGracefully();
            worker.shutdownGracefully();
            log.debug("服务已关闭");
        }
    }
}
```





### 客户端

```java
package mao.t1;

import io.netty.bootstrap.Bootstrap;
import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.extern.slf4j.Slf4j;

import java.util.Random;

/**
 * Project name(项目名称)：Netty_黏包与半包现象
 * Package(包名): mao.t1
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/23
 * Time(创建时间)： 22:10
 * Version(版本): 1.0
 * Description(描述)： 客户端，演示粘包现象
 */

@Slf4j
public class Client
{
    public static void main(String[] args)
    {
        NioEventLoopGroup worker = new NioEventLoopGroup();
        try
        {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.channel(NioSocketChannel.class);
            bootstrap.group(worker);
            bootstrap.handler(new ChannelInitializer<SocketChannel>()
            {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception
                {
                    ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                    {
                        @Override
                        public void channelActive(ChannelHandlerContext ctx) throws Exception
                        {
                            log.debug("sending...");
                            Random r = new Random();
                            char c = 'a';
                            for (int i = 0; i < 10; i++)
                            {
                                ByteBuf buffer = ctx.alloc().buffer();
                                buffer.writeBytes(new byte[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15});
                                ctx.writeAndFlush(buffer);
                            }
                        }
                    });
                }
            });
            ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 8080).sync();
            channelFuture.channel().closeFuture().sync();

        }
        catch (InterruptedException e)
        {
            log.error("client error", e);
        }
        finally
        {
            worker.shutdownGracefully();
        }
    }

}
```





### 服务端运行结果

```sh
2023-03-23  22:28:03.950  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x322fbe77, L:/127.0.0.1:8080 - R:/127.0.0.1:50641] READ: 160B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000010| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000020| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000030| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000040| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000050| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000060| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000070| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000080| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000090| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
```



**服务器端的某次输出，可以看到一次就接收了 160 个字节，而非分 10 次接收**









## 半包现象

### 服务端

```java
package mao.t2;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_黏包与半包现象
 * Package(包名): mao.t2
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/23
 * Time(创建时间)： 22:33
 * Version(版本): 1.0
 * Description(描述)： 服务端，演示半包现象
 */

@Slf4j
public class Server
{
    public static void main(String[] args)
    {
        new Server().start();
    }

    private void start()
    {
        NioEventLoopGroup boss = new NioEventLoopGroup(1);
        NioEventLoopGroup worker = new NioEventLoopGroup();
        try
        {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.channel(NioServerSocketChannel.class);
            serverBootstrap.group(boss, worker);
            //更改接收缓冲区，更改成10字节，可能没用
            serverBootstrap.option(ChannelOption.SO_RCVBUF, 10);
            serverBootstrap.childHandler(new ChannelInitializer<SocketChannel>()
            {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception
                {
                    ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                    ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                    {
                        @Override
                        public void channelActive(ChannelHandlerContext ctx) throws Exception
                        {
                            log.debug("connected {}", ctx.channel());
                            super.channelActive(ctx);
                        }

                        @Override
                        public void channelInactive(ChannelHandlerContext ctx) throws Exception
                        {
                            log.debug("disconnect {}", ctx.channel());
                            super.channelInactive(ctx);
                        }
                    });
                }
            });
            ChannelFuture channelFuture = serverBootstrap.bind(8080);
            log.debug("{} binding...", channelFuture.channel());
            channelFuture.sync();
            log.debug("{} bound...", channelFuture.channel());
            channelFuture.channel().closeFuture().sync();
        }
        catch (InterruptedException e)
        {
            log.error("server error", e);
        }
        finally
        {
            boss.shutdownGracefully();
            worker.shutdownGracefully();
            log.debug("服务已关闭");
        }
    }
}
```





### 客户端

```java
package mao.t2;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import lombok.extern.slf4j.Slf4j;

import java.util.Random;

/**
 * Project name(项目名称)：Netty_黏包与半包现象
 * Package(包名): mao.t2
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/23
 * Time(创建时间)： 22:33
 * Version(版本): 1.0
 * Description(描述)： 客户端，演示半包现象
 */

@Slf4j
public class Client
{
    public static void main(String[] args)
    {
        NioEventLoopGroup worker = new NioEventLoopGroup();
        try
        {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.channel(NioSocketChannel.class);
            bootstrap.group(worker);
            bootstrap.handler(new ChannelInitializer<SocketChannel>()
            {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception
                {
                    ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                    {
                        @Override
                        public void channelActive(ChannelHandlerContext ctx) throws Exception
                        {
                            log.debug("sending...");
                            Random r = new Random();
                            char c = 'a';
                            ByteBuf buffer = ctx.alloc().buffer();
                            for (int i = 0; i < 100; i++)
                            {
                                buffer.writeBytes(new byte[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15});
                            }
                            ctx.writeAndFlush(buffer);
                        }
                    });
                }
            });
            ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 8080).sync();
            channelFuture.channel().closeFuture().sync();

        }
        catch (InterruptedException e)
        {
            log.error("client error", e);
        }
        finally
        {
            worker.shutdownGracefully();
        }
    }
}
```





### 服务端运行结果

```sh
2023-03-23  22:39:06.148  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x1af629bd, L:/127.0.0.1:8080 - R:/127.0.0.1:51157] READ: 1024B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000010| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000020| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000030| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000040| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000050| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000060| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000070| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000080| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000090| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000a0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000b0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000c0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000d0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000e0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000f0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000100| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000110| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000120| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000130| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000140| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000150| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000160| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000170| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000180| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000190| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001a0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001b0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001c0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001d0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001e0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001f0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000200| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000210| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000220| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000230| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000240| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000250| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000260| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000270| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000280| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000290| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000002a0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000002b0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000002c0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000002d0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000002e0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000002f0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000300| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000310| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000320| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000330| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000340| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000350| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000360| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000370| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000380| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000390| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000003a0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000003b0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000003c0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000003d0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000003e0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000003f0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
2023-03-23  22:39:06.148  [nioEventLoopGroup-3-2] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 1024, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-23  22:39:06.148  [nioEventLoopGroup-3-2] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x1af629bd, L:/127.0.0.1:8080 - R:/127.0.0.1:51157].
2023-03-23  22:39:06.149  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x1af629bd, L:/127.0.0.1:8080 - R:/127.0.0.1:51157] READ: 576B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000010| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000020| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000030| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000040| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000050| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000060| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000070| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000080| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000090| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000a0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000b0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000c0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000d0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000e0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000000f0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000100| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000110| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000120| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000130| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000140| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000150| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000160| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000170| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000180| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000190| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001a0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001b0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001c0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001d0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001e0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|000001f0| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000200| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000210| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000220| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
|00000230| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
```



服务器端的某次输出，可以看到接收的消息被分为两节，第一次 1024 字节，第二次 576 字节



**serverBootstrap.option(ChannelOption.SO_RCVBUF, 10) 影响的底层接收缓冲区（即滑动窗口）大小，仅决定了 netty 读取的最小单位，netty 实际每次读取的一般是它的整数倍**









## 现象分析

### 粘包

**现象**：发送 abc 、def两次消息，实际接收 abcdef一次消息



**原因**

* 应用层：接收方 ByteBuf 设置太大（Netty 默认 1024）
* 滑动窗口：假设发送方 256 bytes 表示一个完整报文，但由于接收方处理不及时且窗口大小足够大，这 256 bytes 字节就会缓冲在接收方的滑动窗口中，当滑动窗口中缓冲了多个报文就会粘包
* Nagle 算法：会造成粘包





### 半包

**现象**：发送 abcdef一次消息，接收 abc、 def两次消息



**原因**

* 应用层：接收方 ByteBuf 小于实际发送数据量
* 滑动窗口：假设接收方的窗口只剩了 128 bytes，发送方的报文大小是 256 bytes，这时放不下了，只能先发送前 128 bytes，等待 ack 后才能发送剩余部分，这就造成了半包
* MSS 限制：当发送的数据超过 MSS 限制后，会将数据切分发送，就会造成半包





### 滑动窗口

TCP 以一个段（segment）为单位，每发送一个段就需要进行一次确认应答（ack）处理，但如果这么做，缺点是包的往返时间越长性能就越差

![image-20230324134153236](img/Netty学习笔记/image-20230324134153236.png)



为了解决此问题，引入了窗口概念，窗口大小即决定了无需等待应答而可以继续发送的数据最大值

![image-20230324134205802](img/Netty学习笔记/image-20230324134205802.png)



窗口实际就起到一个缓冲区的作用，同时也能起到流量控制的作用

* 图中深色的部分即要发送的数据，高亮的部分即窗口
* 窗口内的数据才允许被发送，当应答未到达前，窗口必须停止滑动
* 如果 1001~2000 这个段的数据 ack 回来了，窗口就可以向前滑动
* 接收方也会维护一个窗口，只有落在窗口内的数据才能允许接收







### MSS限制

* 链路层对一次能够发送的最大数据有限制，这个限制称之为 MTU（maximum transmission unit），不同的链路设备的 MTU 值也有所不同，例如

 * 以太网的 MTU 是 1500
 * FDDI（光纤分布式数据接口）的 MTU 是 4352
 * 本地回环地址的 MTU 是 65535 - 本地测试不走网卡

* MSS 是最大段长度（maximum segment size），它是 MTU 刨去 tcp 头和 ip 头后剩余能够作为数据传输的字节数

 * ipv4 tcp 头占用 20 bytes，ip 头占用 20 bytes，因此以太网 MSS 的值为 1500 - 40 = 1460
 * TCP 在传递大量数据时，会按照 MSS 大小将数据进行分割发送
 * MSS 的值在三次握手时通知对方自己 MSS 的值，然后在两者之间选择一个小值作为 MSS







### Nagle算法

* 即使发送一个字节，也需要加入 tcp 头和 ip 头，也就是总字节数会使用 41 bytes，非常不经济。因此为了提高网络利用率，tcp 希望尽可能发送足够大的数据，这就是 Nagle 算法产生的缘由
* 该算法是指发送端即使还有应该发送的数据，但如果这部分数据很少的话，则进行延迟发送
  * 如果 SO_SNDBUF 的数据达到 MSS，则需要发送
  * 如果 SO_SNDBUF 中含有 FIN（表示需要连接关闭）这时将剩余数据发送，再关闭
  * 如果 TCP_NODELAY = true，则需要发送
  * 已发送的数据都收到 ack 时，则需要发送
  * 上述条件不满足，但发生超时（一般为 200ms）则需要发送
  * 除上述情况，延迟发送













## 解决方案

1. 短链接，发一个包建立一次连接，这样连接建立到连接断开之间就是消息的边界，缺点效率太低
2. 每一条消息采用固定长度，缺点浪费空间
3. 每一条消息采用分隔符，例如 \n，缺点需要转义
4. 每一条消息分为 head 和 body，head 中包含 body 的长度





### 短链接

#### 服务端

```java
package mao.t1;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.GenericFutureListener;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_黏包与半包现象解决
 * Package(包名): mao.t1
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/24
 * Time(创建时间)： 21:00
 * Version(版本): 1.0
 * Description(描述)： 短链接方式解决黏包与半包问题，服务端
 */

@Slf4j
public class Server
{
    @SneakyThrows
    public static void main(String[] args)
    {
        NioEventLoopGroup boos = new NioEventLoopGroup();
        NioEventLoopGroup worker = new NioEventLoopGroup();
        Channel channel = new ServerBootstrap()
                .group(boos, worker)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                        ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                        {
                            @Override
                            public void channelActive(ChannelHandlerContext ctx) throws Exception
                            {
                                log.debug("连接建立：" + ctx.channel());
                                super.channelActive(ctx);
                            }

                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
                            {
                                ByteBuf byteBuf = (ByteBuf) msg;
                                log.debug("读事件:" + byteBuf);
                                super.channelRead(ctx, msg);
                            }

                            @Override
                            public void channelInactive(ChannelHandlerContext ctx) throws Exception
                            {
                                log.debug("连接关闭：" + ctx.channel());
                                super.channelInactive(ctx);
                            }
                        });
                    }
                })
                .bind(8080)
                .sync().channel();
        channel.closeFuture().addListener(new GenericFutureListener<Future<? super Void>>()
        {
            @Override
            public void operationComplete(Future<? super Void> future) throws Exception
            {
                boos.shutdownGracefully();
                worker.shutdownGracefully();
            }
        });
    }

}
```



#### 客户端

```java
package mao.t1;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_黏包与半包现象解决
 * Package(包名): mao.t1
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/24
 * Time(创建时间)： 21:01
 * Version(版本): 1.0
 * Description(描述)： 短链接方式解决黏包与半包问题，客户端
 */

@Slf4j
public class Client
{
    public static void main(String[] args)
    {
        //发送5次消息
        for (int i = 0; i < 5; i++)
        {
            send();
        }
    }

    private static void send()
    {
        NioEventLoopGroup worker = new NioEventLoopGroup();
        try
        {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.channel(NioSocketChannel.class);
            bootstrap.group(worker);
            bootstrap.handler(new ChannelInitializer<SocketChannel>()
            {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception
                {
                    ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                    ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                    {
                        @Override
                        public void channelActive(ChannelHandlerContext ctx) throws Exception
                        {
                            log.debug("sending...");
                            ByteBuf buffer = ctx.alloc().buffer();
                            buffer.writeBytes(new byte[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15});
                            ctx.writeAndFlush(buffer);
                            // 发完即关
                            ctx.close();
                        }
                    });
                }
            });
            ChannelFuture channelFuture = bootstrap.connect("localhost", 8080).sync();
            channelFuture.channel().closeFuture().sync();

        }
        catch (InterruptedException e)
        {
            log.error("client error", e);
        }
        finally
        {
            worker.shutdownGracefully();
        }
    }
}
```





#### 运行结果

客户端

```sh
2023-03-24  21:14:49.711  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xf5854979, L:/127.0.0.1:49953 - R:localhost/127.0.0.1:8080] WRITE: 16B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:14:49.711  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xf5854979, L:/127.0.0.1:49953 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:14:49.712  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xf5854979, L:/127.0.0.1:49953 - R:localhost/127.0.0.1:8080] CLOSE
2023-03-24  21:14:49.713  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xf5854979, L:/127.0.0.1:49953 ! R:localhost/127.0.0.1:8080] INACTIVE
2023-03-24  21:14:49.713  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xf5854979, L:/127.0.0.1:49953 ! R:localhost/127.0.0.1:8080] UNREGISTERED
2023-03-24  21:14:49.787  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x72a58ebe] REGISTERED
2023-03-24  21:14:49.787  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x72a58ebe] CONNECT: localhost/127.0.0.1:8080
2023-03-24  21:14:49.788  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x72a58ebe, L:/127.0.0.1:49954 - R:localhost/127.0.0.1:8080] ACTIVE
2023-03-24  21:14:49.788  [nioEventLoopGroup-3-1] DEBUG mao.t1.Client:  sending...
2023-03-24  21:14:49.791  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x72a58ebe, L:/127.0.0.1:49954 - R:localhost/127.0.0.1:8080] WRITE: 16B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:14:49.793  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x72a58ebe, L:/127.0.0.1:49954 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:14:49.794  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x72a58ebe, L:/127.0.0.1:49954 - R:localhost/127.0.0.1:8080] CLOSE
2023-03-24  21:14:49.794  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x72a58ebe, L:/127.0.0.1:49954 ! R:localhost/127.0.0.1:8080] INACTIVE
2023-03-24  21:14:49.795  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x72a58ebe, L:/127.0.0.1:49954 ! R:localhost/127.0.0.1:8080] UNREGISTERED
2023-03-24  21:14:49.836  [nioEventLoopGroup-4-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x416d4b44] REGISTERED
2023-03-24  21:14:49.837  [nioEventLoopGroup-4-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x416d4b44] CONNECT: localhost/127.0.0.1:8080
2023-03-24  21:14:49.837  [nioEventLoopGroup-4-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x416d4b44, L:/127.0.0.1:49955 - R:localhost/127.0.0.1:8080] ACTIVE
2023-03-24  21:14:49.837  [nioEventLoopGroup-4-1] DEBUG mao.t1.Client:  sending...
2023-03-24  21:14:49.844  [nioEventLoopGroup-4-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x416d4b44, L:/127.0.0.1:49955 - R:localhost/127.0.0.1:8080] WRITE: 16B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:14:49.844  [nioEventLoopGroup-4-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x416d4b44, L:/127.0.0.1:49955 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:14:49.844  [nioEventLoopGroup-4-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x416d4b44, L:/127.0.0.1:49955 - R:localhost/127.0.0.1:8080] CLOSE
2023-03-24  21:14:49.845  [nioEventLoopGroup-4-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x416d4b44, L:/127.0.0.1:49955 ! R:localhost/127.0.0.1:8080] INACTIVE
2023-03-24  21:14:49.845  [nioEventLoopGroup-4-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x416d4b44, L:/127.0.0.1:49955 ! R:localhost/127.0.0.1:8080] UNREGISTERED
2023-03-24  21:14:49.892  [nioEventLoopGroup-5-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x868b830d] REGISTERED
2023-03-24  21:14:49.892  [nioEventLoopGroup-5-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x868b830d] CONNECT: localhost/127.0.0.1:8080
2023-03-24  21:14:49.893  [nioEventLoopGroup-5-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x868b830d, L:/127.0.0.1:49956 - R:localhost/127.0.0.1:8080] ACTIVE
2023-03-24  21:14:49.893  [nioEventLoopGroup-5-1] DEBUG mao.t1.Client:  sending...
2023-03-24  21:14:49.898  [nioEventLoopGroup-5-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x868b830d, L:/127.0.0.1:49956 - R:localhost/127.0.0.1:8080] WRITE: 16B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:14:49.898  [nioEventLoopGroup-5-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x868b830d, L:/127.0.0.1:49956 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:14:49.898  [nioEventLoopGroup-5-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x868b830d, L:/127.0.0.1:49956 - R:localhost/127.0.0.1:8080] CLOSE
2023-03-24  21:14:49.898  [nioEventLoopGroup-5-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x868b830d, L:/127.0.0.1:49956 ! R:localhost/127.0.0.1:8080] INACTIVE
2023-03-24  21:14:49.898  [nioEventLoopGroup-5-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x868b830d, L:/127.0.0.1:49956 ! R:localhost/127.0.0.1:8080] UNREGISTERED
2023-03-24  21:14:49.954  [nioEventLoopGroup-6-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xe228bc61] REGISTERED
2023-03-24  21:14:49.955  [nioEventLoopGroup-6-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xe228bc61] CONNECT: localhost/127.0.0.1:8080
2023-03-24  21:14:49.955  [nioEventLoopGroup-6-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xe228bc61, L:/127.0.0.1:49957 - R:localhost/127.0.0.1:8080] ACTIVE
2023-03-24  21:14:49.955  [nioEventLoopGroup-6-1] DEBUG mao.t1.Client:  sending...
2023-03-24  21:14:49.960  [nioEventLoopGroup-6-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xe228bc61, L:/127.0.0.1:49957 - R:localhost/127.0.0.1:8080] WRITE: 16B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:14:49.960  [nioEventLoopGroup-6-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xe228bc61, L:/127.0.0.1:49957 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:14:49.960  [nioEventLoopGroup-6-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xe228bc61, L:/127.0.0.1:49957 - R:localhost/127.0.0.1:8080] CLOSE
2023-03-24  21:14:49.960  [nioEventLoopGroup-6-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xe228bc61, L:/127.0.0.1:49957 ! R:localhost/127.0.0.1:8080] INACTIVE
2023-03-24  21:14:49.960  [nioEventLoopGroup-6-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xe228bc61, L:/127.0.0.1:49957 ! R:localhost/127.0.0.1:8080] UNREGISTERED
2023-03-24  21:14:51.935  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.PoolThreadCache:  Freed 1 thread-local buffer(s) from thread: nioEventLoopGroup-2-1
2023-03-24  21:14:52.011  [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.PoolThreadCache:  Freed 1 thread-local buffer(s) from thread: nioEventLoopGroup-3-1
2023-03-24  21:14:52.058  [nioEventLoopGroup-4-1] DEBUG io.netty.buffer.PoolThreadCache:  Freed 1 thread-local buffer(s) from thread: nioEventLoopGroup-4-1
2023-03-24  21:14:52.120  [nioEventLoopGroup-5-1] DEBUG io.netty.buffer.PoolThreadCache:  Freed 1 thread-local buffer(s) from thread: nioEventLoopGroup-5-1
2023-03-24  21:14:52.199  [nioEventLoopGroup-6-1] DEBUG io.netty.buffer.PoolThreadCache:  Freed 1 thread-local buffer(s) from thread: nioEventLoopGroup-6-1
```



服务端

```sh
2023-03-24  21:14:49.706  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xfcb39f29, L:/127.0.0.1:8080 - R:/127.0.0.1:49953] REGISTERED
2023-03-24  21:14:49.706  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xfcb39f29, L:/127.0.0.1:8080 - R:/127.0.0.1:49953] ACTIVE
2023-03-24  21:14:49.707  [nioEventLoopGroup-3-1] DEBUG mao.t1.Server:  连接建立：[id: 0xfcb39f29, L:/127.0.0.1:8080 - R:/127.0.0.1:49953]
2023-03-24  21:14:49.713  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-24  21:14:49.713  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-24  21:14:49.713  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-24  21:14:49.714  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-24  21:14:49.721  [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-24  21:14:49.721  [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-24  21:14:49.724  [nioEventLoopGroup-3-1] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@3c539b8b
2023-03-24  21:14:49.737  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xfcb39f29, L:/127.0.0.1:8080 - R:/127.0.0.1:49953] READ: 16B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:14:49.737  [nioEventLoopGroup-3-1] DEBUG mao.t1.Server:  读事件:PooledUnsafeDirectByteBuf(ridx: 0, widx: 16, cap: 1024)
2023-03-24  21:14:49.737  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 16, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:14:49.738  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xfcb39f29, L:/127.0.0.1:8080 - R:/127.0.0.1:49953].
2023-03-24  21:14:49.738  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xfcb39f29, L:/127.0.0.1:8080 - R:/127.0.0.1:49953] READ COMPLETE
2023-03-24  21:14:49.738  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xfcb39f29, L:/127.0.0.1:8080 - R:/127.0.0.1:49953] READ COMPLETE
2023-03-24  21:14:49.739  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xfcb39f29, L:/127.0.0.1:8080 ! R:/127.0.0.1:49953] INACTIVE
2023-03-24  21:14:49.739  [nioEventLoopGroup-3-1] DEBUG mao.t1.Server:  连接关闭：[id: 0xfcb39f29, L:/127.0.0.1:8080 ! R:/127.0.0.1:49953]
2023-03-24  21:14:49.739  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xfcb39f29, L:/127.0.0.1:8080 ! R:/127.0.0.1:49953] UNREGISTERED
2023-03-24  21:14:49.788  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x649051a8, L:/127.0.0.1:8080 - R:/127.0.0.1:49954] REGISTERED
2023-03-24  21:14:49.789  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x649051a8, L:/127.0.0.1:8080 - R:/127.0.0.1:49954] ACTIVE
2023-03-24  21:14:49.789  [nioEventLoopGroup-3-2] DEBUG mao.t1.Server:  连接建立：[id: 0x649051a8, L:/127.0.0.1:8080 - R:/127.0.0.1:49954]
2023-03-24  21:14:49.801  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x649051a8, L:/127.0.0.1:8080 - R:/127.0.0.1:49954] READ: 16B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:14:49.801  [nioEventLoopGroup-3-2] DEBUG mao.t1.Server:  读事件:PooledUnsafeDirectByteBuf(ridx: 0, widx: 16, cap: 1024)
2023-03-24  21:14:49.801  [nioEventLoopGroup-3-2] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 16, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:14:49.802  [nioEventLoopGroup-3-2] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x649051a8, L:/127.0.0.1:8080 - R:/127.0.0.1:49954].
2023-03-24  21:14:49.802  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x649051a8, L:/127.0.0.1:8080 - R:/127.0.0.1:49954] READ COMPLETE
2023-03-24  21:14:49.802  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x649051a8, L:/127.0.0.1:8080 - R:/127.0.0.1:49954] READ COMPLETE
2023-03-24  21:14:49.802  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x649051a8, L:/127.0.0.1:8080 ! R:/127.0.0.1:49954] INACTIVE
2023-03-24  21:14:49.802  [nioEventLoopGroup-3-2] DEBUG mao.t1.Server:  连接关闭：[id: 0x649051a8, L:/127.0.0.1:8080 ! R:/127.0.0.1:49954]
2023-03-24  21:14:49.802  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x649051a8, L:/127.0.0.1:8080 ! R:/127.0.0.1:49954] UNREGISTERED
2023-03-24  21:14:49.839  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x12c675d4, L:/127.0.0.1:8080 - R:/127.0.0.1:49955] REGISTERED
2023-03-24  21:14:49.839  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x12c675d4, L:/127.0.0.1:8080 - R:/127.0.0.1:49955] ACTIVE
2023-03-24  21:14:49.839  [nioEventLoopGroup-3-3] DEBUG mao.t1.Server:  连接建立：[id: 0x12c675d4, L:/127.0.0.1:8080 - R:/127.0.0.1:49955]
2023-03-24  21:14:49.852  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x12c675d4, L:/127.0.0.1:8080 - R:/127.0.0.1:49955] READ: 16B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:14:49.852  [nioEventLoopGroup-3-3] DEBUG mao.t1.Server:  读事件:PooledUnsafeDirectByteBuf(ridx: 0, widx: 16, cap: 1024)
2023-03-24  21:14:49.852  [nioEventLoopGroup-3-3] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 16, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:14:49.853  [nioEventLoopGroup-3-3] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x12c675d4, L:/127.0.0.1:8080 - R:/127.0.0.1:49955].
2023-03-24  21:14:49.853  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x12c675d4, L:/127.0.0.1:8080 - R:/127.0.0.1:49955] READ COMPLETE
2023-03-24  21:14:49.853  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x12c675d4, L:/127.0.0.1:8080 - R:/127.0.0.1:49955] READ COMPLETE
2023-03-24  21:14:49.853  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x12c675d4, L:/127.0.0.1:8080 ! R:/127.0.0.1:49955] INACTIVE
2023-03-24  21:14:49.853  [nioEventLoopGroup-3-3] DEBUG mao.t1.Server:  连接关闭：[id: 0x12c675d4, L:/127.0.0.1:8080 ! R:/127.0.0.1:49955]
2023-03-24  21:14:49.853  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x12c675d4, L:/127.0.0.1:8080 ! R:/127.0.0.1:49955] UNREGISTERED
2023-03-24  21:14:49.894  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x501aa2ea, L:/127.0.0.1:8080 - R:/127.0.0.1:49956] REGISTERED
2023-03-24  21:14:49.894  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x501aa2ea, L:/127.0.0.1:8080 - R:/127.0.0.1:49956] ACTIVE
2023-03-24  21:14:49.894  [nioEventLoopGroup-3-4] DEBUG mao.t1.Server:  连接建立：[id: 0x501aa2ea, L:/127.0.0.1:8080 - R:/127.0.0.1:49956]
2023-03-24  21:14:49.907  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x501aa2ea, L:/127.0.0.1:8080 - R:/127.0.0.1:49956] READ: 16B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:14:49.907  [nioEventLoopGroup-3-4] DEBUG mao.t1.Server:  读事件:PooledUnsafeDirectByteBuf(ridx: 0, widx: 16, cap: 1024)
2023-03-24  21:14:49.908  [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 16, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:14:49.908  [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x501aa2ea, L:/127.0.0.1:8080 - R:/127.0.0.1:49956].
2023-03-24  21:14:49.908  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x501aa2ea, L:/127.0.0.1:8080 - R:/127.0.0.1:49956] READ COMPLETE
2023-03-24  21:14:49.908  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x501aa2ea, L:/127.0.0.1:8080 - R:/127.0.0.1:49956] READ COMPLETE
2023-03-24  21:14:49.908  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x501aa2ea, L:/127.0.0.1:8080 ! R:/127.0.0.1:49956] INACTIVE
2023-03-24  21:14:49.908  [nioEventLoopGroup-3-4] DEBUG mao.t1.Server:  连接关闭：[id: 0x501aa2ea, L:/127.0.0.1:8080 ! R:/127.0.0.1:49956]
2023-03-24  21:14:49.908  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x501aa2ea, L:/127.0.0.1:8080 ! R:/127.0.0.1:49956] UNREGISTERED
2023-03-24  21:14:49.956  [nioEventLoopGroup-3-5] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x8a8ec4ee, L:/127.0.0.1:8080 - R:/127.0.0.1:49957] REGISTERED
2023-03-24  21:14:49.957  [nioEventLoopGroup-3-5] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x8a8ec4ee, L:/127.0.0.1:8080 - R:/127.0.0.1:49957] ACTIVE
2023-03-24  21:14:49.957  [nioEventLoopGroup-3-5] DEBUG mao.t1.Server:  连接建立：[id: 0x8a8ec4ee, L:/127.0.0.1:8080 - R:/127.0.0.1:49957]
2023-03-24  21:14:49.967  [nioEventLoopGroup-3-5] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x8a8ec4ee, L:/127.0.0.1:8080 - R:/127.0.0.1:49957] READ: 16B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:14:49.967  [nioEventLoopGroup-3-5] DEBUG mao.t1.Server:  读事件:PooledUnsafeDirectByteBuf(ridx: 0, widx: 16, cap: 1024)
2023-03-24  21:14:49.968  [nioEventLoopGroup-3-5] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 16, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:14:49.968  [nioEventLoopGroup-3-5] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x8a8ec4ee, L:/127.0.0.1:8080 - R:/127.0.0.1:49957].
2023-03-24  21:14:49.968  [nioEventLoopGroup-3-5] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x8a8ec4ee, L:/127.0.0.1:8080 - R:/127.0.0.1:49957] READ COMPLETE
2023-03-24  21:14:49.968  [nioEventLoopGroup-3-5] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x8a8ec4ee, L:/127.0.0.1:8080 - R:/127.0.0.1:49957] READ COMPLETE
2023-03-24  21:14:49.968  [nioEventLoopGroup-3-5] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x8a8ec4ee, L:/127.0.0.1:8080 ! R:/127.0.0.1:49957] INACTIVE
2023-03-24  21:14:49.968  [nioEventLoopGroup-3-5] DEBUG mao.t1.Server:  连接关闭：[id: 0x8a8ec4ee, L:/127.0.0.1:8080 ! R:/127.0.0.1:49957]
2023-03-24  21:14:49.968  [nioEventLoopGroup-3-5] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x8a8ec4ee, L:/127.0.0.1:8080 ! R:/127.0.0.1:49957] UNREGISTERED
```





**半包用这种办法还是不好解决，因为接收方的缓冲区大小是有限的**







### 固定长度

让所有数据包长度固定（假设长度为 32 字节），服务器端加入

```java
ch.pipeline().addLast(new FixedLengthFrameDecoder(64));
```

采用这种方法后，客户端什么时候 flush 都可以



缺点是，数据包的大小不好把握

* 长度定的太大，浪费
* 长度定的太小，对某些数据包又显得不够





#### 服务端

```java
package mao.t2;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.FixedLengthFrameDecoder;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.GenericFutureListener;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_黏包与半包现象解决
 * Package(包名): mao.t2
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/24
 * Time(创建时间)： 21:30
 * Version(版本): 1.0
 * Description(描述)： 固定长度方式解决黏包与半包问题，服务端
 */

@Slf4j
public class Server
{
    @SneakyThrows
    public static void main(String[] args)
    {
        NioEventLoopGroup boos = new NioEventLoopGroup();
        NioEventLoopGroup worker = new NioEventLoopGroup();
        Channel channel = new ServerBootstrap()
                .group(boos, worker)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new FixedLengthFrameDecoder(32));
                        ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                        ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                        {
                            @Override
                            public void channelActive(ChannelHandlerContext ctx) throws Exception
                            {
                                log.debug("连接建立：" + ctx.channel());
                                super.channelActive(ctx);
                            }

                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
                            {
                                ByteBuf byteBuf = (ByteBuf) msg;
                                log.debug("读事件:" + byteBuf);
                                super.channelRead(ctx, msg);
                            }

                            @Override
                            public void channelInactive(ChannelHandlerContext ctx) throws Exception
                            {
                                log.debug("连接关闭：" + ctx.channel());
                                super.channelInactive(ctx);
                            }
                        });
                    }
                })
                .bind(8080)
                .sync().channel();
        channel.closeFuture().addListener(new GenericFutureListener<Future<? super Void>>()
        {
            @Override
            public void operationComplete(Future<? super Void> future) throws Exception
            {
                boos.shutdownGracefully();
                worker.shutdownGracefully();
            }
        });
    }
}
```





#### 客户端

```java
package mao.t2;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.extern.slf4j.Slf4j;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_黏包与半包现象解决
 * Package(包名): mao.t2
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/24
 * Time(创建时间)： 21:26
 * Version(版本): 1.0
 * Description(描述)： 固定长度方式解决黏包与半包问题，客户端
 */

@Slf4j
public class Client
{
    public static void main(String[] args)
    {
        NioEventLoopGroup worker = new NioEventLoopGroup();
        try
        {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.channel(NioSocketChannel.class);
            bootstrap.group(worker);
            bootstrap.handler(new ChannelInitializer<SocketChannel>()
            {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception
                {
                    ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                    ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                    {
                        @Override
                        public void channelActive(ChannelHandlerContext ctx) throws Exception
                        {
                            send(ctx, "123");
                            send(ctx, "hello");
                            send(ctx, "12345678901");
                            send(ctx, "j");
                            send(ctx, "dsagsagasdhaerhsdg");
                            send(ctx, "+++45678");
                        }
                    });
                }
            });
            ChannelFuture channelFuture = bootstrap.connect("localhost", 8080).sync();
            channelFuture.channel().closeFuture().sync();

        }
        catch (InterruptedException e)
        {
            log.error("client error", e);
        }
        finally
        {
            worker.shutdownGracefully();
        }
    }

    /**
     * 发送消息,固定长度32字节
     *
     * @param ctx ctx
     * @param msg 消息
     */
    private static void send(ChannelHandlerContext ctx, String msg)
    {
        ByteBuf buffer = ctx.alloc().buffer(32);
        byte[] bytes = msg.getBytes(StandardCharsets.UTF_8);
        buffer.writeBytes(bytes);
        if (bytes.length < 32)
        {
            //补齐剩下的字节
            for (int i = 0; i < 32 - bytes.length; i++)
            {
                buffer.writeByte(0);
            }
        }
        ctx.writeAndFlush(buffer);
    }
}
```





#### 运行结果

客户端

```sh
2023-03-24  21:41:01.855  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] WRITE: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 00 00 00 00 00 00 00 00 00 00 00 00 00 |123.............|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.855  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:41:01.856  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] WRITE: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.856  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:41:01.857  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] WRITE: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30 31 00 00 00 00 00 |12345678901.....|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.857  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:41:01.857  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] WRITE: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 6a 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |j...............|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.857  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:41:01.857  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] WRITE: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 64 73 61 67 73 61 67 61 73 64 68 61 65 72 68 73 |dsagsagasdhaerhs|
|00000010| 64 67 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |dg..............|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.857  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:41:01.857  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] WRITE: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2b 2b 2b 34 35 36 37 38 00 00 00 00 00 00 00 00 |+++45678........|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.858  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xd0250f69, L:/127.0.0.1:55869 - R:localhost/127.0.0.1:8080] FLUSH
```





服务端

```sh
2023-03-24  21:41:01.868  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869] READ: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 00 00 00 00 00 00 00 00 00 00 00 00 00 |123.............|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.868  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 32, widx: 192, cap: 1024))
2023-03-24  21:41:01.868  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 32, widx: 192, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:41:01.868  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [FixedLengthFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869].
2023-03-24  21:41:01.868  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869] READ: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00 |hello...........|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.868  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 64, widx: 192, cap: 1024))
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 64, widx: 192, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [FixedLengthFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869].
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869] READ: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30 31 00 00 00 00 00 |12345678901.....|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 96, widx: 192, cap: 1024))
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 96, widx: 192, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [FixedLengthFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869].
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869] READ: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 6a 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |j...............|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 128, widx: 192, cap: 1024))
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 128, widx: 192, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [FixedLengthFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869].
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869] READ: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 64 73 61 67 73 61 67 61 73 64 68 61 65 72 68 73 |dsagsagasdhaerhs|
|00000010| 64 67 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |dg..............|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 160, widx: 192, cap: 1024))
2023-03-24  21:41:01.869  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 160, widx: 192, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:41:01.870  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [FixedLengthFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869].
2023-03-24  21:41:01.870  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869] READ: 32B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2b 2b 2b 34 35 36 37 38 00 00 00 00 00 00 00 00 |+++45678........|
|00000010| 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 |................|
+--------+-------------------------------------------------+----------------+
2023-03-24  21:41:01.870  [nioEventLoopGroup-3-1] DEBUG mao.t2.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 192, widx: 192, cap: 1024))
2023-03-24  21:41:01.870  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 32, cap: 32/32, unwrapped: PooledUnsafeDirectByteBuf(ridx: 192, widx: 192, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:41:01.870  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [FixedLengthFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869].
2023-03-24  21:41:01.870  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x01a6a02e, L:/127.0.0.1:8080 - R:/127.0.0.1:55869] READ COMPLETE
```









### 固定分隔符

默认以 \n 或 \r\n 作为分隔符，如果超出指定长度仍未出现分隔符，则抛出异常

在服务端加入

```java
ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
```



缺点：处理字符数据比较合适，但如果内容本身包含了分隔符（字节数据常常会有此情况），那么就会解析错误



#### 服务端

```java
package mao.t3;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.FixedLengthFrameDecoder;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.GenericFutureListener;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

/**
 * Project name(项目名称)：Netty_黏包与半包现象解决
 * Package(包名): mao.t3
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/24
 * Time(创建时间)： 21:55
 * Version(版本): 1.0
 * Description(描述)： 固定分隔符方式解决黏包与半包问题，服务端
 */

@Slf4j
public class Server
{
    @SneakyThrows
    public static void main(String[] args)
    {
        NioEventLoopGroup boos = new NioEventLoopGroup();
        NioEventLoopGroup worker = new NioEventLoopGroup();
        Channel channel = new ServerBootstrap()
                .group(boos, worker)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
                        ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                        ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                        {
                            @Override
                            public void channelActive(ChannelHandlerContext ctx) throws Exception
                            {
                                log.debug("连接建立：" + ctx.channel());
                                super.channelActive(ctx);
                            }

                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
                            {
                                ByteBuf byteBuf = (ByteBuf) msg;
                                log.debug("读事件:" + byteBuf);
                                super.channelRead(ctx, msg);
                            }

                            @Override
                            public void channelInactive(ChannelHandlerContext ctx) throws Exception
                            {
                                log.debug("连接关闭：" + ctx.channel());
                                super.channelInactive(ctx);
                            }
                        });
                    }
                })
                .bind(8080)
                .sync().channel();
        channel.closeFuture().addListener(new GenericFutureListener<Future<? super Void>>()
        {
            @Override
            public void operationComplete(Future<? super Void> future) throws Exception
            {
                boos.shutdownGracefully();
                worker.shutdownGracefully();
            }
        });
    }
}
```





#### 客户端

```java
package mao.t3;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.extern.slf4j.Slf4j;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_黏包与半包现象解决
 * Package(包名): mao.t3
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/24
 * Time(创建时间)： 21:51
 * Version(版本): 1.0
 * Description(描述)： 固定分隔符方式解决黏包与半包问题，客户端
 */

@Slf4j
public class Client
{
    public static void main(String[] args)
    {
        NioEventLoopGroup worker = new NioEventLoopGroup();
        try
        {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.channel(NioSocketChannel.class);
            bootstrap.group(worker);
            bootstrap.handler(new ChannelInitializer<SocketChannel>()
            {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception
                {
                    ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                    ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                    {
                        @Override
                        public void channelActive(ChannelHandlerContext ctx) throws Exception
                        {
                            send(ctx, "123");
                            send(ctx, "hello");
                            send(ctx, "123456789012345678901234567890123456789012345678901234567890123456789" +
                                    "01234567890123456789012345678901234567890123456789012345678901234567890");
                            //中间带了\n
                            send(ctx, "321\n654");
                        }
                    });
                }
            });
            ChannelFuture channelFuture = bootstrap.connect("localhost", 8080).sync();
            channelFuture.channel().closeFuture().sync();

        }
        catch (InterruptedException e)
        {
            log.error("client error", e);
        }
        finally
        {
            worker.shutdownGracefully();
        }
    }

    /**
     * 发送消息,以\n结束
     *
     * @param ctx ctx
     * @param msg 消息
     */
    private static void send(ChannelHandlerContext ctx, String msg)
    {
        msg = msg + "\n";
        byte[] bytes = msg.getBytes(StandardCharsets.UTF_8);
        ByteBuf buffer = ctx.alloc().buffer(bytes.length);
        buffer.writeBytes(bytes);
        ctx.writeAndFlush(buffer);
    }
}
```





#### 运行结果

客户端

```sh
2023-03-24  21:56:18.959  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x62ec620f, L:/127.0.0.1:52577 - R:localhost/127.0.0.1:8080] WRITE: 4B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 0a                                     |123.            |
+--------+-------------------------------------------------+----------------+
2023-03-24  21:56:18.960  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x62ec620f, L:/127.0.0.1:52577 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:56:18.960  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x62ec620f, L:/127.0.0.1:52577 - R:localhost/127.0.0.1:8080] WRITE: 6B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 0a                               |hello.          |
+--------+-------------------------------------------------+----------------+
2023-03-24  21:56:18.961  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x62ec620f, L:/127.0.0.1:52577 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:56:18.961  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x62ec620f, L:/127.0.0.1:52577 - R:localhost/127.0.0.1:8080] WRITE: 141B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36 |1234567890123456|
|00000010| 37 38 39 30 31 32 33 34 35 36 37 38 39 30 31 32 |7890123456789012|
|00000020| 33 34 35 36 37 38 39 30 31 32 33 34 35 36 37 38 |3456789012345678|
|00000030| 39 30 31 32 33 34 35 36 37 38 39 30 31 32 33 34 |9012345678901234|
|00000040| 35 36 37 38 39 30 31 32 33 34 35 36 37 38 39 30 |5678901234567890|
|00000050| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36 |1234567890123456|
|00000060| 37 38 39 30 31 32 33 34 35 36 37 38 39 30 31 32 |7890123456789012|
|00000070| 33 34 35 36 37 38 39 30 31 32 33 34 35 36 37 38 |3456789012345678|
|00000080| 39 30 31 32 33 34 35 36 37 38 39 30 0a          |901234567890.   |
+--------+-------------------------------------------------+----------------+
2023-03-24  21:56:18.961  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x62ec620f, L:/127.0.0.1:52577 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  21:56:18.961  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x62ec620f, L:/127.0.0.1:52577 - R:localhost/127.0.0.1:8080] WRITE: 8B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 33 32 31 0a 36 35 34 0a                         |321.654.        |
+--------+-------------------------------------------------+----------------+
2023-03-24  21:56:18.961  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x62ec620f, L:/127.0.0.1:52577 - R:localhost/127.0.0.1:8080] FLUSH
```



服务端

```sh
2023-03-24  21:56:18.959  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577] REGISTERED
2023-03-24  21:56:18.959  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577] ACTIVE
2023-03-24  21:56:18.960  [nioEventLoopGroup-3-1] DEBUG mao.t3.Server:  连接建立：[id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577]
2023-03-24  21:56:18.962  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-24  21:56:18.962  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-24  21:56:18.962  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-24  21:56:18.962  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-24  21:56:18.965  [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-24  21:56:18.965  [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-24  21:56:18.966  [nioEventLoopGroup-3-1] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@7d181ee7
2023-03-24  21:56:18.972  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577] READ: 3B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33                                        |123             |
+--------+-------------------------------------------------+----------------+
2023-03-24  21:56:18.973  [nioEventLoopGroup-3-1] DEBUG mao.t3.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 3, cap: 3/3, unwrapped: PooledUnsafeDirectByteBuf(ridx: 4, widx: 159, cap: 1024))
2023-03-24  21:56:18.973  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 3, cap: 3/3, unwrapped: PooledUnsafeDirectByteBuf(ridx: 4, widx: 159, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:56:18.973  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577].
2023-03-24  21:56:18.973  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577] READ: 5B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f                                  |hello           |
+--------+-------------------------------------------------+----------------+
2023-03-24  21:56:18.973  [nioEventLoopGroup-3-1] DEBUG mao.t3.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 5, cap: 5/5, unwrapped: PooledUnsafeDirectByteBuf(ridx: 10, widx: 159, cap: 1024))
2023-03-24  21:56:18.973  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 5, cap: 5/5, unwrapped: PooledUnsafeDirectByteBuf(ridx: 10, widx: 159, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:56:18.973  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577].
2023-03-24  21:56:18.973  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577] READ: 140B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36 |1234567890123456|
|00000010| 37 38 39 30 31 32 33 34 35 36 37 38 39 30 31 32 |7890123456789012|
|00000020| 33 34 35 36 37 38 39 30 31 32 33 34 35 36 37 38 |3456789012345678|
|00000030| 39 30 31 32 33 34 35 36 37 38 39 30 31 32 33 34 |9012345678901234|
|00000040| 35 36 37 38 39 30 31 32 33 34 35 36 37 38 39 30 |5678901234567890|
|00000050| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36 |1234567890123456|
|00000060| 37 38 39 30 31 32 33 34 35 36 37 38 39 30 31 32 |7890123456789012|
|00000070| 33 34 35 36 37 38 39 30 31 32 33 34 35 36 37 38 |3456789012345678|
|00000080| 39 30 31 32 33 34 35 36 37 38 39 30             |901234567890    |
+--------+-------------------------------------------------+----------------+
2023-03-24  21:56:18.974  [nioEventLoopGroup-3-1] DEBUG mao.t3.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 140, cap: 140/140, unwrapped: PooledUnsafeDirectByteBuf(ridx: 151, widx: 159, cap: 1024))
2023-03-24  21:56:18.974  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 140, cap: 140/140, unwrapped: PooledUnsafeDirectByteBuf(ridx: 151, widx: 159, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:56:18.974  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577].
2023-03-24  21:56:18.974  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577] READ: 3B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 33 32 31                                        |321             |
+--------+-------------------------------------------------+----------------+
2023-03-24  21:56:18.974  [nioEventLoopGroup-3-1] DEBUG mao.t3.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 3, cap: 3/3, unwrapped: PooledUnsafeDirectByteBuf(ridx: 155, widx: 159, cap: 1024))
2023-03-24  21:56:18.974  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 3, cap: 3/3, unwrapped: PooledUnsafeDirectByteBuf(ridx: 155, widx: 159, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:56:18.974  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577].
2023-03-24  21:56:18.974  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577] READ: 3B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 36 35 34                                        |654             |
+--------+-------------------------------------------------+----------------+
2023-03-24  21:56:18.974  [nioEventLoopGroup-3-1] DEBUG mao.t3.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 3, cap: 3/3, unwrapped: PooledUnsafeDirectByteBuf(ridx: 159, widx: 159, cap: 1024))
2023-03-24  21:56:18.974  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 3, cap: 3/3, unwrapped: PooledUnsafeDirectByteBuf(ridx: 159, widx: 159, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  21:56:18.974  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577].
2023-03-24  21:56:18.975  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb3d2ff72, L:/127.0.0.1:8080 - R:/127.0.0.1:52577] READ COMPLETE
```







### 预设长度

在发送消息前，先约定用定长字节表示接下来数据的长度

```java
// 最大长度，长度偏移，长度占用字节，长度调整，剥离字节数
ch.pipeline().addLast(new LengthFieldBasedFrameDecoder(4096,0, 2,0, 2));
```



#### 服务端

```java
package mao.t4;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.LengthFieldBasedFrameDecoder;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.GenericFutureListener;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_黏包与半包现象解决
 * Package(包名): mao.t4
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/24
 * Time(创建时间)： 22:07
 * Version(版本): 1.0
 * Description(描述)： 预设长度方式解决黏包与半包问题，服务端
 */

@Slf4j
public class Server
{
    @SneakyThrows
    public static void main(String[] args)
    {
        NioEventLoopGroup boos = new NioEventLoopGroup();
        NioEventLoopGroup worker = new NioEventLoopGroup();
        Channel channel = new ServerBootstrap()
                .group(boos, worker)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        // 最大长度，长度偏移，长度占用字节，长度调整，剥离字节数
                        ch.pipeline().addLast(new LengthFieldBasedFrameDecoder(4096,
                                0, 2,
                                0, 2));
                        ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                        ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                        {
                            @Override
                            public void channelActive(ChannelHandlerContext ctx) throws Exception
                            {
                                log.debug("连接建立：" + ctx.channel());
                                super.channelActive(ctx);
                            }

                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
                            {
                                ByteBuf byteBuf = (ByteBuf) msg;
                                log.debug("读事件:" + byteBuf);
                                String s = byteBuf.toString(StandardCharsets.UTF_8);
                                log.debug(s);
                                super.channelRead(ctx, msg);
                            }

                            @Override
                            public void channelInactive(ChannelHandlerContext ctx) throws Exception
                            {
                                log.debug("连接关闭：" + ctx.channel());
                                super.channelInactive(ctx);
                            }
                        });
                    }
                })
                .bind(8080)
                .sync().channel();
        channel.closeFuture().addListener(new GenericFutureListener<Future<? super Void>>()
        {
            @Override
            public void operationComplete(Future<? super Void> future) throws Exception
            {
                boos.shutdownGracefully();
                worker.shutdownGracefully();
            }
        });
    }
}
```





#### 客户端

```java
package mao.t4;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.extern.slf4j.Slf4j;

import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_黏包与半包现象解决
 * Package(包名): mao.t4
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/24
 * Time(创建时间)： 22:06
 * Version(版本): 1.0
 * Description(描述)： 预设长度方式解决黏包与半包问题，客户端
 */

@Slf4j
public class Client
{
    public static void main(String[] args)
    {
        NioEventLoopGroup worker = new NioEventLoopGroup();
        try
        {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.channel(NioSocketChannel.class);
            bootstrap.group(worker);
            bootstrap.handler(new ChannelInitializer<SocketChannel>()
            {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception
                {
                    ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                    ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                    {
                        @Override
                        public void channelActive(ChannelHandlerContext ctx) throws Exception
                        {
                            send(ctx, "123");
                            send(ctx, "hello");
                            send(ctx, "123456789012345678901234567890123456789012345678901234567890123456789" +
                                    "01234567890123456789012345678901234567890123456789012345678901234567890");
                            //中间带了\n
                            send(ctx, "321\n654");
                            send(ctx,"您好，我是张三！");
                        }
                    });
                }
            });
            ChannelFuture channelFuture = bootstrap.connect("localhost", 8080).sync();
            channelFuture.channel().closeFuture().sync();

        }
        catch (InterruptedException e)
        {
            log.error("client error", e);
        }
        finally
        {
            worker.shutdownGracefully();
        }
    }

    /**
     * 发送消息,预设长度方式，规定前2字节是长度信息，也就是最大长度65535，规定最大长度4096
     *
     * @param ctx ctx
     * @param msg 消息
     */
    private static void send(ChannelHandlerContext ctx, String msg)
    {
        byte[] bytes = msg.getBytes(StandardCharsets.UTF_8);
        //得到长度
        int length = bytes.length;
        if (4096 - 2 - length < 0)
        {
            log.warn("消息长度太长，自动丢弃");
            return;
        }
        log.debug("消息的长度：" + length + ",消息内容：" + msg);
        ByteBuf buffer = ctx.alloc().buffer(length + 2);
        //先写入长度信息
        buffer.writeShort(length);
        //再写入消息
        buffer.writeBytes(bytes);
        ctx.writeAndFlush(buffer);
    }
}
```





#### 运行结果

客户端

```sh
2023-03-24  22:24:30.818  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732] REGISTERED
2023-03-24  22:24:30.818  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732] CONNECT: localhost/127.0.0.1:8080
2023-03-24  22:24:30.820  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732, L:/127.0.0.1:52844 - R:localhost/127.0.0.1:8080] ACTIVE
2023-03-24  22:24:30.821  [nioEventLoopGroup-2-1] DEBUG mao.t4.Client:  消息的长度：3,消息内容：123
2023-03-24  22:24:30.823  [nioEventLoopGroup-2-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-24  22:24:30.823  [nioEventLoopGroup-2-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-24  22:24:30.823  [nioEventLoopGroup-2-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-24  22:24:30.823  [nioEventLoopGroup-2-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-24  22:24:30.826  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-24  22:24:30.826  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-24  22:24:30.827  [nioEventLoopGroup-2-1] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@409b5926
2023-03-24  22:24:30.833  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732, L:/127.0.0.1:52844 - R:localhost/127.0.0.1:8080] WRITE: 5B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 03 31 32 33                                  |..123           |
+--------+-------------------------------------------------+----------------+
2023-03-24  22:24:30.833  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732, L:/127.0.0.1:52844 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  22:24:30.836  [nioEventLoopGroup-2-1] DEBUG mao.t4.Client:  消息的长度：5,消息内容：hello
2023-03-24  22:24:30.837  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732, L:/127.0.0.1:52844 - R:localhost/127.0.0.1:8080] WRITE: 7B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 05 68 65 6c 6c 6f                            |..hello         |
+--------+-------------------------------------------------+----------------+
2023-03-24  22:24:30.837  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732, L:/127.0.0.1:52844 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  22:24:30.837  [nioEventLoopGroup-2-1] DEBUG mao.t4.Client:  消息的长度：140,消息内容：12345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890
2023-03-24  22:24:30.837  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732, L:/127.0.0.1:52844 - R:localhost/127.0.0.1:8080] WRITE: 142B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 8c 31 32 33 34 35 36 37 38 39 30 31 32 33 34 |..12345678901234|
|00000010| 35 36 37 38 39 30 31 32 33 34 35 36 37 38 39 30 |5678901234567890|
|00000020| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36 |1234567890123456|
|00000030| 37 38 39 30 31 32 33 34 35 36 37 38 39 30 31 32 |7890123456789012|
|00000040| 33 34 35 36 37 38 39 30 31 32 33 34 35 36 37 38 |3456789012345678|
|00000050| 39 30 31 32 33 34 35 36 37 38 39 30 31 32 33 34 |9012345678901234|
|00000060| 35 36 37 38 39 30 31 32 33 34 35 36 37 38 39 30 |5678901234567890|
|00000070| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36 |1234567890123456|
|00000080| 37 38 39 30 31 32 33 34 35 36 37 38 39 30       |78901234567890  |
+--------+-------------------------------------------------+----------------+
2023-03-24  22:24:30.838  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732, L:/127.0.0.1:52844 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  22:24:30.838  [nioEventLoopGroup-2-1] DEBUG mao.t4.Client:  消息的长度：7,消息内容：321
654
2023-03-24  22:24:30.838  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732, L:/127.0.0.1:52844 - R:localhost/127.0.0.1:8080] WRITE: 9B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 07 33 32 31 0a 36 35 34                      |..321.654       |
+--------+-------------------------------------------------+----------------+
2023-03-24  22:24:30.838  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732, L:/127.0.0.1:52844 - R:localhost/127.0.0.1:8080] FLUSH
2023-03-24  22:24:30.838  [nioEventLoopGroup-2-1] DEBUG mao.t4.Client:  消息的长度：24,消息内容：您好，我是张三！
2023-03-24  22:24:30.838  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732, L:/127.0.0.1:52844 - R:localhost/127.0.0.1:8080] WRITE: 26B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 00 18 e6 82 a8 e5 a5 bd ef bc 8c e6 88 91 e6 98 |................|
|00000010| af e5 bc a0 e4 b8 89 ef bc 81                   |..........      |
+--------+-------------------------------------------------+----------------+
2023-03-24  22:24:30.838  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xbc2f4732, L:/127.0.0.1:52844 - R:localhost/127.0.0.1:8080] FLUSH
```



服务端

```sh
2023-03-24  22:24:30.832  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844] REGISTERED
2023-03-24  22:24:30.832  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844] ACTIVE
2023-03-24  22:24:30.833  [nioEventLoopGroup-3-1] DEBUG mao.t4.Server:  连接建立：[id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844]
2023-03-24  22:24:30.839  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-24  22:24:30.839  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-24  22:24:30.839  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-24  22:24:30.839  [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-24  22:24:30.846  [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-24  22:24:30.846  [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-24  22:24:30.847  [nioEventLoopGroup-3-1] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@5ceb6151
2023-03-24  22:24:30.857  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844] READ: 3B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33                                        |123             |
+--------+-------------------------------------------------+----------------+
2023-03-24  22:24:30.857  [nioEventLoopGroup-3-1] DEBUG mao.t4.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 3, cap: 3/3, unwrapped: PooledUnsafeDirectByteBuf(ridx: 5, widx: 189, cap: 1024))
2023-03-24  22:24:30.857  [nioEventLoopGroup-3-1] DEBUG mao.t4.Server:  123
2023-03-24  22:24:30.858  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 3, cap: 3/3, unwrapped: PooledUnsafeDirectByteBuf(ridx: 5, widx: 189, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  22:24:30.858  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LengthFieldBasedFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844].
2023-03-24  22:24:30.858  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844] READ: 5B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f                                  |hello           |
+--------+-------------------------------------------------+----------------+
2023-03-24  22:24:30.858  [nioEventLoopGroup-3-1] DEBUG mao.t4.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 5, cap: 5/5, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 189, cap: 1024))
2023-03-24  22:24:30.858  [nioEventLoopGroup-3-1] DEBUG mao.t4.Server:  hello
2023-03-24  22:24:30.858  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 5, cap: 5/5, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 189, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  22:24:30.858  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LengthFieldBasedFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844].
2023-03-24  22:24:30.859  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844] READ: 140B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36 |1234567890123456|
|00000010| 37 38 39 30 31 32 33 34 35 36 37 38 39 30 31 32 |7890123456789012|
|00000020| 33 34 35 36 37 38 39 30 31 32 33 34 35 36 37 38 |3456789012345678|
|00000030| 39 30 31 32 33 34 35 36 37 38 39 30 31 32 33 34 |9012345678901234|
|00000040| 35 36 37 38 39 30 31 32 33 34 35 36 37 38 39 30 |5678901234567890|
|00000050| 31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36 |1234567890123456|
|00000060| 37 38 39 30 31 32 33 34 35 36 37 38 39 30 31 32 |7890123456789012|
|00000070| 33 34 35 36 37 38 39 30 31 32 33 34 35 36 37 38 |3456789012345678|
|00000080| 39 30 31 32 33 34 35 36 37 38 39 30             |901234567890    |
+--------+-------------------------------------------------+----------------+
2023-03-24  22:24:30.859  [nioEventLoopGroup-3-1] DEBUG mao.t4.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 140, cap: 140/140, unwrapped: PooledUnsafeDirectByteBuf(ridx: 154, widx: 189, cap: 1024))
2023-03-24  22:24:30.859  [nioEventLoopGroup-3-1] DEBUG mao.t4.Server:  12345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890
2023-03-24  22:24:30.859  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 140, cap: 140/140, unwrapped: PooledUnsafeDirectByteBuf(ridx: 154, widx: 189, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  22:24:30.859  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LengthFieldBasedFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844].
2023-03-24  22:24:30.859  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844] READ: 7B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 33 32 31 0a 36 35 34                            |321.654         |
+--------+-------------------------------------------------+----------------+
2023-03-24  22:24:30.859  [nioEventLoopGroup-3-1] DEBUG mao.t4.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 7, cap: 7/7, unwrapped: PooledUnsafeDirectByteBuf(ridx: 163, widx: 189, cap: 1024))
2023-03-24  22:24:30.860  [nioEventLoopGroup-3-1] DEBUG mao.t4.Server:  321
654
2023-03-24  22:24:30.860  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 7, cap: 7/7, unwrapped: PooledUnsafeDirectByteBuf(ridx: 163, widx: 189, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  22:24:30.860  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LengthFieldBasedFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844].
2023-03-24  22:24:30.860  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844] READ: 24B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| e6 82 a8 e5 a5 bd ef bc 8c e6 88 91 e6 98 af e5 |................|
|00000010| bc a0 e4 b8 89 ef bc 81                         |........        |
+--------+-------------------------------------------------+----------------+
2023-03-24  22:24:30.860  [nioEventLoopGroup-3-1] DEBUG mao.t4.Server:  读事件:PooledSlicedByteBuf(ridx: 0, widx: 24, cap: 24/24, unwrapped: PooledUnsafeDirectByteBuf(ridx: 189, widx: 189, cap: 1024))
2023-03-24  22:24:30.860  [nioEventLoopGroup-3-1] DEBUG mao.t4.Server:  您好，我是张三！
2023-03-24  22:24:30.860  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 24, cap: 24/24, unwrapped: PooledUnsafeDirectByteBuf(ridx: 189, widx: 189, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-24  22:24:30.861  [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LengthFieldBasedFrameDecoder#0, LoggingHandler#0, Server$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844].
2023-03-24  22:24:30.861  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x706d6e8d, L:/127.0.0.1:8080 - R:/127.0.0.1:52844] READ COMPLETE
```















# 协议设计与解析

## 为什么需要协议？

TCP/IP 中消息传输基于流的方式，没有边界。

协议的目的就是划定消息的边界，制定通信双方要共同遵守的通信规则





## redis协议

### RESP协议

Redis是一个CS架构的软件，通信一般分两步（不包括pipeline和PubSub）

* 客户端（client）向服务端（server）发送一条命令
* 服务端解析并执行命令，返回响应结果给客户端

因此客户端发送命令的格式、服务端响应结果的格式必须有一个规范，这个规范就是通信协议。



在Redis中采用的是**RESP**（Redis Serialization Protocol）协议



* lRedis 1.2版本引入了RESP协议
* lRedis 2.0版本中成为与Redis服务端通信的标准，称为RESP2
* lRedis 6.0版本中，从RESP2升级到了RESP3协议，增加了更多数据类型并且支持6.0的新特性--客户端缓存



lRedis 2.0版本：

在RESP中，通过首字节的字符来区分不同数据类型，常用的数据类型包括5种

* 单行字符串：首字节是 ‘**+**’ ，后面跟上单行字符串，以CRLF（ "**\r\n**" ）结尾。例如返回"OK"： "+OK\r\n"

* 错误（Errors）：首字节是 ‘**-**’ ，与单行字符串格式一样，只是字符串是异常信息，例如："-Error message\r\n"

* 数值：首字节是 ‘**:**’ ，后面跟上数字格式的字符串，以CRLF结尾。例如：":10\r\n"

* 多行字符串：首字节是 ‘**$**’ ，表示二进制安全的字符串，最大支持512MB。例如：$5\r\nhello***\r\n

  * $5的数字5：字符串占用字节大小
  * hello：真正的字符串数据
  * 如果大小为0，则代表空字符串："$0\r\n\r\n"
  * u如果大小为-1，则代表不存在："$-1\r\n"

* 数组：首字节是 ‘*’，后面跟上数组元素个数，再跟上元素，元素数据类型不限

  ```sh
  *3\r\n
  :10\r\n
  $5\r\nhello\r\n
  ```







### 通信客户端

```java
package mao.t1;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.GenericFutureListener;
import lombok.extern.slf4j.Slf4j;

import java.net.InetSocketAddress;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

/**
 * Project name(项目名称)：Netty_Redis协议通信
 * Package(包名): mao.t1
 * Class(类名): RedisClient
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/25
 * Time(创建时间)： 20:14
 * Version(版本): 1.0
 * Description(描述)： resp2.0协议通信
 */

@Slf4j
public class RedisClient
{
    public static void main(String[] args)
    {
        NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
        ChannelFuture channelFuture = new Bootstrap()
                .group(nioEventLoopGroup)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                        ch.pipeline().addLast(new ChannelInboundHandlerAdapter()
                        {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
                            {
                                ByteBuf byteBuf = (ByteBuf) msg;
                                String s = byteBuf.toString(StandardCharsets.UTF_8);
                                log.info("redis响应结果：" + s);
                                super.channelRead(ctx, msg);
                            }

                            @Override
                            public void channelActive(ChannelHandlerContext ctx) throws Exception
                            {
                                super.channelActive(ctx);
                            }

                            @Override
                            public void channelInactive(ChannelHandlerContext ctx) throws Exception
                            {
                                ctx.close();
                            }
                        });
                    }
                })
                .connect(new InetSocketAddress("127.0.0.1", 6379));
        Channel channel = channelFuture.channel();

        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                Scanner input = new Scanner(System.in);
                while (true)
                {
                    String cmd = input.nextLine();
                    if ("q".equals(cmd))
                    {
                        channel.close();
                        return;
                    }
                    log.debug("命令：" + cmd);
                    write(channel, cmd);
                }
            }
        }, "input");

        channelFuture.addListener(new GenericFutureListener<Future<? super Void>>()
        {
            /**
             * 操作完成(启动完成)
             *
             * @param future future
             * @throws Exception 异常
             */
            @Override
            public void operationComplete(Future<? super Void> future) throws Exception
            {
                if (future.isSuccess())
                {
                    log.info("客户端启动完成");
                    thread.start();
                }
                else
                {
                    log.warn("客户端启动失败：" + future.cause().getMessage());
                }
            }
        });
        channel.closeFuture().addListener(new GenericFutureListener<Future<? super Void>>()
        {
            /**
             * 操作完成(关闭)
             *
             * @param future future
             * @throws Exception 异常
             */
            @Override
            public void operationComplete(Future<? super Void> future) throws Exception
            {
                log.info("关闭客户端");
                nioEventLoopGroup.shutdownGracefully();
            }
        });
    }

    /**
     * 向redis发送命令
     *
     * @param channel Channel
     * @param cmd     命令字符串，中间用空格分开，比如： "set name 张三"
     */
    private static void write(Channel channel, String cmd)
    {
        //判断是否为空
        if (cmd == null)
        {
            return;
        }
        //按空格分割
        String[] split = cmd.split(" ");
        //得到数组长度
        int length = split.length;
        //开辟ByteBuf
        ByteBuf buffer = channel.alloc().buffer();
        buffer.writeBytes(("*" + length).getBytes(StandardCharsets.UTF_8));
        buffer.writeBytes("\r\n".getBytes(StandardCharsets.UTF_8));
        //循环
        for (String s : split)
        {
            buffer.writeBytes(("$" + s.length()).getBytes(StandardCharsets.UTF_8));
            buffer.writeBytes("\r\n".getBytes(StandardCharsets.UTF_8));
            buffer.writeBytes(s.getBytes(StandardCharsets.UTF_8));
            buffer.writeBytes("\r\n".getBytes(StandardCharsets.UTF_8));
        }
        //发送
        channel.writeAndFlush(buffer);
    }
}
```



运行结果：

```sh
2023-03-25  20:48:26.582  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618] REGISTERED
2023-03-25  20:48:26.582  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618] CONNECT: /127.0.0.1:6379
2023-03-25  20:48:26.585  [nioEventLoopGroup-2-1] INFO  mao.t1.RedisClient:  客户端启动完成
2023-03-25  20:48:26.585  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] ACTIVE
get name
2023-03-25  20:48:39.203  [input] DEBUG mao.t1.RedisClient:  命令：get name
2023-03-25  20:48:39.204  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-25  20:48:39.205  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-25  20:48:39.205  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-25  20:48:39.205  [input] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-25  20:48:39.207  [input] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-25  20:48:39.207  [input] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-25  20:48:39.208  [input] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@794284ee
2023-03-25  20:48:39.214  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] WRITE: 23B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2a 32 0d 0a 24 33 0d 0a 67 65 74 0d 0a 24 34 0d |*2..$3..get..$4.|
|00000010| 0a 6e 61 6d 65 0d 0a                            |.name..         |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:48:39.215  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] FLUSH
2023-03-25  20:48:39.219  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ: 34B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2d 4e 4f 41 55 54 48 20 41 75 74 68 65 6e 74 69 |-NOAUTH Authenti|
|00000010| 63 61 74 69 6f 6e 20 72 65 71 75 69 72 65 64 2e |cation required.|
|00000020| 0d 0a                                           |..              |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:48:39.220  [nioEventLoopGroup-2-1] INFO  mao.t1.RedisClient:  redis响应结果：-NOAUTH Authentication required.

2023-03-25  20:48:39.220  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 34, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-25  20:48:39.220  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, RedisClient$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379].
2023-03-25  20:48:39.220  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ COMPLETE
auth 123
2023-03-25  20:48:52.341  [input] DEBUG mao.t1.RedisClient:  命令：auth 123
2023-03-25  20:48:52.342  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] WRITE: 23B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2a 32 0d 0a 24 34 0d 0a 61 75 74 68 0d 0a 24 33 |*2..$4..auth..$3|
|00000010| 0d 0a 31 32 33 0d 0a                            |..123..         |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:48:52.342  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] FLUSH
2023-03-25  20:48:52.342  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ: 23B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2d 45 52 52 20 69 6e 76 61 6c 69 64 20 70 61 73 |-ERR invalid pas|
|00000010| 73 77 6f 72 64 0d 0a                            |sword..         |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:48:52.342  [nioEventLoopGroup-2-1] INFO  mao.t1.RedisClient:  redis响应结果：-ERR invalid password

2023-03-25  20:48:52.342  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 23, cap: 1024) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-25  20:48:52.342  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, RedisClient$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379].
2023-03-25  20:48:52.342  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ COMPLETE
auth 123456
2023-03-25  20:48:58.615  [input] DEBUG mao.t1.RedisClient:  命令：auth 123456
2023-03-25  20:48:58.615  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] WRITE: 26B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2a 32 0d 0a 24 34 0d 0a 61 75 74 68 0d 0a 24 36 |*2..$4..auth..$6|
|00000010| 0d 0a 31 32 33 34 35 36 0d 0a                   |..123456..      |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:48:58.615  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] FLUSH
2023-03-25  20:48:58.616  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ: 5B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2b 4f 4b 0d 0a                                  |+OK..           |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:48:58.616  [nioEventLoopGroup-2-1] INFO  mao.t1.RedisClient:  redis响应结果：+OK

2023-03-25  20:48:58.616  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 5, cap: 512) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-25  20:48:58.616  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, RedisClient$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379].
2023-03-25  20:48:58.616  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ COMPLETE
keys map*
2023-03-25  20:49:08.410  [input] DEBUG mao.t1.RedisClient:  命令：keys map*
2023-03-25  20:49:08.411  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] WRITE: 24B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2a 32 0d 0a 24 34 0d 0a 6b 65 79 73 0d 0a 24 34 |*2..$4..keys..$4|
|00000010| 0d 0a 6d 61 70 2a 0d 0a                         |..map*..        |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:49:08.411  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] FLUSH
2023-03-25  20:49:08.411  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ: 36B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2a 33 0d 0a 24 35 0d 0a 6d 61 70 31 33 0d 0a 24 |*3..$5..map13..$|
|00000010| 34 0d 0a 6d 61 70 31 0d 0a 24 35 0d 0a 6d 61 70 |4..map1..$5..map|
|00000020| 31 34 0d 0a                                     |14..            |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:49:08.412  [nioEventLoopGroup-2-1] INFO  mao.t1.RedisClient:  redis响应结果：*3
$5
map13
$4
map1
$5
map14

2023-03-25  20:49:08.412  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 36, cap: 512) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-25  20:49:08.412  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, RedisClient$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379].
2023-03-25  20:49:08.412  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ COMPLETE
get key1
2023-03-25  20:49:32.529  [input] DEBUG mao.t1.RedisClient:  命令：get key1
2023-03-25  20:49:32.530  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] WRITE: 23B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2a 32 0d 0a 24 33 0d 0a 67 65 74 0d 0a 24 34 0d |*2..$3..get..$4.|
|00000010| 0a 6b 65 79 31 0d 0a                            |.key1..         |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:49:32.530  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] FLUSH
2023-03-25  20:49:32.530  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ: 12B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 24 36 0d 0a e4 bd a0 e5 a5 bd 0d 0a             |$6..........    |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:49:32.530  [nioEventLoopGroup-2-1] INFO  mao.t1.RedisClient:  redis响应结果：$6
你好

2023-03-25  20:49:32.530  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 12, cap: 496) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-25  20:49:32.530  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, RedisClient$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379].
2023-03-25  20:49:32.530  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ COMPLETE
get name
2023-03-25  20:49:55.535  [input] DEBUG mao.t1.RedisClient:  命令：get name
2023-03-25  20:49:55.535  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] WRITE: 23B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2a 32 0d 0a 24 33 0d 0a 67 65 74 0d 0a 24 34 0d |*2..$3..get..$4.|
|00000010| 0a 6e 61 6d 65 0d 0a                            |.name..         |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:49:55.536  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] FLUSH
2023-03-25  20:49:55.536  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ: 5B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 24 2d 31 0d 0a                                  |$-1..           |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:49:55.536  [nioEventLoopGroup-2-1] INFO  mao.t1.RedisClient:  redis响应结果：$-1

2023-03-25  20:49:55.536  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 5, cap: 496) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-25  20:49:55.536  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, RedisClient$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379].
2023-03-25  20:49:55.536  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ COMPLETE
set name lisi
2023-03-25  20:50:14.100  [input] DEBUG mao.t1.RedisClient:  命令：set name lisi
2023-03-25  20:50:14.101  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] WRITE: 33B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2a 33 0d 0a 24 33 0d 0a 73 65 74 0d 0a 24 34 0d |*3..$3..set..$4.|
|00000010| 0a 6e 61 6d 65 0d 0a 24 34 0d 0a 6c 69 73 69 0d |.name..$4..lisi.|
|00000020| 0a                                              |.               |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:50:14.101  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] FLUSH
2023-03-25  20:50:14.101  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ: 5B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2b 4f 4b 0d 0a                                  |+OK..           |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:50:14.101  [nioEventLoopGroup-2-1] INFO  mao.t1.RedisClient:  redis响应结果：+OK

2023-03-25  20:50:14.101  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 5, cap: 480) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-25  20:50:14.101  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, RedisClient$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379].
2023-03-25  20:50:14.101  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ COMPLETE
get name
2023-03-25  20:50:19.587  [input] DEBUG mao.t1.RedisClient:  命令：get name
2023-03-25  20:50:19.588  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] WRITE: 23B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 2a 32 0d 0a 24 33 0d 0a 67 65 74 0d 0a 24 34 0d |*2..$3..get..$4.|
|00000010| 0a 6e 61 6d 65 0d 0a                            |.name..         |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:50:19.588  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] FLUSH
2023-03-25  20:50:19.589  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ: 10B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 24 34 0d 0a 6c 69 73 69 0d 0a                   |$4..lisi..      |
+--------+-------------------------------------------------+----------------+
2023-03-25  20:50:19.589  [nioEventLoopGroup-2-1] INFO  mao.t1.RedisClient:  redis响应结果：$4
lisi

2023-03-25  20:50:19.589  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded inbound message PooledUnsafeDirectByteBuf(ridx: 0, widx: 10, cap: 480) that reached at the tail of the pipeline. Please check your pipeline configuration.
2023-03-25  20:50:19.589  [nioEventLoopGroup-2-1] DEBUG io.netty.channel.DefaultChannelPipeline:  Discarded message pipeline : [LoggingHandler#0, RedisClient$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379].
2023-03-25  20:50:19.589  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] READ COMPLETE
q
2023-03-25  20:50:27.316  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 - R:/127.0.0.1:6379] CLOSE
2023-03-25  20:50:27.316  [nioEventLoopGroup-2-1] INFO  mao.t1.RedisClient:  关闭客户端
2023-03-25  20:50:27.319  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 ! R:/127.0.0.1:6379] INACTIVE
2023-03-25  20:50:27.319  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 ! R:/127.0.0.1:6379] CLOSE
2023-03-25  20:50:27.319  [nioEventLoopGroup-2-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0xb844e618, L:/127.0.0.1:52290 ! R:/127.0.0.1:6379] UNREGISTERED
2023-03-25  20:50:29.567  [nioEventLoopGroup-2-1] DEBUG io.netty.buffer.PoolThreadCache:  Freed 4 thread-local buffer(s) from thread: nioEventLoopGroup-2-1
```











## http协议

### 服务端

```java
package mao.t1;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.http.*;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.GenericFutureListener;
import lombok.extern.slf4j.Slf4j;

import java.nio.charset.StandardCharsets;
import java.util.Map;
import java.util.function.Consumer;

import static io.netty.handler.codec.http.HttpHeaderNames.CONTENT_LENGTH;

/**
 * Project name(项目名称)：Netty_HTTP协议通信
 * Package(包名): mao.t1
 * Class(类名): Server
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/25
 * Time(创建时间)： 21:55
 * Version(版本): 1.0
 * Description(描述)： http协议
 */

@Slf4j
public class Server
{
    public static void main(String[] args)
    {
        NioEventLoopGroup boss = new NioEventLoopGroup(2);
        NioEventLoopGroup worker = new NioEventLoopGroup();
        ChannelFuture channelFuture = new ServerBootstrap()
                .group(boss, worker)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>()
                {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception
                    {
                        ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG))
                                .addLast(new HttpServerCodec())
                                .addLast(new SimpleChannelInboundHandler<HttpRequest>()
                                {
                                    /**
                                     * 处理请求行和请求头
                                     *
                                     * @param ctx         ctx
                                     * @param httpRequest http请求
                                     * @throws Exception 异常
                                     */
                                    @Override
                                    protected void channelRead0(ChannelHandlerContext ctx, HttpRequest httpRequest)
                                            throws Exception
                                    {
                                        log.debug("请求的uri：" + httpRequest.uri());
                                        //请求头
                                        HttpHeaders headers = httpRequest.headers();
                                        headers.forEach(new Consumer<Map.Entry<String, String>>()
                                        {
                                            @Override
                                            public void accept(Map.Entry<String, String> stringStringEntry)
                                            {
                                                String key = stringStringEntry.getKey();
                                                String value = stringStringEntry.getValue();
                                                log.debug(key + " ---> " + value);
                                            }
                                        });
                                        DefaultFullHttpResponse httpResponse = new
                                                DefaultFullHttpResponse(HttpVersion.HTTP_1_1,
                                                HttpResponseStatus.OK);
                                        String respMsg = "<h1>Hello, world!</h1>";
                                        byte[] bytes = respMsg.getBytes(StandardCharsets.UTF_8);
                                        httpResponse.headers().setInt(CONTENT_LENGTH, bytes.length);
                                        httpResponse.headers().set("content-type", "text/html;charset=utf-8");
                                        httpResponse.content().writeBytes(bytes);
                                        //写回响应
                                        ctx.writeAndFlush(httpResponse);
                                    }
                                })
                                .addLast(new SimpleChannelInboundHandler<HttpContent>()
                                {
                                    /**
                                     * 处理请求体
                                     *
                                     * @param ctx ctx
                                     * @param httpContent HttpContent
                                     * @throws Exception 异常
                                     */
                                    @Override
                                    protected void channelRead0(ChannelHandlerContext ctx, HttpContent httpContent)
                                            throws Exception
                                    {
                                        ByteBuf byteBuf = httpContent.content();
                                        String s = byteBuf.toString(StandardCharsets.UTF_8);
                                        log.debug("请求体：" + s);
                                    }
                                });
                    }
                }).bind(8080);
        Channel channel = channelFuture.channel();

        channelFuture.addListener(new GenericFutureListener<Future<? super Void>>()
        {
            @Override
            public void operationComplete(Future<? super Void> future) throws Exception
            {
                if (future.isSuccess())
                {
                    log.info("服务启动完成");
                }
                else
                {
                    log.warn("服务启动失败：" + future.cause().getMessage());
                }
            }
        });

        channel.closeFuture().addListener(new GenericFutureListener<Future<? super Void>>()
        {
            @Override
            public void operationComplete(Future<? super Void> future) throws Exception
            {
                log.info("关闭客户端");
                boss.shutdownGracefully();
                worker.shutdownGracefully();
            }
        });
    }
}

```





### 客户端

```java
package mao.t1;

import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.io.*;
import java.net.URL;
import java.net.URLConnection;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

/**
 * Project name(项目名称)：Netty_HTTP协议通信
 * Package(包名): mao.t1
 * Class(类名): Client
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/25
 * Time(创建时间)： 21:55
 * Version(版本): 1.0
 * Description(描述)： 无
 */

@Slf4j
public class Client
{
    @SneakyThrows
    public static void main(String[] args)
    {
        Scanner input = new Scanner(System.in);
        while (true)
        {
            String requestBody = input.nextLine();
            if ("q".equals(requestBody))
            {
                return;
            }
            log.debug("请求体：" + requestBody);
            String resp = httpGet("http://localhost:8080/index.html", requestBody);
            log.info("响应内容：" + resp);
        }
    }

    /**
     * http get请求
     *
     * @param urlString   url字符串
     * @param requestBody 请求体
     * @return {@link String}
     */
    private static String httpGet(String urlString, String requestBody) throws IOException
    {
        BufferedReader bufferedReader = null;
        InputStreamReader inputStreamReader = null;
        InputStream inputStream = null;
        OutputStream outputStream = null;
        try
        {
            URL url = new URL(urlString);
            URLConnection urlConnection = url.openConnection();
            urlConnection.setDoOutput(true);
            //连接
            urlConnection.connect();
            outputStream = urlConnection.getOutputStream();
            //写请求体
            outputStream.write(requestBody.getBytes(StandardCharsets.UTF_8));
            inputStream = urlConnection.getInputStream();
            inputStreamReader = new InputStreamReader(inputStream, StandardCharsets.UTF_8);
            bufferedReader = new BufferedReader(inputStreamReader);
            StringBuilder stringBuilder = new StringBuilder();
            String resp;
            while ((resp = bufferedReader.readLine()) != null)
            {
                stringBuilder.append(resp);
            }
            return stringBuilder.toString();
        }
        finally
        {
            if (bufferedReader != null)
            {
                bufferedReader.close();
            }
            if (inputStreamReader != null)
            {
                inputStreamReader.close();
            }
            if (inputStream != null)
            {
                inputStream.close();
            }
            if (outputStream != null)
            {
                outputStream.close();
            }
        }
    }
}

```





### 运行结果

浏览器访问

```sh
2023-03-25  21:43:03.421  [nioEventLoopGroup-2-1] INFO  mao.t2.Server:  服务启动完成
2023-03-25  21:43:08.374  [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkAccessible: true
2023-03-25  21:43:08.374  [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.AbstractByteBuf:  -Dio.netty.buffer.checkBounds: true
2023-03-25  21:43:08.375  [nioEventLoopGroup-3-1] DEBUG io.netty.util.ResourceLeakDetectorFactory:  Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@6cf2b2c2
2023-03-25  21:43:08.380  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x7d5b7c34, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65438] REGISTERED
2023-03-25  21:43:08.380  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x3598ea6e, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65439] REGISTERED
2023-03-25  21:43:08.381  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x3598ea6e, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65439] ACTIVE
2023-03-25  21:43:08.381  [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x7d5b7c34, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65438] ACTIVE
2023-03-25  21:43:08.382  [nioEventLoopGroup-3-2] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxCapacityPerThread: 4096
2023-03-25  21:43:08.382  [nioEventLoopGroup-3-2] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.maxSharedCapacityFactor: 2
2023-03-25  21:43:08.382  [nioEventLoopGroup-3-2] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.linkCapacity: 16
2023-03-25  21:43:08.382  [nioEventLoopGroup-3-2] DEBUG io.netty.util.Recycler:  -Dio.netty.recycler.ratio: 8
2023-03-25  21:43:08.388  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x3598ea6e, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65439] READ: 846B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 47 45 54 20 2f 69 6e 64 65 78 2e 68 74 6d 6c 20 |GET /index.html |
|00000010| 48 54 54 50 2f 31 2e 31 0d 0a 48 6f 73 74 3a 20 |HTTP/1.1..Host: |
|00000020| 6c 6f 63 61 6c 68 6f 73 74 3a 38 30 38 30 0d 0a |localhost:8080..|
|00000030| 43 6f 6e 6e 65 63 74 69 6f 6e 3a 20 6b 65 65 70 |Connection: keep|
|00000040| 2d 61 6c 69 76 65 0d 0a 43 61 63 68 65 2d 43 6f |-alive..Cache-Co|
|00000050| 6e 74 72 6f 6c 3a 20 6d 61 78 2d 61 67 65 3d 30 |ntrol: max-age=0|
|00000060| 0d 0a 73 65 63 2d 63 68 2d 75 61 3a 20 22 4d 69 |..sec-ch-ua: "Mi|
|00000070| 63 72 6f 73 6f 66 74 20 45 64 67 65 22 3b 76 3d |crosoft Edge";v=|
|00000080| 22 31 31 31 22 2c 20 22 4e 6f 74 28 41 3a 42 72 |"111", "Not(A:Br|
|00000090| 61 6e 64 22 3b 76 3d 22 38 22 2c 20 22 43 68 72 |and";v="8", "Chr|
|000000a0| 6f 6d 69 75 6d 22 3b 76 3d 22 31 31 31 22 0d 0a |omium";v="111"..|
|000000b0| 73 65 63 2d 63 68 2d 75 61 2d 6d 6f 62 69 6c 65 |sec-ch-ua-mobile|
|000000c0| 3a 20 3f 30 0d 0a 73 65 63 2d 63 68 2d 75 61 2d |: ?0..sec-ch-ua-|
|000000d0| 70 6c 61 74 66 6f 72 6d 3a 20 22 57 69 6e 64 6f |platform: "Windo|
|000000e0| 77 73 22 0d 0a 55 70 67 72 61 64 65 2d 49 6e 73 |ws"..Upgrade-Ins|
|000000f0| 65 63 75 72 65 2d 52 65 71 75 65 73 74 73 3a 20 |ecure-Requests: |
|00000100| 31 0d 0a 55 73 65 72 2d 41 67 65 6e 74 3a 20 4d |1..User-Agent: M|
|00000110| 6f 7a 69 6c 6c 61 2f 35 2e 30 20 28 57 69 6e 64 |ozilla/5.0 (Wind|
|00000120| 6f 77 73 20 4e 54 20 31 30 2e 30 3b 20 57 69 6e |ows NT 10.0; Win|
|00000130| 36 34 3b 20 78 36 34 29 20 41 70 70 6c 65 57 65 |64; x64) AppleWe|
|00000140| 62 4b 69 74 2f 35 33 37 2e 33 36 20 28 4b 48 54 |bKit/537.36 (KHT|
|00000150| 4d 4c 2c 20 6c 69 6b 65 20 47 65 63 6b 6f 29 20 |ML, like Gecko) |
|00000160| 43 68 72 6f 6d 65 2f 31 31 31 2e 30 2e 30 2e 30 |Chrome/111.0.0.0|
|00000170| 20 53 61 66 61 72 69 2f 35 33 37 2e 33 36 20 45 | Safari/537.36 E|
|00000180| 64 67 2f 31 31 31 2e 30 2e 31 36 36 31 2e 35 31 |dg/111.0.1661.51|
|00000190| 0d 0a 41 63 63 65 70 74 3a 20 74 65 78 74 2f 68 |..Accept: text/h|
|000001a0| 74 6d 6c 2c 61 70 70 6c 69 63 61 74 69 6f 6e 2f |tml,application/|
|000001b0| 78 68 74 6d 6c 2b 78 6d 6c 2c 61 70 70 6c 69 63 |xhtml+xml,applic|
|000001c0| 61 74 69 6f 6e 2f 78 6d 6c 3b 71 3d 30 2e 39 2c |ation/xml;q=0.9,|
|000001d0| 69 6d 61 67 65 2f 77 65 62 70 2c 69 6d 61 67 65 |image/webp,image|
|000001e0| 2f 61 70 6e 67 2c 2a 2f 2a 3b 71 3d 30 2e 38 2c |/apng,*/*;q=0.8,|
|000001f0| 61 70 70 6c 69 63 61 74 69 6f 6e 2f 73 69 67 6e |application/sign|
|00000200| 65 64 2d 65 78 63 68 61 6e 67 65 3b 76 3d 62 33 |ed-exchange;v=b3|
|00000210| 3b 71 3d 30 2e 37 0d 0a 53 65 63 2d 46 65 74 63 |;q=0.7..Sec-Fetc|
|00000220| 68 2d 53 69 74 65 3a 20 6e 6f 6e 65 0d 0a 53 65 |h-Site: none..Se|
|00000230| 63 2d 46 65 74 63 68 2d 4d 6f 64 65 3a 20 6e 61 |c-Fetch-Mode: na|
|00000240| 76 69 67 61 74 65 0d 0a 53 65 63 2d 46 65 74 63 |vigate..Sec-Fetc|
|00000250| 68 2d 55 73 65 72 3a 20 3f 31 0d 0a 53 65 63 2d |h-User: ?1..Sec-|
|00000260| 46 65 74 63 68 2d 44 65 73 74 3a 20 64 6f 63 75 |Fetch-Dest: docu|
|00000270| 6d 65 6e 74 0d 0a 41 63 63 65 70 74 2d 45 6e 63 |ment..Accept-Enc|
|00000280| 6f 64 69 6e 67 3a 20 67 7a 69 70 2c 20 64 65 66 |oding: gzip, def|
|00000290| 6c 61 74 65 2c 20 62 72 0d 0a 41 63 63 65 70 74 |late, br..Accept|
|000002a0| 2d 4c 61 6e 67 75 61 67 65 3a 20 7a 68 2d 43 4e |-Language: zh-CN|
|000002b0| 2c 7a 68 3b 71 3d 30 2e 39 2c 65 6e 3b 71 3d 30 |,zh;q=0.9,en;q=0|
|000002c0| 2e 38 2c 65 6e 2d 47 42 3b 71 3d 30 2e 37 2c 65 |.8,en-GB;q=0.7,e|
|000002d0| 6e 2d 55 53 3b 71 3d 30 2e 36 0d 0a 43 6f 6f 6b |n-US;q=0.6..Cook|
|000002e0| 69 65 3a 20 49 64 65 61 2d 32 33 34 37 65 36 38 |ie: Idea-2347e68|
|000002f0| 33 3d 65 37 63 31 39 34 64 36 2d 63 65 66 31 2d |3=e7c194d6-cef1-|
|00000300| 34 61 30 36 2d 38 32 31 35 2d 30 33 39 36 34 34 |4a06-8215-039644|
|00000310| 65 34 39 66 30 31 3b 20 48 6d 5f 6c 76 74 5f 38 |e49f01; Hm_lvt_8|
|00000320| 61 63 65 66 36 36 39 65 61 36 36 66 34 37 39 38 |acef669ea66f4798|
|00000330| 35 34 65 63 64 33 32 38 64 31 66 33 34 38 66 3d |54ecd328d1f348f=|
|00000340| 31 36 37 39 35 35 35 34 34 36 0d 0a 0d 0a       |1679555446....  |
+--------+-------------------------------------------------+----------------+
2023-03-25  21:43:08.395  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  请求的uri：/index.html
2023-03-25  21:43:08.400  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Host ---> localhost:8080
2023-03-25  21:43:08.400  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Connection ---> keep-alive
2023-03-25  21:43:08.400  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Cache-Control ---> max-age=0
2023-03-25  21:43:08.400  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  sec-ch-ua ---> "Microsoft Edge";v="111", "Not(A:Brand";v="8", "Chromium";v="111"
2023-03-25  21:43:08.400  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  sec-ch-ua-mobile ---> ?0
2023-03-25  21:43:08.400  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  sec-ch-ua-platform ---> "Windows"
2023-03-25  21:43:08.400  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Upgrade-Insecure-Requests ---> 1
2023-03-25  21:43:08.400  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  User-Agent ---> Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36 Edg/111.0.1661.51
2023-03-25  21:43:08.400  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Accept ---> text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
2023-03-25  21:43:08.400  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Sec-Fetch-Site ---> none
2023-03-25  21:43:08.401  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Sec-Fetch-Mode ---> navigate
2023-03-25  21:43:08.401  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Sec-Fetch-User ---> ?1
2023-03-25  21:43:08.401  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Sec-Fetch-Dest ---> document
2023-03-25  21:43:08.401  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Accept-Encoding ---> gzip, deflate, br
2023-03-25  21:43:08.401  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Accept-Language ---> zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
2023-03-25  21:43:08.401  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Cookie ---> Idea-2347e683=e7c194d6-cef1-4a06-8215-039644e49f01; Hm_lvt_8acef669ea66f479854ecd328d1f348f=1679555446
2023-03-25  21:43:08.402  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x3598ea6e, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65439] WRITE: 100B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 48 54 54 50 2f 31 2e 31 20 32 30 30 20 4f 4b 0d |HTTP/1.1 200 OK.|
|00000010| 0a 63 6f 6e 74 65 6e 74 2d 6c 65 6e 67 74 68 3a |.content-length:|
|00000020| 20 32 32 0d 0a 63 6f 6e 74 65 6e 74 2d 74 79 70 | 22..content-typ|
|00000030| 65 3a 20 74 65 78 74 2f 68 74 6d 6c 3b 63 68 61 |e: text/html;cha|
|00000040| 72 73 65 74 3d 75 74 66 2d 38 0d 0a 0d 0a 3c 68 |rset=utf-8....<h|
|00000050| 31 3e 48 65 6c 6c 6f 2c 20 77 6f 72 6c 64 21 3c |1>Hello, world!<|
|00000060| 2f 68 31 3e                                     |/h1>            |
+--------+-------------------------------------------------+----------------+
2023-03-25  21:43:08.403  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x3598ea6e, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65439] FLUSH
2023-03-25  21:43:08.403  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  请求体：
2023-03-25  21:43:08.403  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x3598ea6e, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65439] READ COMPLETE
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x3598ea6e, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65439] READ: 746B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 47 45 54 20 2f 66 61 76 69 63 6f 6e 2e 69 63 6f |GET /favicon.ico|
|00000010| 20 48 54 54 50 2f 31 2e 31 0d 0a 48 6f 73 74 3a | HTTP/1.1..Host:|
|00000020| 20 6c 6f 63 61 6c 68 6f 73 74 3a 38 30 38 30 0d | localhost:8080.|
|00000030| 0a 43 6f 6e 6e 65 63 74 69 6f 6e 3a 20 6b 65 65 |.Connection: kee|
|00000040| 70 2d 61 6c 69 76 65 0d 0a 73 65 63 2d 63 68 2d |p-alive..sec-ch-|
|00000050| 75 61 3a 20 22 4d 69 63 72 6f 73 6f 66 74 20 45 |ua: "Microsoft E|
|00000060| 64 67 65 22 3b 76 3d 22 31 31 31 22 2c 20 22 4e |dge";v="111", "N|
|00000070| 6f 74 28 41 3a 42 72 61 6e 64 22 3b 76 3d 22 38 |ot(A:Brand";v="8|
|00000080| 22 2c 20 22 43 68 72 6f 6d 69 75 6d 22 3b 76 3d |", "Chromium";v=|
|00000090| 22 31 31 31 22 0d 0a 73 65 63 2d 63 68 2d 75 61 |"111"..sec-ch-ua|
|000000a0| 2d 6d 6f 62 69 6c 65 3a 20 3f 30 0d 0a 55 73 65 |-mobile: ?0..Use|
|000000b0| 72 2d 41 67 65 6e 74 3a 20 4d 6f 7a 69 6c 6c 61 |r-Agent: Mozilla|
|000000c0| 2f 35 2e 30 20 28 57 69 6e 64 6f 77 73 20 4e 54 |/5.0 (Windows NT|
|000000d0| 20 31 30 2e 30 3b 20 57 69 6e 36 34 3b 20 78 36 | 10.0; Win64; x6|
|000000e0| 34 29 20 41 70 70 6c 65 57 65 62 4b 69 74 2f 35 |4) AppleWebKit/5|
|000000f0| 33 37 2e 33 36 20 28 4b 48 54 4d 4c 2c 20 6c 69 |37.36 (KHTML, li|
|00000100| 6b 65 20 47 65 63 6b 6f 29 20 43 68 72 6f 6d 65 |ke Gecko) Chrome|
|00000110| 2f 31 31 31 2e 30 2e 30 2e 30 20 53 61 66 61 72 |/111.0.0.0 Safar|
|00000120| 69 2f 35 33 37 2e 33 36 20 45 64 67 2f 31 31 31 |i/537.36 Edg/111|
|00000130| 2e 30 2e 31 36 36 31 2e 35 31 0d 0a 73 65 63 2d |.0.1661.51..sec-|
|00000140| 63 68 2d 75 61 2d 70 6c 61 74 66 6f 72 6d 3a 20 |ch-ua-platform: |
|00000150| 22 57 69 6e 64 6f 77 73 22 0d 0a 41 63 63 65 70 |"Windows"..Accep|
|00000160| 74 3a 20 69 6d 61 67 65 2f 77 65 62 70 2c 69 6d |t: image/webp,im|
|00000170| 61 67 65 2f 61 70 6e 67 2c 69 6d 61 67 65 2f 73 |age/apng,image/s|
|00000180| 76 67 2b 78 6d 6c 2c 69 6d 61 67 65 2f 2a 2c 2a |vg+xml,image/*,*|
|00000190| 2f 2a 3b 71 3d 30 2e 38 0d 0a 53 65 63 2d 46 65 |/*;q=0.8..Sec-Fe|
|000001a0| 74 63 68 2d 53 69 74 65 3a 20 73 61 6d 65 2d 6f |tch-Site: same-o|
|000001b0| 72 69 67 69 6e 0d 0a 53 65 63 2d 46 65 74 63 68 |rigin..Sec-Fetch|
|000001c0| 2d 4d 6f 64 65 3a 20 6e 6f 2d 63 6f 72 73 0d 0a |-Mode: no-cors..|
|000001d0| 53 65 63 2d 46 65 74 63 68 2d 44 65 73 74 3a 20 |Sec-Fetch-Dest: |
|000001e0| 69 6d 61 67 65 0d 0a 52 65 66 65 72 65 72 3a 20 |image..Referer: |
|000001f0| 68 74 74 70 3a 2f 2f 6c 6f 63 61 6c 68 6f 73 74 |http://localhost|
|00000200| 3a 38 30 38 30 2f 69 6e 64 65 78 2e 68 74 6d 6c |:8080/index.html|
|00000210| 0d 0a 41 63 63 65 70 74 2d 45 6e 63 6f 64 69 6e |..Accept-Encodin|
|00000220| 67 3a 20 67 7a 69 70 2c 20 64 65 66 6c 61 74 65 |g: gzip, deflate|
|00000230| 2c 20 62 72 0d 0a 41 63 63 65 70 74 2d 4c 61 6e |, br..Accept-Lan|
|00000240| 67 75 61 67 65 3a 20 7a 68 2d 43 4e 2c 7a 68 3b |guage: zh-CN,zh;|
|00000250| 71 3d 30 2e 39 2c 65 6e 3b 71 3d 30 2e 38 2c 65 |q=0.9,en;q=0.8,e|
|00000260| 6e 2d 47 42 3b 71 3d 30 2e 37 2c 65 6e 2d 55 53 |n-GB;q=0.7,en-US|
|00000270| 3b 71 3d 30 2e 36 0d 0a 43 6f 6f 6b 69 65 3a 20 |;q=0.6..Cookie: |
|00000280| 49 64 65 61 2d 32 33 34 37 65 36 38 33 3d 65 37 |Idea-2347e683=e7|
|00000290| 63 31 39 34 64 36 2d 63 65 66 31 2d 34 61 30 36 |c194d6-cef1-4a06|
|000002a0| 2d 38 32 31 35 2d 30 33 39 36 34 34 65 34 39 66 |-8215-039644e49f|
|000002b0| 30 31 3b 20 48 6d 5f 6c 76 74 5f 38 61 63 65 66 |01; Hm_lvt_8acef|
|000002c0| 36 36 39 65 61 36 36 66 34 37 39 38 35 34 65 63 |669ea66f479854ec|
|000002d0| 64 33 32 38 64 31 66 33 34 38 66 3d 31 36 37 39 |d328d1f348f=1679|
|000002e0| 35 35 35 34 34 36 0d 0a 0d 0a                   |555446....      |
+--------+-------------------------------------------------+----------------+
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  请求的uri：/favicon.ico
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Host ---> localhost:8080
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Connection ---> keep-alive
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  sec-ch-ua ---> "Microsoft Edge";v="111", "Not(A:Brand";v="8", "Chromium";v="111"
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  sec-ch-ua-mobile ---> ?0
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  User-Agent ---> Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36 Edg/111.0.1661.51
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  sec-ch-ua-platform ---> "Windows"
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Accept ---> image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Sec-Fetch-Site ---> same-origin
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Sec-Fetch-Mode ---> no-cors
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Sec-Fetch-Dest ---> image
2023-03-25  21:43:08.433  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Referer ---> http://localhost:8080/index.html
2023-03-25  21:43:08.434  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Accept-Encoding ---> gzip, deflate, br
2023-03-25  21:43:08.434  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Accept-Language ---> zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
2023-03-25  21:43:08.434  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  Cookie ---> Idea-2347e683=e7c194d6-cef1-4a06-8215-039644e49f01; Hm_lvt_8acef669ea66f479854ecd328d1f348f=1679555446
2023-03-25  21:43:08.434  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x3598ea6e, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65439] WRITE: 100B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 48 54 54 50 2f 31 2e 31 20 32 30 30 20 4f 4b 0d |HTTP/1.1 200 OK.|
|00000010| 0a 63 6f 6e 74 65 6e 74 2d 6c 65 6e 67 74 68 3a |.content-length:|
|00000020| 20 32 32 0d 0a 63 6f 6e 74 65 6e 74 2d 74 79 70 | 22..content-typ|
|00000030| 65 3a 20 74 65 78 74 2f 68 74 6d 6c 3b 63 68 61 |e: text/html;cha|
|00000040| 72 73 65 74 3d 75 74 66 2d 38 0d 0a 0d 0a 3c 68 |rset=utf-8....<h|
|00000050| 31 3e 48 65 6c 6c 6f 2c 20 77 6f 72 6c 64 21 3c |1>Hello, world!<|
|00000060| 2f 68 31 3e                                     |/h1>            |
+--------+-------------------------------------------------+----------------+
2023-03-25  21:43:08.434  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x3598ea6e, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65439] FLUSH
2023-03-25  21:43:08.434  [nioEventLoopGroup-3-2] DEBUG mao.t2.Server:  请求体：
2023-03-25  21:43:08.434  [nioEventLoopGroup-3-2] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x3598ea6e, L:/[0:0:0:0:0:0:0:1]:8080 - R:/[0:0:0:0:0:0:0:1]:65439] READ COMPLETE
```



客户端访问

```sh
hello
2023-03-25  21:44:17.314  [main] DEBUG mao.t2.Client:  请求体：hello
2023-03-25  21:44:17.324  [main] INFO  mao.t2.Client:  响应内容：<h1>Hello, world!</h1>
你好
2023-03-25  21:44:27.771  [main] DEBUG mao.t2.Client:  请求体：你好
2023-03-25  21:44:27.777  [main] INFO  mao.t2.Client:  响应内容：<h1>Hello, world!</h1>
654
2023-03-25  21:44:32.211  [main] DEBUG mao.t2.Client:  请求体：654
2023-03-25  21:44:32.213  [main] INFO  mao.t2.Client:  响应内容：<h1>Hello, world!</h1>
```

```sh
2023-03-25  21:44:17.319  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x9b850779, L:/127.0.0.1:8080 - R:/127.0.0.1:65455] REGISTERED
2023-03-25  21:44:17.319  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x9b850779, L:/127.0.0.1:8080 - R:/127.0.0.1:65455] ACTIVE
2023-03-25  21:44:17.322  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x9b850779, L:/127.0.0.1:8080 - R:/127.0.0.1:65455] READ: 235B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 50 4f 53 54 20 2f 69 6e 64 65 78 2e 68 74 6d 6c |POST /index.html|
|00000010| 20 48 54 54 50 2f 31 2e 31 0d 0a 55 73 65 72 2d | HTTP/1.1..User-|
|00000020| 41 67 65 6e 74 3a 20 4a 61 76 61 2f 31 36 2e 30 |Agent: Java/16.0|
|00000030| 2e 32 0d 0a 48 6f 73 74 3a 20 6c 6f 63 61 6c 68 |.2..Host: localh|
|00000040| 6f 73 74 3a 38 30 38 30 0d 0a 41 63 63 65 70 74 |ost:8080..Accept|
|00000050| 3a 20 74 65 78 74 2f 68 74 6d 6c 2c 20 69 6d 61 |: text/html, ima|
|00000060| 67 65 2f 67 69 66 2c 20 69 6d 61 67 65 2f 6a 70 |ge/gif, image/jp|
|00000070| 65 67 2c 20 2a 3b 20 71 3d 2e 32 2c 20 2a 2f 2a |eg, *; q=.2, */*|
|00000080| 3b 20 71 3d 2e 32 0d 0a 43 6f 6e 6e 65 63 74 69 |; q=.2..Connecti|
|00000090| 6f 6e 3a 20 6b 65 65 70 2d 61 6c 69 76 65 0d 0a |on: keep-alive..|
|000000a0| 43 6f 6e 74 65 6e 74 2d 74 79 70 65 3a 20 61 70 |Content-type: ap|
|000000b0| 70 6c 69 63 61 74 69 6f 6e 2f 78 2d 77 77 77 2d |plication/x-www-|
|000000c0| 66 6f 72 6d 2d 75 72 6c 65 6e 63 6f 64 65 64 0d |form-urlencoded.|
|000000d0| 0a 43 6f 6e 74 65 6e 74 2d 4c 65 6e 67 74 68 3a |.Content-Length:|
|000000e0| 20 35 0d 0a 0d 0a 68 65 6c 6c 6f                | 5....hello     |
+--------+-------------------------------------------------+----------------+
2023-03-25  21:44:17.323  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  请求的uri：/index.html
2023-03-25  21:44:17.323  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  User-Agent ---> Java/16.0.2
2023-03-25  21:44:17.323  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  Host ---> localhost:8080
2023-03-25  21:44:17.323  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  Accept ---> text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
2023-03-25  21:44:17.323  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  Connection ---> keep-alive
2023-03-25  21:44:17.323  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  Content-type ---> application/x-www-form-urlencoded
2023-03-25  21:44:17.323  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  Content-Length ---> 5
2023-03-25  21:44:17.323  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x9b850779, L:/127.0.0.1:8080 - R:/127.0.0.1:65455] WRITE: 100B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 48 54 54 50 2f 31 2e 31 20 32 30 30 20 4f 4b 0d |HTTP/1.1 200 OK.|
|00000010| 0a 63 6f 6e 74 65 6e 74 2d 6c 65 6e 67 74 68 3a |.content-length:|
|00000020| 20 32 32 0d 0a 63 6f 6e 74 65 6e 74 2d 74 79 70 | 22..content-typ|
|00000030| 65 3a 20 74 65 78 74 2f 68 74 6d 6c 3b 63 68 61 |e: text/html;cha|
|00000040| 72 73 65 74 3d 75 74 66 2d 38 0d 0a 0d 0a 3c 68 |rset=utf-8....<h|
|00000050| 31 3e 48 65 6c 6c 6f 2c 20 77 6f 72 6c 64 21 3c |1>Hello, world!<|
|00000060| 2f 68 31 3e                                     |/h1>            |
+--------+-------------------------------------------------+----------------+
2023-03-25  21:44:17.323  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x9b850779, L:/127.0.0.1:8080 - R:/127.0.0.1:65455] FLUSH
2023-03-25  21:44:17.324  [nioEventLoopGroup-3-3] DEBUG mao.t2.Server:  请求体：hello
2023-03-25  21:44:17.325  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x9b850779, L:/127.0.0.1:8080 - R:/127.0.0.1:65455] READ COMPLETE
2023-03-25  21:44:22.337  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x9b850779, L:/127.0.0.1:8080 - R:/127.0.0.1:65455] READ COMPLETE
2023-03-25  21:44:22.337  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x9b850779, L:/127.0.0.1:8080 ! R:/127.0.0.1:65455] INACTIVE
2023-03-25  21:44:22.337  [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x9b850779, L:/127.0.0.1:8080 ! R:/127.0.0.1:65455] UNREGISTERED
2023-03-25  21:44:27.773  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 - R:/127.0.0.1:65456] REGISTERED
2023-03-25  21:44:27.773  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 - R:/127.0.0.1:65456] ACTIVE
2023-03-25  21:44:27.776  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 - R:/127.0.0.1:65456] READ: 236B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 50 4f 53 54 20 2f 69 6e 64 65 78 2e 68 74 6d 6c |POST /index.html|
|00000010| 20 48 54 54 50 2f 31 2e 31 0d 0a 55 73 65 72 2d | HTTP/1.1..User-|
|00000020| 41 67 65 6e 74 3a 20 4a 61 76 61 2f 31 36 2e 30 |Agent: Java/16.0|
|00000030| 2e 32 0d 0a 48 6f 73 74 3a 20 6c 6f 63 61 6c 68 |.2..Host: localh|
|00000040| 6f 73 74 3a 38 30 38 30 0d 0a 41 63 63 65 70 74 |ost:8080..Accept|
|00000050| 3a 20 74 65 78 74 2f 68 74 6d 6c 2c 20 69 6d 61 |: text/html, ima|
|00000060| 67 65 2f 67 69 66 2c 20 69 6d 61 67 65 2f 6a 70 |ge/gif, image/jp|
|00000070| 65 67 2c 20 2a 3b 20 71 3d 2e 32 2c 20 2a 2f 2a |eg, *; q=.2, */*|
|00000080| 3b 20 71 3d 2e 32 0d 0a 43 6f 6e 6e 65 63 74 69 |; q=.2..Connecti|
|00000090| 6f 6e 3a 20 6b 65 65 70 2d 61 6c 69 76 65 0d 0a |on: keep-alive..|
|000000a0| 43 6f 6e 74 65 6e 74 2d 74 79 70 65 3a 20 61 70 |Content-type: ap|
|000000b0| 70 6c 69 63 61 74 69 6f 6e 2f 78 2d 77 77 77 2d |plication/x-www-|
|000000c0| 66 6f 72 6d 2d 75 72 6c 65 6e 63 6f 64 65 64 0d |form-urlencoded.|
|000000d0| 0a 43 6f 6e 74 65 6e 74 2d 4c 65 6e 67 74 68 3a |.Content-Length:|
|000000e0| 20 36 0d 0a 0d 0a e4 bd a0 e5 a5 bd             | 6..........    |
+--------+-------------------------------------------------+----------------+
2023-03-25  21:44:27.776  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  请求的uri：/index.html
2023-03-25  21:44:27.776  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  User-Agent ---> Java/16.0.2
2023-03-25  21:44:27.776  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  Host ---> localhost:8080
2023-03-25  21:44:27.777  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  Accept ---> text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
2023-03-25  21:44:27.777  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  Connection ---> keep-alive
2023-03-25  21:44:27.777  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  Content-type ---> application/x-www-form-urlencoded
2023-03-25  21:44:27.777  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  Content-Length ---> 6
2023-03-25  21:44:27.777  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 - R:/127.0.0.1:65456] WRITE: 100B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 48 54 54 50 2f 31 2e 31 20 32 30 30 20 4f 4b 0d |HTTP/1.1 200 OK.|
|00000010| 0a 63 6f 6e 74 65 6e 74 2d 6c 65 6e 67 74 68 3a |.content-length:|
|00000020| 20 32 32 0d 0a 63 6f 6e 74 65 6e 74 2d 74 79 70 | 22..content-typ|
|00000030| 65 3a 20 74 65 78 74 2f 68 74 6d 6c 3b 63 68 61 |e: text/html;cha|
|00000040| 72 73 65 74 3d 75 74 66 2d 38 0d 0a 0d 0a 3c 68 |rset=utf-8....<h|
|00000050| 31 3e 48 65 6c 6c 6f 2c 20 77 6f 72 6c 64 21 3c |1>Hello, world!<|
|00000060| 2f 68 31 3e                                     |/h1>            |
+--------+-------------------------------------------------+----------------+
2023-03-25  21:44:27.777  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 - R:/127.0.0.1:65456] FLUSH
2023-03-25  21:44:27.778  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  请求体：你好
2023-03-25  21:44:27.778  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 - R:/127.0.0.1:65456] READ COMPLETE
2023-03-25  21:44:32.212  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 - R:/127.0.0.1:65456] READ: 233B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 50 4f 53 54 20 2f 69 6e 64 65 78 2e 68 74 6d 6c |POST /index.html|
|00000010| 20 48 54 54 50 2f 31 2e 31 0d 0a 55 73 65 72 2d | HTTP/1.1..User-|
|00000020| 41 67 65 6e 74 3a 20 4a 61 76 61 2f 31 36 2e 30 |Agent: Java/16.0|
|00000030| 2e 32 0d 0a 48 6f 73 74 3a 20 6c 6f 63 61 6c 68 |.2..Host: localh|
|00000040| 6f 73 74 3a 38 30 38 30 0d 0a 41 63 63 65 70 74 |ost:8080..Accept|
|00000050| 3a 20 74 65 78 74 2f 68 74 6d 6c 2c 20 69 6d 61 |: text/html, ima|
|00000060| 67 65 2f 67 69 66 2c 20 69 6d 61 67 65 2f 6a 70 |ge/gif, image/jp|
|00000070| 65 67 2c 20 2a 3b 20 71 3d 2e 32 2c 20 2a 2f 2a |eg, *; q=.2, */*|
|00000080| 3b 20 71 3d 2e 32 0d 0a 43 6f 6e 6e 65 63 74 69 |; q=.2..Connecti|
|00000090| 6f 6e 3a 20 6b 65 65 70 2d 61 6c 69 76 65 0d 0a |on: keep-alive..|
|000000a0| 43 6f 6e 74 65 6e 74 2d 74 79 70 65 3a 20 61 70 |Content-type: ap|
|000000b0| 70 6c 69 63 61 74 69 6f 6e 2f 78 2d 77 77 77 2d |plication/x-www-|
|000000c0| 66 6f 72 6d 2d 75 72 6c 65 6e 63 6f 64 65 64 0d |form-urlencoded.|
|000000d0| 0a 43 6f 6e 74 65 6e 74 2d 4c 65 6e 67 74 68 3a |.Content-Length:|
|000000e0| 20 33 0d 0a 0d 0a 36 35 34                      | 3....654       |
+--------+-------------------------------------------------+----------------+
2023-03-25  21:44:32.212  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  请求的uri：/index.html
2023-03-25  21:44:32.212  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  User-Agent ---> Java/16.0.2
2023-03-25  21:44:32.212  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  Host ---> localhost:8080
2023-03-25  21:44:32.212  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  Accept ---> text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
2023-03-25  21:44:32.212  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  Connection ---> keep-alive
2023-03-25  21:44:32.212  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  Content-type ---> application/x-www-form-urlencoded
2023-03-25  21:44:32.212  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  Content-Length ---> 3
2023-03-25  21:44:32.212  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 - R:/127.0.0.1:65456] WRITE: 100B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 48 54 54 50 2f 31 2e 31 20 32 30 30 20 4f 4b 0d |HTTP/1.1 200 OK.|
|00000010| 0a 63 6f 6e 74 65 6e 74 2d 6c 65 6e 67 74 68 3a |.content-length:|
|00000020| 20 32 32 0d 0a 63 6f 6e 74 65 6e 74 2d 74 79 70 | 22..content-typ|
|00000030| 65 3a 20 74 65 78 74 2f 68 74 6d 6c 3b 63 68 61 |e: text/html;cha|
|00000040| 72 73 65 74 3d 75 74 66 2d 38 0d 0a 0d 0a 3c 68 |rset=utf-8....<h|
|00000050| 31 3e 48 65 6c 6c 6f 2c 20 77 6f 72 6c 64 21 3c |1>Hello, world!<|
|00000060| 2f 68 31 3e                                     |/h1>            |
+--------+-------------------------------------------------+----------------+
2023-03-25  21:44:32.212  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 - R:/127.0.0.1:65456] FLUSH
2023-03-25  21:44:32.212  [nioEventLoopGroup-3-4] DEBUG mao.t2.Server:  请求体：654
2023-03-25  21:44:32.213  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 - R:/127.0.0.1:65456] READ COMPLETE
2023-03-25  21:44:37.791  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 - R:/127.0.0.1:65456] READ COMPLETE
2023-03-25  21:44:37.791  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 ! R:/127.0.0.1:65456] INACTIVE
2023-03-25  21:44:37.791  [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler:  [id: 0x68cd2b82, L:/127.0.0.1:8080 ! R:/127.0.0.1:65456] UNREGISTERED
```









## 自定义协议

### 协议要素

* 魔数，用来在第一时间判定是否是无效数据包
* 版本号，可以支持协议的升级
* 序列化算法，消息正文到底采用哪种序列化反序列化方式，可以由此扩展，例如：json、protobuf、hessian、jdk
* 指令类型，是登录、注册、单聊、群聊... 跟业务相关
* 请求序号，为了双工通信，提供异步能力
* 正文长度
* 消息正文





### 协议父类消息

```java
package mao.message;

import lombok.Data;

import java.io.Serializable;
import java.util.HashMap;
import java.util.Map;

/**
 * Project name(项目名称)：Netty_自定义协议
 * Package(包名): mao.message
 * Class(类名): Message
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/26
 * Time(创建时间)： 14:05
 * Version(版本): 1.0
 * Description(描述)： 协议父类消息
 */

@Data
public abstract class Message implements Serializable
{
    /**
     * 根据消息类型字节，获得对应的消息 class
     *
     * @param messageType 消息类型字节
     * @return 消息 class
     */
    public static Class<? extends Message> getMessageClass(int messageType)
    {
        return messageClasses.get(messageType);
    }

    /**
     * 序列id
     */
    private int sequenceId;

    /**
     * 消息类型
     */
    private int messageType;

    /**
     * 得到消息类型
     *
     * @return int
     */
    public abstract int getMessageType();

    public static final int PingMessage = 1;
    public static final int PongMessage = 2;
    public static final int HelloRequestMessage = 3;
    public static final int HelloResponseMessage = 4;


    /**
     * 请求类型 byte 值
     */
    public static final int RPC_MESSAGE_TYPE_REQUEST = 101;

    /**
     * 响应类型 byte 值
     */
    public static final int RPC_MESSAGE_TYPE_RESPONSE = 102;

    /**
     * 消息类
     */
    private static final Map<Integer, Class<? extends Message>> messageClasses = new HashMap<>();

    static
    {
        messageClasses.put(PingMessage, PingMessage.class);
        messageClasses.put(PongMessage, PongMessage.class);
        messageClasses.put(HelloRequestMessage, HelloRequestMessage.class);
        messageClasses.put(HelloResponseMessage, HelloResponseMessage.class);
    }
}
```





### 抽象响应消息

```java
package mao.message;

import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.ToString;

/**
 * Project name(项目名称)：Netty_自定义协议
 * Package(包名): mao.message
 * Class(类名): AbstractResponseMessage
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/26
 * Time(创建时间)： 14:12
 * Version(版本): 1.0
 * Description(描述)： 抽象响应消息
 */


@Data
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
public abstract class AbstractResponseMessage extends Message
{
    /**
     * 是否是成功的消息
     */
    private boolean success;

    /**
     * 如果失败，失败的原因
     */
    private String reason;

    public AbstractResponseMessage()
    {
    }

    public AbstractResponseMessage(boolean success, String reason)
    {
        this.success = success;
        this.reason = reason;
    }
}
```





### ping消息

```java
package mao.message;

import lombok.Data;
import lombok.EqualsAndHashCode;

/**
 * Project name(项目名称)：Netty_自定义协议
 * Package(包名): mao.message
 * Class(类名): PingMessage
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/26
 * Time(创建时间)： 14:17
 * Version(版本): 1.0
 * Description(描述)： ping消息
 */

@Data
@EqualsAndHashCode(callSuper = true)
public class PingMessage extends Message
{
    /**
     * 时间
     */
    private long time;

    @Override
    public int getMessageType()
    {
        return PingMessage;
    }
}
```





###  pong消息

```java
package mao.message;

import lombok.Data;
import lombok.EqualsAndHashCode;
import mao.protocol.SequenceIdGenerator;

/**
 * Project name(项目名称)：Netty_自定义协议
 * Package(包名): mao.message
 * Class(类名): PongMessage
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/26
 * Time(创建时间)： 14:18
 * Version(版本): 1.0
 * Description(描述)： pong 消息
 */


@Data
@EqualsAndHashCode(callSuper = true)
public class PongMessage extends Message
{
    /**
     * 请求时间
     */
    private long time;

    public PongMessage(int time)
    {
        this.time = time;
        setSequenceId(SequenceIdGenerator.nextId());
    }

    public PongMessage()
    {
        setSequenceId(SequenceIdGenerator.nextId());
    }

    @Override
    public int getMessageType()
    {
        return PongMessage;
    }
}
```





### 打招呼的请求消息

```java
package mao.message;

import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.ToString;

/**
 * Project name(项目名称)：Netty_自定义协议
 * Package(包名): mao.message
 * Class(类名): HelloRequestMessage
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/26
 * Time(创建时间)： 14:20
 * Version(版本): 1.0
 * Description(描述)： 打招呼的请求消息
 */


@Data
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
public class HelloRequestMessage extends Message
{
    /**
     * 打招呼的人的姓名
     */
    private String name;

    /**
     * 内容
     */
    private String body;

    @Override
    public int getMessageType()
    {
        return HelloRequestMessage;
    }
}
```





### 打招呼的响应消息

```java
package mao.message;

import lombok.*;
import mao.protocol.SequenceIdGenerator;

/**
 * Project name(项目名称)：Netty_自定义协议
 * Package(包名): mao.message
 * Class(类名): HelloResponseMessage
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/26
 * Time(创建时间)： 14:23
 * Version(版本): 1.0
 * Description(描述)： 打招呼的响应消息
 */


@Data
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
public class HelloResponseMessage extends AbstractResponseMessage
{
    /**
     * 要响应的消息
     */
    private String body;

    public HelloResponseMessage(String body)
    {
        this.body = body;
        setSequenceId(SequenceIdGenerator.nextId());
    }

    public HelloResponseMessage(boolean success, String reason, String body)
    {
        super(success, reason);
        this.body = body;
        setSequenceId(SequenceIdGenerator.nextId());
    }

    public HelloResponseMessage()
    {
        setSequenceId(SequenceIdGenerator.nextId());
    }

    @Override
    public int getMessageType()
    {
        return HelloResponseMessage;
    }

    //这里写静态方法简化类的频繁创建

    /**
     * 成功
     *
     * @param body 消息内容
     * @return {@link HelloResponseMessage}
     */
    public static HelloResponseMessage success(String body)
    {
        return new HelloResponseMessage(true, null, body);
    }


    /**
     * 失败
     *
     * @return {@link HelloResponseMessage}
     */
    public static HelloResponseMessage fail()
    {
        return new HelloResponseMessage(false, "未知", null);
    }

    /**
     * 失败
     *
     * @param reason 原因
     * @return {@link HelloResponseMessage}
     */
    public static HelloResponseMessage fail(String reason)
    {
        return new HelloResponseMessage(false, reason, null);
    }

}
```





### config.properties

```properties
server.port=8080
serializer.algorithm=Json
```





### 服务配置类

```java
package mao.config;

import mao.protocol.SerializerAlgorithm;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

/**
 * Project name(项目名称)：Netty_自定义协议
 * Package(包名): mao.config
 * Class(类名): ServerConfig
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/26
 * Time(创建时间)： 21:18
 * Version(版本): 1.0
 * Description(描述)： 服务配置类
 */

public class ServerConfig
{

    private static Properties properties;

    static
    {
        try (InputStream inputStream =
                     ServerConfig.class.getClassLoader().getResourceAsStream("config.properties"))
        {
            properties = new Properties();
            properties.load(inputStream);
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
    }

    /**
     * 得到服务器端口號
     *
     * @return int
     */
    public static int getServerPort()
    {
        String value = properties.getProperty("server.port");
        if (value == null)
        {
            return 8080;
        }
        else
        {
            return Integer.parseInt(value);
        }
    }

    /**
     * 得到序列化器算法
     *
     * @return {@link SerializerAlgorithm}
     */
    public static SerializerAlgorithm getSerializerAlgorithm()
    {
        String value = properties.getProperty("serializer.algorithm");
        if (value == null)
        {
            return SerializerAlgorithm.Java;
        }
        else
        {
            return SerializerAlgorithm.valueOf(value);
        }
    }
}
```







### Serializer

用于扩展序列化、反序列化算法

```java
package mao.protocol;

import java.io.*;
import java.lang.reflect.Type;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_自定义协议
 * Package(包名): mao.protocol
 * Interface(接口名): Serializer
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/26
 * Time(创建时间)： 21:10
 * Version(版本): 1.0
 * Description(描述)： 用于扩展序列化、反序列化算法
 */

public interface Serializer
{
    /**
     * 反序列化
     *
     * @param clazz clazz
     * @param bytes 字节数组
     * @return {@link T}
     */
    <T> T deserialize(Class<T> clazz, byte[] bytes);

    /**
     * 序列化
     *
     * @param object 对象
     * @return {@link byte[]}
     */
    <T> byte[] serialize(T object);

}
```





### SerializerAlgorithm

```java
package mao.protocol;

import com.alibaba.fastjson.JSON;

import java.io.*;
import java.nio.charset.StandardCharsets;

/**
 * Project name(项目名称)：Netty_自定义协议
 * Package(包名): mao.protocol
 * Enum(枚举名): SerializerAlgorithm
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/26
 * Time(创建时间)： 21:12
 * Version(版本): 1.0
 * Description(描述)： 序列化算法，json采用fastjson
 */

public enum SerializerAlgorithm implements Serializer
{
    Java
            {
                @Override
                public <T> T deserialize(Class<T> clazz, byte[] bytes)
                {
                    try
                    {
                        ObjectInputStream objectInputStream = new ObjectInputStream(new ByteArrayInputStream(bytes));
                        return (T) objectInputStream.readObject();
                    }
                    catch (IOException | ClassNotFoundException e)
                    {
                        throw new RuntimeException("反序列化失败", e);
                    }
                }

                @Override
                public <T> byte[] serialize(T object)
                {
                    try
                    {
                        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
                        ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
                        objectOutputStream.writeObject(object);
                        return byteArrayOutputStream.toByteArray();
                    }
                    catch (IOException e)
                    {
                        throw new RuntimeException("序列化失败", e);
                    }
                }
            },

    Json
            {
                @Override
                public <T> T deserialize(Class<T> clazz, byte[] bytes)
                {
                    String json = new String(bytes, StandardCharsets.UTF_8);
                    return JSON.parseObject(json, clazz);
                }

                @Override
                public <T> byte[] serialize(T object)
                {
                    String jsonString = JSON.toJSONString(object);
                    return jsonString.getBytes(StandardCharsets.UTF_8);
                }
            }
}
```





### 序列ID生成器

比较简单，非全局唯一

```java
package mao.protocol;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * Project name(项目名称)：Netty_自定义协议
 * Package(包名): mao.protocol
 * Class(类名): SequenceIdGenerator
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2023/3/26
 * Time(创建时间)： 21:06
 * Version(版本): 1.0
 * Description(描述)： 序列 ID 生成器，内部采用CAS的方式累加
 */

public class SequenceIdGenerator
{
    /**
     * id
     */
    private static final AtomicInteger id = new AtomicInteger();

    public static int nextId()
    {
        return id.incrementAndGet();
    }
}
```







### 协议解码器
