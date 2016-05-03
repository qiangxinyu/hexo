title: iOS开发中的一些小Tip
---


### <font color = #a2c700>iOS开发中如何检测当前运行商类型(移动、联通、电信) </font>

OS开发中，有时需要检测设备运营商类型，如移动、联通或者电信，本文以检测联通为例。

```objectivec
- (BOOL)checkIsUnicom
{
    CTTelephonyNetworkInfo *info = [[CTTelephonyNetworkInfo alloc] init];
    CTCarrier *carrier = info.subscriberCellularProvider;
    NSString *carrierName = carrier.carrierName;
    NSString *mobileCountryCode = carrier.mobileCountryCode;
    NSString *mobileNetworkCode = carrier.mobileNetworkCode;
    [info release];
    if (!mobileNetworkCode) {
        return NO;
    }
    if ([mobileCountryCode intValue]==460) { //国内
        return [carrierName rangeOfString:@"联通"].length>0 ||
         [mobileNetworkCode isEqualToString:@"01"]   ||
         [mobileNetworkCode isEqualToString:@"06"];
    }
    return [self statusBarCheckIsUnicom];
}
```

运行商对应的NetworkCode

<a href = "/images/iOSCodeSomeTip/iOSSomeTip_1.png"> <img src = "/images/iOSCodeSomeTip/iOSSomeTip_1.png" width = 567 height = 269 alt = "iOSSomeTip_1"/></a>

正常情况下，以上代码可满足正常需求，但是对于美版或者日版卡贴iPhone，检测到的CTCarrier并非sim卡信息，此时就需要通过StatusBar实时检测当前网络运行商

```objectivec
- (BOOL)statusBarCheckIsUnicom
{
    NSArray *subviews = [[[[UIApplication sharedApplication] valueForKey:@"statusBar"]
     valueForKey:@"foregroundView"] subviews];
    UIView *serviceView = nil;
    Class serviceClass = NSClassFromString([NSString stringWithFormat:@"UIStat%@Serv%@%@", 
    @"usBar", @"ice", @"ItemView"]);
    for (UIView *subview in subviews) {
        if([subview isKindOfClass:[serviceClass class]]) {
            serviceView = subview;
            break;
        }
    }
    if (serviceView) {
        NSString *carrierName = [serviceView valueForKey:[@"service" 
        stringByAppendingString:@"String"]];
        return [carrierName rangeOfString:@"联通"].length>0;
    } else {
        return NO;
    }
}
```


### <font color = #a2c700> iOS9 进行第三方跳转 </font>

需要在plist 加入：

	<key>LSApplicationQueriesSchemes</key>
		<array>
		<string>wechat</string>
		<string>local</string>
		<string>weixin</string>
		<string>sinaweibohd</string>
		<string>sinaweibo</string>
		<string>sinaweibosso</string>
		<string>weibosdk</string>
		<string>alisdkdemo</string>
		<string>weibosdk2.5</string>
		<string>mqqapi</string>
		<string>mqqbrowser</string>
		<string>mqq</string>
		<string>mqqOpensdkSSoLogin</string>
		<string>mqqconnect</string>
		<string>mqqopensdkdataline</string>
		<string>mqqopensdkgrouptribeshare</string>
		<string>mqqopensdkfriend</string>
		<string>mqqopensdkapi</string>
		<string>mqqopensdkapiV2</string>
		<string>mqqopensdkapiV3</string>
		<string>mqzoneopensdk</string>
		<string>wtloginmqq</string>
		<string>wtloginmqq2</string>
		<string>mqqwpa</string>
		<string>safepay</string>
		<string>mqzone</string>
		<string>mqqapiwallet</string>
		<string>mqzonev2</string>
		<string>mqzoneshare</string>
		<string>wtloginqzone</string>
		<string>mqzonewx</string>
		<string>mqzoneopensdkapiV2</string>
		<string>mqzoneopensdkapi19</string>
		<string>mqzoneopensdkapi</string>
		<string>mqzoneopensdk</string>
		<string>renrenios</string>
		<string>renrenapi</string>
		<string>renren</string>
		<string>renreniphone</string>
		<string>yixin</string>
		<string>instagram</string>
		<string>whatsapp</string>
		<string>line</string>
		<string>fbapi</string>
		<string>fb-messenger-api</string>
		<string>fbauth2</string>
		<string>fbshareextension</string>
		<string>alipay</string>
		<string>cydia</string>
		<string>safepay</string>
		</array>



### <font color = #a2c700> 判断设备的型号 </font>

```objectivec
#import <sys/utsname.h>

//获得设备型号
+ (NSString *)getCurrentDeviceModel
{
    struct utsname systemInfo;
    uname(&systemInfo);
    NSString *platform = [NSString stringWithCString:systemInfo.machine
                                              encoding:NSUTF8StringEncoding];


    if ([platform isEqualToString:@"iPhone1,1"]) return @"iPhone 2G (A1203)";
    if ([platform isEqualToString:@"iPhone1,2"]) return @"iPhone 3G (A1241/A1324)";
    if ([platform isEqualToString:@"iPhone2,1"]) return @"iPhone 3GS (A1303/A1325)";
    if ([platform isEqualToString:@"iPhone3,1"]) return @"iPhone 4 (A1332)";
    if ([platform isEqualToString:@"iPhone3,2"]) return @"iPhone 4 (A1332)";
    if ([platform isEqualToString:@"iPhone3,3"]) return @"iPhone 4 (A1349)";
    if ([platform isEqualToString:@"iPhone4,1"]) return @"iPhone 4S (A1387/A1431)";
    if ([platform isEqualToString:@"iPhone5,1"]) return @"iPhone 5 (A1428)";
    if ([platform isEqualToString:@"iPhone5,2"]) return @"iPhone 5 (A1429/A1442)";
    if ([platform isEqualToString:@"iPhone5,3"]) return @"iPhone 5c (A1456/A1532)";
    if ([platform isEqualToString:@"iPhone5,4"]) return @"iPhone 5c (A1507/A1516/A1526/A1529)";
    if ([platform isEqualToString:@"iPhone6,1"]) return @"iPhone 5s (A1453/A1533)";
    if ([platform isEqualToString:@"iPhone6,2"]) return @"iPhone 5s (A1457/A1518/A1528/A1530)";
    if ([platform isEqualToString:@"iPhone7,1"]) return @"iPhone 6 Plus (A1522/A1524)";
    if ([platform isEqualToString:@"iPhone7,2"]) return @"iPhone 6 (A1549/A1586)";
    if ([platform isEqualToString:@"iPhone8,1"]) return @"iPhone 6s (A1633/A1688/A1691/A1700)";
    if ([platform isEqualToString:@"iPhone8,2"]) return @"iPhone 6s Plus (A1634/A1687/A1690/A1699)";

    if ([platform isEqualToString:@"iPod1,1"])   return @"iPod Touch 1G (A1213)";
    if ([platform isEqualToString:@"iPod2,1"])   return @"iPod Touch 2G (A1288)";
    if ([platform isEqualToString:@"iPod3,1"])   return @"iPod Touch 3G (A1318)";
    if ([platform isEqualToString:@"iPod4,1"])   return @"iPod Touch 4G (A1367)";
    if ([platform isEqualToString:@"iPod5,1"])   return @"iPod Touch 5G (A1421/A1509)";

    if ([platform isEqualToString:@"iPad1,1"])   return @"iPad 1G (A1219/A1337)";

    if ([platform isEqualToString:@"iPad2,1"])   return @"iPad 2 (A1395)";
    if ([platform isEqualToString:@"iPad2,2"])   return @"iPad 2 (A1396)";
    if ([platform isEqualToString:@"iPad2,3"])   return @"iPad 2 (A1397)";
    if ([platform isEqualToString:@"iPad2,4"])   return @"iPad 2 (A1395+New Chip)";
    if ([platform isEqualToString:@"iPad2,5"])   return @"iPad Mini 1G (A1432)";
    if ([platform isEqualToString:@"iPad2,6"])   return @"iPad Mini 1G (A1454)";
    if ([platform isEqualToString:@"iPad2,7"])   return @"iPad Mini 1G (A1455)";

    if ([platform isEqualToString:@"iPad3,1"])   return @"iPad 3 (A1416)";
    if ([platform isEqualToString:@"iPad3,2"])   return @"iPad 3 (A1403)";
    if ([platform isEqualToString:@"iPad3,3"])   return @"iPad 3 (A1430)";
    if ([platform isEqualToString:@"iPad3,4"])   return @"iPad 4 (A1458)";
    if ([platform isEqualToString:@"iPad3,5"])   return @"iPad 4 (A1459)";
    if ([platform isEqualToString:@"iPad3,6"])   return @"iPad 4 (A1460)";

    if ([platform isEqualToString:@"iPad4,1"])   return @"iPad Air (A1474)";
    if ([platform isEqualToString:@"iPad4,2"])   return @"iPad Air (A1475)";
    if ([platform isEqualToString:@"iPad4,3"])   return @"iPad Air (A1476)";
    if ([platform isEqualToString:@"iPad4,4"])   return @"iPad Mini 2G (A1489)";
    if ([platform isEqualToString:@"iPad4,5"])   return @"iPad Mini 2G (A1490)";
    if ([platform isEqualToString:@"iPad4,6"])   return @"iPad Mini 2G (A1491)";

    if ([platform isEqualToString:@"i386"])      return @"iPhone Simulator";
    if ([platform isEqualToString:@"x86_64"])    return @"iPhone Simulator";
    return platform;
}
```


### <font color = #a2c700> 隐藏返回按钮的文字 </font>

```objectivec
[[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:UIOffsetMake(0, -60)
                                                         forBarMetrics:UIBarMetricsDefault];   ```
                                                         
  
  
### <font color = #a2c700> 判断 pickerView 是否正在滑动 </font>

```objectivec
- (BOOL)isScrolling:(UIView *)view
{
    if ([view isKindOfClass:UIScrollView.class]) {
        UIScrollView * scrollView = (UIScrollView *)view;
        if (scrollView.dragging || scrollView.decelerating) {
            return YES;
        }
    }

    for (UIView * aView in view.subviews) {
        if ([self isScrolling:aView]) {
            return YES;
        }
    }
    return NO;
}
```

### <font color = #a2c700> ios8系统 点击设置隐私定位功能直接崩溃的问题 </font>

<a href = "/images/iOSCodeSomeTip/iOSSomeTip_2.png"> <img src = "/images/iOSCodeSomeTip/iOSSomeTip_2.png" width = 626 height = 285 alt = "iOSSomeTip_2"/> </a>                                                         
                                                                                                                                                                           

### <font color = #a2c700> 改变webView上的图片大小 </font>

```objectivec
NSString * js = [NSString stringWithFormat: @"var script = document.createElement('script');"
                     "script.type = 'text/javascript';"
                     "script.text = \"function ResizeImages() { "
                     "var myimg,oldwidth;"
                     "var maxwidth=%f;" //缩放系数
                     "for(i=0;i <document.images.length;i++){"
                     "myimg = document.images[i];"
                     "if(myimg.width > maxwidth){"
                     "oldwidth = myimg.width;"
                     "myimg.width = maxwidth;"
                     "myimg.style.width = maxwidth+'px';"
                     "myimg.height = myimg.height * (maxwidth/oldwidth);"
                      "myimg.style.height = 'auto';"
                     "}"
                     "}"
                     "}\";"
                     "document.getElementsByTagName('head')[0].appendChild(script);",kScreenWidth-10];
    [webView stringByEvaluatingJavaScriptFromString:js];
    [webView stringByEvaluatingJavaScriptFromString:@"ResizeImages();"];
```      



### <font color = #a2c700> C文件申明冲突 </font>

头件的中的#ifndef，这是一个很关键的东西。比如你有两个C文件，这两个C文件都include了同一个头文件。而编译时，这两个C文件要一同编译成一个可运行文件，于是问题来了，大量的声明冲突。

还是把头文件的内容都放在#ifndef和#endif中吧。不管你的头文件会不会被多个文件引用，你都要加上这个。一般格式是这样的：

```c
#ifndef <标识>
#define <标识>
......
......
#endif
```


<标识>在理论上来说可以是自由命名的，但每个头文件的这个“标识”都应该是唯一的。标识的命名规则一般是头文件名全大写，前后加下划线，并把文件名中的“.”也变成下划线，如：stdio.h

```c
#ifndef _STDIO_H_
#define _STDIO_H_
......
#endif  
```

### <font color = #a2c700> Navigation 在 pop 和push 的时候奔溃 </font>

一般是因为 delegate的问题

```objectivec
- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    if ([self isMovingFromParentViewController])
    {
        if (self.navigationController.delegate == self)
        {
            self.navigationController.delegate = nil;
        }
    }
}
```

### <font color = #a2c700>  pop回来后取消选中的cell </font>

在 `viewWillAppear` 方法中加入：

```objectivec
[self.tableView deselectRowAtIndexPath:[self.tableView indexPathForSelectedRow] animated:YES];                                                                                                       
```


### <font color = #a2c700> CollectionView的cell太少无法拖动出来下拉刷新 </font>

```objectivec
self.collectionView.alwaysBounceVertical = YES;
```

### <font color = #a2c700> 导入C文件发生冲突  </font>

在.pch文件加入

```c
#ifdef __OBJC__
... oc的import
#endif
```