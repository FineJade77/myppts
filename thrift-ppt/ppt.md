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
* 一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议
* 通俗一点就是：调用远程计算机上的程序，就像调用本地程序一样。
<!-- * ![](/img/rpc.png) -->
* 流程： <img src="/img/rpc.png" height="300">
* 常见的RPC框架: Apache的 *Thrift*、google的gRPC、阿里的Dubbo、Apache的Avro等

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
* 由Fackbook开发,已经开源并且加入到Apache项目 {:&.moveIn}
* 支持多种编程语言的RPC(Remote Procedure Call远程过程调用)框架
* 利用自身的协议栈和代码生成工具，根据接口定义语言(IDL Interface definition lanuage)构建,可以在C++,Java,Python等程序语言之间进行无缝结合的高效服务
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
    </br> <span class="blue">list&lt;t&gt;  set&lt;t&gt;  map&lt;t, t&gt;</span>
* 结构体(struct)
* 异常(exception)
* 服务(service)


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
            <span class="gray3" style="font-size: large">根据 Thrift 定义的服务接口描述文件生成的客户端和服务器端代码</span>
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


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## Ttransport 传输层 {:.gray3}
----
{:.gray3}
* 负责Thrift的字节流(Byte Stream)数据在IO模块上的传输
* 主要传输类型:
    * TSocket 使用阻塞的Socket进行IO传输 {:&.zoomIn.blue}
    * TFramedTransport 使用一个带Buffer的Socket进行IO传输，使用NoblockingServer的时候会需要使用TFramedTransport {:.blue}
    * TFileTransport 使用文件对象进行IO传输 {:.blue}
    * TZlibTransport 与其他的TTransport配合使用，完成数据的压缩传输 {:.blue}

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
    * TSimpleServer 单线程服务器端使用标准的阻塞式 I/O {:&.zoomIn.blue}
    * TThreadPoolServer 多线程服务器端使用标准的阻塞式 I/O {:.blue}
    * TNonblockingServer 多线程服务器端使用非阻塞式 I/O {:.blue}
    * THttpServer 基于HTTP的服务器

[note]
<img src="/img/thrift-architecture.jpg" height="500">
[/note]


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
    transport = TSocket.TServerSocket(host="0.0.0.0" ,port=9090)
    tfactory = TTransport.TBufferedTransportFactory()
    pfactory = TBinaryProtocol.TBinaryProtocolFactory()

    server = TServer.TSimpleServer(processor, transport, tfactory, pfactory)
    server.serve()
```


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## 优点和缺点 {:.gray3}
---
{:.blue}
* <span class="green">优点</span>
    * 通过thrift文件生成目标代码，简单易用
    * 支持多语言 支持多种序列化方式 支持多种通讯协议 支持多线程和非阻塞式服务端
    * 客户端调用更直观，初步的类型和结构校验
    * 性能比较好
* <span class="gray2">缺点</span>：
    * 文档少文档少
    * 只能由客户端发起请求，不支持双向通讯
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
## Thrift vs Protobuf(Protocol Bufffers) {:.gray3}
----
* Protobuf


[slide style="background-image:url('/img/le-ppt.png');background-size:100% 100%"]

## End {:.gray3}
----
# ***Thanks***