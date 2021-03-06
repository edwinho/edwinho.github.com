---
layout: post
title: "使用cocos2dx游戏引擎做一个ARPG小游戏"
category : game
tags : [cocos2dx, C++, game]
---

用cocos2dx2.1.4作为游戏引擎做一个ARPG小游戏，包含基本的寻路、战斗、射击、碰撞检测等。

### 1、使用A星算法寻路，避开障碍物选择最优路线移动
　　指定一个目标位置，玩家角色绕开障碍物走到该位置，首先要判断目标位置是否在障碍物内或在地图外。根据`F=G+H`来判断每次八个方向哪个最优，并检测这个点是否在关闭（已走过位置）的集合内，每个点都用struct结构来保存，包含这个点的坐标、G和F。每移动一次将这个位置放入关闭的集合直到终点。

<!-- more -->

### 2、地图、角色的移动
&#160;&#160;&#160;&#160;地图放大原来的大小，人物在移动时，始终保持在镜头正中间，当移动到地图边缘时，玩家角色移动，可以不在镜头中央，移向地图中间时玩家角色又要保持在镜头正中间。  
&emsp;&emsp;这个问题我纠结了好久，刚开始我的实现方法是保持玩家角色在镜头中间，只做奔跑的Action，而地图做移动Action，向人物奔跑相反的方向移动，到边缘时地图就是移动了，改为玩家角色做相对的移动，刚开始时是没有问题的，到地图边缘时就会出bug，这个处理方法其实是挺麻烦的，要保存玩家角色和地图的位移量，每次移动也要更新，不好维护和处理，这可能是出bug的问题所在。  
&ensp;&ensp;&ensp;&ensp;自己一个人搞不掂，就请教了同事和导师，他们的解决方案与我的很不相同，玩家角色是在地图的子层，人物移动一帧，地图向相反方向移动一帧，通过这样来保持玩家角色在镜头正中间。而且A星的寻路也大相径庭，我之前是先计算好之后要走的路径，然后通过Action走过去，这样不好，因为后面加上AI怪后，要有追踪功能，target位置会移动，就会变得好难处理，改为在update函数和自己写个计时器里实现，每刷一帧，玩家角色和地图都移一下。

### 3、点击事件
&emsp;&emsp;使用`ccTouchBegin`实现点击移动事件，ccTouchBegin点击捕获的坐标是当前窗口，即this的坐标，使用`map->convertTouchToNodeSpace(pTouch)`将坐标转换为地图的坐标位置。

### 4、增加AI怪物
&emsp;&emsp;增加4个sprite作为怪物，当初是想使用类似cocos2dx3.0中的`Vector<Sprite*> enemy`来动态保存怪物，使用这种方法的好处是以后可以方便增加怪物，但发现2.1.4中的CCArrary没有这个功能，作为代替方法是用`CCSprite* enemy[4]`写死数量。  
&emsp;&emsp;用计时器schedule实现怪物的自由活动，使用CCRect给每个怪物设定一个攻击范围，玩家角色进入攻击范围内会被攻击`enemyRect.containsPoint(player->getPosition())`

### 5、旋转炮塔射击
&emsp;&emsp;在障碍物上增设炮塔Sprite，炮塔射击的方向`(CCRANDOM_0_1() * range)`和速度`(CCRANDOM_0_1() * rangeDuration)`都是随机的，最后在子弹的移动Action结束时加个回调函数，`CCCallFuncN::create(this,callfuncN_selector(HelloWorld:: spriteMoveFinished))`， 当子弹移出地图时，清除这个CCSprite避免消耗太多内在导致溢出。当玩家角色的Rect和子弹的Rect发生重叠后`playRect.intersectsRect(bulletRect)`，就产生碰撞，子弹停止当前所有动作`opAllAction()`，播放攻击特效Action，并设置回调函数，清除这个子弹sprite。

### 6、添加玩家角色HP、技能CD、怪物技能CD、玩家得分
&emsp;&emsp;作为一个角色，应该要有生命，被怪攻击后HP减少一个点，直到HP为0结束游戏并统计玩家得分，分数是由玩家击杀敌人获得，杀一个敌人得一个分数点。技能CD在schedule里更新，每一秒刷新一次，敌人的CD比玩家角色技能CD时间在短，但玩家角色手长。

### 7、加入菜单层
&emsp;&emsp;在进入游戏前做一个菜单，让玩家选择相应功能，这个功能是次要的，所以界面UI做得很简单。

### 8、AI怪有活动范围、视野
&emsp;&emsp;给每个怪物设定一个视野范围`isualRangeRect`，玩家进入其范围后，怪物会追向玩家，当达到攻击范围后`enemyAttackRect`就会停止之前的行动转向玩家攻击，玩家也可以及时避开，因为每个怪都有自己的活动范围。  
&emsp;&emsp;另外，玩家角色也可以追击敌人，玩家点击中敌人后，玩家角色会主动走向敌人，进入敌人视野范围后，敌人也会向玩家走来，这时是相向移动，但玩家角色的攻击范围更大，在技能为激活状态时会先攻击敌人。

### 9、后续工作
&emsp;&emsp;玩家可以在击杀一定数量的怪物后可以提升等级，从而提高玩家生命值，减少技能CD，激活新技能，经过一定时间后玩家HP自动回复；给AI怪物设定生命值，需要多次攻击才能完成击杀，生命越多的敌人被击杀后获得的分数就越多。等等。

### 查看完整项目

- [ARPG Game Demo](https://github.com/edwinho/ARPGDemo)
