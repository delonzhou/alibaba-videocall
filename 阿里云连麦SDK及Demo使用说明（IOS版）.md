#IOS版本连麦SDK#
<br>
##概述##

AlivcVideoChat是一款适用于 iOS 平台的、提供了网络连麦功能的SDK，它提供了丰富的接口供开发者进行二次开发。它不仅支持单纯的主播推流、观众播放的功能，还支持主播与主播之间、主播与观众之间的视频互动，同时第三方观众还可以通过服务器提供的混流功能观看这些互动的场景。它的特点是短延时、支持弱网环境、抗网络抖动能力强。

###功能特点###

* 使用标准的rtmp协议传输，视频格式为h264，音频格式为aac
* 支持armv7和arm64指令
* 支持美颜功能
* 支持多种功能选择与参数配置。如：支持打开/关闭美颜功能、打开/关闭静音功能，支持推流和播放器的参数设置等。
* 提供接口实时获取有关主播端和观众端的状态信息。如：推流端的上传帧率和码率、播放端的下载帧率和码率、播放延迟等，有助于监控连麦的整个流程，给使用者提供实时的状态信息和合理的提醒（如：网络差，建议退出连麦等）。
* 推流过程中会根据实时的网络带宽动态调整编码器参数，支持弱网环境下使用。
* 连麦过程中采用音频优先上传的策略，减少卡顿率；提供丢帧策略，减少累积延迟。

###注意事项###

* 目前连麦的编码器使用的是软编码，为避免CPU的性能不足或手机温度过高，视频分辨率暂时只支持848x480和640x360。 
* 音频编码采用硬件编码，格式为AAC-LC。由于目前使用的回声消除算法对于音频采样率最大支持到32000Hz，过高的采样率对于音频的质量并无帮助，所以当前版本中音频的采样率固定为32000Hz。
* 操作系统版本要求ios8.0以上。 

##快速接入##
###开发环境配置###

* 运行环境（XCode6.0以上版本，iOS SDK8.0以上版本）
* 硬件CPU支持ARMv7、ARMv7s或ARM64

###sdk下载###

###快速安装###

##demo使用说明##

demo实现了连麦的基本场景，包括主播端的推流、用户端的播放、主播与主播之间的连麦、主播与观众之间的连麦。

程序整体使用了一个简易的MVC架构，其中view处于Views文件夹下，Controller处于Controller文件夹下。

信号传递这里依赖了第三方通信库——环信，主要使用了环信的聊天室功能及个人消息功能。如果用户有其他的选择，可以替换。

网络层主要使用开源框架AFNetworking，建议用户开发时做一层manager的封装，方便使用。

###基础业务###
**登录**

POST请求登录接口——NSString *createURLString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"login"];


```
    __weak __typeof__(self) weakSelf = self;
    [[AFHTTPSessionManager manager] POST:createURLString parameters:param progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        
        // 用户uid
        NSString *uid = [[responseObject[@"data"] objectForKey:@"id"] stringValue];
        // 用户名
        NSString *name = [responseObject[@"data"] objectForKey:@"name"];
        
        // 环信用户名（环信的注册在appSever完成）
        NSDictionary *emDictionary = [responseObject[@"data"] objectForKey:@"em"];
        NSLog(@"%@", [emDictionary objectForKey:@"username"]);
    	  // 登录环信(先退出本地保存的环信账号)
        [[EMClient sharedClient] logout:YES];
        EMError *error = [[EMClient sharedClient] loginWithUsername:[emDictionary objectForKey:@"username"] password:@"12345678"];
        if (!error)
        {
            [[EMClient sharedClient].options setIsAutoLogin:NO];
        } else {
            NSLog(@"登录EM失败:%u", error.code);
        }
        
        dispatch_async(dispatch_get_main_queue(), ^{
            ...// 省略
        });
        
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        
        dispatch_async(dispatch_get_main_queue(), ^{
            ...// 省略
        });
    }];
```

**直播列表**

POST请求直播列表接口——NSString *urlString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"live/list"];
    
```
    __weak __typeof__(self) weakSelf = self;
    [[AFHTTPSessionManager manager] POST:urlString parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        
        // 清空data数据
        [self.listDataArray removeAllObjects];
        // Array data数据赋值
        for (NSDictionary *dictionary in [responseObject objectForKey:@"data"]) {
            [self.listDataArray addObject:dictionary];
        }
        // 停止下拉刷新
        [weakSelf.refreshHeaderView endRefreshing];
        // TableView重新加载数据
        [weakSelf.listTableView reloadData];
        
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        
        NSLog(@"播放列表获取失败:%@", error);
        [weakSelf.refreshHeaderView endRefreshing];
        
    }];

```
###主播业务###

在StartLiveViewController中调用下列代码，需要引入头文件#import \<AlivcVideoChat/alivcVideoChatHost.h>

**创建直播**

```
	/**
 	*  创建直播
 	*/
	- (void)createPublisher {
    
    // init
    self.publiserVideoCall = [[AlivcVideoChatHost alloc] init];
    // 视频宽高
    int width = 640; // 480
    int height = 360; // 848

    // 参数配置
    ...//省略
    
    // 预览
    int ret;
    ret = [self.publiserVideoCall prepareToPublish:self.startLiveView.publisherView width:width height:height publisherParam:self.publisherParam];
    
    // error Notification
    [self addVideoChatObserver];
}
```
            
**开始推流**

POST请求创建直播接口--NSString *createURLString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"live/create"];
	            
```
	NSString *createURLString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"live/create"];
    NSDictionary *param = @{@"uid":self.uid,
                            @"desc":self.startLiveView.descTextFeild.text};
    
    __weak __typeof__(self) weakSelf = self;
    [[AFHTTPSessionManager manager] POST:createURLString parameters:param progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
       // 创建成功 获取推流URL
        weakSelf.rtmpURLString = [responseObject[@"data"] objectForKey:@"rtmpUrl"];
        weakSelf.roomId = [responseObject[@"data"] objectForKey:@"roomId"];
        
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        
        dispatch_async(dispatch_get_main_queue(), ^{
           ...//省略
        });
    }];
    
        
    //获取推流URL后，调用startToPublish接口开始推流
    int ret = [self.publiserVideoCall startToPublish:self.rtmpURLString];

```

                     
**开始连麦：主播作为邀请方**

在连麦的过程中，主播可以作为邀请方邀请观众和其他主播参与连麦。

1） 发送连麦邀请

调用邀请主播连麦接口--NSString *urlString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"videocall/invite"];

```
    NSString *urlString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"videocall/invite"];
    // 邀请和被邀请主播的uid 混流类型（画中画和左右模式） 邀请类型（1：观众 2：主播）
    NSDictionary *param = @{@"inviterUid":self.uid,
                            @"inviteeUid":inviteeUidString,
                            @"type":@"picture_in_picture",
                            @"inviterType":@"2"};
    
    [[AFHTTPSessionManager manager] POST:urlString parameters:param progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        ...//发送成功
        
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        
        dispatch_async(dispatch_get_main_queue(), ^{
          ...//发送失败
    }];
```

2） 收到允许连麦的通知，获取对方视频URL

```
	- (void)inviteFeedBackWithStatus:(NSString *)status inviterUid:(NSString *)inviterUid {
	    
	    NSString *urlString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"videocall/feedback"];
	    
	    NSDictionary *param = @{@"inviterUid":inviterUid,
	                            @"inviteeUid":self.uid,
	                            @"status":status,
	                            @"type":@"picture_in_picture"};    
	    __weak __typeof__(self)weakSelf = self;
	    [[AFHTTPSessionManager manager] POST:urlString parameters:param progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
	    
	    // 判断是否允许连麦
	        if ([[responseObject allKeys] containsObject:@"data"] && [status intValue] == 1) {
	        // 获取对方的视频播放地址，并开始连麦
	            NSString *inviterPlayUrl = [responseObject[@"data"] objectForKey:@"inviterPlayUrl"];
	            [weakSelf createLiveCallWithInviteePlayUrl:inviterPlayUrl];
	        }
	        
	        // code 3404 错误码 当前未推流
	        if ([responseObject[@"code"] intValue] == 3404) {
	            AlivcLiveAlertView *alert = [[AlivcLiveAlertView alloc] initWithTitle:@"提示" icon:nil message:@"当前您的推流状态异常，请点击推流按钮重新推流" delegate:nil buttonTitles:@"OK", nil];
	            [alert showInView:self.startLiveView];
	        }
	        
	    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
	        
	        dispatch_async(dispatch_get_main_queue(), ^{
	            ...//省略
	        });
	    }];
	
	}
```

3） 开始连麦，播放对方视频

```
    // 参数配置             
    ...//省略
	[self.publiserVideoCall setPlayerParam:self.playerParam];
	
    // 开始连麦            
	int ret = [self.publiserVideoCall launchChat:[NSURL URLWithString:inviteePlayUrl] view:self.startLiveView.inviteView];
	if (ret != 0) {
       // SDK初始化失败             
		AlivcLiveAlertView *alert = [[AlivcLiveAlertView alloc] initWithTitle:@"提示" icon:nil message:[NSString stringWithFormat:@"SDK连麦初始化失败,ret=%d", ret] delegate:nil buttonTitles:@"OK",nil];
		[alert showInView:self.startLiveView];
   }
```

**开始连麦：主播作为被邀请方**

在连麦的过程中，主播可以作为被邀请方和其他主播进行连麦。

1） 收到连麦邀请

```
    - (void)didReceiveMessages:(NSArray *)aMessages {
    	 // 收到推送 解析推送数据
        for (EMMessage *message in aMessages) {
        EMMessageBody *msgBody = message.body;
        
        switch (msgBody.type) {
            case EMMessageBodyTypeText:
            {
                // 收到的文字消息
                EMTextMessageBody *textBody = (EMTextMessageBody *)msgBody;
                NSString *txt = textBody.text;
                NSLog(@"收到的文字是 txt -- %@",txt);
                // NSString转换为NSDictionary
                NSData *jsonData = [txt dataUsingEncoding:NSUTF8StringEncoding];
                NSError *err;
                NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:jsonData options:NSJSONReadingMutableContainers error:&err];
                if (err) {
                    NSLog(@"推送信息解析失败");
                    return;
                }
                // 收到多条连麦请求信息。需要抢麦
                if ([[dic objectForKey:@"type"] integerValue] == 1 && aMessages.count > 1) {
                    
                    isRob = YES;
                    [self.customConnectArray addObject:dic];
                    
                	}
            	}
                break;
                
            default:
                break;
        	}
    	}
    
    	// 抢麦
    	if (isRob) {
        	AlivcLiveAlertView *alert = [[AlivcLiveAlertView alloc] initWithTitle:@"提示" icon:nil message:@"多人抢麦" delegate:self buttonTitles:@"全部否决",@"查看列表",nil];
        	alert.tag = 30003;
        	[alert showInView:self.startLiveView];
    	}
	}

	/**
	*  消息区分类型，做相应操作
	*/
	- (void)classifyMessagesWithMessageDic:(NSDictionary *)dic {
	    
	    NSInteger type = [[dic objectForKey:@"type"] integerValue];
	    
	    switch (type) {
	        case 1:
	        {
	            // 收到连麦通知
	            NSString *inviterName = [dic[@"data"] objectForKey:@"inviterName"];
	
	            if (self.startLiveView.connectBtn.selected) {
	                // 当前在连麦状态  直接拒绝连麦
	                AlivcLiveAlertView *alert = [[AlivcLiveAlertView alloc] initWithTitle:@"提示" icon:nil message:[NSString stringWithFormat:@"%@向你发起连麦,由于当前在连麦中，已拒绝", inviterName] delegate:nil buttonTitles:@"OK",nil];
	                [alert showInView:self.startLiveView];
	                [self inviteFeedBackWithStatus:@"2" inviterUid:[dic[@"data"] objectForKey:@"inviterUid"]];
	                return;
	            }
	            
	            // 弹窗提示连麦请求操作
	            NSString *inviterUid = [dic[@"data"] objectForKey:@"inviterUid"];
	            AlivcLiveAlertView *alert = [[AlivcLiveAlertView alloc] initWithTitle:@"收到连麦请求" icon:nil message:inviterUid delegate:self buttonTitles:@"拒绝连麦", @"同意连麦",nil];
	            alert.tag = 30001;
	            [alert showInView:self.startLiveView];
	        }
	            break;
	}

```   

2） 反馈是否同意连麦

```  
- (void)inviteFeedBackWithStatus:(NSString *)status inviterUid:(NSString *)inviterUid {
    
    NSString *urlString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"videocall/feedback"];
    
    NSDictionary *param = @{@"inviterUid":inviterUid,
                            @"inviteeUid":self.uid,
                            @"status":status,
                            @"type":@"picture_in_picture"};// 混流类型 picture_in_picture 画中画 side_by_side 左右两边;
    
    __weak __typeof__(self)weakSelf = self;
    [[AFHTTPSessionManager manager] POST:urlString parameters:param progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        
        NSLog(@"连麦反馈:%@", responseObject);
        
        if ([[responseObject allKeys] containsObject:@"data"] && [status intValue] == 1) {
            NSString *inviterPlayUrl = [responseObject[@"data"] objectForKey:@"inviterPlayUrl"];
            
            [weakSelf createLiveCallWithInviteePlayUrl:inviterPlayUrl];
        }        
	}
}
```  

3） 收到连麦成功通知，开始播放对方视频

```  
- (void)createLiveCallWithInviteePlayUrl:(NSString *)inviteePlayUrl {

	...//省略

	int ret = [self.publiserVideoCall launchChat:[NSURL URLWithString:inviteePlayUrl] view:self.startLiveView.inviteView];

}
```  
	                                                 
**结束连麦**

```  
	/**
	 *  关闭连麦请求
	 */
	 - (void)closeInviteWithcloseRoomId:(NSString *)closeRoomId notifiedRoomId:(NSString *)notifiedRoomId{
	    
	    NSString *urlString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"videocall/close"];
	    // 关闭连麦的双方roomId
	    NSDictionary *param = @{@"closeRoomId":closeRoomId,
	                            @"notifiedRoomId":notifiedRoomId};
	    
	    [[AFHTTPSessionManager manager] POST:urlString parameters:param progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
	        
	        ...//关闭请求成功 结束连麦
	        
	    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
	        
	        dispatch_async(dispatch_get_main_queue(), ^{
	            ...//省略
	        });
	    }];
	}

	/**
 	 *  结束连麦
 	 */
	- (void)closeLiveCall {
	    self.inviteState = ALIVC_INVITE_NONE;
	    [self.startLiveView.connectBtn setSelected:NO];
	    [self.startLiveView.inviteView removeFromSuperview];
	    if (self.publiserVideoCall) {
	        int ret = [self.publiserVideoCall abortChat];
	    }
	}
```


**停止推流**
    
```
	// 结束本次推流
    [self.publiserVideoCall stopPublishing];
```
                     
**关闭直播**
    
```
	/**
	 *  离开直播（完全关闭）
	 */
	- (void)finishLive {
	    // 移除通知
	    [self removeVideoChatObserver];
	    if (self.publiserVideoCall) {
	    	 // 结束直播
	        [self.publiserVideoCall finishPublishing];
	        self.publiserVideoCall = nil;
	    }
	    
	    // 移除环信delegate
	    [[EMClient sharedClient].chatManager removeDelegate:self];
	}

```                     

**异常处理**

添加Notification，监测直播的状态，对异常情况进行处理，部分重要通知如下

```
	- (void) addVideoChatObserver {
		// 推流连接失败
	    [[NSNotificationCenter defaultCenter] addObserver:self
                                         selector:@selector(OnPublishConnectError:)                                       name:AlivcVideoChatPublisherOpenFailed object:self.publiserVideoCall];
                                         
		// 推流连接超时
	    [[NSNotificationCenter defaultCenter] addObserver:self                 selector:@selector(OnNetworkTimeout:) name:AlivcVideoChatPublisherSendDataTimeout object:self.publiserVideoCall];
	    
		// 网速太慢
	    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(OnNetworkSlow:)                     name:AlivcVideoChatPublisherNetSpeedPoor object:self.publiserVideoCall];
	    
		// 播放器获取不到视频（播放失败）
	    [[NSNotificationCenter defaultCenter] addObserver:self             selector:@selector(OnPlayerOpenFailed:) name:AlivcVideoChatPlayerOpenFailed object:self.publiserVideoCall];
	}
```

###观众业务###

参考LiveRoomViewController，需要引入头文件#import \<AlivcVideoChat/alivcVideoChatParter.h> \<AlivcVideoChat/alivcVideoChatParam.h>

**进入观看**

创建播放器

```
	/**
	 *  创建player
	 */
	- (void)createMediaPlayer {
		// init
	    self.mediaPlayerCall = [[AlivcVideoChatParter alloc] init];
	    
		//参数配置
		...//省略
		
	    [self.mediaPlayerCall setPlayerParam:self.playerParam];
	    // 开始播放
	    int err = [self.mediaPlayerCall startToPlay:[NSURL URLWithString:self.playUrl] view:self.liveRoomView.mediaPalyerView];
	    if(err != 0) {
	        NSLog(@"preprare failed,error code is %d",(int)err);
	        return;
	    }
	    // 添加播放器通知
	    [self addVideoChatObserver];
	}

```

                     
**发送评论**

1）调用评论接口--NSString *urlString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"live/comment"];

```
	/**
	 *  评论Request
	 */
	- (void)commentRequestWithComment:(NSString *)comment {
	    NSString *urlString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"live/comment"];
	    // 用户uid 当前直播间roomId 评论内容
	    NSDictionary *param = @{@"uid":self.uid,
	                            @"roomId":self.roomId,
	                            @"comment":comment};
	    [[AFHTTPSessionManager manager] POST:urlString parameters:param progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
	        
	    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
	        
	        NSLog(@"评论失败 error:%@", error);
	    }];
	}
```
2）收到评论之后在TableView聊天界面进行展示

```
	// 评论
	NSString *name = [dic[@"data"] objectForKey:@"name"];
	NSString *comment = [dic[@"data"] objectForKey:@"comment"];
	// 如果当前弹幕超过30条则清屏
	if (self.chatDataArray.count >= 30) {
		[self.chatDataArray removeAllObjects];
	}
	// data添加数据
	[self.chatDataArray addObject:[NSString stringWithFormat:@"%@:%@", name, comment]];
	// 重新加载TableView
	[self.liveRoomView.chatTableView reloadData];
	// TableView滑动到底部 使用滑动动画
	[self.liveRoomView.chatTableView scrollToRowAtIndexPath:[NSIndexPath indexPathForRow:(self.chatDataArray.count - 1) inSection:0] atScrollPosition:(UITableViewScrollPositionBottom) animated:YES];
```
                     
**点赞**

1）调用点赞接口--NSString *urlString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"live/like"];

```
	/**
	 *  点赞Request
	 */
	- (void)spotRequest {
	    
	    NSString *urlString = [NSString stringWithFormat:@"%@%@", kQPLiveHost, @"live/like"];
	    // 用户uid 当前直播间roomId
	    NSDictionary *param = @{@"uid":self.uid,
	                            @"roomId":self.roomId};
	    [[AFHTTPSessionManager manager] POST:urlString parameters:param progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
	        
	    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
	        
	        NSLog(@"点赞失败 error:%@", error);
	    }];
	}
```

2）点赞动画展示（屏幕底部弹出随机颜色的心形图案，按照轨迹飘动效果）

```
	- (void)spotButtonAction:(UIButton *)sender {
	    // init飘心动画组件
	    AlivcLiveSpotView* spot = [[AlivcLiveSpotView alloc] initWithFrame:CGRectMake(0, 0, 36, 36)];
	    // 添加
	    [self.view addSubview:spot];
	    // frame
	    CGPoint fountainSource = CGPointMake(kAlivcLiveScreenWidth - 100, kAlivcLiveScreenHeight - 36/2.0 - 10);
	    spot.center = fountainSource;
	    // 开始动画
	    [spot animateInView:self.view];
	}
```



**邀请主播连麦**

1）发送连麦邀请

参考主播业务的连麦邀请代码

2）收到连麦反馈

参考主播业务的连麦反馈代码

3）开始连麦，进行推流

```
	/**
	 *  开始连麦
	 */
	- (void)createLiveCallWithPushUrl:(NSString *)url playUrl:(NSString *)playUrl {
	    
	    // 连麦配置dictionary
		self.publisherParam = [[NSMutableDictionary alloc] init];

		//参数配置
		...//省略

		// 开始连麦 参数：推流url width height 预览view；拉流url 播放view
		int ret = [self.mediaPlayerCall onlineChat:url width:width height:height preview:self.pushView publisherParam:self.publisherParam playerUrl:[NSURL URLWithString:playUrl]];
		
		...//UI省略
	}
```


**结束连麦**

网络请求部分参考主播连麦，观众端SDK结束连麦操作如下

```
	/**
	 *  结束连麦
	 */
	- (void)closeLiveCall {
	    // 结束连麦
	    [self.mediaPlayerCall offlineChat];
	    // 移除推流窗口 推流功能button
	    [self.pushView removeFromSuperview];
	    self.liveRoomView.mediaPalyerView.frame = [UIScreen mainScreen].bounds;
	    [self.liveRoomView.skinButton removeFromSuperview];
	    [self.liveRoomView.toggleButton removeFromSuperview];
	    
	    [self.liveRoomView.connectBtn setSelected:NO];
	}
```


**退出观看**

```
	/**
	 *  关闭player
	 */
	- (void)closeMediaPlayer {
	    
	    if(self.mediaPlayerCall != nil) {
	    	  // 如果在连麦中，先停止连麦
	        [self.mediaPlayerCall offlineChat];
	        // 停止播放
	        [self.mediaPlayerCall stopPlaying];
	    }
	    // 移除通知
	    [self removeVideoChatObserver];
	    // 置空
	    self.mediaPlayerCall = nil;
	    // 退出环信直播间
	    EMError *error = nil;
	    [[EMClient sharedClient].roomManager leaveChatroom:self.roomId error:&error];
	}
```


**异常处理**

添加Notification，监测直播的状态，对异常情况进行处理，部分重要通知如下

```
	- (void) addVideoChatObserver
	{
		// 推流连接失败
	    [[NSNotificationCenter defaultCenter] addObserver:self                         selector:@selector(OnPublishConnectError:)   name:AlivcVideoChatPublisherOpenFailed object:self.publiserVideoCall];
	    
		// 推流连接超时
	    [[NSNotificationCenter defaultCenter] addObserver:self             selector:@selector(OnNetworkTimeout:) name:AlivcVideoChatPublisherSendDataTimeout object:self.publiserVideoCall];
	    
		// 网速太慢
	    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(OnNetworkSlow:) name:AlivcVideoChatPublisherNetSpeedPoor object:self.publiserVideoCall]; 
	    
		// 播放器获取不到视频（播放失败）
	    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(OnPlayerOpenFailed:) name:AlivcVideoChatPlayerOpenFailed object:self.publiserVideoCall];
	}
	
```

###参数配置与功能选择###
推流和播放可以进行一些参数配置，以及一些其他扩展功能如摄像头的调整、静音等可以选择使用。

**推流参数配置**

```
    //设置前置摄像头是否镜像
    NSNumber* frontCameraMirror = [[NSNumber alloc] initWithBool:YES];
    [self.publisherParam setObject:frontCamera forKey:ALIVC_PUBLISHER_PARAM_FRONTCAMERAMIRROR];
    
    //设置推流超时时间
    NSNumber* uploadTimeout = [[NSNumber alloc] initWithInt:5000];
    [self.publisherParam setObject:uploadTimeout forKey:ALIVC_PUBLISHER_PARAM_UPLOADTIMEOUT];
    
    //设置推流最大码率
    NSNumber* maxBitRate = [[NSNumber alloc] initWithInt:600];
    [self.publisherParam setObject:maxBitRate forKey:ALIVC_PUBLISHER_PARAM_MAXBITRATE];
    
    //设置推流最小码率
    NSNumber* minBitRate = [[NSNumber alloc] initWithInt:200];
    [self.publisherParam setObject:minBitRate forKey:ALIVC_PUBLISHER_PARAM_MINBITRATE];
  
  //设置推流初始码率
    NSNumber* originalBitRate = [[NSNumber alloc] initWithInt:400];
    [self.publisherParam setObject:originalBitRate forKey:ALIVC_PUBLISHER_PARAM_ORIGINALBITRATE];
    
    //设置是横屏推流还是竖屏推流
    NSNumber* landscape = [[NSNumber alloc] initWithInt:NO];
    [self.publisherParam setObject:landscape forKey:ALIVC_PUBLISHER_PARAM_LANDSCAPE];
    
    //设置推流是前置摄像头还是后置摄像头
    NSNumber* cameraPosition = [[NSNumber alloc] initWithInt:cameraPositionFront];
    [self.publisherParam setObject:cameraPosition forKey:ALIVC_PUBLISHER_PARAM_CAMERAPOSITION];
    
```

**播放参数配置**

```
    self.playerParam = [[NSMutableDictionary alloc] init];
    
    //设置播放丢帧的时间阈值，缓冲区超过5s，则进行丢帧
    NSNumber* dropBufferDuration = [[NSNumber alloc] initWithInt:5000];
    [self.playerParam setObject:dropBufferDuration forKey:ALIVC_PLAYER_PARAM_DROPBUFFERDURATION];
    
    //设置播放渲染模式
    NSNumber* scalingMode = [NSNumber numberWithInt:scalingModeAspectFit];
    [self.playerParam setObject:scalingMode forKey:ALIVC_PLAYER_PARAM_SCALINGMODE];
    
    //设置播放下载超时时间
    NSNumber* downloadTimeout = [[NSNumber alloc] initWithInt:15000];
    [self.playerParam setObject:downloadTimeout forKey:ALIVC_PLAYER_PARAM_DOWNLOADIMEOUT];
    
    [self.publiserVideoCall setPlayerParam:self.playerParam];
    
```

**Filter配置**

使用NSDictionary的方式，以便于后续的扩展.目前暂时只支持“打开/关闭美颜”这一个配置。

```
    NSNumber* number = [[NSNumber alloc] initWithBool:sender.selected];
    NSString* key = ALIVC_FILTER_PARAM_BEAUTY_ON;
    NSDictionary* dic = [[NSDictionary alloc] initWithObjectsAndKeys:number, key,nil];
    [self.publiserVideoCall setFilterParam:dic];
    
```

**摄像头功能配置**

摄像头的功能包括有：切换前后摄像头、放大、聚焦等功能。具体的使用请参考接口说明。

1）切换前后摄像头

``` 
- (int) switchCamera;

```

2）放大摄像头

``` 
- (int) zoomCamera:(CGFloat)zoom;

```

3）聚焦摄像头

``` 
- (int) focusCameraAtAdjustedPoint:(CGPoint)point autoFocus:(BOOL)autoFocus;

```

**静音功能**

静音则指推流的时候不把音频发送出去，观众则听不到音频。

``` 
@property(nonatomic, readwrite)  BOOL publisherMuteMode;

```

**性能参数**

在推流和播放的时候，可以获取到一些性能参数，以便能够知道推流和播放的状态，异常情况，以及性能情况等。具体请参考接口文档。

``` 
//获取推流的性能参数值
-(AlivcPublisherPerformanceInfo*) getPublisherPerformanceInfo;

//获取播放的性能参数值
-(AlivcPlayerPerformanceInfo*) getPlayerPerformanceInfo;

```


##接口说明##

针对连麦场景中存在的主播和观众这两个不同的角色，SDK中提供了两个类AliVcVideoChatHost、AlivcVideoChatParter分别来实现主播端、观众端的各项功能。同时，我们对连麦过程中的各种通知进行了定义，并且提供了接口来获取连麦过程中推流和播放的状态信息。

###AlivcVideoChatHost的接口和事件通知###

接口名称|功能描述
-------|------
prepareToPublish	| 准备推流，建立预览界面
startToPublish	| 开始推流
stopPublishing	| 结束推流
finishPublishing	| 结束推流预览
launchChat	| 开始连麦
abortChat	| 结束连麦
setPlayerParam	| 设置连麦后播放参数
switchCamera	| 切换前后摄像头
zoomCamera	| 缩放摄像头
focusCameraAtAdjustedPoint	| 聚焦摄像头到某个位置
setFilterParam	| 设置滤镜参数
getPublisherPerformanceInfo	| 获取推流性能参数
getPlayerPerformanceInfo	| 获取连麦后播放性能参数
publisherMuteMode	| 推流是否静音
getSDKVersion	 | 获取SDK版本号

事件通知 | 内容描述
-------|------
AlivcVideoChatMemoryPool |	内存不足
AlivcVideoChatPublisherOpenFailed |	推流端打开失败。可能是网络未连接或推流地址错误
AlivcVideoChatPublisherSendDataTimeout |	推流端发送数据超时
AlivcVideoChatPublisherNetworkPool	| 推流端网络差
AlivcVideoChatPublisherVideoCaptureDisabled	| 视频采集被禁止
AlivcVideoChatPublisherAudioCaptureDisabled	| 音频采集被禁止
AlivcVideoChatPublisherVideoEncoderInitFailed	| 视频端初始化失败
AlivcVideoChatPublisherAudioEncoderInitFailed	| 音频端初始化失败
AlivcVideoChatPublisherEncodeVideoFailed	| 推流端视频编码失败
AlivcVideoChatPublisherEncodeAudioFailed	| 推流端音频编码失败
AlivcVideoChatPlayerOpenFailed	| 播放器打开失败。可能是网络未连接或播放地址错误
AlivcVideoChatPlayerStartBuffering	| 播放端缓冲开始
AlivcVideoChatPlayerEndBuffering	| 播放端缓冲结束
AlivcVideoChatPlayerFirstFrameRender	| 播放端首帧显示
AlivcVideoChatPlayerReadPacketTimeout	| 播放端下载数据超时
AlivcVideoChatPlayerNoDisplayViewer	| 播放端无显示窗口
AlivcVideoChatPlayerInvalidCodec	| 播放端音视频格式无效

<br>
接口的具体描述如下：

**prepareToPublish**

```
- (int) prepareToPublish: (UIView*)view 
width:(int)width 
height:(int)height 
publisherParam:(NSDictionary*) publlisherParam;
```
功能：准备推流。调用该函数后将对音视频采集的硬件设备进行初始化，对音视频编码器进行初始化，并开启美颜等滤镜。同时，主播可以预览到经过滤镜处理以后的视频效果。

参数：

view：推流或连麦过程中供主播预览的view。

width、height：推流视频的宽和高。

publisherParam：主播推流的参数。使用NSDictionary的方式，以便于后续的扩展。目前可以设置的参数如下：

* ALIVC_ PUBLISHER_ PARAM_ UPLOADTIMEOUT：	推流上传超时时间，单位ms,默认8000。
* ALIVC_ PUBLISHER_ PARAM_ CAMERAPOSITION：	选择前后摄像头，枚举成员：
cameraPositionFront = 0,
cameraPositionBack = 1。默认前置。
* ALIVC_ PUBLISHER_ PARAM_ LANDSCAPE：推流横屏/竖屏，NO为竖屏，YES为竖竖。默认竖屏。
* ALIVC_ PUBLISHER_ PARAM_ MAXBITRATE：推流最大码率，单位Kbps。默认1500。
* ALIVC_ PUBLISHER_ PARAM_ MINBITRATE：推流最小码率，单位Kbps。默认200。
* ALIVC_ PUBLISHER_ PARAM_ ORIGINALBITRATE：	推流初始码率，单位Kbps。默认500。
* ALIVC_ PUBLISHER_ PARAM_ AUDIOSAMPLERATE：	推流音频采样率，单位Hz。固定32000，暂不可调。
* ALIVC_ PUBLISHER_ PARAM_ AUDIOBITRATE：	推流音频码率，单位Kbps。固定96，暂不可调。
* ALIVC_ PUBLISHER_ PARAM_ FRONTCAMERAMIRROR：	前置摄像头是否镜像。

备注：目前视频编码采用的是软编码，软编码条件下只支持两种分辨率：360x640、480x848（横屏推流的时候为640x360、848x480）。

**startToPublish**

```
- (int) startToPublish: (NSString*)url;
```
功能：开始推流。调用该函数将启动音视频的编码，并将压缩后的音视频流打包上传到服务器。

参数：

url：主播推流地址。

备注：此处的推流仅仅是主播单向的直播推流，与连麦这种双向互动没有关系。必须先调用函数prepareToPublish后才能调用该函数。

**stopPublishing**

```
- (int) stopPublishing;
```

功能：结束推流。调用该函数将结束本次的直播推流，并关闭音视频编码功能，但采集、滤镜功能仍然运行，预览功能仍然保留。

参数：无。

备注：若在连麦状态下调用该函数，则sdk会先停止连麦，再结束推流。


**finishPublishing**

```
- (int) finishPublishing;
```

功能：退出推流直播。调用该函数将停止采集、滤镜功能，销毁预览窗口，释放所有资源。

参数：无。

备注：若调用了函数startToPublish，则必须调用函数stopPublishing以后才可以调用该函数。


**launchChat**

```
- (int) launchChat:(NSURL*)url view:(UIView*)view;
```

功能：开始连麦。这种连麦可以是与观众的连麦，也可以是与另一个主播进行的连麦。调用该函数启动播放器，将连麦参与方上传的视频在另一个窗口中播放出来。

参数：

url：用于播放连麦参与方的视频的地址。

view：播放连麦参与方视频的窗口。

备注：必须调用函数startToPublish后才能调用该函数。


**abortChat**

```
- (int) abortChat;
```

功能：结束连麦。调用该函数将关闭播放器，销毁用于播放的窗口。

参数：无。

备注：无。


**setPlayerParam**

```
-(void) setPlayerParam:(NSDictionary*)playerParam;
```

功能：设置连麦过程中播放器相关的配置参数。

参数：

playerParam：播放器相关的配置参数。使用NSDictionary的方式，以便于后续的扩展。目前可以设置的参数如下：

* ALIVC_ PLAYER_ PARAM_ DOWNLOADTIMEOUT：	连麦播放缓冲超时时间，单位ms。默认15000。
* ALIVC_ PLAYER_ PARAM_ DROPBUFFERDURATION：	连麦播放开始丢帧阈值，单位ms。默认1000。
* ALIVC_ PLAYER_ PARAM_ SCALINGMODE：	连麦播放显示模式，目前支持2种。枚举成员scalingModeAspectFit = 0，代表等比例缩放，若显示窗口宽高比与视频不同则会有黑边；scalingModeAspectFitWithCropping = 1，代表带切边的等比例缩放，若显示窗口宽高比与视频不同，则自动对视频裁边以撑满显示窗口。 默认值为1。

备注：这些参数可以在连麦开始之前设置，也可以在连麦过程中进行调整。

**switchCamera**

```
- (int) switchCamera;
```

功能：切换摄像头。

参数：无。

备注：该函数可以在推流和连麦的过程中随时进行调用。


**zoomCamera**

```
- (int) zoomCamera:(CGFloat)zoom;
```

功能：摄像头放大倍率。调用该函数将对当前视频进行光学放大。放大后的视频将显示在预览窗口。

参数：

zoom：放大倍率。最小值为1.0，表示不放大；最大值maxZoom与设备本身相关，且上限设置为3.0，即maxZoom = min（maxZoom，3.0）。

备注：该函数仅对后置摄像头有效。


**focusCameraAtAdjustedPoint**

```
- (int) focusCameraAtAdjustedPoint:(CGPoint)point autoFocus:(BOOL)autoFocus;
```

功能：聚焦到某个设置的点。调用该函数可以聚焦到预览窗口上人为指定的某个点。

参数：

point：需要聚焦到的点的位置。（0.0，0.0）代表左上角，（1.0，1.0）代表右下角，（0.5，0.5）代表中心点。

autofocus：自动聚焦模式。0代表只自动聚焦一次，以后将按照固定的景深来进行聚焦；1代表持续自动聚焦，当拍摄的物体变换时仍然会自动调整景深来聚焦。

备注：无。

**setFilterParam**

```
-(void) setFilterParam:(NSDictionary*)param;
```
功能：设置滤镜的相关参数。

参数：

param：滤镜相关的配置参数。使用NSDictionary的方式，以便于后续的扩展。目前只有一个美颜的滤镜，可以设置的参数如下：

* ALIVC_ FILTER_ PARAM_ BEAUTY_ON：	美颜是否开启

备注：该函数可以在推流前调用，也可以在推流和连麦过程中调用。

**getPublisherPerformanceInfo**

```
-(AlivcPublisherPerformanceInfo*) getPublisherPerformanceInfo;
```

功能：获得与推流相关的性能参数

参数：无

返回值：推流性能参数。具体如下：

* audioEncodedBitrate：	音频编码速度，单位Kbps
* videoEncodedBitrate：	视频编码速度，单位Kbps
* audioUploadedBitrate：	音频上传速度，单位kbps
* videoUploadedBitrate：	视频上传速度，单位kbps
* audioPacketsInBuffer：	缓冲的音频帧数
* videoPacketsInBuffer：	缓冲的视频帧数
* videoEncodedFps：	视频编码帧率
* videoUploadedFps：	视频上传帧率
* videoCaptureFps：	视频采集帧率
* videoEncoderParamOfBitrate：	当前视频编码器的设置码率，单位kbps
* currentlyUploadedVideoFramePts：	当前上传的视频帧的pts，单位ms
* currentlyUploadedAudioFramePts：	当前上传的音频帧的pts，单位ms
* previousKeyframePts：	上一个关键帧的pts，单位ms
* totalFramesOfEncodedVideo：	视频编码总帧数
* totalTimeOfEncodedVideo：	视频编码总耗时，单位ms
* totalSizeOfUploadedPackets：	上传的音视频流总量，单位Kbyte
* totalTimeOfPublishing：	当前推流的总时间，单位ms
* totalFramesOfVideoUploaded：	上传的视频帧总数
* dropDurationOfVideoFrames：	视频丢帧的累计时长，单位ms
* audioDurationFromCaptureToUpload：	当前音频帧从采集到上传的耗时，单位ms
* videoDurationFromCaptureToUpload：	当前视频帧从采集到上传的耗时，单位ms

**getPlayerPerformanceInfo**

```
-(AlivcPlayerPerformanceInfo*) getPlayerPerformanceInfo;
```

功能：获得与播放相关的性能参数

参数：无

返回值：播放性能参数。具体如下：

* videoPacketsInBuffer：	缓冲的视频帧数
* audioPacketsInBuffer：	缓冲的音频帧数
* videoDurationFromDownloadToRender：	视频从下载到播放的耗时，单位ms
* audioDurationFromDownloadToRender：	音频从下载到播放的耗时，单位ms
* videoPtsOfLastPacketInBuffer：	缓冲区中最后一帧视频的pts
* audioPtsOfLastPacketInBuffer：	缓冲区中最后一帧音频的pts

**publisherMuteMode**

```
BOOL publisherMuteMode;
```

功能：设置推流端的静音模式。

参数：无。

备注：无

**getSDKVersion**

```
- (NSString *) getSDKVersion;
```

功能：获取版本信息。

参数：无

返回值：版本号。


###AlivcVideoChatParter的接口和事件通知###

接口名称|功能描述
----|----
startToPlay	| 开始观看直播
stopPlaying	| 结束观看直播
onlineChat	| 开始连麦
offlineChat	| 结束连麦
setPlayerParam	| 设置播放参数
switchCamera	| 切换前后摄像头
zoomCamera	| 缩放摄像头
focusCameraAtAdjustedPoint	| 聚焦摄像头到某个位置
setFilterParam	| 设置滤镜参数
getPublisherPerformanceInfo	| 获取推流性能参数
getPlayerPerformanceInfo	| 获取连麦后播放性能参数
publisherMuteMode	| 推流是否静音
getSDKVersion	| 获取SDK版本号

事件通知|内容描述
----|----
AlivcVideoChatMemoryPool 	| 内存不足
AlivcVideoChatPublisherOpenFailed |	推流端打开失败。可能是网络未连接或推流地址错误
AlivcVideoChatPublisherSendDataTimeout	| 推流端发送数据超时
AlivcVideoChatPublisherNetworkPool	| 推流端网络差
AlivcVideoChatPublisherVideoCaptureDisabled | 	视频采集被禁止
AlivcVideoChatPublisherAudioCaptureDisabled | 	音频采集被禁止
AlivcVideoChatPublisherVideoEncoderInitFailed | 	视频端初始化失败
AlivcVideoChatPublisherAudioEncoderInitFailed | 	音频端初始化失败
AlivcVideoChatPublisherEncodeVideoFailed | 	推流端视频编码失败
AlivcVideoChatPublisherEncodeAudioFailed | 	推流端音频编码失败
AlivcVideoChatPlayerOpenFailed	| 播放器打开失败。可能是网络未连接或播放地址错误
AlivcVideoChatPlayerStartBuffering | 	播放端缓冲开始
AlivcVideoChatPlayerEndBuffering	| 播放端缓冲结束
AlivcVideoChatPlayerFirstFrameRender	| 播放端首帧显示
AlivcVideoChatPlayerReadPacketTimeout	| 播放端下载数据超时
AlivcVideoChatPlayerNoDisplayViewer	| 播放端无显示窗口
AlivcVideoChatPlayerInvalidCodec	| 播放端音视频格式无效


<br>
接口的具体描述如下：

**startToPlay**

```
-(int) startToPlay:(NSURL*)url view:(UIView*)view;
```

功能：开始观看直播。调用该函数将启动直播播放器，并播放获取到的直播流。

参数：

url：直播地址。

view：直播播放器的渲染窗口。

备注：无。

**stopPlaying**

```
-(int) stopPlaying;
```

功能：结束观看直播。调用该函数将关闭直播播放器，并销毁所有资源。

参数：无。

备注：若在连麦状态下调用该函数，则sdk会先停止连麦，再结束播放。


**onlineChat**

```
-(int) onlineChat:(NSString*)publisherUrl width:(int)width height:(int)height 
preview:(UIView*)preview publisherParam:(NSDictionary*)publisherParam
playerUrl:(NSURL*)playerUrl;
```
功能：开始连麦。调用该函数将开启音视频的采集设备、启动预览功能、启动音视频编码功能并将压缩后的音视频流上传。同时将播放地址切换到具备短延时功能的新地址。

参数：

publisherUrl：连麦时推流的地址。

playerUrl：连麦时切换到的短延时播放地址。

width，height：编码视频的宽和高。

preview：连麦时推流的预览窗口。

publisherParam：连麦时推流的参数。使用NSDictionary的方式，以便于后续的扩展。目前可以设置的参数如下：（与类
AlivcVideoChatHost中接口函数prepareToPublish的参数publisherParam相同）

* ALIVC_ PUBLISHER_ PARAM_ UPLOADTIMEOUT	推流上传超时时间，单位ms,默认
* ALIVC_ PUBLISHER_ PARAM_ CAMERAPOSITION	选择前后摄像头，枚举成员：
cameraPositionFront = 0,
cameraPositionBack = 1。默认前置。
* ALIVC_ PUBLISHER_ PARAM_ LANDSCAPE	推流横屏/竖屏，NO为竖屏，YES为竖竖。默认竖屏。
* ALIVC_ PUBLISHER_ PARAM_ MAXBITRATE	推流最大码率，单位Kbps。默认1500。
* ALIVC_ PUBLISHER_ PARAM_ MINBITRATE	推流最小码率，单位Kbps。默认200。
* ALIVC_ PUBLISHER_ PARAM_ ORIGINALBITRATE	推流初始码率，单位Kbps。默认500。
* ALIVC_ PUBLISHER_ PARAM_ AUDIOSAMPLERATE	推流音频采样率，单位Hz。固定32000，暂不可调。
* ALIVC_ PUBLISHER_ PARAM_ AUDIOBITRATE	推流音频码率，单位Kbps。固定96，暂不可调。
* ALIVC_ PUBLISHER_ PARAM_ FRONTCAMERAMIRROR	前置摄像头是否镜像。

备注：目前视频编码采用的是软编码，软编码条件下只支持两种分辨率：360x640、480x848（横屏推流的时候为640x360、848x480）。

**offlineChat**

```
-(int) offlineChat;
```

功能：结束连麦。调用该函数将结束观众的推流，销毁推流的所有资源，并将播放地址切换到连麦之前的地址。

参数：无。

备注：无。

**setPlayerParam**

```
-(void) setPlayerParam:(NSDictionary*)playerParam;
```

功能：设置播放器相关的配置参数。

参数：

playerParam：播放器相关的配置参数。使用NSDictionary的方式，以便于后续的扩展。目前可以设置的参数如下：

* ALIVC_ PLAYER_ PARAM_ DOWNLOADTIMEOUT：	播放器缓冲超时时间，单位ms。默认15000。
* ALIVC_ PLAYER_ PARAM_ DROPBUFFERDURATION：	播放器开始丢帧阈值，单位ms。连麦过程中默认值为1000，非连麦过程中默认值为8000。
* ALIVC_ PLAYER_ PARAM_ SCALINGMODE：	播放器显示模式，目前支持2种。枚举成员scalingModeAspectFit = 0，代表等比例缩放，若显示窗口宽高比与视频不同则会有黑边；scalingModeAspectFitWithCropping = 1，代表带切边的等比例缩放，若显示窗口宽高比与视频不同，则自动对视频裁边以撑满显示窗口。 默认值为1。

备注：与类AlivcVideoChatHost中接口函数setPlayerParam功能与参数设置相同。

**switchCamera**

```
- (int) switchCamera;
```

功能：切换摄像头。

参数：无。

备注：该函数可以在连麦的过程中随时进行调用。


**zoomCamera**

```
- (int) zoomCamera:(CGFloat)zoom;
```

功能：摄像头放大倍率。调用该函数将对当前视频进行光学放大。放大后的视频将显示在预览窗口。

参数：

zoom：放大倍率。最小值为1.0，表示不放大；最大值maxZoom与设备本身相关，且上限设置为3.0，即maxZoom = min（maxZoom，3.0）。

备注：该函数仅对后置摄像头有效。


**focusCameraAtAdjustedPoint**

```
- (int) focusCameraAtAdjustedPoint:(CGPoint)point autoFocus:(BOOL)autoFocus;
```

功能：聚焦到某个设置的点。调用该函数可以聚焦到预览窗口上人为指定的某个点。

参数：

point：需要聚焦到的点的位置。（0.0，0.0）代表左上角，（1.0，1.0）代表右下角，（0.5，0.5）代表中心点。

autofocus：自动聚焦模式。0代表只自动聚焦一次，以后将按照固定的景深来进行聚焦；1代表持续自动聚焦，当拍摄的物体变换时仍然会自动调整景深来聚焦。

备注：无。

**setFilterParam**

```
-(void) setFilterParam:(NSDictionary*)param;
```
功能：设置滤镜的相关参数。

参数：

param：滤镜相关的配置参数。使用NSDictionary的方式，以便于后续的扩展。目前只有一个美颜的滤镜，可以设置的参数如下：

* ALIVC_ FILTER_ PARAM_ BEAUTY_ON：	美颜是否开启

备注：该函数可以在连麦过程中随时进行调用。

**getPublisherPerformanceInfo**

```
-(AlivcPublisherPerformanceInfo*) getPublisherPerformanceInfo;
```

功能：获得与推流相关的性能参数

参数：无

返回值：推流性能参数。具体如下：（与类AlivcVideoChatHost中相同）

* audioEncodedBitrate：	音频编码速度，单位Kbps
* videoEncodedBitrate：	视频编码速度，单位Kbps
* audioUploadedBitrate：	音频上传速度，单位kbps
* videoUploadedBitrate：	视频上传速度，单位kbps
* audioPacketsInBuffer：	缓冲的音频帧数
* videoPacketsInBuffer：	缓冲的视频帧数
* videoEncodedFps：	视频编码帧率
* videoUploadedFps：	视频上传帧率
* videoCaptureFps：	视频采集帧率
* videoEncoderParamOfBitrate：	当前视频编码器的设置码率，单位kbps
* currentlyUploadedVideoFramePts：	当前上传的视频帧的pts，单位ms
* currentlyUploadedAudioFramePts：	当前上传的音频帧的pts，单位ms
* previousKeyframePts：	上一个关键帧的pts，单位ms
* totalFramesOfEncodedVideo：	视频编码总帧数
* totalTimeOfEncodedVideo：	视频编码总耗时，单位ms
* totalSizeOfUploadedPackets：	上传的音视频流总量，单位Kbyte
* totalTimeOfPublishing：	当前推流的总时间，单位ms
* totalFramesOfVideoUploaded：	上传的视频帧总数
* dropDurationOfVideoFrames：	视频丢帧的累计时长，单位ms
* audioDurationFromCaptureToUpload：	当前音频帧从采集到上传的耗时，单位ms
* videoDurationFromCaptureToUpload：	当前视频帧从采集到上传的耗时，单位ms

**getPlayerPerformanceInfo**

```
-(AlivcPlayerPerformanceInfo*) getPlayerPerformanceInfo;
```

功能：获得与播放相关的性能参数

参数：无

返回值：播放性能参数。具体如下：（与类AlivcVideoChatHost中相同）

* videoPacketsInBuffer：	缓冲的视频帧数
* audioPacketsInBuffer：	缓冲的音频帧数
* videoDurationFromDownloadToRender：	视频从下载到播放的耗时，单位ms
* audioDurationFromDownloadToRender：	音频从下载到播放的耗时，单位ms
* videoPtsOfLastPacketInBuffer：	缓冲区中最后一帧视频的pts
* audioPtsOfLastPacketInBuffer：	缓冲区中最后一帧音频的pts

**publisherMuteMode**

```
BOOL publisherMuteMode;
```

功能：设置推流端的静音模式，仅在连麦过程中使用。

参数：无。

备注：无

**getSDKVersion**

```
- (NSString *) getSDKVersion;
```

功能：获取版本信息。

参数：无

返回值：版本号。
