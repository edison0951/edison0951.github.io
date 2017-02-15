---
title: 通过代码获取LaunchImage
date: 2017-01-11 09:50:00
tags: LaunchImage 启动图
---

相信大多数开发者都会遇到这样一个需求：在AppDelegate的

```objective-c
(BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
```

方法执行完毕之后，想让启动图再停留一会，以便程序中再做一些初始化工作，这样的体验会更好一点。

程序中访问bundle中的图片资源一般有两种方式：

1. 通过 [UIImage imageNamed:name] 方法获取
2. 通过 [[NSBundle mainBundle] pathForResource:@"name" ofType:@"type"] 方法获取

但是通过以上两个方法获取LaunchImage，返回的结果都是nil。经过一番查找之后，发现LaunchImage.launchimage中图片的名字发生了变化。具体代码如下：

```objective-c
CGSize viewSize = self.window.bounds.size;

NSString *viewOrientation = @"Portrait";    //横屏请设置成 @"Landscape"

NSString *launchImage = nil;

NSArray *dictArray = [[[NSBundle mainBundle] infoDictionary] valueForKey:@"UILaunchImages"];

for (NSDictionary* dict in dictArray){

	CGSize imageSize = CGSizeFromString(dict[@"UILaunchImageSize"]);

 	if (CGSizeEqualToSize(imageSize, viewSize) && [viewOrientation isEqualToString:dict[@"UILaunchImageOrientation"]]){

    	launchImage = dict[@"UILaunchImageName"];

    }
}
```

​ 其输出的格式大致是这样的：   

```objective-c
UILaunchImageMinimumOSVersion = "7.0";
UILaunchImageName = "LaunchImage-700-568h";
UILaunchImageOrientation = Portrait;
UILaunchImageSize = "{320, 568}";
```

通过变量名字，我们能大概猜出他们分别是干什么的，UILaunchImageMinimumOSVersion表示启动图支持的最低版本，这个地方需要特别说明的是，对于iPhone6以上设备的分辨率，默认的最低版本是8.0，具体的解释可以移步这个[地址](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html#//apple_ref/doc/uid/TP40009252-SW28)查看。

通过以上代码，你就能根据你当前屏幕的分辨率获取对应的启动图了。

多说一句，你还可以手动设置这些key的名字，具体请看这个[链接](http://stackoverflow.com/questions/18976412/launch-screens-supporting-ios6-and-ios7-forced-to-splash-screen)。

