---
layout: post
title: 在iOS App中使用Cocos2d-x v 3.0
categories: Cocos2d-x
---

1. 将cocos2d文件夹拖到UIKit项目文件夹中, 将libs.xcodeproj拖到Xcode项目里面
2. Target Dependencies加入cocos2d的5个lib
3. Link Binary With Libraries加入cocos2d的5个iOS lib
4. 对照cocos2d Helloworld项目添加缺失的库,
5. libs.xcodeproj的Architectures改成和项目一致(arm64这些)
6. PROJECT的Always Search User Paths改为Yes, Header Search Path加入:

```
	$(inherited)
	$(SRCROOT)/cocos2d
	$(SRCROOT)/cocos2d/cocos
	$(SRCROOT)/cocos2d/cocos/base
	$(SRCROOT)/cocos2d/cocos/physics
	$(SRCROOT)/cocos2d/cocos/math
	$(SRCROOT)/cocos2d/cocos/2d
	$(SRCROOT)/cocos2d/cocos/ui
	$(SRCROOT)/cocos2d/cocos/network
	$(SRCROOT)/cocos2d/cocos/audio/include
	$(SRCROOT)/cocos2d/cocos/editor-support
	$(SRCROOT)/cocos2d/extensions
	$(SRCROOT)/cocos2d/external
	$(SRCROOT)/cocos2d/external/chipmunk/include/chipmunk
```

7. TARGETS的Always Search User Paths改为Yes, Header Search Path加入以下两句:

```
	$(SRCROOT)/cocos2d/cocos/platform/ios
	$(SRCROOT)/cocos2d/cocos/platform/ios/Simulation
```

8. 引用"cocos2d.h"的objc类后缀名都要改成.mm

---
> 注:

1. 从Cocos2d Scene的页面返回时会崩溃的bug, 代码修改CCDirectorCaller:(v3.0, v3.2以后版本已修复)

```Objective-C
	+ (void)destroy
	{
	    [s_sharedDirectorCaller destroy]; // modified
	    [s_sharedDirectorCaller release];
	    s_sharedDirectorCaller = nil;
	}

	- (void)destroy
	{// modified
	    [displayLink invalidate];
	    displayLink = nil;
	    CCLOG("caller destroy");
	}
```

2. CCEAGLView.mm在dealloc中removeObserver, 否则退出Cocos界面后所有弹出键盘都会崩溃:(v3.0, v3.2以后版本已修复)

```Objective-C
	[[NSNotificationCenter defaultCenter] removeObserver:self];
```
