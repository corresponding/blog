---
title: Okio源码解析
date: 2017-12-20 19:39:26
tags: [库, Android, 源码]
categories: Android
---

{% asset_img 1.jpg 深秋 %}


大家一定听说或者使用过鼎鼎大名的OKHttp和Retrofit，它们内部都调用了Okio。
它们三个都是Square团队开源的库，主要处理I/O操作和网络请求。
今天我们就来读下Okio的源码，看看Okio有什么神奇之处。



# Okio是什么?
Okio是一个完善java.io和java.nio的库，可以更方便的读、写、处理数据。

看到上面的官方介绍，我们可以发现两个重点：
1.完善java.io和java.nio；
2.Okio的主要功能是，读、写、处理数据。

再深入思考下，java.io和java.nio有哪些尚未完善的地方？Okio有做了哪些优化？

java.io能处理所有的IO信息，但是还是有些不完美之处：
1.需要自己去管理byte数组；
2.java.io类和继承关系过于复杂，使用起来更加不便；
3.java.io操作可能会某些原因一直等待无法返回，需要另外开线程去监听；
4.操作字符流和字节流方法不同，需要区别对待；
5.涉及到多个流之间的数据传递，需要反复拷贝，效率不高。

被Android官方青睐的IO库，Okio一一解决了这些问题：
1.使用ByteString去封装byte数组；
2.简化类关系，主要使用Buffer、Sink、Source等类，相互关系简单；
3.设置timeout超时控制；
4.可用相同方法去操作字符流和字节流；
5.提高多个流之间的数据传递速度。

带着这些问题，我们继续前进。

# ByteString
记得我最开始编程时，C语言中没有字符串类型，只能使用char*的方式存储字符串，而且经常涉及数组和指针的转换，
需要使用memcpy和memset等等函数，容易出错，而且还需要加入大量的检测。
后面使用面向对象语言后，发现用String对象直接管理字符串，良好的封装char数组的常用方法，使用起来各种清爽。

同样使用过java.io的开发者，肯定会想起被byte数组笼罩的恐惧。
需要不厌其烦的处理一串串byte数组，完成encode和decode函数，还要特别关注编码问题。
这个时候就希望，能有一个对象来管理byte数组，并且帮我们封装好各种函数，使用时直接调用下即可。

Okio就满足开发者这个愿望，提供了ByteString类，把对字节数组的常用方法都封装好了。
使用Okio库后，开发者可以以ByteString作为最小粒度进行操作。

先来看下ByteString的组成
```
// 0.实现了Serializable和Comparable接口
public class ByteString implements Serializable, Comparable<ByteString> {
  // 1.byte数组，所以为ByteString
  final byte[] data;
  // 2.hashCode，在需要时才计算出。不会被序列化
  transient int hashCode;
  // 3.data utf8化后生成的String，在需要时才计算出。不会被序列化
  transient String utf8; 
}
```
ByteString的内部也不复杂，主要就是byte数组。
这里要看下String的结构，其实String可以命名为CharString。
因为一般开发中，字符串用的比较多，所以java官方直接命名为String。
```
// 0.同样实现了Serializable和Comparable接口，额外实现了CharSequence处理常规char数组数据
public final class String implements Serializable, Comparable<String>, CharSequence {
    // 1.char数组，所以String等于CharString
    private final char value[];
	// 2.hashCode
    private int hash; 
}
```

接下去看下，ByteString的读和写
```
public static ByteString read(InputStream in, int byteCount) throws IOException {
    // 1.检查合法性
    if (in == null) throw new IllegalArgumentException("in == null");
    if (byteCount < 0) throw new IllegalArgumentException("byteCount < 0: " + byteCount);
    // 2.分配空间
    byte[] result = new byte[byteCount];
    // 3.循环读取inputStream中的数据
    for (int offset = 0, read; offset < byteCount; offset += read) {
      // 3.1.每次尽可能多的读取inputStream数据，放入result中
      read = in.read(result, offset, byteCount - offset);
      // 3.2.读取inputStream异常
      if (read == -1) throw new EOFException();
    }
    // 4.生成ByteString
    return new ByteString(result);
  }
```

```
public void write(OutputStream out) throws IOException {
	// 1.检查合法性
    if (out == null) throw new IllegalArgumentException("out == null");
	// 2.直接写入byte数组
    out.write(data);
}
```


# Buffer
IO操作的最重要部分就是对流的处理，所以定义好一个流非常关键。
Okio中，就是用Buffer去表示流。

{% asset_img 2.png Buffer类关系图 %}

先看下翻译
source：来源；水源；
sink：水槽；洗涤槽；
一下子就明白了Source表示输入源，Sink表示输出源。
对应java.io中的inputStream和outputStream。

```
public final class Buffer implements BufferedSource, BufferedSink, Cloneable {
  Segment head;
  long size;    
}
```
size表示Buffer中的字节数量。
主要成员变量也非常简单，就是segment。
源码上注释非常简洁清晰：A segment of a buffer，翻译过来就是Buffer的片段。
```
final class Segment {
  Segment prev;			// 指向双向链表上一节点
  Segment next;			// 指向双向链表下一节点
  final byte[] data;    // data表示本segment存储的数据
  int pos;              // pos表示data中下一个读取字节的index
  int limit;            // limit表示data中下一个写入字节的index。为什么用limit命名呢，因为这个index也是读取字节的上限
  boolean shared;       // 是否共享。Okio为了提高效率，在某些时候不拷贝整个segment，而是采用弱引用方式指向segment
  boolean owner;        // 因为加入共享功能后，就需要确定持有者，只有持有者才能往这个segment中写数据
  ...
}
```
从prev和next就可以发现这是双向链表。在Buffer中就持有head，通过head去访问整个链表。
通过后续的代码阅读，发现这是一个双向循环链表。

{% asset_img 3.png segment双向链表 %}



我们分析下Okio中是如何写入和读取byte数据的。

## Buffer.writeByte 
先来看看Buffer怎么样写入一个byte
```
// 0.注意入参是int格式，不过其中有效数据的只有8 bit
@Override public Buffer writeByte(int b) {
	// 1.寻找可供写入的segment
    Segment tail = writableSegment(1);
	// 2.在segment中写入字节byte。limit的含义在上文提及，表示下一个写入字节的index
    tail.data[tail.limit++] = (byte) b;
    // 3.buffer中size变化
	size += 1;
    return this;
}
```
writeByte函数的代码都很简单，我们继续看看如何寻找可供写入的segment。

## Buffer.writableSegment
```
// 0.入参minimumCapacity表示允许写的最小空间
Segment writableSegment(int minimumCapacity) {
    // 1.检查合法性
    if (minimumCapacity < 1 || minimumCapacity > Segment.SIZE) throw new IllegalArgumentException();
    // 2.当前head为空时，先新建一个
    if (head == null) {
      // 2.1.从SegmentPool中取出一个segment
      head = SegmentPool.take();
      // 2.2.设置为双向循环链表
      return head.next = head.prev = head;
    }
    // 3.选择链表最尾部的segment
    Segment tail = head.prev;
    // 4.如果尾部segment中没有足够的可写空间，或者当前Buffer不是尾部segment的持有者，重新从SegmentPool取出segment插入链表
    if (tail.limit + minimumCapacity > Segment.SIZE || !tail.owner) {
      tail = tail.push(SegmentPool.take());
    }
    return tail;
}
```
这里引入了SegmentPool这个类，主要用来管理Segment对象，回收旧segment，分配segment。
SegmentPool减少了segment的新建和释放次数，缓解java GC的压力。

注：类似于Handler中message.obtain()，其背后也有一个pool去管理所有的message对象。


## SegmentPool
这个类代码很短，全部贴出来
```
final class SegmentPool {
  // pool 容量上限，最大为64KiB
  static final long MAX_SIZE = 64 * 1024;
  // 单向链表
  static Segment next;
  // pool中总字节数
  static long byteCount;

  private SegmentPool() {
  }

  static Segment take() {
    // take和recycle中next都被synchronize包含，所以是线程安全的
    synchronized (SegmentPool.class) {
      // 如果当前pool不为空，链表还有数据，从链表头取出segment
      if (next != null) {
        Segment result = next;
        next = result.next;
        result.next = null;
        byteCount -= Segment.SIZE;
        return result;
      }
    }
    // 如果当前pool部为空，链表中没有数据，新建一个segment
    return new Segment();
  }

  static void recycle(Segment segment) {
    if (segment.next != null || segment.prev != null) throw new IllegalArgumentException();
    // 如果当前segment被共享，则放弃回收
    if (segment.shared) return;
    // take和recycle中next都被synchronize包含，所以是线程安全的
    synchronized (SegmentPool.class) {
      // pool达到上限的，放弃回收
      if (byteCount + Segment.SIZE > MAX_SIZE) return;
      byteCount += Segment.SIZE;
      segment.next = next;
      next = segment;
      // 重新设置segment的pos和limit
      segment.pos = segment.limit = 0;
    }
  }
}
```
segmentPool这个类非常简单，内部维护一个单向列表。
take()会从头部取出segment，recycler()将待回收的segment放回到链表头部。

## Buffer.readByte
我们来看下readByte，其方法和writeByte完全对称。
```
public byte readByte() {
    // 1.检查合法性
    if (size == 0) throw new IllegalStateException("size == 0");
    // 2.取出segment中数据
    Segment segment = head;
    int pos = segment.pos;
    int limit = segment.limit;
    byte[] data = segment.data;
    byte b = data[pos++];
    size -= 1;
    // 3.如果当前segment中数据被读取完毕
    if (pos == limit) {
      // 3.1.从双向链表中pop出来
      head = segment.pop();
      // 3.2.回收这个segment，放入到segmentPool中
      SegmentPool.recycle(segment);
    } else {
      // 4.如果当前segment中数据未被读取完毕，只更新pos信息
      segment.pos = pos;
    }
    return b;
  }
```

# Source
IO最重要的就是对流的处理，Okio中对应的数据结构是Source和Sink

```
// 0.Closeable接口中方法为close()
public interface Source extends Closeable {
  // 1.读取Buffer中byteCount长度的数据，返回值为本次读取的字节长度
  long read(Buffer sink, long byteCount) throws IOException;
  // 2.超时控制
  Timeout timeout();
  // 3.关闭source并且释放资源，允许调用多次
  @Override void close() throws IOException;
}
```
这是一个接口，后续都是使用其实现类作为输入流。

## Okio.Source
我们看下Okio.Source，开发者一般都使用这个函数生成Source。
```
private static Source source(final InputStream in, final Timeout timeout) {
    // 1.检查合法性
    if (in == null) throw new IllegalArgumentException("in == null");
    if (timeout == null) throw new IllegalArgumentException("timeout == null");
    // 2.新建了一个匿名类，实现Source接口
    return new Source() {
      @Override public long read(Buffer sink, long byteCount) throws IOException {
        // 2.1.检查合法性
        if (byteCount < 0) throw new IllegalArgumentException("byteCount < 0: " + byteCount);
        if (byteCount == 0) return 0;
        try {
          // 2.2.检查是否超时
          timeout.throwIfReached();
          // 2.3.寻找可供写入的segment，在之前详细解释过
          Segment tail = sink.writableSegment(1);
          // 3.3.当前segment剩余可以写入大小，和byteCount两者间选择较小的值，设置为maxToCopy
          int maxToCopy = (int) Math.min(byteCount, Segment.SIZE - tail.limit);
          // 3.4.从InputStream中尽可能的提取maxToCopy长度，写入到segment中
          int bytesRead = in.read(tail.data, tail.limit, maxToCopy);
          // 3.5.读取异常
          if (bytesRead == -1) return -1;
          // 3.6.调整segment和buffer中的参数
          tail.limit += bytesRead;
          sink.size += bytesRead;
          return bytesRead;
        } catch (AssertionError e) {
          if (isAndroidGetsocknameError(e)) throw new IOException(e);
          throw e;
        }
      }

      @Override public void close() throws IOException {
        in.close();
      }

      @Override public Timeout timeout() {
        return timeout;
      }

      @Override public String toString() {
        return "source(" + in + ")";
      }
    };
}
```
以上代码逻辑非常清晰，流水型执行下去即可。
这里出现了一个新的对象timeout，用来管理超时，我们看看其内部结构。

## Timeout
```
public class Timeout {
  // 1.判断deadlineNanoTime是否被定义。如果缺少该变量的话，deadlineNanoTime或者timeoutNanos为0时，无法判断是未设置还是设置为0
  private boolean hasDeadline;
  // 2.截止时间，单位为纳秒
  private long deadlineNanoTime;
  // 3.设定的超时时间间隔，单位为纳秒
  private long timeoutNanos;
}
```
Timeout主要用来设置超时时间，对Stream的读取超过指定时间后，认定为失败，开发者需要选择close或者重新操作这个Stream。
不使用超时机制的话，Stream出现异常则会陷入无限等待中。

继续看下上面被调用的throwIfReached()
```
public void throwIfReached() throws IOException {
    // 1.判断当前线程是否被中断
    if (Thread.interrupted()) {
      throw new InterruptedIOException("thread interrupted");
    }
    // 2.判断当前是否超时
    if (hasDeadline && deadlineNanoTime - System.nanoTime() <= 0) {
      throw new InterruptedIOException("deadline reached");
    }
}
```

## Okio.buffer
```
// 0.这里传入上面通过Okio.source生成的Source
public static BufferedSource buffer(Source source) {
    // 1.生成BufferSource，真正处理输入流，你看命名中都写着real
    return new RealBufferedSource(source);
}
```

# RealBufferedSource
终于找到开发者最后操作的Source类，先来看下它的成员吧。
```
// 0.实现BufferedSource接口
final class RealBufferedSource implements BufferedSource {
  // 1.初始化时自动生成
  public final Buffer buffer = new Buffer();
  // 2.Okio.Buffer中传入的source对象
  public final Source source;
  // 3.判断source是否关闭
  boolean closed;
}
```

## RealBufferedSource.read
对于输入流来说，我们需要紧紧抓住read方法。
```
@Override public int read(byte[] sink, int offset, int byteCount) throws IOException {
    // 1.检查合法性
    checkOffsetAndCount(sink.length, offset, byteCount);
    // 2.当前buffer为空时
    if (buffer.size == 0) {
      // 2.1.尽可能读取一个segment放入到buffer中
      long read = source.read(buffer, Segment.SIZE);
      // 2.2.读取异常
      if (read == -1) return -1;
    }
    // 3.选择buffer中数据和byteCount中较小值
    int toRead = (int) Math.min(byteCount, buffer.size);
    // 4.将buffer中数据输出到byte数组
    return buffer.read(sink, offset, toRead);
  }
```
主要流程就是，现将source数据读取到buffer中，再将buffer提取到byte数组中。buffer.read()这个函数这里就不展开讨论了，其原理和buffer.readByte非常相近。


# Sink
同理，完全对应的，也可以按照Okio.sink-> Okio.buffer(Sink)-> RealBufferedSink-> RealBufferedSink.write()流程走完一遍Okio中关于写的操作。


总结，Okio这个库本身并不复杂，将常用的流和字节数据进行良好的封装，加之源码中相近的注释，阅读起来非常流畅，想必使用起来的体验也会非常棒。


