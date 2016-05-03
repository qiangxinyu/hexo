title:  右滑返回 失效
---



ios7开始 苹果增加了页面 右滑返回的效果；具体的是以 `UINavigationController` 为容器的 `ViewController`间右滑切换页面。代码里的设置是：

`self.navigationController.interactivePopGestureRecognizer.enabled = YES;（default is YES）` 可以看到苹果给 `navigationController` 添加了一个手势（具体为 `UIScreenEdgePanGestureRecognizer`（边缘手势，同样是ios7以后才有的）），就是利用这个手势实现的 ios7的侧滑返回。

问题1：

然而事情并非我们想的那么简单。

1. 当我们用系统的 `UINavigationController` ，并且也是利用系统的 `navigateBar` 的时候，是完全没有问题的
2. 但是当我们没有用系统的 `navigateBar` 或者自定义了返回按钮的时候，这个时候 右滑返回是失效的。

解决（问题1）办法：

对于这种失效的情况，考虑到 `interactivePopGestureRecognizer` 也有 `delegate` 属性，替换默认的
`self.navigationController.interactivePopGestureRecognizer.delegate`来配置右滑返回的表现也是可行的。
我们可以在主`NavigationController`中设置一下：

`self.navigationController.interactivePopGestureRecognizer.delegate =(id)self`

问题2：

但是出现很多问题，比如说在 `rootViewController` 的时候这个手势也可以响应，导致整个程序页面不响应；`push` 了多层后，快速的触发两次手势，也会错乱

解决（问题2）办法：

```objectivec
@interface NavRootViewController : UINavigationController
@property(nonatomic,weak) UIViewController* currentShowVC;
@end

@implementation NavRootViewController
-(id)initWithRootViewController:(UIViewController *)rootViewController
{
	NavRootViewController* nvc = [super initWithRootViewController:rootViewController];
	self.interactivePopGestureRecognizer.delegate = self;
	nvc.delegate = self;
	return nvc;
}

-(void)navigationController:(UINavigationController *)navigationController 
willShowViewController:(UIViewController *)viewController animated:(BOOL)animated
{}

-(void)navigationController:(UINavigationController *)navigationController 
didShowViewController:(UIViewController *)viewController animated:(BOOL)animated
{
if (navigationController.viewControllers.count == 1) {
	self.currentShowVC = Nil;
} else {
	self.currentShowVC = viewController;
}

-(BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer
{
if (gestureRecognizer == self.interactivePopGestureRecognizer) {
	return (self.currentShowVC == self.topViewController); //the most important
}
	return YES;
}

@end
```

借鉴了别人的方法：具体是通过 获取当前 `pushView` 栈里的当前显示的VC,根据这个VC来决定 是否开启手势（如果currentShowVC 是当前显示的，则开启手势；如果 currentShowVC为nil,则代表在主页面，关闭手势）

注：

当时试了一种方法 就是滑动的时候来回设置 `interactivePopGestureRecognizer的delegate；` 发现 会有crash，原因就是 因为 我把 当前显示的VC设置为了这个手势的 `delegate` ，但当这个VC消失的时候，这个 `delegate` 便被释放了，导致crash

至此，觉得ios7上的右滑返回大功告成了，心里正happy，妈蛋，发现了一个可耻的bug：
UIScrollView 上 右滑返回的手势失灵了，靠！！！！！！

问题三：
UIScrollView上手势失灵：
经研究，发现是UIScrollView上已经添加了 `panGestureRecognizer`（滑动）手势


<a href = "/images/rightSlipBackFailure/rightSlipBackFailure.jpg"> <img src = "/images/rightSlipBackFailure/rightSlipBackFailure.jpg" width = 690 height = 77 alt = "rightSlipBackFailure"/></a>

解决（问题三）办法：

参考：<http://www.cnblogs.com/lexingyu/p/3702742.html>

【解决方案】

苹果以UIGestureRecognizerDelegate的形式，支持多个UIGestureRecognizer共存。其中的一个方法是：

```objectivec
// called when the recognition of one of gestureRecognizer or otherGestureRecognizer would be 
blocked by the other
// return YES to allow both to recognize simultaneously. the default implementation returns NO 
(by default no two gestures can be recognized simultaneously)
//
// note: returning YES is guaranteed to allow simultaneous recognition. returning NO is not 
guaranteed to prevent simultaneous recognition, as the other gesture's delegate may return YES

- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer 
shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer;
```

一句话总结就是此方法返回YES时，手势事件会一直往下传递，不论当前层次是否对该事件进行响应。

```objectivec
@implementation UIScrollView (AllowPanGestureEventPass)

- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer 
  shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer
{
    if ([gestureRecognizer isKindOfClass:[UIPanGestureRecognizer class]]    
        && [otherGestureRecognizer isKindOfClass:[UIScreenEdgePanGestureRecognizer class]])
    {
        return YES;
    }
    else
    {
        return  NO;
    }
}
```

事实上，对UIGestureRecognizer来说，它们对事件的接收顺序和对事件的响应是可以分开设置的，即存在接收链和响应链。接收链如上文所述，和UIView绑定，由UIView的层次决定接收顺序。

而响应链在apple君的定义下，逻辑出奇的简单，只有一个方法可以设置多个gestureRecognizer的响应关系：

```objectivec
// create a relationship with another gesture recognizer that will prevent this gesture's 
actions from being called until otherGestureRecognizer transitions to 
UIGestureRecognizerStateFailed // if otherGestureRecognizer transitions to 
UIGestureRecognizerStateRecognized or UIGestureRecognizerStateBegan then this recognizer will 
instead transition to UIGestureRecognizerStateFailed // example usage: a single ap may 
require a double tap to fail
- (void)requireGestureRecognizerToFail:(UIGestureRecognizer *)otherGestureRecognizer;
```

每个UIGesturerecognizer都是一个有限状态机，上述方法会在两个gestureRecognizer间建立一个依托于state的依赖关系，当被依赖的gestureRecognizer.state = failed时，另一个gestureRecognizer才能对手势进行响应。

所以，只需要

`[_scrollView.panGestureRecognizer requireGestureRecognizerToFail:screenEdgePanGestureRecognizer];`

```objectivec
- (UIScreenEdgePanGestureRecognizer *)screenEdgePanGestureRecognizer
{
    UIScreenEdgePanGestureRecognizer *screenEdgePanGestureRecognizer = nil;
    if (self.view.gestureRecognizers.count > 0)
    {
        for (UIGestureRecognizer *recognizer in self.view.gestureRecognizers)
        {
            if ([recognizer isKindOfClass:[UIScreenEdgePanGestureRecognizer class]])
            {
                screenEdgePanGestureRecognizer = (UIScreenEdgePanGestureRecognizer *)recognizer;
                break;
            }
        }
    }

    return screenEdgePanGestureRecognizer;

}
```
