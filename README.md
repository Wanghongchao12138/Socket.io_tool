## iOS Socket.io使用 ||  socket添加请求头 || extraHeaders || 安卓 transport 相对iOS 处理方法
# socket.io 使用

socket.io 现在已经没有OC版本，只有swift版本，如需使用需要添加桥接文件

-  **socketIO.h文件导入**
-  **桥接文件的添加**
-  **socket配置选项**
-  **socket使用**

## socket.io 下载

>  [下载地址(点击跳转)](https://github.com/socketio/socket.io-client-swift)

 可直接通过cocoapods下载 
  >**注意:** use_frameworks! 这个需要添加上

  > pod 'Socket.IO-Client-Swift', '~> 13.2.0'

**use_frameworks!遇到了library not found for -lXXXXX 的解决方法**
使用 use_frameworks！ 之后 原本的pod 生成的.a文件变成的Framework，链接器就找不到你需要的这个库，所以和常规处理方法可能不太一致
> **第一步** 
 在工程的 Build Settings-> Other Linker Flags 直接删除pod的三方库
 ![这里写图片描述](https://img-blog.csdn.net/20180712142706787?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIzOTA3NDY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 **第二步**
 在工程的 Build Settings->Header Search Path 中添加缺失链接库的所在文件夹的路径。(这个因为是cocoapods 生成的库，直接添加$(inherited)）
![这里写图片描述](https://img-blog.csdn.net/20180712143031146?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIzOTA3NDY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
**第三步**
如果还有头文件报错，可以更换头文件引用方式，如图所示：
![这里写图片描述](https://img-blog.csdn.net/2018071214333515?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIzOTA3NDY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


## socketIO.h文件导入
sockeIO.h文件不会自动生成，也不会pod出来，需要自己添加或者是把github里面的类文件直接拖入,代码文件如下所示
```
#import <Foundation/Foundation.h>

//! Project version number for SocketIO-Mac.
FOUNDATION_EXPORT double SocketIO_MacVersionNumber;

//! Project version string for SocketIO-Mac.
FOUNDATION_EXPORT const unsigned char SocketIO_MacVersionString[];

// In this header, you should import all the public headers of your framework using statements like #import <SocketIO_Mac/PublicHeader.h>
```

## 添加桥接文件
socket.io 代码文件添加之后，需要创建项目桥接文件
1.文件格式： 项目名称-Bridging-Header.h
2.在项目target下的build setting -> Swift Compiler-General ->Objective-C Bridging Header将上面建立文件的目录设置上去。我的Demo头文件目录为SocketTest-Bridging-Header.h
3.在桥接文件中添加sockeIO头文件，如图
![桥接文件添加头文件引用](https://img-blog.csdn.net/20180710185648483?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIzOTA3NDY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


## socket配置选项

| 参数			     |  	  默认值 	|     作用     |
|--------------------| ---------------- | ------------|
| connect timeout | 5000 		| 设置创建连接所接收的超时时间，单位是毫秒|
| try multiple transports | true 		| 当连接超时后是否允许Socket.io以其他连接方式尝试连接|
| reconnect | true 		| 当连接终止后，是否允许Socket.io自动进行重连|
| reconnection delay | 500 		| 为Socket.io的重连设置一个时间间隔，内部会在多次重连尝试时采用该值的指数值间隔，用来避免性能损耗（500 > 1000 > 2000 > 4000 > 8000）|
| max reconnection attempts | 10 		| 设置一个重连的最大尝试次数，超过这个值后Socket.io会使用所有允许的其他连接方式尝试重连，直到最终失败|
| transports |   ['websocket', 'flashsocket', 'htmlfile', 'xhr-multipart', 'xhr-polling', 'jsonp-polling']	| 默认支持的链接方式（顺序敏感）|

这些如果需要可以选择添加，我的Demo中添加的是需要在socket连接的时候添加请求头，如果不需要可以直接不添加相关参数，直接以(2)方式直接连接即可。
(1). 需要添加请求头方法 1 
```
NSURL *url = [NSURL URLWithString:@"服务器地址:端口号"];
NSDictionary *dic =@{@"log": @YES,
                         @"forceWebsockets": @NO,
                         @"forcePolling": @NO,
                         @"compress": @YES,
                         @"reconnectAttempts":@(-1),
                         @"forceNew": @YES,
                         @"reconnectAttempts": @5,
                         @"extraHeaders": @{@"请求头key": @"请求头value"},
                         };
    _manager = [[SocketManager alloc] initWithSocketURL:url config:dic];
    _socket = [_manager socketForNamespace:@"/buyGrab"];//更改namespace 后面连接需要调整
    [_socket connect];
```
(2) 不需要添加请求头 可以直接使用
```
NSURL* url = [[NSURL alloc] initWithString:@"http://localhost:8080"];
SocketManager* manager = [[SocketManager alloc] initWithSocketURL:url config:@{@"log": @YES, @"compress": @YES}];
SocketIOClient* socket = manager.defaultSocket;
```

## socket 使用

```
// socket 连接服务器
	[socket on:@"connect" callback:^(NSArray* data, SocketAckEmitter* ack) {
	    NSLog(@"socket connected ");
	            [weakSelf.socket handleEvent:@"authenticated" data:data isInternalMessage:NO withAck:1]; // 添加接收鉴权的方法
	}];
	[socket connect];
// socket 鉴权 看情况使用
    [socket on:@"authenticated" callback:^(NSArray * data, SocketAckEmitter * ack) {
        if (data.count > 0) {
            NSLog(@"socket 鉴权结果 - %@",data);
        }
    }];
// 心跳包 具体情况看服务端发送和接收方式
    [socket on:@"ping" callback:^(NSArray * _Nonnull data, SocketAckEmitter * _Nonnull ack) {
        NSLog(@"socket - 心跳包");
        // 查看socket 连接状态
        NSLog(@"connect manager status  : %ld",(long)weakSelf.manager.status);
    }];
  // socket 断开连接
    [socket on:@"disconnect" callback:^(NSArray* data, SocketAckEmitter* ack) {
        // 断开连接
        NSLog(@"socket.io disconnect   断开连接");
        // 添加条件判断 超过5次断开连接
        if (weakSelf.connectCount > 5) {
            [weakSelf closeSocket];
        }
    }];
    
    [socket on:@"reconnect" callback:^(NSArray *data, SocketAckEmitter *ack) {
        NSLog(@"重连中...");
        if (data.count > 0) {
        // 连接成功
            weakSelf.connectCount = 0;
        }else
        {
	        // 连接失败
            weakSelf.connectCount++;
        }
        if (weakSelf.connectCount > 5) {
            [weakSelf closeSocket];
        }
        
    }];
    //连接出错了
    [_socket on:@"error" callback:^(NSArray * data, SocketAckEmitter * ack) {
        NSLog(@"socket.io error %@",data);
    }];

// 关闭socket
-(void)closeSocket{
    if (_socket) { // 如果socket 存在 再关闭 
        self.connectCount = 0;
        // 发送关闭socket 请求
        [_socket emit:@"disconnect_request" with:@[]]; 
        [_socket disconnect];
        _socket = nil;
    }
}
```
其他[^footnote].
  [^footnote]: *具体的参数调用和获取的方法可参考上面调用*.
## 添加桥接文件
socket.io 代码文件添加之后，需要创建项目桥接文件
1.文件格式： 项目名称-Bridging-Header.h
2.在项目target下的build setting -> Swift Compiler-General ->Objective-C Bridging Header将上面建立文件的目录设置上去。我的Demo头文件目录为SocketTest-Bridging-Header.h
3.在桥接文件中添加sockeIO头文件，如图
![桥接文件添加头文件引用](https://img-blog.csdn.net/20180710185648483?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIzOTA3NDY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


## socket配置选项

| 参数			     |  	  默认值 	|     作用     |
|--------------------| ---------------- | ------------|
| connect timeout | 5000 		| 设置创建连接所接收的超时时间，单位是毫秒|
| try multiple transports | true 		| 当连接超时后是否允许Socket.io以其他连接方式尝试连接|
| reconnect | true 		| 当连接终止后，是否允许Socket.io自动进行重连|
| reconnection delay | 500 		| 为Socket.io的重连设置一个时间间隔，内部会在多次重连尝试时采用该值的指数值间隔，用来避免性能损耗（500 > 1000 > 2000 > 4000 > 8000）|
| max reconnection attempts | 10 		| 设置一个重连的最大尝试次数，超过这个值后Socket.io会使用所有允许的其他连接方式尝试重连，直到最终失败|
| transports |   ['websocket', 'flashsocket', 'htmlfile', 'xhr-multipart', 'xhr-polling', 'jsonp-polling']	| 默认支持的链接方式（顺序敏感）|

这些如果需要可以选择添加，我的Demo中添加的是需要在socket连接的时候添加请求头，如果不需要可以直接不添加相关参数，直接以(3)方式直接连接即可。
(1). 需要添加请求头方法 1 
```
NSURL* url = [[NSURL alloc] initWithString:@"http://localhost:端口号"];
 SocketManager *manager = [[SocketManager alloc] initWithSocketURL:url config:@{@"log": @YES,@"compress": @YES,@"extraHeaders":@{@"请求头key":@"请求头value"}}];
 /* 
	 这里是添加的服务器地址和端口号拼接的参数，如不需要可舍弃
	 例： http://www.aaaaa.com/xxxx  
 */
 self.socket = [self.manager socketForNamespace:@"/xxxx"]; 
```
(2). 需要添加请求头方法 2
```
NSURL* url = [[NSURL alloc] initWithString:[NSString stringWithFormat:@"http://服务器地址:端口号/xxx?请求头key=%@",请求头value]];
    
 NSMutableDictionary *messageServer = [[NSMutableDictionary alloc]init];
 [messageServer setObject:@NO forKey:@"log"];
 [messageServer setObject:@YES forKey:@"compress"];
 SocketManager *manager = [[SocketManager alloc] initWithSocketURL:url config:messageServer];
 SocketIOClient *socket = manager.defaultSocket;
```
(3) 不需要添加请求头 可以直接使用
```
NSURL* url = [[NSURL alloc] initWithString:@"http://localhost:8080"];
SocketManager* manager = [[SocketManager alloc] initWithSocketURL:url config:@{@"log": @YES, @"compress": @YES}];
SocketIOClient* socket = manager.defaultSocket;
```

## socket 使用

```
// socket 连接服务器
	[socket on:@"connect" callback:^(NSArray* data, SocketAckEmitter* ack) {
	    NSLog(@"socket connected ");
	}];
	[socket connect];
// socket 鉴权 看情况使用
    [socket on:@"authenticated" callback:^(NSArray * data, SocketAckEmitter * ack) {
        if (data.count > 0) {
            NSLog(@"socket 鉴权结果 - %@",data);
        }
    }];
// 心跳包 具体情况看服务端发送和接收方式
    [socket on:@"ping" callback:^(NSArray * _Nonnull data, SocketAckEmitter * _Nonnull ack) {
        NSLog(@"socket - 心跳包");
        // 查看socket 连接状态
        NSLog(@"connect manager status  : %ld",(long)weakSelf.manager.status);
    }];
  // socket 断开连接
    [socket on:@"disconnect" callback:^(NSArray* data, SocketAckEmitter* ack) {
        // 断开连接
        NSLog(@"socket.io disconnect   断开连接");
        // 添加条件判断 超过5次断开连接
        if (weakSelf.connectCount > 5) {
            [weakSelf closeSocket];
        }
    }];
    
    [socket on:@"reconnect" callback:^(NSArray *data, SocketAckEmitter *ack) {
        NSLog(@"重连中...");
        if (data.count > 0) {
        // 连接成功
            weakSelf.connectCount = 0;
        }else
        {
	        // 连接失败
            weakSelf.connectCount++;
        }
        if (weakSelf.connectCount > 5) {
            [weakSelf closeSocket];
        }
        
    }];
    //连接出错了
    [_socket on:@"error" callback:^(NSArray * data, SocketAckEmitter * ack) {
        NSLog(@"socket.io error %@",data);
    }];

// 关闭socket
-(void)closeSocket{
    if (_socket) { // 如果socket 存在 再关闭 
        self.connectCount = 0;
        // 发送关闭socket 请求
        [_socket emit:@"disconnect_request" with:@[]]; 
        [_socket disconnect];
        _socket = nil;
    }
}
```
其他[^footnote].
  [^footnote]: *具体的参数调用和获取的方法可参考上面调用*.
