---
layout: post
title: 'iOS WebSocket 长链接'
date: 2018-03-31-iOS WebSocket
author: 李大鹏
cover: ''
tags: iOS
---
### 一、什么是 websocket ？
WebSocket protocol 是HTML5一种新的协议。它实现了浏览器与服务器全双工通信(full-duplex)。
在 WebSocket API，浏览器和服务器只需要要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。两者之间就直接可以数据互相传送。
![socket连接过程](http://files.pmpanda.com/890ac151bb9865f78ebe5940e0027f42.png)

### 二、SocketRocket（Facebook）- 最好的 WebSocket 框架
1.使用podpod管理库  

```
pod 'SocketRocket'
pod install
```
2.首先创建一个名为WebSocketManager的单例类  

```
+(instancetype)shared;
```
3.创建一个枚举,分别表示WebSocket的链接状态  

```
typedef NS_ENUM(NSUInteger,WebSocketConnectType){
    WebSocketDefault = 0,   //初始状态,未连接,不需要重新连接
    WebSocketConnect,       //已连接
    WebSocketDisconnect    //连接后断开,需要重新连接
};
```
4.创建连接  

```
//建立长连接
- (void)connectServer;
```
5.处理连接成功的结果;  

```
-(void)webSocketDidOpen:(RMWebSocket *)webSocket; //连接成功回调
```
6.处理连接失败的结果  

```
- (void)webSocket:(SRWebSocket *)webSocket didFailWithError:(NSError *)error;//连接失败回调
```
7.接收消息  

```
///接收消息回调,需要提前和后台约定好消息格式.
- (void)webSocket:(SRWebSocket *)webSocket didReceiveMessageWithString:(nonnull NSString *)string
```
8.关闭连接  

```
- (void)webSocket:(SRWebSocket *)webSocket didCloseWithCode:(NSInteger)code reason:(NSString *)reason wasClean:(BOOL)wasClean;///关闭连接回调的代理
```
9.为保持长链接的连接状态,需要定时向后台发送消息,就是俗称的:心跳包.
需要创建一个定时器,固定时间发送消息.
10.链接断开情况处理:
首先判断是否是主动断开,如果是主动断开就不作处理.
如果不是主动断开链接,需要做重新连接的逻辑.

11.补充
使用 senderheartBeat/ sendDataToServer: 方法前，需要判断 socket 是否连接，断开则重连
（如息屏或返回主页导致的链接断开）  

```
- (void)applicationDidBecomeActive:(UIApplication *)application {
    // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
    if ([WebSocketManager shared].connectType == WebSocketDisconnect) {
        [[WebSocketManager shared] connectServer];        
    }
}
```
12.代码实现  

```
// WebSocketManager.h
#import <Foundation/Foundation.h>
#import "RMWebSocket.h"

typedef NS_ENUM(NSUInteger,WebSocketConnectType){
    WebSocketDefault = 0, //初始状态,未连接
    WebSocketConnect,      //已连接
    WebSocketDisconnect    //连接后断开
};

@class WebSocketManager;
@protocol WebSocketManagerDelegate <NSObject>

- (void)webSocketManagerDidReceiveMessageWithString:(NSString *)string;

@end

NS_ASSUME_NONNULL_BEGIN

@interface WebSocketManager : NSObject

@property (nonatomic, strong) RMWebSocket *webSocket;
@property(nonatomic,weak)  id<WebSocketManagerDelegate > delegate;
@property (nonatomic, assign)   BOOL isConnect;  //是否连接
@property (nonatomic, assign)   WebSocketConnectType connectType;

+(instancetype)shared;
- (void)connectServer;//建立长连接
- (void)reConnectServer;//重新连接
- (void)RMWebSocketClose;//关闭长连接
- (void)sendDataToServer:(NSString *)data;//发送数据给服务器

@end

NS_ASSUME_NONNULL_END


// WebSocketManager.m
#import "WebSocketManager.h"

@interface WebSocketManager ()<RMWebSocketDelegate>
@property (nonatomic, strong) NSTimer *heartBeatTimer; //心跳定时器
@property (nonatomic, strong) NSTimer *netWorkTestingTimer; //没有网络的时候检测网络定时器
@property (nonatomic, assign) NSTimeInterval reConnectTime; //重连时间
@property (nonatomic, strong) NSMutableArray *sendDataArray; //存储要发送给服务端的数据
@property (nonatomic, assign) BOOL isActivelyClose;    //用于判断是否主动关闭长连接，如果是主动断开连接，连接失败的代理中，就不用执行 重新连接方法

@end
@implementation WebSocketManager

+(instancetype)shared{
    static WebSocketManager *_instance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instance = [[self alloc]init];
    });
    return _instance;
}

- (instancetype)init
{
    self = [super init];
    if(self){
        self.reConnectTime = 0;
        self.isActivelyClose = NO;

        self.sendDataArray = [[NSMutableArray alloc] init];
    }
    return self;
}

//建立长连接
- (void)connectServer{
    self.isActivelyClose = NO;

    self.webSocket.delegate = nil;
    [self.webSocket close];
    _webSocket = nil;
//    self.webSocket = [[RMWebSocket alloc] initWithURL:[NSURL URLWithString:@"https://dev-im-gateway.runxsports.com/ws/token=88888888"]];
    self.webSocket = [[RMWebSocket alloc] initWithURL:[NSURL URLWithString:@"ws://chat.workerman.net:7272"]];
    self.webSocket.delegate = self;
    [self.webSocket open];
}

- (void)sendPing:(id)sender{
    [self.webSocket sendPing:nil error:NULL];
}

#pragma mark --------------------------------------------------
#pragma mark - socket delegate
///开始连接
-(void)webSocketDidOpen:(RMWebSocket *)webSocket{

    NSLog(@"socket 开始连接");
    self.isConnect = YES;
    self.connectType = WebSocketConnect;

    [self initHeartBeat];///开始心跳

}

///连接失败
-(void)webSocket:(RMWebSocket *)webSocket didFailWithError:(NSError *)error{
    NSLog(@"连接失败");
    self.isConnect = NO;
    self.connectType = WebSocketDisconnect;

    DLog(@"连接失败，这里可以实现掉线自动重连，要注意以下几点");
    DLog(@"1.判断当前网络环境，如果断网了就不要连了，等待网络到来，在发起重连");
    DLog(@"3.连接次数限制，如果连接失败了，重试10次左右就可以了");

    //判断网络环境
    if (AFNetworkReachabilityManager.sharedManager.networkReachabilityStatus == AFNetworkReachabilityStatusNotReachable){ //没有网络

        [self noNetWorkStartTestingTimer];//开启网络检测定时器
    }else{ //有网络

        [self reConnectServer];//连接失败就重连
    }
}

///接收消息
-(void)webSocket:(RMWebSocket *)webSocket didReceiveMessageWithString:(NSString *)string{

    NSLog(@"接收消息----  %@",string);
    if ([self.delegate respondsToSelector:@selector(webSocketManagerDidReceiveMessageWithString:)]) {
        [self.delegate webSocketManagerDidReceiveMessageWithString:string];
    }
}


///关闭连接
-(void)webSocket:(RMWebSocket *)webSocket didCloseWithCode:(NSInteger)code reason:(NSString *)reason wasClean:(BOOL)wasClean{

    self.isConnect = NO;

    if(self.isActivelyClose){
        self.connectType = WebSocketDefault;
        return;
    }else{
        self.connectType = WebSocketDisconnect;
    }

    DLog(@"被关闭连接，code:%ld,reason:%@,wasClean:%d",code,reason,wasClean);

    [self destoryHeartBeat]; //断开连接时销毁心跳

    //判断网络环境
    if (AFNetworkReachabilityManager.sharedManager.networkReachabilityStatus == AFNetworkReachabilityStatusNotReachable){ //没有网络
        [self noNetWorkStartTestingTimer];//开启网络检测
    }else{ //有网络
        NSLog(@"关闭连接");
        _webSocket = nil;
        [self reConnectServer];//连接失败就重连
    }
}

///ping
-(void)webSocket:(RMWebSocket *)webSocket didReceivePong:(NSData *)pongData{
    NSLog(@"接受pong数据--> %@",pongData);
}


#pragma mark - NSTimer

//初始化心跳
- (void)initHeartBeat{
    //心跳没有被关闭
    if(self.heartBeatTimer) {
        return;
    }
    [self destoryHeartBeat];
    dispatch_main_async_safe(^{
        self.heartBeatTimer  = [NSTimer timerWithTimeInterval:10 target:self selector:@selector(senderheartBeat) userInfo:nil repeats:true];
        [[NSRunLoop currentRunLoop]addTimer:self.heartBeatTimer forMode:NSRunLoopCommonModes];
    })

}
//重新连接
- (void)reConnectServer{
    if(self.webSocket.readyState == RM_OPEN){
        return;
    }

    if(self.reConnectTime > 1024){  //重连10次 2^10 = 1024
        self.reConnectTime = 0;
        return;
    }

    WS(weakSelf);
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(self.reConnectTime *NSEC_PER_SEC)), dispatch_get_main_queue(), ^{

        if(weakSelf.webSocket.readyState == RM_OPEN && weakSelf.webSocket.readyState == RM_CONNECTING) {
            return;
        }

        [weakSelf connectServer];
        //        CTHLog(@"正在重连......");

        if(weakSelf.reConnectTime == 0){  //重连时间2的指数级增长
            weakSelf.reConnectTime = 2;
        }else{
            weakSelf.reConnectTime *= 2;
        }
    });

}

//发送心跳
- (void)senderheartBeat{
    //和服务端约定好发送什么作为心跳标识，尽可能的减小心跳包大小
    WS(weakSelf);
    dispatch_main_async_safe(^{
        if(weakSelf.webSocket.readyState == RM_OPEN){
            [weakSelf sendPing:nil];
        }
    });
}

//没有网络的时候开始定时 -- 用于网络检测
- (void)noNetWorkStartTestingTimer{
    WS(weakSelf);
    dispatch_main_async_safe(^{
        weakSelf.netWorkTestingTimer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:weakSelf selector:@selector(noNetWorkStartTesting) userInfo:nil repeats:YES];
        [[NSRunLoop currentRunLoop] addTimer:weakSelf.netWorkTestingTimer forMode:NSDefaultRunLoopMode];
    });
}
//定时检测网络
- (void)noNetWorkStartTesting{
    //有网络
    if(AFNetworkReachabilityManager.sharedManager.networkReachabilityStatus != AFNetworkReachabilityStatusNotReachable)
    {
        //关闭网络检测定时器
        [self destoryNetWorkStartTesting];
        //开始重连
        [self reConnectServer];
    }
}

//取消网络检测
- (void)destoryNetWorkStartTesting{
    WS(weakSelf);
    dispatch_main_async_safe(^{
        if(weakSelf.netWorkTestingTimer)
        {
            [weakSelf.netWorkTestingTimer invalidate];
            weakSelf.netWorkTestingTimer = nil;
        }
    });
}


//取消心跳
- (void)destoryHeartBeat{
    WS(weakSelf);
    dispatch_main_async_safe(^{
        if(weakSelf.heartBeatTimer)
        {
            [weakSelf.heartBeatTimer invalidate];
            weakSelf.heartBeatTimer = nil;
        }
    });
}


//关闭长连接
- (void)RMWebSocketClose{
    self.isActivelyClose = YES;
    self.isConnect = NO;
    self.connectType = WebSocketDefault;
    if(self.webSocket)
    {
        [self.webSocket close];
        _webSocket = nil;
    }

    //关闭心跳定时器
    [self destoryHeartBeat];

    //关闭网络检测定时器
    [self destoryNetWorkStartTesting];
}


//发送数据给服务器
- (void)sendDataToServer:(NSString *)data{
    [self.sendDataArray addObject:data];

    //[_webSocket sendString:data error:NULL];

    //没有网络
    if (AFNetworkReachabilityManager.sharedManager.networkReachabilityStatus == AFNetworkReachabilityStatusNotReachable)
    {
        //开启网络检测定时器
        [self noNetWorkStartTestingTimer];
    }
    else //有网络
    {
        if(self.webSocket != nil)
        {
            // 只有长连接OPEN开启状态才能调 send 方法，不然会Crash
            if(self.webSocket.readyState == RM_OPEN)
            {
//                if (self.sendDataArray.count > 0)
//                {
//                    NSString *data = self.sendDataArray[0];
                    [_webSocket sendString:data error:NULL]; //发送数据
//                    [self.sendDataArray removeObjectAtIndex:0];
//
//                }
            }
            else if (self.webSocket.readyState == RM_CONNECTING) //正在连接
            {
                DLog(@"正在连接中，重连后会去自动同步数据");
            }
            else if (self.webSocket.readyState == RM_CLOSING || self.webSocket.readyState == RM_CLOSED) //断开连接
            {
                //调用 reConnectServer 方法重连,连接成功后 继续发送数据
                [self reConnectServer];
            }
        }
        else
        {
            [self connectServer]; //连接服务器
        }
    }
}

@end
```
