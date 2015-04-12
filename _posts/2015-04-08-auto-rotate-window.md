---
layout: post
category : game
title: '游戏窗口自动翻转'
tags : [cocos2dx, Android, Java, game]
---
{% include JB/setup %}

### 实现功能：

* 竖屏状态不提供屏幕翻转。
* 横屏状态，如果玩家手机的翻转开关为开启，则根据玩家需要提供屏幕翻转。如果翻转开关为关闭，则不提供翻转。

### 实现方法：

##Android

1.首先自定义一个类继承ContentObserver。在onChange()方法里面去获取手机方向Settings的值，每次改变方向锁定的状态都会重设手机屏幕旋转方式

<!--more-->

	public void onChange(boolean selfChange) {
		super.onChange(selfChange);
		int flag = Settings.System.getInt(getContentResolver(),Settings.System.ACCELEROMETER_ROTATION, 0);
		if (flag == 1)
		{
			setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_SENSOR_LANDSCAPE);
		}
		else if (flag == 0)
		{
			setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
		}
	}

	
2.注册监听事件

	private SettingsValueChangeContentObserver mContentOb;
	protected void onStart() {
		super.onStart();
		mContentOb=new SettingsValueChangeContentObserver();
		getContentResolver().registerContentObserver(Settings.System.getUriFor(Settings.System.ACCELEROMETER_ROTATION), true, mContentOb);
	}


3.当应用退出的时候取消监听

	protected void onStop() {
		super.onStop();
		getContentResolver().unregisterContentObserver(mContentOb);
	}

##IOS

在RootViewController.mm文件中写入配置屏幕旋转方式：

	// Override to allow orientations other than the default portrait orientation.
	// This method is deprecated on ios6
	- (BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation {
	    return UIInterfaceOrientationIsLandscape( interfaceOrientation );
	}

	// For ios6, use supportedInterfaceOrientations & shouldAutorotate instead
	- (NSUInteger) supportedInterfaceOrientations{
	#ifdef __IPHONE_6_0
	    //return UIInterfaceOrientationMaskAllButUpsideDown;
	    return UIInterfaceOrientationMaskLandscape;
	#endif
	}

	- (BOOL) shouldAutorotate {
	    return YES;
	}

	- (void)didReceiveMemoryWarning {
	    // Releases the view if it doesn't have a superview.
	    [super didReceiveMemoryWarning];
	    // Release any cached data, images, etc that aren't in use.
	}

	- (void)viewDidUnload {
	    [super viewDidUnload];
	    // Release any retained subviews of the main view.
	    // e.g. self.myOutlet = nil;
	}

	- (void)dealloc {
	    [super dealloc];
	}


### 扩展

- [关于Android设置屏幕旋转的官方文档](http://developer.android.com/reference/android/content/pm/ActivityInfo.html#SCREEN_ORIENTATION_SENSOR)
