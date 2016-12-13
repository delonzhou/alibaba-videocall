#Android版本连麦SDK#
<br>
##概述##

AlivcVideoChat是一款适用于 android 平台的、提供了网络连麦功能的SDK，它提供了丰富的接口供开发者进行二次开发。它不仅支持单纯的主播推流、观众播放的功能，还支持主播与主播之间、主播与观众之间的视频互动，同时第三方观众还可以通过服务器提供的混流功能观看这些互动的场景。它的特点是短延时、支持弱网环境、抗网络抖动能力强。

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

* 运行环境（android 4.0及以上版本）
* 硬件CPU支持ARMv7、ARMv7s或ARM64

###sdk下载###

###权限说明###
```javascript    
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.VIBRATE"/>
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_MOCK_LOCATION"/>    //release时可去掉
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.GET_TASKS"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```
###混淆配置###
```javascript
####### 连麦SDK ######## 
-keepclasseswithmembernames class * {
    native <methods>;
} -keep public class com.alivc.** {
    *;
}

######### android websocket #########
-keep class com.koushikdutta.async.**{ *;}
-keep class com.alibaba.sdk.** {*;}
-dontwarn javax.annotation.**
-dontwarn net.jcip.annotations.**

######### okhttp ########
-dontwarn com.squareup.okhttp.**
-keep class com.squareup.okhttp.** { *;}
-dontwarn okio.**

########## gson ###########
-keepattributes Signature 
# For using GSON @Expose annotation
-keepattributes *Annotation*
 # Gson specific classes
-keep class sun.misc.Unsafe { *; }
-keep class com.google.gson.examples.android.model.** { *; } 

-keep class * implements com.google.gson.TypeAdapterFactory
-keep class * implements com.google.gson.JsonSerializer
-keep class * implements com.google.gson.JsonDeserializer

########## retorfit ########
# Platform calls Class.forName on types which do not exist on Android to determine platform.
-dontnote retrofit2.Platform
# Platform used when running on RoboVM on iOS. Will not be used at runtime.
-dontnote retrofit2.Platform$IOS$MainThreadExecutor
# Platform used when running on Java 8 VMs. Will not be used at runtime.
-dontwarn retrofit2.Platform$Java8
# Retain generic type information for use by reflection by converters and adapters.
-keepattributes Signature
# Retain declared checked exceptions for use by a Proxy instance.

-keepattributes Exceptions
 ######### rxjava #########
-dontwarn sun.misc.**
-keepclassmembers class
rx.internal.util.unsafe.*ArrayQueue*Field* {
 long producerIndex;
 long consumerIndex;
}
-keepclassmembers class
rx.internal.util.unsafe.BaseLinkedQueueProducerNodeRef {
 rx.internal.util.atomic.LinkedQueueNode producerNode;
}
-keepclassmembers class
rx.internal.util.unsafe.BaseLinkedQueueConsumerNodeRef {
 rx.internal.util.atomic.LinkedQueueNode consumerNode;
}

```
###快速安装###

##demo使用说明##

demo实现了连麦的基本场景，包括主播端的推流、用户端的播放、主播与主播之间的连麦、主播与观众之间的连麦。

程序整体使用了一个简易的MVP架构，其中view处于*com.alibaba.livecloud.videocall.ui.view*这个包中，presenter处于*com.alibaba.livecloud.videocall.presenter*这个包中，而model处于com.alibaba.livecloud.videocall.bi这个包中。

信号传递这里依赖了第三方通信库——环信，主要使用了环信的聊天室功能及个人消息功能；并且针对Demo的业务场景进行了进一步的封装，主要在*com.alibaba.livecloud.videocall.im*这个包中。如果用户有其他的选择，可以替换。

RestAPI主要依赖了**Retrofit**框架，并且结合**RxJava**一直使用，主要在*com.alibaba.livecloud.videocall.http*这个包中。

###基础业务###
**登录**

调用登陆接口——*com.alibaba.livecloud.videocall.presenter.LoginPresenter#login()*


```javascript
public void login(String username, String password) {
	.......//省略
		@Override
		public void onNext(LoginResult o) {
			//登陆成功，保存登陆信息
			mView.saveLoginInfo(o.getId());
                
			//初始化ImManager
			mView.initImManager(o.getImUserInfo());
                
			//跳转到主页
			mView.gotoMainActivity();
		}
	.......//省略
}
```

**直播列表**

调用列表加载接口——*com.alibaba.livecloud.videocall.presenter.MainPresenter#loadLiveList()*
    
```javascript
/**
 * 加载直播列表
 */
public void loadLiveList() {
	.......//省略

		@Override
		public void onNext(List<LiveItemResult> liveItemResults) {
			mMainView.showLiveList(liveItemResults);
		}
        
	......//省略
}
```

**权限检查**

针对android 6.0以上的设备（这里需要用到*Manifest.permission.CAMERA*,
*Manifest.permission.RECORD_AUDIO*这两个权限）,*com.alibaba.livecloud.videocall.ui.LiveActivity#onCreate()*中调用
 
```javascript
@Override
protected void onCreate(Bundle savedInstanceState) {
	........//省略
        
	if(permissionCheck()) {
		mHasPermission = true;
	}else {
		if(Build.VERSION.SDK_INT >= 23) {
			ActivityCompat.requestPermissions(this,permissionManifest,PERMISSION_REQUEST_CODE);
		}else {
			showNoPermissionTip(getString(noPermissionTip[mNoPermissionIndex]));
			finish();
		}
	}
	.......//省略
}
```
```javascript
/**
 * 权限检查（适配6.0以上手机）
 */
private boolean permissionCheck() {
	int permissionCheck = PackageManager.PERMISSION_GRANTED;
	String permission = null;
	for (int i = 0;i<permissionManifest.length;i++) {
		permission = permissionManifest[i];
		mNoPermissionIndex = i;
		if (PermissionChecker.checkSelfPermission(this, permission)
			!= PackageManager.PERMISSION_GRANTED) {
			permissionCheck = PackageManager.PERMISSION_DENIED;
		}
	}
	if (permissionCheck != PackageManager.PERMISSION_GRANTED) {
		return false;
	} else {
		return true;
	}
}
```
重写onRequestPermissionsResult方法

```javascript
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
	super.onRequestPermissionsResult(requestCode, permissions, grantResults);
	switch (requestCode) {
		case PERMISSION_REQUEST_CODE:
			boolean hasPermission = true;
			for (int i = 0; i < permissions.length; i++) {
				if (grantResults[i] == PackageManager.PERMISSION_DENIED) {
					int toastTip = noPermissionTip[i];
					mNoPermissionIndex = i;
						if (toastTip != 0) {
							mLiveView.showToast(toastTip);
							hasPermission = false;
							finish();
						}
					}
				}
				mHasPermission = hasPermission;
				break;
			}
		}
	}
}
```
###主播业务###


**初始化**

在*com.alibaba.livecloud.videocall.ui.LiveActivity#onOnCreate()*中调用初始化代码
    
```javascript
@Override
protected void onCreate(Bundle savedInstanceState) {
	.......//省略
        
	initRecorder(); 
        
	.......//省略 
}
```
```javascript
/**
 * 主播端初始化
 */
private void initRecorder() {
	//设置推流器推流相关参数
	......		// 省略

	mChatHost = new AlivcVideoChatHost();
	mChatHost.init(this);
	mChatHost.setHostViewScalingMode(IMediaPublisher.VideoScalingMode.VIDEO_SCALING_MODE_SCALE_TO_FIT_WITH_CROPPING);
	mChatHost.setParterViewScalingMode(MediaPlayer.VideoScalingMode.VIDEO_SCALING_MODE_SCALE_TO_FIT_WITH_CROPPING);
	mChatHost.setErrorListener(mOnErrorListener);
	mChatHost.setInfoListener(mInfoListener);

	//设置美颜开启
	mFilterMap.put(AlivcVideoChatHost.ALIVC_FILTER_PARAM_BEAUTY_ON, Boolean.toString(true));
	mChatHost.setFilterParam(mFilterMap);
}
```

**创建直播**

由于Demo中是进入直播创建界面就开启预览，因此需要在预览SurfaceView的SurfaceHolder$Callback的surfaceCreated中执行开启预览的逻辑

```javascript
public void surfaceCreated(final SurfaceHolder holder) {
	startPreView(holder);
	.......//省略        
}
```        
```javascript
/**
 * 开启预览
 */
private void startPreView(final SurfaceHolder holder) {
	//需要先检查是否已经授权（6.0的动态权限请求是异步行为）
	if (mHasPermission) {
		//创建直播
		mChatHost.prepareToPublish(holder.getSurface(), 360, 640, mMediaParam);
		if (mCameraFacing == AlivcMediaFormat.CAMERA_FACING_FRONT) {
			mChatHost.setFilterParam(mFilterMap);
			if (mLiveBottomFragment != null) {
				mLiveBottomFragment.setBeautyUI(true);
			}
		}
	} else {
		/**
		 * 如果没有授权，需要判断当前系统版本是否是6.0以上，如果是6.0以上，因为动态请求权限属于异步行为，所以需要等待授权结果，
		 * 采用postDelay的方式，一秒后再重新请求一次，如果是低于6.0则直接给出没有权限的提醒，并且finish掉
		 */
		if (Build.VERSION.SDK_INT < 23) {
			showNoPermissionTip(getString(mNoPermissionIndex));
			finish();
		} else {
			mPermissionRun = new Runnable() {
				@Override
				public void run() {
					mPermissionRun = null;
					startPreView(holder);
				}
			};
			mHandler.postDelayed(mPermissionRun, PERMISSION_DELAY);
		}
	}
}
```

            
**开始推流**

调用创建直播的REST API——*com.alibaba.livecloud.videocall.presenter.CreateLivePresenter*
	            
```javascript    
/**
 * 创建直播
 * @param uid
 * @param desc
 */
public void createLive(String uid, String desc) {
	........//省略
        
		@Override
		public void onNext(LiveCreateResult result) {
		mView.showToast(result.getRtmpUrl());

		//调用连麦SDK，开始推流
		mChatHost.startToPublish(result.getRtmpUrl());

		//切换到推流界面
		mView.gotoRecord(result.getRoomID(),
                        result.getName(),
                        result.getUid());

		}
        
	......//省略
}
```

                     

**停止推流**
    
```javascript
/**
 * 停止推流
 */
public void stopPublish() {
	if (null != mChatHost) {
		......
 	   mChatHost.stopPublishing();
 	   mChatHost.finishPublishing();
 	   ......
 	}
}
```
                     
**关闭直播**
    
结束掉Activity，*com.alibaba.videocall.ui.LiveActivity#finish()*，并且在onDestroy中调用相关代码
    
```javascript
@Override
protected void onDestroy() {
	super.onDestroy();
        
	if (!mIsPublishStop) {
		stopPublish();
	}
	if (mChatHost != null) {
		mChatHost.release();
		mChatHost = null;
	}
        
	if (!mIsChatting) {
		mFeedbackPresenter.closeVideoCall(mRoomID, mChatRoomID);
	}
}
```                     

**异常处理**

需要在初始化时分别设置错误信息回调和状态信息回调，具体每个Error Code和Info Code的含义请参考*com.alivc.publisher.MediaError*接口文档

```javascript
/**
 * 初始化推流器
 */
private void initRecorder() {
	......//省略
        
	mChatHost.setErrorListener(mOnErrorListener);
	mChatHost.setInfoListener(mInfoListener);

	......//省略
}
```

###观众业务###

**初始化**

需要在Activity创建时执行初始化操作，参考*com.alibaba.videocall.ui.WatchLiveActivity#onCreate()*和
*com.ablibaba.videocall.presenter.WatchLivePresenter#onCreate()*

```javascript
/**
 * 对应Activity的生命周期回调{@link android.app.Activity#onCreate(Bundle)}
 */
public void onCreate() {
	.......//省略       
       
	initPlayer(); 
}
```
```javascript
/**
 * 观众端初始化
 */
private void initPlayer() {
	mChatParter = new AlivcVideoChatParter();
	mChatParter.setErrorListener(mPlayerErrorListener);
	mChatParter.init(mContext);
	mChatParter.setInfoListener(mPlayerInfoListener);

	mFilterMap.put(AlivcVideoChatParter.ALIVC_FILTER_PARAM_BEAUTY_ON, Boolean.toString(true)); //设置连麦预览/推流时开启美颜
	mChatParter.setFilterParam(mFilterMap);
}
```

**观看直播**

获取播放时渲染视频所用的SurfaceView，并且将其与*com.alibaba.videocall.ui.WatchLiveActivity#mPlaySurfaceCB*
绑定，在mPlaySurfaceCB的surfaceCreated中执行播放的逻辑
    
```javascript    
@Override
protected void onCreate(Bundle savedInstanceState) {
	......//省略
	mPlaySurfaceView.getHolder().addCallback(mPlaySurfaceCB);
}
```
```javascript
SurfaceHolder.Callback mPlaySurfaceCB = new SurfaceHolder.Callback() {
	@Override
	public void surfaceCreated(SurfaceHolder holder) {
		Log.d(TAG, "parter player surface create.");
		holder.setType(SurfaceHolder.SURFACE_TYPE_GPU);
		holder.setKeepScreenOn(true);
		if (mPlaySurfaceStatus == SurfaceStatus.UNINITED) {
			mPlaySurfaceStatus = SurfaceStatus.CREATED;
			mWatchLivePresenter.startToPlay(mPlaySurfaceView);
		} else if (mPlaySurfaceStatus ==
					SurfaceStatus.DESTROYED) {
			mPlaySurfaceStatus = SurfaceStatus.RECREATED;
		}
	} 

	@Override
	public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
		Log.d(TAG, "parter player surface change.");
		mPlaySurfaceStatus = SurfaceStatus.CHANGED;
		if ((mPreviewSurfaceStatus == null
			|| mPreviewSurfaceStatus ==
			SurfaceStatus.UNINITED
			|| mPreviewSurfaceStatus ==
			SurfaceStatus.CHANGED
			) && mPlaySurfaceStatus ==
			SurfaceStatus.CHANGED) {
			mWatchLivePresenter.mediaResume(mPlaySurfaceView, mPreviewSurfaceView);
		}
		if (shouldOffLine) {
			mWatchLivePresenter.sdkOfflineChat();   //在这里调用真正的offlineChat，保证渲染出得最后一帧数据是正常播放的尺寸，而不是小窗播放的尺寸
			shouldOffLine = false;
		}
	} 
	@Override
	public void surfaceDestroyed(SurfaceHolder holder) {
		mPlaySurfaceStatus = SurfaceStatus.DESTROYED;
		Log.d(TAG, "parter player surface destroy.");
	}
};
```
在surfaceView的surfaceCreated回调中调用SDK的播放接口startToPlay
```javascript
/**
 * 开始直播播放（大窗）
 * @param surfaceView
 */
public void startToPlay(final SurfaceView surfaceView) {
	if (mChatParter == null) {
            
		initPlayer();
	}
        
	......//省略

	mChatParter.startToPlay(mPlayUrl, surfaceView); //开始直播
        
	......//省略
}
```
                     
**发送评论**

发送评论，参考*com.alibaba.videocall.presenter.WatchBottomPresenter#sendComment()*

    
```javascript    
/**
 * 发送评论
 * @param uid
 * @param roomID
 * @param comment
 */
public void sendComment(String uid, String roomID, String comment) {
	mCommentSub = new Subscriber() {
		......//省略
            
	};
	ServiceBIFactory.getInteractionServiceBI().sendComment(uid, roomID, comment, mCommentSub);
}
```

收到评论显示，参考 *com.alibaba.videocall.ui.InteractionFragment#mCommentFunc*

```javascript
/**
 * 收到评论消息处理的Action
 */
private ImHelper.Func<MsgDataComment> mCommentFunc = new ImHelper.Func<MsgDataComment>(){

	@Override
	public void action(MsgDataComment msgDataComment) {
		.......//省略
            
		mCommentView.post(new Runnable() {
			@Override
			public void run() {
				mAdapter.addComment(commentBean);
				mCommentView.smoothScrollToPosition(mAdapter.getItemCount() - 1);
			}
		});
	}
};
```
                     
**点赞**

发送赞，参考*com.alibaba.videocall.presenter.WatchBottomPresenter#sendLike()*
    
```javascript    
/**
 * 发送赞
 * @param roomID
 * @param uid
 */
public void sendLike(String roomID, String uid){
	mLikeSub = new Subscriber() {
		......//省略
	};
	ServiceBIFactory.getInteractionServiceBI().sendLike(roomID, uid, mLikeSub);
}
```


收到赞显示，参考*com.alibaba.videocall.ui.InteractionFragment#mLikeFunc*

```javascript    
/**
 * 收到点赞消息处理Action
 */
private ImHelper.Func<MsgDataLike> mLikeFunc = new ImHelper.Func<MsgDataLike>(){

	@Override
	public void action(MsgDataLike o) {
		if(!mUID.equals(o.getUid())) {
			showLikeUI();
		}
	}
};
```


**退出观看**

```javascript
/**
 * 对应生命周期函数{@link Activity#onDestroy()}
 */
public void onDestroy() {
	if (mChatParter != null) {
		mChatParter.setErrorListener(null);
		if(isChatting()) {
			mChatParter.offlineChat();      //关闭连麦推流
			closeVideoCall();               //调用结束连麦REST API
		}               
		mChatParter.stopPlaying();      //停止播放
		mChatParter.release();          //释放资源
     
	}       
	.......//省略
}
```


**异常处理**

需要在初始化时分别设置错误信息回调和状态信息回调，具体每个Error Code和Info Code的含义请参考
*com.alivc.publisher.MediaError*接口文档

```javascript
/**
 * 初始化播放器
 */
private void initPlayer() {
	......//省略
        
	mChatParter.setErrorListener(mOnErrorListener);
	mChatParter.setInfoListener(mInfoListener);
	......//省略
}
```

###连麦过程###

demo中一共设计了三种连麦的方式：主播邀请主播连麦，主播邀请观众连麦，观众邀请主播连麦。

**主播邀请主播连麦**

1） 发送邀请--- **邀请方（主播A）**
调用邀请直播的REST API——*com.alibaba.livecloud.videocall.presenter.LivePresenter*

```javascript
/**
 * 邀请连麦
 * @param inviterUID
 * @param inviteeUID
 * @param inviterType
 */
public void inviteVideoChat(String inviterUID, final String inviteeUID, int inviterType) {
	if (mVideoChatStatus == VideoChatStatus.UNCHAT) {
		........//省略        
            
			@Override
			public void onNext(Object o) {
				mView.showToast(R.string.invite_succeed);
				mView.hideAnchorList();
			}
        
		......//省略
	}
}
```


2） 收到邀请消息，显示邀请处理Dialog---**被邀请方（主播B）**
参考*com.alibaba.livecloud.videocall.presenter.LivePresenter*

```javascript
/**
 * 连麦邀请的消息处理Action
 */
ImHelper.Func<MsgDataInvite> mInviteFunc = new ImHelper.Func<MsgDataInvite>() {

	@Override
	public void action(final MsgDataInvite msgDataInvite) {
		mChatterName = msgDataInvite.getInviterName();
		mInviterType = msgDataInvite.getInviterType();
		runOnUiThread(new Runnable() {
			@Override
			public void run() {
				mLiveView.showFeedbackChooseDialog(msgDataInvite.getInviterName());
				updateChatState(VideoChatStatus.RECEIVED_INVITE);   //更新当前连麦状态为收到邀请，等待反馈的状态 				mHandler.sendEmptyMessageDelayed(MSG_WHAT_PROCESS_INVITING_TIMEOUT, INVITE_CHAT_TIMEOUT_DELAY); //超过10s自动拒绝连麦
			}
		});
	}
};
```     

3） 反馈邀请---**被邀请方（主播B）**
调用Rest API *com.alibaba.livecloud.videocall.presenter.LivePresenter*

```javascript
/**
 * 反馈邀请
 * @param inviterType
 * @param inviteeType
 * @param inviterUID
 * @param inviteeUID
 * @param status
 */
private void internalRESTFeedback(int inviterType,
                             int inviteeType,
                             String inviterUID,
                             final String inviteeUID,
                             final int status) {
	.......//省略
		@Override
		public void onNext(InviteFeedbackResult result) {          
			if (status == FeedbackForm.STATUS_AGREE 				&& result != null) { 				mLiveView.showFeedbackSuccessfulUI(true); 				mChatRoomID = result.getInviterRoomID();    //缓存邀请方的RoomID 				/** 				 * 所谓的短延时URL实际上就是未经转码的原始流播放地址也就是，主播连麦观众时，主播端看到的观众的小窗画面，应该使用的播放地址 				 * 				 * 注意：这里没有直接就开始播放小窗，是因为这个时候观众端实际上还没有推流成功，需要等到收到推流成功的通知才开始播放 				 */ 				setSmallDelayPlayUrl(result.getInviteePlayUrl());       //缓存短延时URL 				updateChatState(VideoChatStatus.TRY_MIX);   //更新连麦状态为开始混流， 等待混流成功 				mHandler.sendEmptyMessageDelayed(MSG_WHAT_MIX_STREAM_TIMEOUT, MIX_STREAM_TIMEOUT);  //开始等待混流成功超时的倒计时 			} else if (status == FeedbackForm.STATUS_AGREE) { 				onError(new RuntimeException("Feedback Result is Null")); 			} else { 				mLiveView.showFeedbackSuccessfulUI(false); 			}       
			.......//省略
		}
	......//省略
}
```

4） 邀请方和被邀请方收到混流成功通知，开始播放混流画面
参考*com.alibaba.livecloud.videocall.presenter.LivePresenter*

```javascript    
/**
 * 混流成功的消息处理Action
 */
ImHelper.Func<MsgDataMergeStream> mMergeStreamSuccFunc = new ImHelper.Func<MsgDataMergeStream>() {

	@Override
	public void action(MsgDataMergeStream msgDataMergeStream) {
		if (mVideoChatStatus == VideoChatStatus.TRY_MIX) {  //如果当前是开始混流并且等待混流成功的状态，则处理这条消息，否则视为无效的消息，不作处理
			mHandler.removeMessages(MSG_WHAT_MIX_STREAM_TIMEOUT);   //移除等待混流成功倒计时的消息
			if (mRoomID.equals(msgDataMergeStream.getInviteeRoomID())) {
				mChatRoomID = msgDataMergeStream.getInviterRoomID();
			} else if (mRoomID.equals(msgDataMergeStream.getInviterRoomID())) {
				mChatRoomID = msgDataMergeStream.getInviteeRoomID();
			} else {
				mLiveView.showToast(R.string.merge_stream_failed);
				updateChatState(VideoChatStatus.UNCHAT);   //更新连麦状态为未连麦
				return;
			}
			mLiveView.showLaunchChatUI();               //显示连麦状态的UI
			updateChatState(VideoChatStatus.MIX_SUCC);      //更新当前连麦状态为混流成功状态
		}	
	}
};
```

5）结束连麦

参考——*com.alibaba.videocall.presenter.LivePresenter*
    
```javascript    
/**
 * 调用结束连麦的REST API
 */
public void closeLiveChat() {
	Subscriber closeChatSub = new Subscriber() {
		@Override
		public void onCompleted() {}

		@Override
		public void onError(Throwable e) {
			mLiveView.showCloseChatFailedUI();      //显示结束连麦失败的UI
		}

		@Override
		public void onNext(Object o) {
			abortChat(true);       //调用SDK结束连麦
		}
	};
	mServiceBI.terminateCall(mRoomID, mChatRoomID, closeChatSub);
}
```
```javascript
/**
 * 终止连麦
 */
public void abortChat(boolean isShowUI) {
	if (mChatHost != null && isChatting()) {
		mChatHost.abortChat();
		updateChatState(VideoChatStatus.UNCHAT);
		if(isShowUI) {
			mLiveView.showAbortChatUI();
		}
	}
}
```
                
                
**主播邀请观众连麦**

1）发送邀请---**邀请方（主播）**
调用邀请直播的REST API——*com.alibaba.livecloud.videocall.presenter.LivePresenter*

```javascript
/**
 * 邀请连麦
 * @param inviterUID
 * @param inviteeUID
 * @param inviterType
 */
public void inviteVideoCall(String inviterUID, String inviteeUID, int inviterType) {
	if (mVideoChatStatus == VideoChatStatus.UNCHAT) {
		........//省略        
            
			@Override
			public void onNext(Object o) {
				mView.showToast(R.string.invite_succeed);
				mView.hideAnchorList();
			}
        
		......//省略
	}
}
```

2） 收到邀请，显示处理邀请Dialog---**被邀请方（观众）**
参考*com.alibaba.livecloud.videocall.presenter.WatchLivePresenter*

```javascript
/**
 * 连麦邀请的消息处理Action
 */
ImHelper.Func<MsgDataInvite> mInviteFunc = new ImHelper.Func<MsgDataInvite>() {
@Override
        public void action(final MsgDataInvite msgDataInvite) {
            mChatterName = msgDataInvite.getInviterName();
            mInviterUID = msgDataInvite.getInviterUID();
            mInviterType = msgDataInvite.getInviterType();

            //显示处理连麦邀请的Dialog
            mView.showFeedbackChooseDialog(msgDataInvite.getInviterName());
            updateChatState(VideoChatStatus.RECEIVED_INVITE);   //更新当前连麦状态为收到邀请等待反馈状态
            mHandler.sendEmptyMessageDelayed(MSG_WHAT_PROCESS_INVITING_TIMEOUT, INVITE_CHAT_TIMEOUT_DELAY); //超过10s自动拒绝连麦
        }
};
```

3）反馈邀请---**被邀请方（观众）**

调用Rest API 
*com.alibaba.livecloud.videocall.presenter.WatchLivePresenter*

```javascript
/**
 * 反馈邀请
 * @param status 反馈的结果：同意-1， 不同意-2
 */
public void feedbackInvite(final int status) {  
	if (mChatStatus == VideoChatStatus.RECEIVED_INVITE) {     
		if (mFeedbackSub != null) {
                mFeedbackSub.unsubscribe();
            }
            if (status == FeedbackForm.STATUS_NOT_AGREE) {
                updateChatState(VideoChatStatus.UNCHAT);    //不同意的情况需要更新当前状态为未连麦状态
            }
            mFeedbackSub = new Subscriber<InviteFeedbackResult>() {
                @Override
                public void onCompleted() {}

                @Override
                public void onError(Throwable e) {mView.showFeedbackReqFailedUI(e);}

                @Override
                public void onNext(InviteFeedbackResult result) {
                    if (status == FeedbackForm.STATUS_AGREE
                            && result != null) {
                        //缓存推流URL和主播的短延迟播放URL
                        mPushUrl = result.getRtmpUrl();
                        mSmallDelayPlayUrl = result.getInviteePlayUrl();

                        mView.showFeedbackReqSuccessUI();   //显示连麦反馈成功的UI
                        startLaunchChat();  //开始连麦
                    } else if (status == FeedbackForm.STATUS_AGREE) {
                        onError(new RuntimeException("Feedback Result is Null"));
                    } else {//不同意
                        mView.showFeedbackReqSuccessUI();   //显示反馈成功的UI
                        updateChatState(VideoChatStatus.UNCHAT);    //更新当前连麦状态为未连麦状态
                    }
                }
            };
            mInviteServiceBI.feedback(FeedbackForm.INVITE_TYPE_WATCHER, mInviterType, mInviterUID, mUid,
                    InviteForm.TYPE_PIC_BY_PIC, status, mFeedbackSub);
            mHandler.removeMessages(MSG_WHAT_PROCESS_INVITING_TIMEOUT); //移除倒计时的消息	}
}
```

4）反馈成功，开始连麦推流,并且播放主播的短延时流（小窗）---**被邀请方（观众）**

参考*com.alibaba.livecloud.videocall.presenter.WatchLivePresenter*

```javascript
/**
* 开始连麦
*/
private void startLaunchChat() {
	updateChatState(VideoChatStatus.TRY_MIX); //更新当前连麦状态为开始推流并尝试混流，等待混流成功
	mView.changePlayViewToChatMode();   //UI层从普通播放模式更改到连麦模式
	/**
	 * 注意： 这里推流输出视频尺寸必须是360 * 640
	 */
	mChatParter.onlineChat(mPushUrl,
								360,
								640,
								mView.getPreviewSurface(), 
								mMediaParam, 
								mSmallDelayPlayUrl);
}
```
5）邀请方收到成功混流通知，开始小窗播放---**邀请方（主播）**

参考*com.alibaba.livecloud.videocall.presenter.LivePresenter*

```javascript    
/**
 * 混流成功的消息处理Action
 */
ImHelper.Func<MsgDataMergeStream> mMergeStreamSuccFunc = new ImHelper.Func<MsgDataMergeStream>(){
	@Override
	public void action(MsgDataMergeStream msgDataMergeStream) {
		if (mVideoChatStatus == VideoChatStatus.TRY_MIX) {  //如果当前是开始混流并且等待混流成功的状态，则处理这条消息，否则视为无效的消息，不作处理
			mHandler.removeMessages(MSG_WHAT_MIX_STREAM_TIMEOUT);   //移除等待混流成功倒计时的消息
			if (mRoomID.equals(msgDataMergeStream.getInviteeRoomID())) {
				mChatRoomID = msgDataMergeStream.getInviterRoomID();
			} else if (mRoomID.equals(msgDataMergeStream.getInviterRoomID())) {
				mChatRoomID = msgDataMergeStream.getInviteeRoomID();
			} else {
				mLiveView.showToast(R.string.merge_stream_failed);
				updateChatState(VideoChatStatus.UNCHAT);   //更新连麦状态为未连麦
				return;
			}
			mLiveView.showLaunchChatUI();               //显示连麦状态的UI
			updateChatState(VideoChatStatus.MIX_SUCC);      //更新当前连麦状态为混流成功状态
		}
	}
};
```

6）主播结束连麦


参考——*com.alibaba.videocall.presenter.LivePresenter*
    
```javascript    
/**
 * 调用结束连麦的REST API
 */
public void closeLiveChat() {
	Subscriber closeChatSub = new Subscriber() {
		@Override
		public void onCompleted() {}

		@Override
		public void onError(Throwable e) {
			mLiveView.showCloseChatFailedUI();      //显示结束连麦失败的UI
		}

		@Override
		public void onNext(Object o) {
			abortChat(true);       //调用SDK结束连麦
		}
	};
	mServiceBI.terminateCall(mRoomID, mChatRoomID, closeChatSub);
}
```
```javascript
/**
 * 终止连麦
 */
public void abortChat(boolean isShowUI) {
	if (mChatHost != null && isChatting()) {
		mChatHost.abortChat();
		updateChatState(VideoChatStatus.UNCHAT);
		if(isShowUI) {
			mLiveView.showAbortChatUI();
		}
	}
}
```
7）观众结束连麦

参考——*com.alibaba.videocall.presenter.LivePresenter*

```javascript
/**
* 结束连麦
*/
public void closeVideoCall() {
	Subscriber closeChatSub = new Subscriber() {
		@Override
		public void onCompleted() {}

		@Override
		public void onError(Throwable e) {mView.showCloseChatFailedUI();}

		@Override
		public void onNext(Object o) {
			if (isChatting()) {
				abortChat(true);
			}
		}
	};
	mInviteServiceBI.terminateCall(mRoomID, mChatRoomID, closeChatSub);
}
```
```javascript
/**
* 结束连麦
* @param isShowUI 是否需要显示相应的UI
* 这里只是更新了UI，改变了surface的大小
* 注意：连麦播放和普通播放使用的是同一个surface，不同的是连麦播放的surface是小窗的大小，位于右下角
* 而普通播放则要动态修改SurfaceView的大小为全屏大小。所以这里真正调用SDK结束连麦的地方是在SurfaceView的
* surfaceChanged回调函数中。之所以要放在那里，是因为必须保证Surface先从小窗变成全屏，再结束，这样底层SDK
* 再遇到从短延时地址切换回普通播放地址时，普通播放地址拉流卡住的情况下，SDK会以全屏的大小重新渲染最后一帧画面。
* 否则会出现最后一帧画面依然是小窗的大小，但是Surface恢复到了全屏大小，则会有一大块Surface处于黑的状态。
*/
private void abortChat(boolean isShowUI) {
	if (isShowUI && !isDestoyed) {
		mView.changePlayViewToNormalMode(); //播放的View切换到正常的播放模式（小窗变大窗）
		mView.closeVideoChatSmallView();  //关闭小窗播放的UI更新
	}
}
```
参考——*com.alibaba.videocall.ui.WatchLiveActivity*
```javascript
SurfaceHolder.Callback mPlaySurfaceCB = new SurfaceHolder.Callback() {
	.......//省略

	@Override
	public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
		.......//省略
		if (shouldOffLine) {
			mWatchLivePresenter.sdkOfflineChat(); //在这里调用真正的offlineChat，保证渲染出得最后一帧数据是正常播放的尺寸，而不是小窗播放的尺寸
			shouldOffLine = false;
		}
	}
	.......//省略
};
```

**观众邀请主播连麦**

1）发送连麦邀请---**邀请方（观众）**

调用REST API，参考*com.alibaba.videocall.presenter.WatchLivePresenter*
    
```javascript    
/**
 * 发起连麦邀请
 */
public void inviteChat() {
	if (mChatStatus == VideoChatStatus.UNCHAT) {
		if (mInviteSub != null) {
			mInviteSub.unsubscribe();
		}
		mInviteSub = new Subscriber() {
			@Override
			public void onCompleted() {}

			@Override
			public void onError(Throwable e) {
				mChatStatus = VideoChatStatus.UNCHAT;   //更新当前连麦状态为未连麦
				mView.showInviteRequestFailedUI(e); //显示邀请连麦失败的UI
			}

			@Override
			public void onNext(Object o) {
				mHandler.sendEmptyMessageDelayed(MSG_WHAT_INVITE_CHAT_TIMEOUT, INVITE_CHAT_TIMEOUT_DELAY);//倒计时，10s后未收到回复，自动认为对方决绝。
				mView.showInviteRequestSuccessUI(); //显示邀请连麦成功的UI
			}
		};
		mInviteServiceBI.inviteCall(mUid, mAnchorUID, InviteForm.TYPE_PIC_BY_PIC, FeedbackForm.INVITE_TYPE_WATCHER, 										mRoomID, mInviteSub);
		mChatStatus = VideoChatStatus.INVITE_FOR_RES;   //更新当前连麦状态为邀请等待响应的状态
	} else {
		mView.showToast(R.string.not_allow_repeat_call);
	}
}
```

2）收到连麦邀请，显示连麦邀请处理Dialog---**被邀请方（主播）**

参考*com.alibaba.videocall.presenter.WatchLivePresenter#mInviteFunc*
    
```javascript    
/**
 * 连麦邀请的消息处理Action
 */
ImHelper.Func<MsgDataInvite> mInviteFunc = new ImHelper.Func<MsgDataInvite>() {
@Override
        public void action(final MsgDataInvite msgDataInvite) {
            mChatterName = msgDataInvite.getInviterName();
            mInviterUID = msgDataInvite.getInviterUID();
            mInviterType = msgDataInvite.getInviterType();

            //显示处理连麦邀请的Dialog
            mView.showFeedbackChooseDialog(msgDataInvite.getInviterName());
            updateChatState(VideoChatStatus.RECEIVED_INVITE);   //更新当前连麦状态为收到邀请等待反馈状态
            mHandler.sendEmptyMessageDelayed(MSG_WHAT_PROCESS_INVITING_TIMEOUT, INVITE_CHAT_TIMEOUT_DELAY); //超过10s自动拒绝连麦
        }
};
```

3）反馈邀请---**被邀请方（主播）**

调用REST API，参考*com.alibaba.videocall.presenter.LivePresenter*

```javascript    
/**
 * 反馈邀请
 * @param inviterType
 * @param inviteeType
 * @param inviterUID
 * @param inviteeUID
 * @param status
 */
private void internalRESTFeedback(int inviterType,
                             int inviteeType,
                             String inviterUID,
                             final String inviteeUID,
                             final int status) {
	.......//省略
		@Override
		public void onNext(InviteFeedbackResult result) {          
			if (status == FeedbackForm.STATUS_AGREE 				&& result != null) { 				mLiveView.showFeedbackSuccessfulUI(true); 				mChatRoomID = result.getInviterRoomID();    //缓存邀请方的RoomID 				/** 				 * 所谓的短延时URL实际上就是未经转码的原始流播放地址也就是，主播连麦观众时，主播端看到的观众的小窗画面，应该使用的播放地址 				 * 				 * 注意：这里没有直接就开始播放小窗，是因为这个时候观众端实际上还没有推流成功，需要等到收到推流成功的通知才开始播放 				 */ 				setSmallDelayPlayUrl(result.getInviteePlayUrl());       //缓存短延时URL 				updateChatState(VideoChatStatus.TRY_MIX);   //更新连麦状态为开始混流， 等待混流成功 				mHandler.sendEmptyMessageDelayed(MSG_WHAT_MIX_STREAM_TIMEOUT, MIX_STREAM_TIMEOUT);  //开始等待混流成功超时的倒计时 			} else if (status == FeedbackForm.STATUS_AGREE) { 				onError(new RuntimeException("Feedback Result is Null")); 			} else { 				mLiveView.showFeedbackSuccessfulUI(false); 			}       
			.......//省略
		}
	......//省略
}
```

4）收到同意连麦消息，开始连麦推流，并且播放主播的短延时流（小窗）---**邀请方（观众）**
参考——*com.alibaba.livecloud.videocall.presenter.WatchLivePresenter*

```javascript    
/**
 * 同意连麦的消息处理Action
 */
ImHelper.Func<MsgDataAgreeVideoCall> mAgreeFunc = new ImHelper.Func<MsgDataAgreeVideoCall>() {
	@Override
	public void action(final MsgDataAgreeVideoCall msgDataAgreeVideoCall) {
		mHandler.removeMessages(MSG_WHAT_INVITE_CHAT_TIMEOUT);//移除邀请等待响应超时倒计时的消息
		if (mChatStatus == VideoChatStatus.INVITE_FOR_RES) {
			mPushUrl = msgDataAgreeVideoCall.getRtmpUrl();
			mSmallDelayPlayUrl = msgDataAgreeVideoCall.getInviteePlayUrl();
			mChatterName = msgDataAgreeVideoCall.getInviteeName();
			mView.showFeedbackResultDialog(true,                    //显示对方回应邀请的结果UI
												msgDataAgreeVideoCall.getInviteeName(),
												msgDataAgreeVideoCall.getInviteeUID());
			mView.showChattingView();           //展示正在连麦的提醒UI
			startLaunchChat();                 //开始连麦，并且播放主播的短延时流（小窗）
		}	
	}
};
```
```javascript
/**
* 开始连麦，并且播放主播的短延时流（小窗）
*/
private void startLaunchChat() {
	updateChatState(VideoChatStatus.TRY_MIX); //更新当前连麦状态为开始推流并尝试混流，等待混流成功
	mView.changePlayViewToChatMode();   //UI层从普通播放模式更改到连麦模式
	/**
	 * 注意： 这里推流输出视频尺寸必须是360 * 640
	 */
	mChatParter.onlineChat(mPushUrl,
								360,
								640,
								mView.getPreviewSurface(), 
								mMediaParam, 
								mSmallDelayPlayUrl);
}
```

5）被邀请方收到成功混流通知，开始小窗播放---**被邀请方（主播）**

参考*com.alibaba.livecloud.videocall.ui.LiveActivity#mMergeStreamSuccFunc*

```javascript    
/**
 * 混流成功的消息处理Action
 */
ImHelper.Func<MsgDataMergeStream> mMergeStreamSuccFunc = new ImHelper.Func<MsgDataMergeStream>(){
	@Override
	public void action(MsgDataMergeStream msgDataMergeStream) {
		if (mVideoChatStatus == VideoChatStatus.TRY_MIX) {  //如果当前是开始混流并且等待混流成功的状态，则处理这条消息，否则视为无效的消息，不作处理
			mHandler.removeMessages(MSG_WHAT_MIX_STREAM_TIMEOUT);   //移除等待混流成功倒计时的消息
			if (mRoomID.equals(msgDataMergeStream.getInviteeRoomID())) {
				mChatRoomID = msgDataMergeStream.getInviterRoomID();
			} else if (mRoomID.equals(msgDataMergeStream.getInviterRoomID())) {
				mChatRoomID = msgDataMergeStream.getInviteeRoomID();
			} else {
				mLiveView.showToast(R.string.merge_stream_failed);
				updateChatState(VideoChatStatus.UNCHAT);   //更新连麦状态为未连麦
				return;
			}
			mLiveView.showLaunchChatUI();               //显示连麦状态的UI
			updateChatState(VideoChatStatus.MIX_SUCC);      //更新当前连麦状态为混流成功状态
		}
	}
};
```

6) 观众结束连麦

参考——*com.alibaba.videocall.presenter.LivePresenter*

```javascript
/**
* 结束连麦
*/
public void closeVideoCall() {
	Subscriber closeChatSub = new Subscriber() {
		@Override
		public void onCompleted() {}

		@Override
		public void onError(Throwable e) {mView.showCloseChatFailedUI();}

		@Override
		public void onNext(Object o) {
			if (isChatting()) {
				abortChat(true);
			}
		}
	};
	mInviteServiceBI.terminateCall(mRoomID, mChatRoomID, closeChatSub);
}
```
```javascript
/**
* 结束连麦
* @param isShowUI 是否需要显示相应的UI
* 这里只是更新了UI，改变了surface的大小
* 注意：连麦播放和普通播放使用的是同一个surface，不同的是连麦播放的surface是小窗的大小，位于右下角
* 而普通播放则要动态修改SurfaceView的大小为全屏大小。所以这里真正调用SDK结束连麦的地方是在SurfaceView的
* surfaceChanged回调函数中。之所以要放在那里，是因为必须保证Surface先从小窗变成全屏，再结束，这样底层SDK
* 再遇到从短延时地址切换回普通播放地址时，普通播放地址拉流卡住的情况下，SDK会以全屏的大小重新渲染最后一帧画面。
* 否则会出现最后一帧画面依然是小窗的大小，但是Surface恢复到了全屏大小，则会有一大块Surface处于黑的状态。
*/
private void abortChat(boolean isShowUI) {
	if (isShowUI && !isDestoyed) {
		mView.changePlayViewToNormalMode(); //播放的View切换到正常的播放模式（小窗变大窗）
		mView.closeVideoChatSmallView();  //关闭小窗播放的UI更新
	}
}
```
参考——*com.alibaba.videocall.ui.WatchLiveActivity*
```javascript
SurfaceHolder.Callback mPlaySurfaceCB = new SurfaceHolder.Callback() {
	.......//省略

	@Override
	public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
		.......//省略
		if (shouldOffLine) {
			mWatchLivePresenter.sdkOfflineChat(); //在这里调用真正的offlineChat，保证渲染出得最后一帧数据是正常播放的尺寸，而不是小窗播放的尺寸
			shouldOffLine = false;
		}
	}
	.......//省略
};
```
7）主播结束连麦


参考——*com.alibaba.videocall.presenter.LivePresenter*
    
```javascript    
/**
 * 调用结束连麦的REST API
 */
public void closeLiveChat() {
	Subscriber closeChatSub = new Subscriber() {
		@Override
		public void onCompleted() {}

		@Override
		public void onError(Throwable e) {
			mLiveView.showCloseChatFailedUI();      //显示结束连麦失败的UI
		}

		@Override
		public void onNext(Object o) {
			abortChat(true);       //调用SDK结束连麦
		}
	};
	mServiceBI.terminateCall(mRoomID, mChatRoomID, closeChatSub);
}
```
```javascript
/**
 * 终止连麦
 */
public void abortChat(boolean isShowUI) {
	if (mChatHost != null && isChatting()) {
		mChatHost.abortChat();
		updateChatState(VideoChatStatus.UNCHAT);
		if(isShowUI) {
			mLiveView.showAbortChatUI();
		}
	}
}
```


###参数配置与功能选择###
推流和播放可以进行一些参数配置，以及一些其他扩展功能如摄像头的调整、静音等可以选择使用。

**推流参数配置**

```javascript
//设置推流器推流相关参数      
mMediaParam.put(MediaConstants.PUBLISHER_PARAM_INIT_BITRATE, "" + 800000);        //初始码率
mMediaParam.put(MediaConstants.PUBLISHER_PARAM_MIN_BITRATE, "" + 600000);        //最小码率
mMediaParam.put(MediaConstants.PUBLISHER_PARAM_MAX_BITRATE, "" + 1000000);        //最大码率
mChatHost.setHostViewScalingMode(IMediaPublisher.VideoScalingMode.VIDEO_SCALING_MODE_SCALE_TO_FIT_WITH_CROPPING);
```

**播放参数配置** 

```
暂无
    
```

**Filter配置**

使用Map的方式，以便于后续的扩展.目前暂时只支持“打开/关闭美颜”这一个配置。

```javascript
//设置美颜开启        
mFilterMap.put(AlivcVideoChatHost.ALIVC_FILTER_PARAM_BEAUTY_ON, Boolean.toString(true));
mChatHost.setFilterParam(mFilterMap);    
```

**摄像头功能配置**

摄像头的功能包括有：切换前后摄像头、放大、聚焦等功能。具体的使用请参考接口说明。

1）切换前后摄像头

```javascript 
public void switchCamera();
```

2）放大摄像头

```javascript 
public void zoomCamera(float zoom);
```

3）聚焦摄像头

```javascript 
public void focusCameraAtAdjustedPoint(float xRatio, float yRatio);
```

**静音功能**

静音则指推流的时候不把音频发送出去，观众则听不到音频。

```javascript 
public void setPublisherMuteModeOn(boolean on);
```

**性能参数**

在推流和播放的时候，可以获取到一些性能参数，以便能够知道推流和播放的状态，异常情况，以及性能情况等。具体请参考接口文档。

```javascript 
//获取推流的性能参数值
public AlivcPublisherPerformanceInfo getPublisherPerformanceInfo();

//获取播放的性能参数值
public AlivcPlayerPerformanceInfo getPlayerPerformanceInfo();
```

**退到后台、锁屏、电话等中断的处理** 

以观众端为例：

1）进入中断——参考*com.alibaba.videocall.presenter.WatchLivePresenter*

```javascript
/**
 * 暂停播放 or 连麦
 */
public void mediaPause() {
	if (mChatParter != null && !mIsPublishPaused) {
		mChatParter.pause();
		mIsPublishPaused = true;
	}
}
```    

2）退出中断

参考*com.alibaba.videocall.ui.WatchLiveActivity*

```javascript
@Override
protected void onResume() {
	super.onResume();
	mConnectivityMonitor.register(this);        //注册对网络状态的监听
	mHeadsetMonitor.register(this);        //注册对耳机状态的监听

	mWatchLivePresenter.onResume();
	if ((mPlaySurfaceStatus != null
					&& mPlaySurfaceStatus != SurfaceStatus.DESTROYED)) {
		mWatchLivePresenter.mediaResume(mPlaySurfaceView, null);//恢复播放 or 连麦
	}
	......//省略
}
```
参考*com.alibaba.videocall.presenter.WatchLivePresenter*

```
/**
 * 恢复播放/连麦
 * @param previewSurf
 * @param playSurf
 */
public void mediaResume(SurfaceView previewSurf, SurfaceView playSurf) {
	if (mChatParter != null && mIsPublishPaused) {
		mChatParter.resume(previewSurf, playSurf);
		mIsPublishPaused = false;
	}
}
```
