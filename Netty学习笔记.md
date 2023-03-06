





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

