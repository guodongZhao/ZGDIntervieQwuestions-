# - iOS经典面试题
=================
# [iOS精英研发大神群](336274987@qq.com) ---QQ群：336274987    
=============
 # iOS经典面试题 -- 汇总 （持续更新中）

## 汇总来自各大神整理的经典面试问题
-----------------------------------

### 临时文件  --来自 果冻的分享的文件

* 1.请解释以下keywords的区别： assign vs weak, __block vs __weak
assign适用于基本数据类型，weak是适用于NSObject对象，并且是一个弱引用。

首先__block是用来修饰一个变量，这个变量就可以在block中被修改（参考block实现原理）

__block：使用__block修饰的变量在block代码快中会被retain（ARC下，MRC下不会retain）

__weak：使用__weak修饰的变量不会在block代码块中被retain ，同时，在ARC下，要避免block出现循环引用 

```
objc __weak typedof(self)weakSelf = self; 
```

* 2.使用nonatomic一定是线程安全的吗？

  答案是 ： 不是的。 

  atomic原子操作，系统会为setter方法加锁。 具体使用 @synchronized(self){//code } 

  nonatomic不会为setter方法加锁。 

  atomic：线程安全，需要消耗大量系统资源来为属性加锁 
  
  nonatomic：非线程安全，适合内存较小的移动设备

** 关于ARC的细节可以看下面的网址：[http://developer.apple.com/library/ios/#releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html]
(http://developer.apple.com/library/ios/#releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)
* 3.描述一个你遇到过的retain cycle例子。
 
 首先我们主要看ARC下retain cycle的问题--

 ARC中，变量可以用四个关键字修饰：
 
__strong： 赋值给这个变量的对象会自动被retain一次，如果在block中引用它，block也会retain它一次。

__unsafe_unretained： 赋值给这个变量不会被retain，也就是说被他修饰的变量的存在不能保证持有对象的可靠性，它可能已经被释放了，而且留下了一个不安全的指针。也不会被block retain。 iOS5 __weak出现之前使用 

__week：类似于__unsafe_unretained，只是如果所持有的对象被释放后，变量会自动被设置为nil，这样更安全些，不过只在IOS5.0以上的系统支持，同样不会被block retain。

__block：修饰一个变量，表示这个变量能在block中被修改（值修改，而不是修改对象中的某一个属性，可以理解为修改指针的指向）。会被自动retain。于其他变量不同的是被 __block 修饰的变量在块中保存的是变量的地址。（其他为变量的值）

  block中的循环引用：一个viewController

 ```objc 
 
 typedef void (^ABlock)(void); //定义一个简单的Block
 @interface ViewController : UIViewController {
    NSMutableArray *_items;
    ABlock _block;
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    _items = [[NSMutableArray alloc] init];
    _block = ^{
        [_items addObject:@"Hello!"]; //_block引用了_items，导致retain cycle。
    };
}
 ```
** Xcode在编译以上程序的时候会给出一个警告：Captureing ‘self’ strongly in this block is likely to lead to a retain cycle。

原因是_items实际上是self->items。_block对象在创建的时候会被retain一次，因此会导致self也被retain一次。这样就形成了一个retain cycle。

`解决方法`：
-----------
```objc
 //方法一 :
 __block ViewController *blockSelf = self;
 _block = ^{
    [blockSelf.items addObject:@"Hello!"];
};
 //方法二 : (德哥友情推荐!!  不用你会后悔的！)
 写法1：__weak ViewController *weakSelf = self; 
 写法2：__weak typeof(self) weakSelf = self;//IOS 5+    
 写法3：__unsafe_unretained typeof(self) unsafeSelf = self;//IOS 4+
 _block = ^{
    [weakSelf.items addObject:@"Hello!"];
};
```
 然而此时问题来了  使用__block 和 __weak 有什么区别呢?

ARC模式下 -- block会retain items变量，但是，由于__block变量保存更为底层的变量地址， 因此当此变量被指向其他对象时，block便不对原来的对象负责，引发的结果就是之前对象被release掉，retain cycle被破坏。

__week 也可以用 __unsafe_unretained 替代，但是 __week 更安全些，虽然它不支持IOS5.0以下的系统。

被 __week 或者 __unsafe_unretained 修饰的变量不会被block retain，所以不会形成retain cycle，但是小心，保证你的对象不会在complete之前被释放，否则会得到你意想不到的结果。

* 4. +(void)load;  +(void)initialize；有什么用处和区别？

在Objective-C中，runtime会自动调用每个类的两个方法。

+load会在类初始加载时调用，+initialize会在第一次调用类的类方法或实例方法之前被调用。

这两个方法是可选的，且只有在实现了它们时才会被调用。 

共同点：两个方法都只会被调用一次。


* 5.如何高性能的给UIImageView加个圆角？（不准说layer.cornerRadius!）
* 
我觉得应该是使用Quartz2D直接绘制图片,得把这个看看。 

步骤： 

 a、创建目标大小(cropWidth，cropHeight)的画布。
 
 b、使用UIImage的drawInRect方法进行绘制的时候，指定rect为(-x，-y，width，height)。
 
 c、从画布中得到裁剪后的图像。
 
```objc
- (UIImage*)cropImageWithRect:(CGRect)cropRect {     
   CGRect drawRect = CGRectMake(-cropRect.origin.x , -cropRect.origin.y, self.size.width * self.scale, self.size.height * self.scale);   
   UIGraphicsBeginImageContext(cropRect.size);   
   CGContextRef context = UIGraphicsGetCurrentContext();     
   CGContextClearRect(context, CGRectMake(0, 0, cropRect.size.width, cropRect.size.height));    
   [self drawInRect:drawRect];     
   UIImage *image = UIGraphicsGetImageFromCurrentImageContext();   
   UIGraphicsEndImageContext();   
   return image; 
} 
``` 

* 6. 我知道你大学毕业过后就没接触过算法数据结构了，但是请你一定告诉我什么是Binary search tree? search的时间复杂度是多少？

Binary search tree:二叉搜索树。 （又：二叉搜索树，二叉排序树）它或者是一棵空树，或者是具有下列性质的二叉树：

若它的左子树不空，则左子树上所有结点的值均小于它的根结点的值；

若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值； 它的左、右子树也分别为二叉排序树

多说无用  上图----23333 

![二叉搜索树](http://c.hiphotos.baidu.com/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=79dcefded70735fa85fd46ebff3864d6/8644ebf81a4c510f0b3dafdf6359252dd52aa57e.jpg "二叉搜索树")


主要由四个方法：（用C语言实现或者Python）

1.search：时间复杂度为O(h)，h为树的高度

2.traversal：时间复杂度为O(n)，n为树的总结点数。

3.insert：时间复杂度为O(h)，h为树的高度。

4.delete：最坏情况下，时间复杂度为O(h)+指针的移动开销。

可以看到，二叉搜索树的dictionary operation的时间复杂度与树的高度h相关。所以需要尽可能的降低树的高度，由此引出平衡二叉树Balanced binary tree。它要求左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。这样就可以将搜索树的高度尽量减小。常用算法有红黑树、AVL、Treap、伸展树等。

@end

