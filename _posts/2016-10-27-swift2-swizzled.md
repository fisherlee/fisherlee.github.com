---
layout: post
title:  "Swift2 Swizzled"
date:   2016-10-27 09:58:11 +0800
categories: Swift
---

new file `AppLoad.swif`

{% highlight swift %}
import UIKit

extension UIViewController {
    public override class func initialize() {

        // make sure this isn't a subclass
        if self !== UIViewController.self {
            return
        }

        struct DispatchToken {
            static var token: dispatch_once_t = 0
        }

        dispatch_once(&DispatchToken.token) {
            let originalSelector = #selector(UIViewController.viewDidLoad)
            let swizzledSelector = #selector(self.lw_viewDidLoad)

            let originalMethod = class_getInstanceMethod(self, originalSelector)
            let swizzledMethod = class_getInstanceMethod(self, swizzledSelector)

            let addMethod = class_addMethod(self, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod))

            if addMethod {
                class_replaceMethod(self, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod))
            }else {
                method_exchangeImplementations(originalMethod, swizzledMethod)
            }

        }
    }

    func lw_viewDidLoad() {
        print("viewDidLoad: \(NSStringFromClass(self.classForCoder))")
        let albumClassName = NSStringFromClass(self.classForCoder)
        if albumClassName.containsString("SwiftAlbum") {
            self.view.backgroundColor = UIColor.init(colorLiteralRed: 244/255, green: 244/255, blue: 244/255, alpha: 1)
        }
    }
}


class AppLoad: NSObject {


}

{% endhighlight %}