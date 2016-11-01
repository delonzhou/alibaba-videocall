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
                ActivityCompat.requestPermissions(this, permissionManifest, PERMISSION_REQUEST_CODE);
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
    
```
	// 结束本次推流
    [self.publiserVideoCall stopPublishing];
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

需要在初始化时分别设置错误信息回调和状态信息回调，具体每个Error Code和Info Code的含义请参考
*com.alivc.publisher.MediaError*接口文档

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

每次播放都需要创建一个新的SurfaceView，并且将其与*com.alibaba.videocall.ui.WatchLiveActivity#mPlaySurfaceCB*
绑定，在mPlaySurfaceCB的surfaceCreated中执行播放的逻辑
    
```javascript    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ......//省略
    
        startToPlay(); // 一进入界面就播放主播直播
    }
```
创建一个新的SurfaceView，并且绑定mPlaySurfaceCB

```javascript
    `/**
     * 开始播放直播（非连麦）
     */
    public void startToPlay() {
        if(mBigSurfaceView != null) {
            mBigSurfaceView.getHolder().removeCallback(mPublishSurfaceCB);
        }
        mBigContainer.removeAllViews();
        mBigSurfaceView = new SurfaceView(this);
        LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.MATCH_PARENT, LinearLayout.LayoutParams.MATCH_PARENT);
        params.gravity = Gravity.CENTER;
        mBigSurfaceView.setLayoutParams(params);
        mBigContainer.addView(mBigSurfaceView);
        mBigSurfaceView.setZOrderOnTop(false);
        mBigSurfaceView.getHolder().addCallback(mPlaySurfaceCB);
        mPlayInBigView = true;
    }
```
```javascript
    SurfaceHolder.Callback mPlaySurfaceCB = new SurfaceHolder.Callback() {
        @Override
        public void surfaceCreated(SurfaceHolder holder) {
            holder.setType(SurfaceHolder.SURFACE_TYPE_GPU);
            holder.setKeepScreenOn(true);
            if (mPlayInBigView) {
                mWatchLivePresenter.startToPlay(mBigSurfaceView); //大窗播放，正常的直播流
            } else {
                mWatchLivePresenter.launchChatPlay(mSmallSurfaceView); //小窗播放，连麦对方的短延时直播流
            }
        }

        .......//省略
    };

```
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
调用邀请直播的REST API——*com.alibaba.livecloud.videocall.presenter.InvitePresenter*

```javascript
    /**
     * 邀请连麦
     * @param inviterUID
     * @param inviteeUID
     * @param inviterType
     */
    public void inviteVideoCall(String inviterUID, String inviteeUID, int inviterType) {
        ........//省略        
            
            @Override
            public void onNext(Object o) {
                mView.showToast(R.string.invite_succeed);
                mView.hideAnchorList();
            }
        
        ......//省略
    }
```


2） 收到邀请消息，显示邀请处理Dialog---**被邀请方（主播B）**
参考*com.alibaba.livecloud.videocall.ui.LiveActivity#mInviteFunc*

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
                    showFeedbackChooseDialog(msgDataInvite.getInviterName(), msgDataInvite.getInviterUID(), getUid());
                }
            });
        }
    };
```     

3） 反馈邀请---**被邀请方（主播B）**
调用Rest API *com.alibaba.livecloud.videocall.presenter.InviteFeedbackPresenter#feedbackInvite()*

```javascript
    /**
     * 反馈邀请
     * @param inviterType
     * @param inviteeType
     * @param inviterUID
     * @param inviteeUID
     * @param status
     */
    public void feedbackInvite(int inviterType,
                               int inviteeType,
                               String inviterUID,
                               final String inviteeUID,
                               final int status) {
        .......//省略
        
            @Override
            public void onNext(InviteFeedbackResult result) {
                if(status == FeedbackForm.STATUS_AGREE
                    && result != null) {
                    mView.hideFeedbackUI();
                    
                    //开始连麦
                    mView.startVideoCall(result.getInviteePlayUrl(), result.getRtmpUrl()); 
                }else if(status == FeedbackForm.STATUS_AGREE) {
                    onError(new RuntimeException("Feedback Result is Null"));
                }
            }
        
        .......//省略
    }
```

4） 邀请方和被邀请方收到混流成功通知，开始播放混流画面
参考*com.alibaba.livecloud.videocall.ui.LiveActivity#mMergeStreamSuccFunc*

```javascript    
    /**
     * 混流成功的消息处理Action
     */
    ImHelper.Func<MsgDataMergeStream> mMergeStreamSuccFunc = new ImHelper.Func<MsgDataMergeStream>() {

        @Override
        public void action(MsgDataMergeStream msgDataMergeStream) {
            if (mRoomID.equals(msgDataMergeStream.getInviteeRoomID())) {
                mChatRoomID = msgDataMergeStream.getInviterRoomID();
            } else {
                mChatRoomID = msgDataMergeStream.getInviteeRoomID();
            }
            mPreChatRoomID = mChatRoomID;
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    //播放混流画面
                    launchChat();
                }
            });
        }
    };
```

5）结束连麦

参考——*com.alibaba.videocall.ui.LiveActivity#abortChat()*
    
```javascript    
    /**
     * 结束连麦播放,并且隐藏小窗的UI（mIvAbortChat, mParterViewContainer）
     * 注意：因为下一次再连麦播放时需要一个新的SurfaceView，
     * 因此，这里需要将当前的playSurfaceView与mPlayCallback解绑，
     * 并且将当前的playSurfaceView从mParterViewContainer移除
     */
    private void abortChat() {
        if (mIsChatting && mChatHost != null) {
            
            //终止连麦
            mChatHost.abortChat();
            mIsChatting = false;
            if (mPlaySurfaceView != null) {
                mParterViewContainer.removeAllViews();
                mPlaySurfaceView.getHolder().removeCallback(mPlayCallback);
                mPlaySurfaceView = null;
            }
        }
        mIvAbortChat.setVisibility(View.GONE);
        mParterViewContainer.setVisibility(View.GONE);
    }
```
                
                
**主播邀请观众连麦**

1）发送邀请---**邀请方（主播）**
调用邀请直播的REST API——*com.alibaba.livecloud.videocall.presenter.InvitePresenter*

```javascript
    /**
     * 邀请连麦
     * @param inviterUID
     * @param inviteeUID
     * @param inviterType
     */
    public void inviteVideoCall(String inviterUID, String inviteeUID, int inviterType) {
        ........//省略        
            
            @Override
            public void onNext(Object o) {
                mView.showToast(R.string.invite_succeed);
                mView.hideAnchorList();
            }
        
        ......//省略
    }
```

2） 收到邀请，显示处理邀请Dialog---**被邀请方（观众）**

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
        }
    };
```

3）反馈邀请---**被邀请方（观众）**

调用Rest API 
*com.alibaba.livecloud.videocall.presenter.InviteFeedbackPresenter#feedbackInvite()*

```javascript
    /**
     * 反馈邀请
     * @param inviterType
     * @param inviteeType
     * @param inviterUID
     * @param inviteeUID
     * @param status
     */
    public void feedbackInvite(int inviterType,
                               int inviteeType,
                               String inviterUID,
                               final String inviteeUID,
                               final int status) {
        .......//省略
        
            @Override
            public void onNext(InviteFeedbackResult result) {
                if(status == FeedbackForm.STATUS_AGREE
                    && result != null) {
                    mView.hideFeedbackUI();
                    
                    //开始连麦推流
                    mView.startVideoCall(result.getInviteePlayUrl(), result.getRtmpUrl()); 
                }else if(status == FeedbackForm.STATUS_AGREE) {
                    onError(new RuntimeException("Feedback Result is Null"));
                }
            }
        
        .......//省略
    }
```

4）反馈成功，开始连麦推流---**被邀请方（观众）**

参考*com.alibaba.livecloud.videocall.ui.WatchLiveActivity#mFeedbackView*

```javascript
    private InviteFeedbackView mFeedbackView = new InviteFeedbackView() {
        ......//省略
        
        @Override
        public void startVideoCall(String smallDelayUrl, String pushUrl) {
            Log.d(TAG, "WatchLiveActivity-->startVideoCall, pushUrl:  "+pushUrl);
            mWatchLivePresenter.mPushUrl = pushUrl;
            mWatchLivePresenter.mSmallDelayPlayUrl = smallDelayUrl;
            startPublish();
        }
        
        ......//省略
    };
```
5）邀请方收到成功混流通知，开始小窗播放---**邀请方（主播）**

参考*com.alibaba.livecloud.videocall.ui.LiveActivity#mMergeStreamSuccFunc*

```javascript    
    /**
     * 混流成功的消息处理Action
     */
    ImHelper.Func<MsgDataMergeStream> mMergeStreamSuccFunc = new ImHelper.Func<MsgDataMergeStream>() {

        @Override
        public void action(MsgDataMergeStream msgDataMergeStream) {
            if (mRoomID.equals(msgDataMergeStream.getInviteeRoomID())) {
                mChatRoomID = msgDataMergeStream.getInviterRoomID();
            } else {
                mChatRoomID = msgDataMergeStream.getInviteeRoomID();
            }
            mPreChatRoomID = mChatRoomID;
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    //播放混流画面
                    launchChat();
                }
            });
        }
    };
```

6）被邀请方收到混流成功通知，开始小窗播放---**被邀请方（观众）**

参考——*com.alibaba.videocall.presenter.WatchLivePresenter#mMergeStreamSuccFunc*，*com.alibaba.videocall.ui.WatchLiveActivity#mView*

```javascript    
    /**
     * 混流成功的消息处理Action
     */
    private ImHelper.Func<MsgDataMergeStream> mMergeStreamSuccFunc = new ImHelper.Func<MsgDataMergeStream>() {
        @Override
        public void action(MsgDataMergeStream msgDataMergeStream) {
            /**
             * 这里之所以要加个判断，是因为混流成功的消息推送应用了环信聊天室，
             * 而环信聊天室，对于进入聊天室的用户都会推送最近的十条数据（可能是
             * 之前已经处理过的），所以这里为了避免对非正常推送数据进行处理，加入
             * 了mHasRequestChat作为一个flag，来判断是否有主动请求或者被动
             * 邀请的连麦，如果有，则处理混流的逻辑，如果没有则视为重复推送的无效
             * 消息
             */
            if (!mHasRequestChat) {
                Log.d(TAG, "not request chat");
                return;
            }
            mHasRequestChat = false;
            if (mRoomID.equals(msgDataMergeStream.getInviteeRoomID())) {
                mChatRoomID = msgDataMergeStream.getInviterRoomID();
            } else {
                mChatRoomID = msgDataMergeStream.getInviteeRoomID();
            }
            
            //小窗播放
            mView.callToChat();
            mIsChatting = true;
        }
    };
```

```javascript    
    WatchLiveView mView = new WatchLiveView() {
        .......//省略

        @Override
        public void callToChat() {
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    if (mPlayInBigView) {
                        //播放的时候无论是原始流地址还是混流地址，每次都需要创建一个新的SurfaceView
                        mSmallSurfaceView = new SurfaceView(WatchLiveActivity.this);
                        LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.MATCH_PARENT, LinearLayout.LayoutParams.MATCH_PARENT);
                        params.gravity = Gravity.CENTER;
                        mSmallSurfaceView.setLayoutParams(params);
                        mSmallContainer.addView(mSmallSurfaceView);
                        mSmallSurfaceView.setZOrderMediaOverlay(true);
                        mPlayInBigView = false;
                        mSmallSurfaceView.getHolder().addCallback(mPlaySurfaceCB);
                        mIvChatClose.setVisibility(View.VISIBLE);
                        mBottomFragment.showRecordView();
                    }
                }
            });
        }

        ......//省略
    }; 
```

7）结束连麦


参考——*com.alibaba.videocall.ui.LiveActivity#abortChat()*
    
```javascript    
    /**
     * 结束连麦播放,并且隐藏小窗的UI（mIvAbortChat, mParterViewContainer）
     * 注意：因为下一次再连麦播放时需要一个新的SurfaceView，
     * 因此，这里需要将当前的playSurfaceView与mPlayCallback解绑，
     * 并且将当前的playSurfaceView从mParterViewContainer移除
     */
    private void abortChat() {
        if (mIsChatting && mChatHost != null) {
            
            //终止连麦
            mChatHost.abortChat();
            mIsChatting = false;
            if (mPlaySurfaceView != null) {
                mParterViewContainer.removeAllViews();
                mPlaySurfaceView.getHolder().removeCallback(mPlayCallback);
                mPlaySurfaceView = null;
            }
        }
        mIvAbortChat.setVisibility(View.GONE);
        mParterViewContainer.setVisibility(View.GONE);
    }
```


**观众邀请主播连麦**

1）发送连麦邀请---**邀请方（观众）**

调用REST API，参考*com.alibaba.videocall.presenter.InvitePresenter#inviteVideoCall()*
    
```javascript    
    /**
     * 邀请连麦
     * @param inviterUID
     * @param inviteeUID
     * @param inviterType
     */
    public void inviteVideoCall(String inviterUID, String inviteeUID, int inviterType) {
        if(mInviteSub != null) {
            mInviteSub.unsubscribe();
        }
        mInviteSub = new Subscriber() {
           .......//省略
        };
        mServiceBI.inviteCall(inviterUID, inviteeUID, InviteForm.TYPE_PIC_BY_PIC, inviterType, mInviteSub);
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
            ......//省略

            //显示处理连麦邀请的Dialog
            mView.showFeedbackChooseDialog(msgDataInvite.getInviterName());
        }
    };
```

3）反馈邀请---**被邀请方（主播）**

调用REST API，参考*com.alibaba.videocall.presenter.InviteFeedbackPresenter#feedbackInvite()*

```javascript    
    /**
     * 反馈邀请
     * @param inviterType
     * @param inviteeType
     * @param inviterUID
     * @param inviteeUID
     * @param status
     */
    public void feedbackInvite(int inviterType,
                               int inviteeType,
                               String inviterUID,
                               final String inviteeUID,
                               final int status) {
        if(mFeedbackSub != null) {
            mFeedbackSub.unsubscribe();
        }
        mFeedbackSub = new Subscriber<InviteFeedbackResult>() {
            .......//省略

            @Override
            public void onNext(InviteFeedbackResult result) {
                if(status == FeedbackForm.STATUS_AGREE
                    && result != null) {
                    mView.hideFeedbackUI();

                    mView.startVideoCall(result.getInviteePlayUrl(), result.getRtmpUrl()); //同意，开始连麦
                }else if(status == FeedbackForm.STATUS_AGREE) {
                    onError(new RuntimeException("Feedback Result is Null"));
                }
            }
        };
        mServiceBI.feedback(inviteeType, inviterType, inviterUID, inviteeUID,
                InviteForm.TYPE_PIC_BY_PIC, status, mFeedbackSub);
    }
```

4）收到同意连麦消息，开始连麦推流---**邀请方（观众）**

```javascript    
    /**
     * 同意连麦的消息处理Action
     */
    ImHelper.Func<MsgDataAgreeVideoCall> mAgreeFunc = new ImHelper.Func<MsgDataAgreeVideoCall>() {
        @Override
        public void action(final MsgDataAgreeVideoCall msgDataAgreeVideoCall) {
            .......//省略
            
            mView.showFeedbackResultDialog(
                    String.format(mContext.getString(R.string.agree_message),
                            msgDataAgreeVideoCall.getInviteeName() + "(" + msgDataAgreeVideoCall.getInviteeUID() + ")"), true);

        }

    };
```
```javascript
   /**
     * 开始连麦推流
     *
     * 需要先停止播放，并且解除mBigSurfaceView与mPlaySurfaceCB绑定，然后创建一个新的SurfaceView,
     * 并且使这个SurfaceView绑定mPublishSurfaceCB
     */
        public void startPublish() {

        mBigSurfaceView = new SurfaceView(WatchLiveActivity.this);
        LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.MATCH_PARENT, LinearLayout.LayoutParams.MATCH_PARENT);
        params.gravity = Gravity.CENTER;
        mBigSurfaceView.setLayoutParams(params);
        mBigContainer.addView(mBigSurfaceView);
        mBigSurfaceView.setZOrderOnTop(false);
        mBigSurfaceView.getHolder().addCallback(mPublishSurfaceCB);
    }    
```

在*com.alibaba.videocall.ui.WatchLiveActivity#mPublishSurfaceCB*的surfaceCreated方法中执行连麦推流的逻辑

```javascript
    SurfaceHolder.Callback mPublishSurfaceCB = new SurfaceHolder.Callback() {
        @Override
        public void surfaceCreated(SurfaceHolder holder) {
            holder.setType(SurfaceHolder.SURFACE_TYPE_GPU);
            holder.setKeepScreenOn(true);

            /**
             * 注意： 这里推流输出视频尺寸必须是360 * 640
             */
            mWatchLivePresenter.launchChat(360,
                    640,
                    mBigSurfaceView.getHolder().getSurface());
            
            .......//省略
        }

        .......//省略
    };
```
```javascript
    /**
     * 开始连麦推流
     * @param width
     * @param height
     * @param surface
     */
    public void launchChat(int width, int height, Surface surface) {
        Log.d(TAG, "WatchActivity --> onlineChat");
        mChatParter.onlineChat(mPushUrl,
                width,
                height,
                surface, mMediaParam,mSmallDelayPlayUrl);
    }
```

5）被邀请方收到成功混流通知，开始小窗播放---**被邀请方（主播）**

参考*com.alibaba.livecloud.videocall.ui.LiveActivity#mMergeStreamSuccFunc*

```javascript    
    /**
     * 混流成功的消息处理Action
     */
    ImHelper.Func<MsgDataMergeStream> mMergeStreamSuccFunc = new ImHelper.Func<MsgDataMergeStream>() {

        @Override
        public void action(MsgDataMergeStream msgDataMergeStream) {
            ......//省略
        
        runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    launchChat(); //播放混流画面
                }
            });
        }
    };
```

6）邀请方收到混流成功通知，开始小窗播放---**邀请方（观众）**

参考——*com.alibaba.videocall.presenter.WatchLivePresenter#mMergeStreamSuccFunc*，*com.alibaba.videocall.ui.WatchLiveActivity#mView*

```javascript    
    /**
     * 混流成功的消息处理Action
     */
    private ImHelper.Func<MsgDataMergeStream> mMergeStreamSuccFunc = new ImHelper.Func<MsgDataMergeStream>() {
        @Override
        public void action(MsgDataMergeStream msgDataMergeStream) {
            /**
             * 这里之所以要加个判断，是因为混流成功的消息推送应用了环信聊天室，
             * 而环信聊天室，对于进入聊天室的用户都会推送最近的十条数据（可能是
             * 之前已经处理过的），所以这里为了避免对非正常推送数据进行处理，加入
             * 了mHasRequestChat作为一个flag，来判断是否有主动请求或者被动
             * 邀请的连麦，如果有，则处理混流的逻辑，如果没有则视为重复推送的无效
             * 消息
             */
            if (!mHasRequestChat) {
                Log.d(TAG, "not request chat");
                return;
            }
            mHasRequestChat = false;
            
            .......//省略
            
            mView.callToChat(); //更新连麦成功后UI
            
            ......//省略
        }
    };
```

7) 结束连麦

```javascript
    /**
     * 结束连麦
     */
    private void abortChat() {

        //中断连麦推流
        mWatchLivePresenter.abortChatPush();

        mView.closeVideoChatSmallView(); //关闭小窗播放的UI更新
    }
```


###参数配置与功能选择###
推流和播放可以进行一些参数配置，以及一些其他扩展功能如摄像头的调整、静音等可以选择使用。

**推流参数配置**

```javascript
    
        //设置推流器推流相关参数      mMediaParam.put(MediaConstants.PUBLISHER_PARAM_INIT_BITRATE, "" + 800000);        //初始码率
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

```

        //设置美颜开启        
        mFilterMap.put(AlivcVideoChatHost.ALIVC_FILTER_PARAM_BEAUTY_ON, Boolean.toString(true));
        
        mChatHost.setFilterParam(mFilterMap);    
```

**摄像头功能配置**

摄像头的功能包括有：切换前后摄像头、放大、聚焦等功能。具体的使用请参考接口说明。

1）切换前后摄像头

``` 
public void switchCamera();

```

2）放大摄像头

``` 
public void zoomCamera(float zoom);

```

3）聚焦摄像头

``` 
public void focusCameraAtAdjustedPoint(float xRatio, float yRatio);

```

**静音功能**

静音则指推流的时候不把音频发送出去，观众则听不到音频。

``` 
public void setPublisherMuteModeOn(boolean on);

```

**性能参数**

在推流和播放的时候，可以获取到一些性能参数，以便能够知道推流和播放的状态，异常情况，以及性能情况等。具体请参考接口文档。

``` 
//获取推流的性能参数值
public AlivcPublisherPerformanceInfo getPublisherPerformanceInfo();

//获取播放的性能参数值
public AlivcPlayerPerformanceInfo getPlayerPerformanceInfo();

```

**退到后台、锁屏、电话等中断的处理** 

以观众端为例：

1）进入中断

```
@Override
    protected void onPause() {
        super.onPause();
        mWatchLivePresenter.onPause();
        dismissLogInfoUI();
    }
```

```
/**
     * 对应{@link Activity#onPause()}
     */
    public void onPause() {
        mImHelper.unRegister(MessageType.LIVE_COMPLETE);
        mImHelper.unRegister(MessageType.AGREE_CALLING);
        mImHelper.unRegister(MessageType.NOT_AGREE_CALLING);
        mImHelper.unRegister(MessageType.CALLING_FAILED);
        mImHelper.unRegister(MessageType.CALLING_SUCCESS);
        mImHelper.unRegister(MessageType.TERMINATE_CALLING);
        mImHelper.unRegister(MessageType.INVITE_CALLING);

        if (mChatParter != null && !mIsPublishPaused) {
            mChatParter.pause();
            Log.d(TAG, "WatchLiveActivity--> mChatParter.pause()");
            mIsPublishPaused = true;
        }
        isCaching = false;
        mErrorHandler.removeCallbacks(mShowInterruptRun);
    }
```    

2）退出中断

```
@Override
    protected void onResume() {
        super.onResume();
        mWatchLivePresenter.onResume();
        if ((mSurfaceStatus != SurfaceStatus.DESTROYED)
                && mBigSurfaceView != null) {
            mWatchLivePresenter.chatResume(mBigSurfaceView, mSmallSurfaceView);
        }

        if (mAppSettings.isShowLogInfo(false)) {
            showLogInfoUI();
        } else {
            dismissLogInfoUI();
        }
    }
```
```
/**
     * 恢复播放/连麦
     *
     * @param previewSurf
     * @param playSurf
     */
    public void chatResume(SurfaceView previewSurf, SurfaceView playSurf) {
        if (mChatParter != null && mIsPublishPaused) {
            Log.d(TAG, "WatchLiveActivity-->mChatParter.resume()");
            mChatParter.resume(previewSurf, playSurf);
            mIsPublishPaused = false;
        }
    }
```


##接口说明##

针对连麦场景中存在的主播和观众这两个不同的角色，SDK中提供了两个类AliVcVideoChatHost、AlivcVideoChatParter分别来实现主播端、观众端的各项功能。同时，我们对连麦过程中的各种通知进行了定义，并且提供了接口来获取连麦过程中推流和播放的状态信息。

###AlivcVideoChatHost的接口和事件通知###

接口名称|功能描述
-------|------
init | 初始化
prepareToPublish	| 准备推流，建立预览界面
startToPublish	| 开始推流
stopPublishing	| 结束推流
finishPublishing	| 结束推流预览
launchChat	| 开始连麦
abortChat	| 结束连麦
release | 释放资源
pause | 暂停推流或连麦
resume | 继续推流或连麦
setPlayerParam	| 设置连麦后播放参数
switchCamera	| 切换前后摄像头
zoomCamera	| 缩放摄像头
focusCameraAtAdjustedPoint	| 聚焦摄像头到某个位置
setFilterParam	| 设置滤镜参数
getPublisherPerformanceInfo	| 获取推流性能参数
getPlayerPerformanceInfo	| 获取连麦后播放性能参数
setPublisherMuteMode	| 推流是否静音
getSDKVersion	 | 获取SDK版本号

错误通知 | 内容描述
-------|------
MediaError.ALIVC_ ERR_ MEMORY_ POOL |	内存不足
MediaError.ALIVC_ ERR_ PUBLISHER_ OPEN_ FAILED |	推流端打开失败。可能是网络未连接或推流地址错误
MediaError.ALIVC_ ERR_ PUBLISHER_ SEND_ DATA_ TIMEOUT |	推流端发送数据超时
MediaError.ALIVC_ ERR_ PUBLISHER_ NETWORK_ POOL	| 推流端网络差
MediaError.ALIVC_ ERR_ PUBLISHER_ VIDEO_ CAPTURE_ DISABLED	| 视频采集被禁止
MediaError.ALIVC_ ERR_ PUBLISHER_ AUDIO_ CAPTURE_ DISABLED	| 音频采集被禁止
MediaError.ALIVC_ ERR_ PUBLISHER_ VIDEO_ ENCODER_ INIT_ FAILED	| 视频编码器初始化失败
MediaError.ALIVC_ ERR_ PUBLISHER_ AUDIO_ ENCODER_ INIT_ FAILED	| 音频编码器初始化失败
MediaError.ALIVC_ ERR_ PUBLISHER_ ENCODE_ VIDEO_ FAILED	| 推流端视频帧编码错误
MediaError.ALIVC_ ERR_ PUBLISHER_ ENCODE_ AUDIO_ FAILED	| 推流端音频帧编码错误
MediaError.ALIVC_ ERR_ PLAYER_ OPEN_ FAILED	| 播放器打开失败。可能是网络未连接或播放地址错误
MediaError.ALIVC_ ERR_ PLAYER_ READ_ PACKET_ TIMEOUT	| 播放端下载数据超时
MediaError.ALIVC_ ERR_ PLAYER_ NO_ SURFACEVIEW	| 播放器无效SurfaceView
MediaError.ALIVC_ ERR_ PLAYER_ INVALID_ CODEC	| 播放端音视频格式无效

监听事件 | 内容描述
-------|------
MediaError.ALIVC_ INFO_ PLAYER_ FIRST_ FRAME_ RENDER |	播放器首帧渲染通知
MediaError.ALIVC_ INFO_ PLAYER_ BUFFERING_ START | 	播放器缓冲开始通知
MediaError.ALIVC_ INFO_ PLAYER_ BUFFERING_ END |	播放器缓冲结束通知
MediaError.ALIVC_ INFO_ PLAYER_ STOP_ PROCESS_ FINISHED |	播放器stop操作完成通知
MediaError.ALIVC_ INFO_ PLAYER_ PREPARED_ PROCESS_ FINISHED |	播放器prepare操作完成通知
MediaError.ALIVC_ INFO_ PLAYER_ INTERRUPT_ PLAYING |	播放器视频播放中断通知

<br>
接口的具体描述如下：

**init**

```
public int init(Context context);
```

功能：初始化

参数：
context: Android应用上下文

返回值：0 表示成功，非0为失败。


**prepareToPublish**

```
public int prepareToPublish(Surface previewSurface, int width, int height, Map<String,String> publisherParam);

```
功能：准备推流。调用该函数后将对音视频采集的硬件设备进行初始化，对音视频编码器进行初始化，并开启美颜等滤镜。同时，主播可以预览到经过滤镜处理以后的视频效果。

参数：

preivewSurface：推流或连麦过程中供主播预览的view。

width、height：推流视频的宽和高。

publisherParam：主播推流的参数。使用Map的方式，以便于后续的扩展。目前可以设置的参数如下：

* MediaConstants.PUBLISHER_ PARAM_ UPLOAD_ TIMEOUT：	推流上传超时时间，单位ms,默认8000。
* MediaConstants.PUBLISHER_ PARAM_ CAMERA_ POSITION：	选择前后摄像头，枚举成员：
cameraPositionFront = 0,
cameraPositionBack = 1。默认前置。
* MediaConstants.PUBLISHER_ PARAM_ SCREEN_ ROTATION：屏幕旋转角度：
竖屏：0；横屏（左侧是头部Home键在右边）：90；竖屏（反向）：180；横屏（头部在右边）：270。默认为0。
* MediaConstants.PUBLISHER_ PARAM_ MAX_ BITRATE：推流最大码率，单位Kbps。默认1500。
* MediaConstants.PUBLISHER_ PARAM_ MIN_ BITRATE：推流最小码率，单位Kbps。默认200。
* MediaConstants.PUBLISHER_ PARAM_ ORIGINAL_ BITRATE：	推流初始码率，单位Kbps。默认500。
* MediaConstants.PUBLISHER_ PARAM_ AUDIO_ SAMPLE_ RATE：	推流音频采样率，单位Hz。固定32000，暂不可调。
* MediaConstants.PUBLISHER_ PARAM_ AUDIO_ BITRATE：	推流音频码率，单位Kbps。固定96，暂不可调。
* MediaConstants.PUBLISHER_ PARAM_ FRONT_ CAMERA_ MIRROR：	前置摄像头是否镜像。

返回值：0表示成功，非0表示失败。

备注：目前视频编码采用的是软编码，软编码条件下只支持两种分辨率：360x640、480x848（横屏推流的时候为640x360、848x480）。

**startToPublish**

```
public int startToPublish(String url);

```
功能：开始推流。调用该函数将启动音视频的编码，并将压缩后的音视频流打包上传到服务器。

参数：

url：主播推流地址。

返回值： 0表示成功；非0表示失败。

备注：此处的推流仅仅是主播单向的直播推流，与连麦这种双向互动没有关系。必须先调用函数prepareToPublish后才能调用该函数。

**stopPublishing**

```
public int stopPublishing();

```

功能：结束推流。调用该函数将结束本次的直播推流，并关闭音视频编码功能，但采集、滤镜功能仍然运行，预览功能仍然保留。

参数：无。

返回值： 0表示成功；非0表示失败。

备注：若在连麦状态下调用该函数，则sdk会先停止连麦，再结束推流。


**finishPublishing**

```
public int finishPublishing();

```

功能：退出推流直播。调用该函数将停止采集、滤镜功能，销毁预览窗口，释放所有资源。

参数：无。

返回值： 0表示成功；非0表示失败。

备注：若调用了函数startToPublish，则必须调用函数stopPublishing以后才可以调用该函数。


**release**

```

public int release();

```

功能：释放资源

参数：无

返回值：0 表示成功，非0为失败。


**launchChat**

```
public int launchChat(String url, SurfaceView parterView);

```

功能：开始连麦。这种连麦可以是与观众的连麦，也可以是与另一个主播进行的连麦。调用该函数启动播放器，将连麦参与方上传的视频在另一个窗口中播放出来。

参数：

url：用于播放连麦参与方的视频的地址。

parterView：连麦参与方视频的播放窗口。

返回值： 0表示成功；非0表示失败。

备注：必须调用函数startToPublish后才能调用该函数。


**abortChat**

```
public int abortChat（）;

```

功能：结束连麦。调用该函数将关闭播放器，销毁用于播放的窗口。

参数：无。

返回值： 0表示成功；非0表示失败。

备注：无。

**pause**

```

public int pause();

```

功能：暂停推流和连麦。

参数：无

返回值：0 表示成功，非0为失败。

备注：仅限于退到后台、锁屏或电话等中断出现时调用。正常推流和连麦过程中请勿调用该函数。

**resume**

```

public int resume(SurfaceView hostView,SurfaceView parterView);

```

功能：恢复推流和连麦。

参数：

hostView: 当前主播预览的SurfaceView。

parterView：连麦参与方的SurfaceView。

返回值：0 表示成功，非0为失败。

备注：若发生退到后台、锁屏或电话等中断时，系统调用了pause函数。此时，如果需要继续进行推流或连麦，可以调用该函数。正常推流和连麦过程中请勿调用该函数。



**setPlayerParam**

```
public void setPlayerParam(Map<String,String> param); // 参数与说明不匹配，param 还是 playerParam

```

功能：设置连麦过程中播放器相关的配置参数。

参数：

playerParam：播放器相关的配置参数。使用Map的方式，以便于后续的扩展。目前可以设置的参数如下：

* MediaConstants.PLAYER_ PARAM_ DOWNLOAD_ TIMEOUT：	连麦播放缓冲超时时间，单位ms。默认15000。
* MediaConstants.PLAYER_ PARAM_ DROP_ BUFFER_ DURATION：	连麦播放开始丢帧阈值，单位ms。默认1000。
* MediaConstants.PLAYER_ PARAM_ SCALING_ MODE：	连麦播放显示模式，目前支持2种。枚举成员VIDEO_ SCALING_ MODE_ SCALE_ TO_ FIT = 0，代表等比例缩放，若显示窗口宽高比与视频不同则会有黑边；VIDEO_ SCALING_ MODE_ SCALE_ TO_ FIT_ WITH_ CROPPING = 1，代表带切边的等比例缩放，若显示窗口宽高比与视频不同，则自动对视频裁边以撑满显示窗口。 默认值为1。

备注：这些参数可以在连麦开始之前设置，也可以在连麦过程中进行调整。

**switchCamera**

```
public void switchCamera（）;

```

功能：切换摄像头。

参数：无。

备注：该函数可以在推流和连麦的过程中随时进行调用。


**zoomCamera**

```
public void zoomCamera（float zoom）;
```

功能：设置摄像头缩放倍率。调用该函数将对当前视频进行光学缩放。缩放后的视频将显示在预览窗口。

参数：

zoom：缩放倍率。0~1之间时，表示缩小；大于1，表示放大。

备注：该函数仅对后置摄像头有效。


**focusCameraAtAdjustedPoint**

```
public void focusCameraAtAdjustedPoint(float xRatio, float yRatio);

```

功能：聚焦到某个设置的点。调用该函数可以聚焦到预览窗口上人为指定的某个点。

参数：

xRatio,yRatio：需要聚焦到的点的位置。（0.0，0.0）代表左上角，（1.0，1.0）代表右下角，（0.5，0.5）代表中心点。

备注：无。

**setFilterParam**

```
public void setFilterParam(Map<String, String> param);

```
功能：设置滤镜的相关参数。

参数：

param：滤镜相关的配置参数。使用Map的方式，以便于后续的扩展。目前只有一个美颜的滤镜，可以设置的参数如下：

* MediaConstants.FILTER_ PARAM_ BEAUTY_ON：	美颜是否开启

备注：该函数可以在推流前调用，也可以在推流和连麦过程中调用。

**getPublisherPerformanceInfo**

```
public AlivcPublisherPerformanceInfo getPublisherPerformanceInfo();

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
public AlivcPlayerPerformanceInfo getPlayerPerformanceInfo();

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

**setPublisherMuteMode**

```
public void setPublisherMuteModeOn(boolean on);

```

功能：设置推流端的静音模式。

参数：

on：  true 表示打开静音模式；false 表示关闭静音模式。

备注：无

**getSDKVersion**

```
public String getSDKVersion();

```

功能：获取版本信息。

参数：无

返回值：版本号。


###AlivcVideoChatParter的接口和事件通知###

接口名称|功能描述
----|----
init | 初始化
startToPlay	| 开始观看直播
stopPlaying	| 结束观看直播
onlineChat	| 开始连麦
offlineChat	| 结束连麦
release | 释放资源
pause | 暂停推流或连麦
resume | 继续推流或连麦
setPlayerParam	| 设置播放参数
switchCamera	| 切换前后摄像头
zoomCamera	| 缩放摄像头
focusCameraAtAdjustedPoint	| 聚焦摄像头到某个位置
setFilterParam	| 设置滤镜参数
getPublisherPerformanceInfo	| 获取推流性能参数
getPlayerPerformanceInfo	| 获取连麦后播放性能参数
setPublisherMuteMode	| 推流是否静音
getSDKVersion	| 获取SDK版本号

错误通知 | 内容描述
-------|------
MediaError.ALIVC_ ERR_ MEMORY_ POOL |	内存不足
MediaError.ALIVC_ ERR_ PUBLISHER_ OPEN_ FAILED |	推流端打开失败。可能是网络未连接或推流地址错误
MediaError.ALIVC_ ERR_ PUBLISHER_ SEND_ DATA_ TIMEOUT |	推流端发送数据超时
MediaError.ALIVC_ ERR_ PUBLISHER_ NETWORK_ POOL	| 推流端网络差
MediaError.ALIVC_ ERR_ PUBLISHER_ VIDEO_ CAPTURE_ DISABLED	| 视频采集被禁止
MediaError.ALIVC_ ERR_ PUBLISHER_ AUDIO_ CAPTURE_ DISABLED	| 音频采集被禁止
MediaError.ALIVC_ ERR_ PUBLISHER_ VIDEO_ ENCODER_ INIT_ FAILED	| 视频编码器初始化失败
MediaError.ALIVC_ ERR_ PUBLISHER_ AUDIO_ ENCODER_ INIT_ FAILED	| 音频编码器初始化失败
MediaError.ALIVC_ ERR_ PUBLISHER_ ENCODE_ VIDEO_ FAILED	| 推流端视频帧编码错误
MediaError.ALIVC_ ERR_ PUBLISHER_ ENCODE_ AUDIO_ FAILED	| 推流端音频帧编码错误
MediaError.ALIVC_ ERR_ PLAYER_ OPEN_ FAILED	| 播放器打开失败。可能是网络未连接或播放地址错误
MediaError.ALIVC_ ERR_ PLAYER_ READ_ PACKET_ TIMEOUT	| 播放端下载数据超时
MediaError.ALIVC_ ERR_ PLAYER_ NO_ SURFACEVIEW	| 播放器无效SurfaceView
MediaError.ALIVC_ ERR_ PLAYER_ INVALID_ CODEC	| 播放端音视频格式无效

监听事件 | 内容描述
-------|------
MediaError.ALIVC_ INFO_ PLAYER_ FIRST_ FRAME_ RENDER |	播放器首帧渲染通知
MediaError.ALIVC_ INFO_ PLAYER_ BUFFERING_ START | 	播放器缓冲开始通知
MediaError.ALIVC_ INFO_ PLAYER_ BUFFERING_ END |	播放器缓冲结束通知
MediaError.ALIVC_ INFO_ PLAYER_ STOP_ PROCESS_ FINISHED |	播放器stop操作完成通知
MediaError.ALIVC_ INFO_ PLAYER_ PREPARED_ PROCESS_ FINISHED |	播放器prepare操作完成通知
MediaError.ALIVC_ INFO_ PLAYER_ INTERRUPT_ PLAYING |	播放器视频播放中断通知

<br>
接口的具体描述如下：


**init**

```
public int init(Context context);

```

功能：初始化

参数：

context: Android应用上下文

返回值：0 表示成功，非0为失败。


**startToPlay**

```
public int startToPlay(String url, SurfaceView surfaceView);

```

功能：开始观看直播。调用该函数将启动直播播放器，并播放获取到的直播流。

参数：

url：直播地址。

surfaceView：直播播放器的渲染窗口。

返回值： 0表示成功；非0表示失败。

备注：无。

**stopPlaying**

```
public int stopPlaying();

```

功能：结束观看直播。调用该函数将关闭直播播放器，并销毁所有资源。

参数：无。

返回值： 0表示成功；非0表示失败。

备注：若在连麦状态下调用该函数，则sdk会先停止连麦，再结束播放。

**release**

```
public int release();

```

功能：释放资源

参数：无

返回值：0 表示成功，非0为失败。


**onlineChat**

```
public int onlineChat(String publisherUrl, int width, int height, Surface previewSurface, Map<String,String> publisherParam，String playerUrl

```
功能：开始连麦。调用该函数将开启音视频的采集设备、启动预览功能、启动音视频编码功能并将压缩后的音视频流上传。同时将播放地址切换到具备短延时功能的新地址。

参数：

publisherUrl：连麦时推流的地址。

playerUrl：连麦时切换到的短延时播放地址。

width，height：编码视频的宽和高。

previewSurface：连麦时推流的预览窗口。

publisherParam：连麦时推流的参数。使用Map的方式，以便于后续的扩展。目前可以设置的参数如下：（与类
AlivcVideoChatHost中接口函数prepareToPublish的参数publisherParam相同）

* MediaConstants.PUBLISHER_ PARAM_ UPLOAD_TIMEOUT：	推流上传超时时间，单位ms,默认8000。
* MediaConstants.PUBLISHER_ PARAM_ CAMERA_ POSITION：	选择前后摄像头，枚举成员：
cameraPositionFront = 0,
cameraPositionBack = 1。默认前置。
* MediaConstants.PUBLISHER_ PARAM_ SCREEN_ ROTATION：屏幕旋转角度：
竖屏：0；横屏（左侧是头部Home键在右边）：90；竖屏（反向）：180；横屏（头部在右边）：270。默认为0。
* MediaConstants.PUBLISHER_ PARAM_ MAX_ BITRATE：推流最大码率，单位Kbps。默认1500。
* MediaConstants.PUBLISHER_ PARAM_ MIN_ BITRATE：推流最小码率，单位Kbps。默认200。
* MediaConstants.PUBLISHER_ PARAM_ ORIGINAL_ BITRATE：	推流初始码率，单位Kbps。默认500。
* MediaConstants.PUBLISHER_ PARAM_ AUDIO_ SAMPLE_ RATE：	推流音频采样率，单位Hz。固定32000，暂不可调。
* MediaConstants.PUBLISHER_ PARAM_ AUDIO_ BITRATE：	推流音频码率，单位Kbps。固定96，暂不可调。
* MediaConstants.PUBLISHER_ PARAM_ FRONT_ CAMERA_ MIRROR：	前置摄像头是否镜像。 // 此处ALIVC不要

返回值： 0表示成功；非0表示失败。

备注：目前视频编码采用的是软编码，软编码条件下只支持两种分辨率：360x640、480x848（横屏推流的时候为640x360、848x480）。

**offlineChat**

```
public int offlineChat();

```

功能：结束连麦。调用该函数将结束观众的推流，销毁推流的所有资源，并将播放地址切换到连麦之前的地址。

参数：无。

返回值： 0表示成功；非0表示失败。

备注：无。

**pause**

```
public int pause();

```

功能：暂停播放或连麦。

参数：无

返回值：0 表示成功，非0为失败。

备注：仅限于退到后台、锁屏或电话等中断出现时调用。正常播放或连麦过程中请勿调用该函数。

**resume**

```

public int resume(SurfaceView hostView,SurfaceView parterView);

```

功能：恢复播放或连麦。

参数：

hostView: 当前主播预览的SurfaceView。

parterView：连麦参与方的SurfaceView。

返回值：0 表示成功，非0为失败。

备注：若发生退到后台、锁屏或电话等中断时，系统调用了pause函数。此时，如果需要继续进行推流或连麦，可以调用该函数。正常播放或连麦过程中请勿调用该函数。

**setPlayerParam**

```
public void setPlayerParam(Map<String,String> param); // 参数与说明不匹配，param 还是 playerParam

```

功能：设置连麦过程中播放器相关的配置参数。

参数：

playerParam：播放器相关的配置参数。使用Map的方式，以便于后续的扩展。目前可以设置的参数如下：

* MediaConstants.PLAYER_ PARAM_ DOWNLOAD_ TIMEOUT：	连麦播放缓冲超时时间，单位ms。默认15000。
* MediaConstants.PLAYER_ PARAM_ DROP_ BUFFER_ DURATION：	连麦播放开始丢帧阈值，单位ms。默认1000。
* MediaConstants.PLAYER_ PARAM_ SCALING_ MODE：	连麦播放显示模式，目前支持2种。枚举成员VIDEO_ SCALING_ MODE_ SCALE_ TO_ FIT = 0，代表等比例缩放，若显示窗口宽高比与视频不同则会有黑边；VIDEO_ SCALING_ MODE_ SCALE_ TO_ FIT_ WITH_ CROPPING = 1，代表带切边的等比例缩放，若显示窗口宽高比与视频不同，则自动对视频裁边以撑满显示窗口。 默认值为1。

备注：与类AlivcVideoChatHost中接口函数setPlayerParam功能与参数设置相同。

**switchCamera**

```
public void switchCamera;
```

功能：切换摄像头。

参数：无。

备注：该函数可以在连麦的过程中随时进行调用。


**zoomCamera**

```
public void zoomCamera(float zoom);

```

功能：设置摄像头缩放倍率。调用该函数将对当前视频进行光学缩放。缩放后的视频将显示在预览窗口。

参数：

zoom：缩放倍率. 0~1之间，表示缩小；大于1，表示放大。

备注：该函数仅对后置摄像头有效。


**focusCameraAtAdjustedPoint**

```
public void focusCameraAtAdjustedPoint(float xRatio, float yRatio);

```

功能：聚焦到某个设置的点。调用该函数可以聚焦到预览窗口上人为指定的某个点。

参数：

xRatio,yRatio：需要聚焦到的点的位置。（0.0，0.0）代表左上角，（1.0，1.0）代表右下角，（0.5，0.5）代表中心点。

备注：无。

**setFilterParam**

```
public void setFilterParam(Map<String, String> param);

```
功能：设置滤镜的相关参数。

参数：

param：滤镜相关的配置参数。使用Map的方式，以便于后续的扩展。目前只有一个美颜的滤镜，可以设置的参数如下：

* MediaConstants.FILTER_ PARAM_ BEAUTY_ON：	美颜是否开启

备注：该函数可以在连麦过程中随时进行调用。

**getPublisherPerformanceInfo**

```
public AlivcPublisherPerformanceInfo getPublisherPerformanceInfo();

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
public AlivcPlayerPerformanceInfo getPlayerPerformanceInfo();

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

**setPublisherMuteMode**

```
public void setPublisherMuteModeOn(boolean on);

```

功能：设置推流端的静音模式，仅在连麦过程中使用。

参数：

on：  true 表示打开静音模式；false 表示关闭静音模式。

备注：无

**getSDKVersion**

```
public String getSDKVersion();

```

功能：获取版本信息。

参数：无

返回值：版本号。
