---
layout: post
title:  "Runtime AssociatedObject"
date:   2017-02-22 15:50:11 +0800
categories: runtime
---

#### AssociatedObject 类关联对象
像对象中添加、获取、删除关联值
> 方法：
  接收：objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
  获取：objc_getAssociatedObject(id object, const void *key)
  移除：objc_removeAssociatedObjects(id object)

*简单的应用*

demo地址：[RuntimeAssociated](https://github.com/fisherlee/Runtime)

``` objc

- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    return 50;
}

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {

    UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath];
    NSString *text = objc_getAssociatedObject(cell, @"kCellTextLabelKey");
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"Alert"
                                                                             message:text
                                                                      preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *okAlert = [UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        objc_removeAssociatedObjects(cell);
    }];
    [alertController addAction:okAlert];


    [self presentViewController:alertController animated:YES completion:^{

    }];
}

```
