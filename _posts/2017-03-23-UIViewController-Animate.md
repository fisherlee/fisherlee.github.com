---
layout: post
title:  "Runtime简单应用-UIViewController切换动画的小框架"
date:   2017-03-23 15:50:10 +0800
categories: objc
---

#### 简单的UIViewController切换动画的小框架

使用方法很简单，工程中引入，“UIViewController+ABC.h” 和“ABCAnimator.h”，
在“ABCAnimator.m”中的“- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext”加入自己写的动画即可。
代码一看即懂。
GitHub地址：https://github.com/fisherlee/ViewControllerEffects

#### ABCAnimator

实现了UIViewController的动画

“- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext”写了一个简单的动画效果，可以随便修改和增加。

#### UIViewController+ABC

要是实现UINavigationControllerDelegate
主要思路就是在app启动时候
- 替换“viewWillAppear:”，实现UINavigationControllerDelegate
- 替换“navigationController:animationControllerForOperation:fromViewController:toViewController:” 实现push和pop的动画调用。

{% highlight objc %}

+ (void)load {

    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{

        SEL origin_villAppear_sel = @selector(viewWillAppear:);
        SEL new_villAppear_sel = @selector(abc_viewWillAppear:);
        [UIViewController abc_exchangeImpSEL1:origin_villAppear_sel sel2:new_villAppear_sel];

        SEL origin_disappear_sel = @selector(viewDidDisappear:);
        SEL new_disappear_sel = @selector(abc_viewDidDisappear:);
        [UIViewController abc_exchangeImpSEL1:origin_disappear_sel sel2:new_disappear_sel];

        SEL origin_nav_sel = @selector(navigationController:animationControllerForOperation:fromViewController:toViewController:);
        SEL new_nav_sel = @selector(abc_navigationController:animationControllerForOperation:fromViewController:toViewController:);
        [UIViewController abc_exchangeImpSEL1:origin_nav_sel sel2:new_nav_sel];
    });
}

- (void)abc_viewWillAppear:(BOOL)animated {
    if (self.navigationController) {
        self.navigationController.delegate = self;
    }

    NSString *cls = NSStringFromClass([self class]);
    NSLog(@"cls: %@", cls);

    [self abc_viewWillAppear:YES];
}

- (void)abc_viewDidDisappear:(BOOL)animated {

    self.animator_type = @(0);

    [self abc_viewDidDisappear:YES];
}

- (id <UIViewControllerAnimatedTransitioning>)abc_navigationController:(UINavigationController *)navigationController
                                            animationControllerForOperation:(UINavigationControllerOperation)operation
                                                         fromViewController:(UIViewController *)fromVC
                                                           toViewController:(UIViewController *)toVC {
    NSInteger type = [self.animator_type integerValue];
    double interval = 0.5;
    if ([self.time_interval doubleValue] > 0) {
        interval = [self.time_interval doubleValue];
    }

    if (type != 0) {
        if (operation == UINavigationControllerOperationPush) {
            ABCAnimator *animator = [[ABCAnimator alloc] initWithTimeInterval:interval];
            animator.transition = type;
            return animator;
        }
        if (operation == UINavigationControllerOperationPop) {
            ABCAnimator *animator = [[ABCAnimator alloc] initWithTimeInterval:interval];
            animator.transition = type;
            return animator;
        }
    }else {
        toVC.view.alpha = 1;
    }

    return nil;
}

+ (void)abc_exchangeImpSEL1:(SEL)sel1 sel2:(SEL)sel2 {
    Class cls = [self class];

    Method origin_method = class_getInstanceMethod(cls, sel1);
    Method new_method = class_getInstanceMethod(cls, sel2);

    BOOL add = class_addMethod(cls, sel1, method_getImplementation(new_method), method_getTypeEncoding(new_method));
    if (add) {
        class_replaceMethod(cls, sel2, method_getImplementation(origin_method), method_getTypeEncoding(origin_method));
    }else {
        method_exchangeImplementations(origin_method, new_method);
    }
}

- (void)setTime_interval:(NSNumber *)time_interval {
    objc_setAssociatedObject(self, &kTimeIntervalAssociatedKey, time_interval, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (NSNumber *)time_interval {
    return objc_getAssociatedObject(self, &kTimeIntervalAssociatedKey);
}

- (void)setAnimator_type:(NSNumber *)animator_type {
    objc_setAssociatedObject(self, &kAnimatorAssociatedKey, animator_type, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (NSNumber *)animator_type {
    return objc_getAssociatedObject(self, &kAnimatorAssociatedKey);
}

{% endhighlight %}
