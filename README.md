# -
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
__weak：使用__weak修饰的变量不会在block代码块中被retain 
同时，在ARC下，要避免block出现循环引用 
** ```objc __weak typedof(self)weakSelf = self; ```


