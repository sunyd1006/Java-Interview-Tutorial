就像很多标准的架构模式都被各种专用框架所支持一样，常见的数据处理模式往往也是目标实现的很好的候选对象，它可以节省开发人员大量的时间和精力。
当然这也适应于本文的主题:编码和解码，或者数据从一种特定协议的格式到另一种格式的转 换。这些任务将由通常称为`编解码器`的组件来处理
Netty 提供了多种组件，简化了为了支持广泛 的协议而创建自定义的编解码器的过程
例如，如果你正在构建一个基于 Netty 的邮件服务器，那 么你将会发现 Netty 对于编解码器的支持对于实现 POP3、IMAP 和 SMTP 协议来说是多么的宝贵
# 0 什么是编解码器
每个网络应用程序都必须定义
- 如何解析在两个节点之间来回传输的原始字节
- 如何将其和目标应用程序的数据格式做相互转换

这种转换逻辑由编解码器处理，编解码器由编码器和解码器组成，它们每种都可以将字节流从一种格式转换为另一种格式

那么它们的区别是什么呢?
如果将消息看作是对于特定的应用程序具有具体含义的结构化的字节序列— 它的数据。那 么编码器是将消息转换为适合于传输的格式(最有可能的就是字节流);而对应的解码器则是将 网络字节流转换回应用程序的消息格式。因此，编码器操作出站数据，而解码器处理入站数据。
记住这些背景信息，接下来让我们研究一下 Netty 所提供的用于实现这两种组件的类。
# 1 Netty解码概述
![](https://img-blog.csdnimg.cn/img_convert/73d354c67617fafbc34db7c863205981.png)
## 1.1 本文目标
- 解码器抽象的解码过程
- Netty里面有哪些拆箱即用的解码器

Netty 的解码器类：
- 将字节解码为消息
ByteToMessageDecoder 和 ReplayingDecoder
- 将一种消息类型解码为另一种
MessageToMessageDecoder

解码器负责`将入站数据从一种格式转到另一种`，所以 Netty 解码器实
现了 `ChannelInboundHandler` 也很自然。
 
- 什么时候会用解码器?
每当需为 `ChannelPipeline` 中的下一个 `ChannelInboundHandler` 转换入站数据时。

得益于` ChannelPipeline` 的设计，可以将多个解码器连接在一起，以实现任意复杂的转换逻辑，这也是 Netty 是如何支持代码的模块化以及复用的一个很好的例子。


## 案例代码
![](https://img-blog.csdnimg.cn/img_convert/4e72393c5f0b6a264ad65bc0c0d02f3a.png)
![](https://img-blog.csdnimg.cn/img_convert/81577ea4f55f3f4993783bf0dffb1134.png)
![](https://img-blog.csdnimg.cn/img_convert/f3a2ffd8af44b091dce499012ea3c21d.png)

# 2 抽象解码器 ByteToMessageDecoder
## 2.1 示例
Netty 提供抽象基类：ByteToMessageDecoder，将字节解码为消息(或另一个字节序列)。
由于`你不可能知道远程节点是否会一次性发送一个完整消息`，所以该类会`缓冲入站数据`，直到它准备好处理。

- ByteToMessageDecoderAPI![](https://img-blog.csdnimg.cn/img_convert/e4017422b311a0952cf57b3e73eb818a.png)
假设你接收了一个包含简单 int 的字节流，每个 int 都需要被单独处理
在这种情况下，你需要从入站` ByteBuf `中读取每个 int，并将它传递给` ChannelPipeline` 中的下一个 `ChannelInboundHandler`
为了解码这个字节流，你要扩展 `ByteToMessageDecoder `类(原子类型的 int 在被添加到 List 中时，会被自动装箱为 Integer)
![ToIntegerDecoder](https://img-blog.csdnimg.cn/img_convert/6270b1e7d6ac2d0bc9f56266a051e4ab.png)
每次从入站 ByteBuf 中读取 4 字节，将其解码为一个 int，然后将它添加到一个 List 中
 当没有更多的元素可以被添加到该 List 中时，它的内容将会被发送给下一个 Channel- InboundHandler
- ToIntegerDecoder类扩展了ByteToMessageDecoder
![](https://img-blog.csdnimg.cn/img_convert/ce607f642b6ede33d3f52657db83767a.png)
虽然` ByteToMessageDecoder `可以很简单地实现这种模式，但是你可能会发现，在调用 `readInt()`前不得不验证所输入的 ByteBuf 是否具有足够的数据有点繁琐
在下一节中， 我们将讨论 ReplayingDecoder，它是一个特殊的解码器，以少量的开销消除了这个步骤
## 2.2 源码解析
![](https://img-blog.csdnimg.cn/img_convert/2ae19fff14d236a1bccbe6cc74051368.png)

下面开始解析解码流程的源码：
### 2.2.1 累加字节流
![](https://img-blog.csdnimg.cn/20201212225856230.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNTg5NTEw,size_1,color_FFFFFF,t_70)

其中的` cumulator` 为
![](https://img-blog.csdnimg.cn/img_convert/13fc4116527e0b0825a5f6c43432ea9e.png)
看一下这个`MERGE_CUMULATOR`
```java
public static final Cumulator MERGE_CUMULATOR = new Cumulator() {
    
    @Override
    public ByteBuf cumulate(ByteBufAllocator alloc, ByteBuf cumulation, ByteBuf in) {
        ByteBuf buffer;
        // 当前写指针后移一定字节,若超过最大容量,则扩容
        if (cumulation.writerIndex() > cumulation.maxCapacity() - in.readableBytes()
                || cumulation.refCnt() > 1) {
            // Expand cumulation (by replace it) when either there is not more room in the buffer
            // or if the refCnt is greater then 1 which may happen when the user use slice().retain() or
            // duplicate().retain().
            //
            // See:
            // - https://github.com/netty/netty/issues/2327
            // - https://github.com/netty/netty/issues/1764
            buffer = expandCumulation(alloc, cumulation, in.readableBytes());
        } else {
            buffer = cumulation;
        }
        // 将当前数据写到累加器
        buffer.writeBytes(in);
        // 释放读进的数据对象
        in.release();
        return buffer;
    }
};
```

### 2.2.2 调用子类 decode 方法进行解析
- 进入该方法查看源码
![](https://img-blog.csdnimg.cn/img_convert/7172ed1a9fe9bf3cdb77712f7f0d34e6.png)

![](https://img-blog.csdnimg.cn/20201212230404103.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNTg5NTEw,size_1,color_FFFFFF,t_70)

### 2.2.2 将解析到的 ByteBuf 向下传播
![](https://img-blog.csdnimg.cn/20201212230544100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNTg5NTEw,size_1,color_FFFFFF,t_70)注意到上图中的如下代码段：
![](https://img-blog.csdnimg.cn/20201212230918251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNTg5NTEw,size_1,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20201212230843978.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNTg5NTEw,size_1,color_FFFFFF,t_70)

## 编解码器中的引用计数
对于编码器和解码器，一旦消息被编码或解码，它就会被 `ReferenceCountUtil.release(message)`调用自动释放。
若需要保留引用以便稍后使用，可调用 `ReferenceCountUtil.retain(message) `，这会增加该引用计数，从而防止该消息被释放。
# 3 固定长度解码器
![](https://img-blog.csdnimg.cn/20201212231239235.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNTg5NTEw,size_1,color_FFFFFF,t_70)

# 4 行解码器
- 非丢弃模式处理
![](https://img-blog.csdnimg.cn/img_convert/97dfef06eb142219f3b1ca3a7dcd40e1.png)
## 4.1 定位行尾
![](https://img-blog.csdnimg.cn/img_convert/3e00d3b065acb26acce782511366ed8b.png)
![](https://img-blog.csdnimg.cn/img_convert/fc94a7b05d2735fdb5437a8e63c18bba.png)
![](https://img-blog.csdnimg.cn/img_convert/f7ae56c55745af7468c21ddbabc0959d.png)
## 4.2 非丢弃模式
![](https://img-blog.csdnimg.cn/img_convert/bf5041999fccf4def3b13c771ca8b45d.png)
![](https://img-blog.csdnimg.cn/img_convert/77a8881d0372955d31a88e83cf9b04a8.png)
### 4.2.1 找到换行符情况下
![](https://img-blog.csdnimg.cn/img_convert/5c827a8d7d8214de7cf48ff1429d6396.png)
### 4.2.2 找不到换行符情况下
![](https://img-blog.csdnimg.cn/img_convert/3261f798a7ccb0121a9907b7a8633ce0.png)
![](https://img-blog.csdnimg.cn/img_convert/9cbaae71cda15677083039332900276b.png)
解析出长度超过最大可解析长度
直接进入丢弃模式,读指针移到写指针位(即丢弃)
并传播异常
## 4.3  丢弃模式
![](https://img-blog.csdnimg.cn/img_convert/8f44d11e3447205ca02cfb2824ff9d17.png)
### 找到换行符
![](https://img-blog.csdnimg.cn/img_convert/b14416c313dfec49d91f8916b06faa09.png)
记录当前丢弃了多少字节(已丢弃 + 本次将丢弃的)
锁定换行符类型
将读指针直接移到换行符后
丢弃字节置零
重置为非丢弃状态
所有字节丢弃后才触发快速失败机制
### 找不到换行符
![](https://img-blog.csdnimg.cn/img_convert/b76e642486e377a3cd882dd2df8565fd.png)
直接记录当前丢弃字节(已丢弃 + 当前可读字节数)
将读指针直接移到写指针


参考
- 《Netty实战》