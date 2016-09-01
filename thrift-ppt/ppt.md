title: thrift框架讲解
speaker: 白晓琦
url: https://github.com/xiaoshaozi52/myppts/thrift-ppt
transition: cards
files: /css/ppt.css

[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

# thrift框架讲解 {:.text-dark}
## 演讲者：白晓琦 {:.gray3}


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## 问题 {:.gray3}
----
# 什么是RPC? {:.text-left.red}
{:.green.text-left}
* 远程过程调用(Remote Procedure Call)是一个计算机通信协议 {:&.zoomIn}
* 允许运行于一台计算机的程序调用另一台计算机的子程序，而程序员无需额外地为这个交互作用编程
* 通俗一点就是：调用远程计算机上的程序，就像调用本地程序一样。
<!-- * ![](/img/rpc.png) -->
* 流程： <img src="/img/rpc.png" height="300">
* 常见的RPC框架: Apache的 *Thrift*、google的gRPC、阿里的Dubbo、Apache的Avro、微信的phxrpc等

[note]
1. 调用客户端句柄；执行传送参数
2. 调用本地系统内核发送网络消息
3. 消息传送到远程主机
4. 服务器句柄得到消息并取得参数
5. 执行远程过程
6. 执行的过程将结果返回服务器句柄
7. 服务器句柄返回结果，调用远程系统内核
8. 消息传回本地主机
9. 客户句柄由内核接收消息
10. 客户接收句柄返回的数据
[/note]


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## thrift定义 {:.gray3}
----
{:.gray3}
* 由Fackbook开发, 贡献给Apache并已经开源 {:&.moveIn}
* 支持多种编程语言的RPC(Remote Procedure Call远程过程调用)框架
* 利用自身的协议栈和代码生成工具，根据接口定义语言(IDL Interface definition lanuage)构建可以在C++,Java,Python等程序语言之间进行无缝结合的高效服务
* github地址:
    </br><i class="fa fa-github"></i>https://github.com/apache/thrift
    </br><i class="fa fa-github"></i>https://github.com/facebook/fbthrift


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## 数据类型 {:.gray3}
----
{:.gray3}
* 基本类型
    </br> <span class="blue">bool、byte、i16、i32、i64、double、string、binarry</span>
* 枚举(enum)
* 容器
    </br> <span class="blue">list&lt;t&gt;  set&lt;t&gt;  map&lt;t1, t2&gt;</span>
* 结构体(struct)
* 异常(exception)
* 服务(service)
* 类型定义(Typedefs)
</br><span class="blue">Thrift supports C/C++ style typedefs.</span>
```
typedef i32 MyInteger   // 1
typedef Tweet ReTweet   // 2
```


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## 字段约束 {:.gray3}
----
{:.gray3}
* required: 必须赋值字段禁止为空
* optional: 可选字段可以为空
* oneway: 对client请求后不返回响应

[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## Sample {:.gray3}
`pushservice.thrift` {:.text-left}
```
namespace py pushservice
enum AfterOpen { GOAPP = 1, GOURL = 2, GOACTIVITY = 3, GOCUSTOM = 4}
enum DisplayType {MESSAGE = 1, NOTIFICATION = 2}
struct AndroidMsgBody {
    1: optional string ticker,  ...
    7: optional AfterOpen after_open = AfterOpen.GOAPP,
    8: optional string open_content
}
struct IosMsgAps {
    1: string alert,  ...
}
struct Policy {
    1: optional string start_time,
    2: optional string expire_time,  ...
}
exception ParamError {
    1: string message,
}
exception ServerError {
    1: i16 code,
    2: string message,
}
service PushService {
    void ping(),
    oneway void zip(),  //client发送请求后不返回响应，无返回值
    bool unicast_notification_android(1:string app, 2:DisplayType display_type, 3:string device_tokens, 4:AndroidMsgBody body, 5:map<string, string> extra, 6:Policy policy, 7:string description, 8:bool production_mode) throws (1: ParamError pe, 2: ServerError se),
    bool unicast_notification_ios(1:string app, 2:string device_token, 3:IosMsgAps aps, 4:map<string, string> extra, 5:Policy policy, 6:string description, 7:bool production_mode) throws (1: ParamError pe, 2: ServerError se),
    ...
}
```
```thrift --gen py pushservice.thrift``` {:.text-left}


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## thrift架构 {:.gray3}
----
<div style="clear:both;">
    <div style="width: 59%;float: left">
        <div class="text-left">
            <span class="bg-yellow">&nbsp;&nbsp;</span>
            <span class="gray3" style="font-size: large">需要实现的业务逻辑</span>
        </div>
        <br>
        <div class="text-left">
            <span class="bg-brown">&nbsp;&nbsp;</span>
            <span class="gray3" style="font-size: large">根据 Thrift 的接口定义文件生成的客户端和服务器端代码</span>
        </div>
        <br>
        <div class="text-left">
            <span class="bg-red">&nbsp;&nbsp;</span>
            <span class="gray3" style="font-size: large">根据 Thrift 文件生成代码实现数据的读写操作</span>
        </div>
        <br>
        <div class="text-left">
            <span class="bg-purple">&nbsp;&nbsp;</span>
            <span class="gray3" style="font-size: large">数据的序列化和反序列化</span>
        </div>
        <br>
        <div class="text-left">
            <span class="bg-blue">&nbsp;&nbsp;</span>
            <span class="gray3" style="font-size: large">字节流(Byte Stream)数据在IO模块上的传输</span>
        </div>
        <br>
        <div class="text-left">
            <span class="bg-gray">&nbsp;&nbsp;</span>
            <span class="gray3" style="font-size: large">底层 I/O 通信</span>
        </div>
    </div>
    <div style="width: 39%;float: left">
        <img src="/img/thrift-architecture.png" height="350">
    </div>
</div>

[note]
```
service PushService {
    void ping(),
    oneway void zip(),  //client发送请求后不返回响应，无返回值
    bool unicast_notification_android(1:string app, 2:DisplayType display_type, 3:string device_tokens, 4:AndroidMsgBody body, 5:map<string, string> extra, 6:Policy policy, 7:string description, 8:bool production_mode) throws (1: ParamError pe, 2: ServerError se),
    bool unicast_notification_ios(1:string app, 2:string device_token, 3:IosMsgAps aps, 4:map<string, string> extra, 5:Policy policy, 6:string description, 7:bool production_mode) throws (1: ParamError pe, 2: ServerError se),
    ...
}
```
[/note]


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## Ttransport 传输层 {:.gray3}
----
{:.gray3}
* 负责Thrift的字节流(Byte Stream)数据在IO模块上的传输
* 主要传输类型:
    * TSocket 使用阻塞的Socket进行IO传输 {:&.zoomIn.blue}
    * TFramedTransport 使用一个带Buffer的Socket进行IO传输，使用NoblockingServer的时候会需要使用TFramedTransport {:.blue}
    * TFileTransport 使用类文件对象进行IO传输 {:.blue}
    * TZlibTransport 使用zlib对数据的压缩传输 {:.blue}

[note]
<img src="/img/thrift-architecture.jpg" height="500">
[/note]


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## TProtocol 协议 {:.gray3}
----
{:.gray3}
* 客户端与服务端之间传输的通信协议，不同的传输协议对应不同的数据编码
* 根据需求常见的协议有:
    * TBinaryProtocol 二进制编码格式进行数据传输 {:&.zoomIn.blue}
    * TCompactProtocol 高效率的、密集的二进制编码格式进行数据传输 {:.blue}
    * TJSONProtocol 使用 JSON 的数据编码协议进行数据传输 {:.blue}
    * TSimpleJSONProtocol 只提供 JSON 只写的协议，适用于通过脚本语言解析 {:.blue}

[note]
<img src="/img/thrift-architecture.jpg" height="500">
[/note]


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## TServer {:.gray3}
----
{:.gray3}
* 负责接收Client的请求，并将请求转发到Processor进行处理
* 常见的服务端类型有以下几种：
    * TSimpleServer 单进程服务器端 使用标准的阻塞式 I/O {:&.zoomIn.blue}
    * TProcessPoolServer 多进程服务器端 使用标准的阻塞式 I/O {:.blue}
    * TNonblockingServer 多线程服务器端 使用非阻塞式 I/O {:.blue}

[note]
* TProcessPoolServer:每一个进程的工作内容都是一样,读消息然后进行转发处理，在进行IO操作时，如果堵塞住了，进程只能等待(闲置)
* TNonblockingServer:workers并不是拿着socket连接进行IO操作，而是主线程管理所有的连接，通过系统调用select，获取所有不用等待的连接，放在队列中，分配给workers处理，让workers在请求量比较大的时候都有活干。
* TProcessPoolServer处理高并发时我们不得不增加workers数量，导致系统在调度进程上产生了很大的开销，很快就会达到性能瓶颈。TNonblockingServer使用多线程实现，可以设置更多的workers并能够更高效的利用CPU，由于夹杂了select及其他调度操作， 对于单个请求的处理会时间比TProcessPoolServer长。
[/note]


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

##　Server Code Sample {:.gray3}
----
`server.py` {:.text-left}
```
from thrift import Thrift
from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TBinaryProtocol
from thrift.server import TServer
from pushservice import PushService
from pushservice.ttypes import ParamError, ServerError

class PushServiceHandler:
    def __init__(self):
        self.log = {}
    def ping(self):
        print 'ping pang pong'
    def unicast_notification_ios(self, app, device_tokens, aps, extra, policy, description, production_mode):
        print '业务逻辑实现部分'

if __name__ == '__main__':
    handler = PushServiceHandler()
    processor = PushService.Processor(handler)
    ts = TSocket.TServerSocket(host="0.0.0.0" ,port=9090)
    tfactory = TTransport.TBufferedTransportFactory()
    pfactory = TBinaryProtocol.TBinaryProtocolFactory()

    server = TServer.TSimpleServer(processor, ts, tfactory, pfactory)
    server.serve()
```


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

##　Client Code Sample {:.gray3}
----
`client.py` {:.text-left}
```
from thrift import Thrift
from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TBinaryProtocol
from pushservice import PushService
from pushservice.ttypes import IosMsgAps, AndroidMsgBody

def main():
    ts = TSocket.TSocket('localhost', 9090)
    transport = TTransport.TBufferedTransport(ts)
    protocol = TBinaryProtocol.TBinaryProtocol(transport)
    client = PushService.Client(protocol)
    transport.open()
    client.ping()
    aps = IosMsgAps("server测试")
    flag = client.unicast_notification_ios('leying', '851807bf98181e22cd1e6f7db7d43ff8095b5d4ab354bc5370f7e9a204f8a176', aps, {}, None, 'test', False)
    transport.close()

if __name__ == '__main__':
    try:
        main()
    except Thrift.TException as tx:
        print '%s' % tx
```

[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## 优点和缺点 {:.gray3}
---
{:.blue}
* <span class="green">优点</span>
    * 通过thrift文件生成目标代码，简单易用
    * 支持多语言 支持多种序列化方式 支持多种通讯协议 支持多进程和非阻塞式服务端
    * 客户端调用更直观，初步的类型和结构校验
    * 性能比较好
* <span class="gray2">缺点</span>：
    * 文档少文档少
    * 只能由客户端发起请求，不支持双向通信
    * 不具备动态特性(可以通过动态定义生成消息或者动态编译)


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## 适用场景 {:.gray3}
----
{:.gray3}
* 对内部提供的Service&API {:&.zoomIn}
* 传输的数据比较大，对性能要求比较高的Service&API


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## 应用 {:.gray3}
----
{:.gray3}
* Facebook的开源的日志收集系统scribe<br>
<i class="fa fa-github"></i>https://github.com/facebook/scribe
* Evernote开放接口<br><i class="fa fa-github"></i>https://github.com/evernote/evernote-thrift
* HBase <br>http://wiki.apache.org/hadoop/Hbase/ThriftApi
<img src="/img/thrift-clients.png" height="300" />


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]
## Protobuf(Protocol Bufffers) {:.gray3}
----
{:.gray3}
* Google公司开发并开源的
* 一种轻便高效的结构化数据存储格式，可以用于结构化数据(序列化), 类似json和XML
* 根据接口定义语言(IDL Interface definition lanuage)构建
* 与语言和平台无关
* 适合做数据存储或 RPC 数据交换格式
* 消息数据序列化后的大小，比json、XML小很多
* <i class="fa fa-github"></i>github地址：https://github.com/google/protobuf
`helloworld.proto`
```
 package lm;
 message helloworld
 {
    required int32     id = 1;  // ID
    required string    str = 2;  // str
    optional int32     opt = 3;  //optional field
 }
```


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]
## Thrift vs Protobuf {:.gray3}
----
|Thrift| Protobuf
:-------|:------|:------
语言支持|Java C++ Python C++ D Dart <br> Go javascript Node.js Cocoa <br> Erlang Haskell OCaml Perl <br> PHP Ruby Smalltalk| C++ Java Python Objective-C <br> C# JavaNano JavaScript Ruby <br> Go PHP(TBD)
基本类型|bool byte integers(16/32/64) double string map<t1,t2> list<t> set<t> | bool integers(32/64) float double string bytes repeated
枚举 | yes {:.green} | yes {:.green}
常量 | yes {:.green} | no {:.red}
结构化类型 | struct | message
异常处理 | yes {:.green} | no {:.red}
编译语言 | C++ | C++


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]
## Thrift vs Protobuf {:.gray3}
----
|Thrift| Protobuf
:-------|:------|:------
RPC实现 | yes {:.green} | no (gRPC) {:.red}
结构化类型扩展 | no {:.red} | yes {:.green}
文档 | lacking | good
优点 | 更多的语言支持, 丰富的数据结构(map, set), 包括了RPC服务的实现| 数据序列化更小，文档丰富
缺点 | 文档匮乏 | 没有RPC服务实现


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## End {:.gray3}
----
# ***Thanks***