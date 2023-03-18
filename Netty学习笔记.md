





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







## NioEventLoop处理io事件

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











## handler执行中何换工人



