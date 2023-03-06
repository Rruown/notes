# 基础概念

[GStreamer](https://gstreamer.freedesktop.org/documentation/tutorials/index.html?gi-language=c)是C语言框架，使用了`GObject`（面向对象）和`Glib`（算法库）。`GStreamer`的函数、结构体和类型以`gst_`为前缀，`GLib`和`GObject`使用`g_`

`Gstreamer`是处理多媒体流的框架。媒体从“source”元件（生产者）”传输到“sink”元件（消费者），中间途径多个执行各种任务的元件。所有相连元件的集合称为**管道**。

  

## 抽象解释

`GStreamer`由`Element`、`Pads`、`Bin（Pipleline）`、`Communication`组成。

可以将`Pipleline`看作一个树形结构

`Element`是树的结点

`Pads`是结点的入口和出口，可以将`Element`的入口和出口相连，并负责数据的接收与发送

`Bin`抽象了树的内部结构，将树形结构模块化。本质上`Bin`包含多个Element，其表现得行为与`Element`类似。最顶层得Bin称为“Pipleline”

## 架构设计

## 自动解析Pipleline

初始化gstreamer

```C
 gst_init (&argc, &argv);
```

gstreamer提供了序列化工具，从文本转换成pipleline

```C
GstElement *pipeline;
 pipeline =
      gst_parse_launch
      ("playbin uri=https://wwwfreedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm",NULL);
```

  

## 手动构建Pipeline

> 具体API可以参考https://gstreamer.freedesktop.org/documentation/libs.html?gi-language=c

element工厂方法

-   参数1：元件的类型
    
-   参数2：元件名
    

```C
source = gst_element_factory_make ("videotestsrc", "source");
sink = gst_element_factory_make ("autovideosink", "sink");
```

初始化pipeline

```C
GstElement *pipeline = gst_pipeline_new("test_pipeline");
```

连接元件（结点），只有同一个bin内的元件才可以相连

```C
/* 多个元件添加到bin，以NULL结束 */
gst_bin_add_many (GST_BIN (pipeline), source, sink, NULL);

/* 单个元件添加到bin */
gst_bin_add(GST_BIN(pipleline), source);

/* 同组内的元件相连 */
gst_element_link(source,Esink);
```

## 元件（lement）

元件是pipeline最基本的构成（结点），元件提供了丰富功能，如读取数据、解码数据、过滤等。

将不同功能的元件连接在一起可以组成执行特定任务的pipeline。

针对特殊场景，可以开发新元件。

### 多线程

gstreamer提供了`queue`元件支持多线程（分支结点并行能力）

样例如：[Basic tutorial 7: Multithreading and Pad Availability](https://gstreamer.freedesktop.org/documentation/tutorials/basic/multithreading-and-pad-availability.html?gi-language=c)

### 插件

元件是继承自`GstElement`的类，提供了一些功能。

插件本质上就是一个可加载的代码块（动态链接库）

## Pad

### Pad Capabilities

描述Pad的“能力”，作用类似于trait

**相连的元件，caps具有公共子集（兼容的）**

如SINK的caps模板：

> 从element factory可以查看caps模板

```Plain
SINK template: 'sink'
  Availability: Always
  Capabilities:
    audio/x-raw
               format: S16LE
                 rate: [ 1, 2147483647 ]
             channels: [ 1, 2 ]
    audio/x-raw
               format: U8
                 rate: [ 1, 2147483647 ]
             channels: [ 1, 2 ]
```

### Pad Availability

#### Always Pads

大多数的pads都是always的，换句话说，这类的pads一开始就是固定不变的

#### Request Pads

经典元件如`tee`（功能是复制流），sink的pads是不固定的，每调用

```C
 GstPad *tee_audio_pad = gst_element_request_pad_simple(tee, "src_%u");
```

创建一个sink pad（复制一份）

#### Sometimes Pads

## 通信

![](https://vastaitech.feishu.cn/space/api/box/stream/download/asynccode/?code=OGUzNmY5YWFhYjMzZjBiMWM2NDQyNmJjZTZlNTdmOWNfeE5CaGtWQTJydklNNXFqa2xIUDNUZWQ5Zm9XSEpzSVlfVG9rZW46Ym94Y25EME80cjB6RjlKTFI0UDJUU2VERUdJXzE2NzgwOTcxMjY6MTY3ODEwMDcyNl9WNA)

### 缓存

> 元件和元件之间的数据传输

数据以块的形式（Buffer）在Pipeline中传输，是实际存储媒体数据的基本单位。

“源”pad负责生产Buffer，“终”pad负责消费Buffer，数据的处理交给元件。

Buffer带有时间戳和持续时间描述其何时被解码或播放。

### 事件

> 应用程序与元件、元件与元件之间

事件与缓存一起在Pipeline中传输，存储控制信息。

### 消息

bus是Pipeline所有元件消息的汇总处，应用层监听bus获取消息

bus线程负责将流线程的消息转发给应用程序

#### 错误检查

监听bus获取错误消息

```C
  /* 监听pipeline的bus，设置监听时间和消息类型*/
  bus = gst_element_get_bus (pipeline);
  msg =
      gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
      GST_MESSAGE_ERROR | GST_MESSAGE_EOS);
  /* 解析消息 */
  if (msg != NULL) {
    GError *err;
    gchar *debug_info;switch (GST_MESSAGE_TYPE (msg)) {case GST_MESSAGE_ERROR:
        gst_message_parse_error (msg, &err, &debug_info);
        g_printerr ("Error received from element %s: %s\n",
            GST_OBJECT_NAME (msg->src), err->message);
        g_printerr ("Debugging information: %s\n",
            debug_info ? debug_info : "none");
        g_clear_error (&err);
        g_free (debug_info);break;case GST_MESSAGE_EOS:
        g_print ("End-Of-Stream reached.\n");break;default:
        /* We should not reach here because we only asked for ERRORs and EOS */
        g_printerr ("Unexpected message received.\n");break;}
    gst_message_unref (msg);
 }
```

### 查询

> 应用程序与元件、元件与元件

查询总是**同步的**

#### 时间管理

`GstQuery`提供访问元件信息的机制，如查询某pipeline是否可以“快进”

略

## GObject

### 修改和访问成员变量

```C
/* 修改成员变量
   参数：    1. 对象
            2. 属性名1
            3. 属性值1
            4. 属性名2
            5. 属性名2
            6. 。。。
            7. 。。。

*/
g_object_set(source, "pattern", 0, NULL)


g_object_get(source, 。。，。。。)
```

# GStreamer工具

## gst-launch-1.0

`gst-launch-1.0`是gstreamer用于快速构建简易pipline的debug工具，以此来检验pipline是否能够正常工作。

**`!`****构建串行pipeline**

```Shell
gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink
```

**元件属性设置**

```Shell
gst-launch-1.0 videotestsrc pattern=11 ! videoconvert ! autovideosink
```

**命名构建分支pipeline**

每个元件都是一个结点，`!`用于串行连接元件。tee被命名为`t`以标记（命名）tee结点，t.（注意逗号）是tee结点的另一串分支

```Bash
gst-launch-1.0 videotestsrc ! videoconvert ! tee name=t ! queue ! autovideosink t. ! queue ! autovideosink
```

**指定pads**

以`.`的方式指定连接的pad

```Bash
gst-launch-1.0 souphttpsrc location=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.webm ! matroskademux name=d d.video_0 ! matroskamux ! filesink location=sintel_video.mkv
```

## gst-inspect-1.0

1.  不指定参数时，列出所有元件
    
2.  以文件名作为参数，列出插件内元件的信息
    
3.  以元件名作为参数，列出元件的信息
    

## gst-discover-1.0

该工具是`GstDiscoverer`的包装，用于解析URI的媒体信息

# 插件开发

  

# GstRTSPServer

gst-rtsp-server是GStreamer上层的库用来搭建（ _**Real Time Streaming Protocol**_ ）RTSP服务

## 设计理念

Server的上层组件是`GstRTSPServer`类型，它会创建并绑定socket，然后附加到mainloop

对于每个请求，都会创建一个新的 `GstRTSPClient`对象来接受请求，并启动一个线程来处理与客户端的进一步通信，直到连接关闭。

客户端发出SETUP请求时，将创建`GstRTSPSession`类型追踪客户端状态，直到客户端发出TEARDOWN请求。

维护了一个从URL到媒体pipeline映射的池，使用pipeline支持流实时传输或按需文件流传输或流的按需转码

维护了一个当前活动的管道池。通常活动管道由一个或多个 `GstRTSPSession` 对象使用。当没有更多会话引用它时，活动管道变为非活动状态。客户端可以选择启动新管道或加入当前活动的管道。无法加入某些活动管道（例如点播流），但可以创建该管道的新实例。