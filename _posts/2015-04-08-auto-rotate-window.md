---
layout: post
category : game
title: 'Android游戏窗口自动翻转'
tags : [cocos2dx, Android, Java, game]
---
{% include JB/setup %}

### 实现功能：

* 竖屏状态不提供屏幕翻转。
* 横屏状态，如果玩家手机的翻转开关为开启，则根据玩家需要提供屏幕翻转。如果翻转开关为关闭，则不提供翻转。

### 实现方法：

1.首先自定义一个类继承ContentObserver。在onChange()方法里面去获取手机方向Settings的值，每次改变方向锁定的状态都会重设手机屏幕旋转方式。

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
	

### 查看完整项目

- [ARPG Game Demo](https://github.com/edwinho/ARPGDemo)
