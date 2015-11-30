# article
some article about programm.
***

##2015-11-20

> have fun with coreImage and GPUImage.

###添加GPUImage

 * git clone [GPUImage](https://github.com/BradLarson/GPUImage.git)
 * 将这个项目拷贝到工程的目录下
 	![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151120gpuimage.png)
 * 打开项目的framework目录，将**GPUImage.xcodeproj**直接拉到工程中
 ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151120project.png) 
 * 文档工程中，选择TARGETS--Build Phases--Target Dependencies--点击**+**加号添加**GPUImage**,**注意图片是白色的小房子**
 * 继续同一个页面下面，Link Binary With Libraries添加图中相对应的静态库
 
 ![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151120lib.png)
 * 指定**库的路径**。选择PROJECT-Build Setting--搜索:**Header Search Paths**,双击后面的路径。点击加号。填上**framework/Source**的**绝对路径**或者**相对路径**，最后点击选择**recursive**。使用**相对路径**，则填上**$(SRCROOT)/masonry/lib/GPUImage/framework/Source**,最后点击选择**recursive**.**$(SRCROOT)**就是你工程最顶层的目录。
 
![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151120path.png)
 * 最后，在工程的ViewController.m添加**#import "GPUImage.h"**。And run这个工程。
 
###测试GPUImage

 
	GPUImageSepiaFilter *filter = [[GPUImageSepiaFilter alloc] init];
    self.catImageView.image = [filter imageByFilteringImage:[UIImage imageNamed:@"cat.jpg"]]; 
    
###测试结果
	
![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151120result.png)

### 根据GPUImage的demo学习滤镜
	
####拍照图片添加滤镜
**SimplePhotoFilter**

####视频录像添加滤镜
**SimpleVideoFileFilter**，视频的文件可以使用Xcode直接读取沙盒的文件得到对应的录像文件。

#####更多的学习可以直接在对应的github文件中获得。
---

##2015-11-07

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/encourage.jpg) 

### coreText框架学习

> 学习coreText，让我想起了大学那段学习液晶屏幕驱动显示的时光。

 * 原文地址 [f* GFW](http://www.zoomfeng.com/blog/coretextshi-yong-jiao-cheng-%5B%3F%5D.html)
 
 
####概览

#####简介
 coteText是一个apple的文版排版框架。它直接将文本的内容，颜色，字体等全部属性交给coreGraphics进行渲染。由于直接与coreGraphics交互，因此渲染非常的高效。
 
#####组成架构
 
![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/coreTextArchi.png) 

#####coteText与WebView在排版的差异

*  coretext占用内存更少，渲染更快。
*  coretext在渲染之前就已经知道了文本的全部属性，其中包含了文本的高度等等属性。但是WebView只可以在渲染之后才能知道如何文本的高度。
* coretext毫无疑问的是具有最好的原生交互效果。然后WebView只可以通过JS进行深度交互，而且交互效果效果也是比不上原生的。
* 但是coretext并不能想WebView那样支持文本的复制与粘贴。
* coretext在排版上面的逻辑更加复杂。当然，你想控制的地方更多更加深入，逻辑的复杂度也是伴随而来的。




####绘制纯文本

	 // 1.获取上下文
    CGContextRef contextRef = UIGraphicsGetCurrentContext();
    
    // 2.转换坐标系
    CGContextSetTextMatrix(contextRef, CGAffineTransformIdentity);
    CGContextTranslateCTM(contextRef, 0, self.bounds.size.height);
    CGContextScaleCTM(contextRef, 1.0, -1.0);
    
    // 3.创建绘制区域，可以对path进行个性化裁剪以改变显示区域
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathAddRect(path, NULL, self.bounds);
    
    // 4.创建需要绘制的文字
    NSMutableAttributedString *attributed = [[NSMutableAttributedString alloc] initWithString:self.text];
    
    // 设置行距等样式
    [[self class] addGlobalAttributeWithContent:attributed font:self.font];
    
    // 加点料
    [attributed addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:30] range:NSMakeRange(10, 5)];
    
    [attributed addAttribute:NSForegroundColorAttributeName value:[UIColor greenColor] range:NSMakeRange(5, 10)];
    
    // 5.根据NSAttributedString生成CTFramesetterRef
    CTFramesetterRef framesetter = CTFramesetterCreateWithAttributedString((CFAttributedStringRef)attributed);
    
    CTFrameRef ctFrame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, attributed.length), path, NULL);
    
    // 6.绘制
    CTFrameDraw(ctFrame, contextRef);
    
   
#####1 获取上下文

	CGContextRef contextRef = UIGraphicsGetCurrentContext();
   就是获取当前画布的上下文，因为会将这个上下文传递给coreGraphics进行渲染的。
上下文，这个词第一次出现我大学学习RTOS的时候。当系统切换线程的时候，就保存当前线程的上下文到栈里面，然后又从栈中得到将要切换到的线程的上下文并进行执行。哈哈，当初翻译这个context的前辈是怎样想到这个词的呢?好奇怪但是又觉得好贴切。

#####2 切换坐标系
	CGContextSetTextMatrix(contextRef, CGAffineTransformIdentity);
因为coreGraphics的原点是在左下角，coretext的是在左上角。因此，要对画布用当前的矩阵进行翻转坐标系。在代码中我们可以log出每个CTRun的rect.origin.y就可以知道是从最底部开始draw到最顶部的。
	
	CGContextTranslateCTM(contextRef, 0, self.bounds.size.height);
将当前的画布的原点平移到**{0，self.bounds.size.height}**

如果没有进行翻转坐标系，会发现文本产生镜像式的倒转显示。

##### 3 创建绘制区域
	    CGMutablePathRef path = CGPathCreateMutable();
        CGPathAddRect(path, NULL, self.bounds);
        
对path进行个性化裁剪从而改变显示区域。如果代码的一样，一般都是与当前view的bounds保持一致，就是满显示。

##### 4 创建需要绘制的文本
	NSMutableAttributedString *attributed = [[NSMutableAttributedString alloc] initWithString:self.text];
    
 这时候，准备开始生产东西了，当然就是从文本这个材料开始了。
 
 	[[self class] addGlobalAttributeWithContent:attributed font:self.font];
 	
 设置文本的全局属性，比如行距，字体大小，默认颜色。当然后面可以对文本进行再次的加工
 
 	[attributed addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:30] range:NSMakeRange(10, 5)];
    
   	 [attributed addAttribute:NSForegroundColorAttributeName value:[UIColor greenColor] range:NSMakeRange(5, 10)];


#####5 根据前面的材料生产得到最终渲染的全部属性材料
 
	CTFramesetterRef framesetter =CTFramesetterCreateWithAttributedString((CFAttributedStringRef)attributed);
	
先用之前的AttributeString创建CTFramesetterRef

	CTFrameRef ctFrame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, attributed.length), path, NULL);
    
再用上面创建的frameRef以及path作为参数创建得到最后的渲染的frame

#####6 绘制
	CTFrameDraw(ctFrame, contextRef);
    
#### 图文混排

---

从纯文本的排版到图文混排，需要知道的是整个文本的结构。

 * CTLine:每一行的文本
 * CTRun：每一行不同的属性的文本


如何实现图文混排呢？其实就是先用一个占位符填充到需要显示图片的位置，并设置这个空白占位符的宽高等。当图片下载或者从本地读取之后就使用平时渲染Image的方式渲染到对应的地方。
从代码中可以知道对于**本地**以及**网络**上的图片，设置占位符的方式是不一样的。**本地**或者**网络**的使用**" "**作为占位符，也可以使用**0xfffc**。最后都是通过遍历整个文本的CTRun来得到图片的位置，根据这个CTRun的属性使用**CGContextDrawImage**进行渲染。

####调整文本高度

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/word.png) 

	* 根据文本保持baseLined对齐，高度是根据每一行的CTRun的属性确定。在代码中，第一行是不需要计算文本对齐的，而是从第二行开始的。
	* 根据文本自定义高度，实现整个文本的统一高度
	
####实现文本的点击交互
	思路:为这个view添加手势事件，然后通过手势获取当前屏幕的**点击点的位置**。然后，在这个手势事件中使用正则表达式来获取整个文本中会出发点击事件的**文本块**，最后通过匹配**点击点的位置**与**文本块**，**点**是否落在某个**文本块**中，最后对具体的**块**做出相应的事件出发


##2015-10-08

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20151008.jpg) 

###UIWebView与JS的深度交互


>  博主一直以来是我的学习榜样，很多很好的解决方法。

 * 原文地址[f*ck GFW](http://kittenyang.com/webview-javascript-bridge/)

记得以前刚开始的CRM系统也是使用JS来开发的。当时自己也是尝试解决原生与JS之间的交互问题，但是效果还不是很理想。思路其实也是跟当前博主的思路一样的， 可惜当时自己的能力与任务繁重问题导致没有更好地去解决。
 曾记得，之前微博也是转发了一个第三方库用来解决原生与JS之间的深度交互，需要找找。**最后也是暴露了自己需要找个时间来学习前端开发才可以了。**


=======
##2015-09-25

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20150925.jpg) 

> iOS Block vs delegate

 * 原文地址: [f*ck GFW](http://blog.stablekernel.com/blocks-or-delegates/),在CocoaChina中也有对应的[译文](http://www.cocoachina.com/ios/20150925/13525.html)
 

###从delegate到Block

刚开始学习不同页面的时候,当然学习的是使用代理传值了.对于刚开始的学习者来说这种比较容易理解.后面学习使用AFNetworking,才看到别人家是使用Block来传值的.之后在自己的开发的项目中也使用过Block传值.用在对AFN的一些方法进行了进一步的封装以及两个界面之间的传值.

###什么时候使用Block,什么时候使用delegate呢?

跟着上面的描述,用了一段时间之后自己心中也提出了这样一个问题.虽然一直都会去搞清楚,似乎还是似懂非懂.也是可能是基础还是很差呢,在钻牛角尖!aha~今天特地去stackoverflow找到这么一段非常好的[回答](http://stackoverflow.com/questions/21771606/objective-c-delegate-or-c-style-block-callback).

>Each has its use.

Delegates should be used when there are ***multiple "events"*** to tell the delegate about and/or when the class needs to get data from the delegate. A good example is with **UITableView**.

A block is best used when there is ***only one (or maybe two) event***. A **completion block** (and maybe a failure block) are a good example of this. A good example is with NSURLConnection sendAsynchronousRequest:queue:completionHandler:.

A 3rd option is notifications. This is best used when there are possibly multiple (and unknown) interested parties in the event(s). The other two are only useful when there is one (and known) interested party.


从上面的回答中我们可以知道对于需要传递多个不同的数据时候使用代理代理,就像举例说的**UITableview**,一个代理包含可以声明多个代理方法实现不同数据的传递.对于只有单一事件或者数据传递可以直接使用Block,我自己也是认为在这种情况使用Block可以更加简洁.比如在网络请求部分就是使用了Block,非常简洁明了.

###那么对于多人开发的时候呢?应该选择代理还是Block?

这个问题我实在微博看到别人对这边[译文](http://www.cocoachina.com/ios/20150925/13525.html)的一个评论.暂时没有思考得到其中的原因.为什么这么提问,怎样去回答??先留着.

---


##2015-09-20

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20150920.jpg) 


>UIScrollView 实践经验

 * 原文地址:这篇文章不需要爬梯  [f*ck GFW](http://tech.glowing.com/cn/practice-in-uiscrollview/)
 
如标题所讲,是关于UIScrollView的实践经验的.读罢全篇文章,我觉得这是对UISCrollview的最佳实践之一了.当我看到这篇文章的时候,一个词:**awesome**
 
文中是从UIScrollView的关于触摸,滑动等事件以及对应过程的代理协议出发,讨论了**UITableView**(UIScrollView的子类)在图片优化方面的处理.UIScrollView的**分页实现方式**,竟然还有**ViewController重用**等问题.其中关于**UITableView**的优化让我收获最多.同时,我自己也跟着里面的分析过程写了一个测试[Demo](https://github.com/zhaiyjgithub/article-2015-09-20.git).

在[Demo](https://github.com/zhaiyjgithub/article-2015-09-20.git)中,我并没有使用文章中使用SDWebImage来请求图片并测试.而是使用最简单的检测方法.详情请看[Demo](https://github.com/zhaiyjgithub/article-2015-09-20.git)的实现过程.当然你也可以直接到一些图片网站测试更加好了,比如我之前使用的是[500px](http://www.jianshu.com/p/f1208b5e42d9) , 这个就是当时学习**swift**的网络请求库**alamoFire**找到的.当然,这个关于**alamoFire**的两篇教程质量还是杠杠的.So,enjoy it!
	
	cell.textLabel.text = [NSString stringWithFormat:@"row:%d",indexPath.row];
	
当然最好是按照文中提到的**问题1,2,3**一步步来跟着分析过程来测试.记得好好留意这个代理方法:

    - (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset
    
最后多谢博主的分享.happy everyday!!

---
=======

##2015-09-12

![](https://github.com/zhaiyjgithub/article/raw/master/titlePic/20150912.jpg)   

>Swift:The rules of being weak and unowned in clusures and capture lists(Xcode 6 Beta 7)

### swift：weak和unowned关键字在闭包以及捕获列表中的使用规则
* 原文地址(需要自备梯子):[f*ck GFW](http://sketchytech.blogspot.com/2014/09/swift-rules-of-weak-and-unowned.html)

从早前文章提出的**weak**,**wnowned** 和 **lazy**关键字，我应该在在这里好好地思考它们在**闭包**中的使用方法

####一、为什么我们要考虑如何在闭包中使用weak以及unwonted？

第一件需要做的事情就是解释为什么我们要再**闭包**中使用weak和**unowned**.然而那些牛人已经对此写好了文档，因此我们只要阅读这些文档就可以知道上述问题的所在了

>如果你要用一个闭包为一个类的实例中的属性赋值，此时闭包就会通过引用这个实例或者或者该实例的成员来捕获得到这个实例。那么你就会在闭包与实例之间建立了强引用循环。Swift语言就使用捕获列表来打破这种强引用循环 ---		Apple

####二、如何利用捕获列表来解决问题呢？
这令人感觉到一种强烈的责任感而不去在一个闭包中建立强引用循环。但是，只要我们花费一些些时间去理解这个捕获列表就可大大缓解这种焦虑感：

>一个闭包表达式可以指定通过捕获列表来捕获周围得到的值。捕获列表写在一个方括号里，不同的值之间使用逗号分隔开。如果要你需要用到捕获列表，就必须使用**in**关键字，即使你漏掉了参数的名称，参数的类型以及返回值的类型。

每一个捕获列表的入口，都使用**marked**或者**unowned**来标记使用它们引用的值

	myFunction { print(self.title) }                    // strong capture
	myFunction { [weak self] in print(self!.title) }    // weak capture
	myFunction { [unowned self] in print(self.title) }  // unowned capture

在捕获列表中，你也可以使用一个随机表达式来命名一个值。当一个闭包形成时就会计算这些表达式以及得到它指定的强弱引用程度，比如：

	// Weak capture of "self.parent" as "parent"
	myFunction { [weak parent = self.parent] in print(parent!.title) }" (Apple)


#### 三、在闭包中weak和unowned的使用规则
首先，最重要的是要弄清楚只有在相关的闭包中哪个地方我们要用一个闭包为类的实例赋值。因此，我们要记得一下这些规则

* 1 如果类的实例或者属性是可选类型的，就使用**weak**
* 如果类的实例或者属性不是可选类型并且永远都不会被赋值为nil,使用**unwonted**
* 必须使用**in**关键字，即使你没有参数名称，参数类型或者返回值类型
*

**Note:** 在下面的列子中的闭包都是关于**lazy**属性变量的。因为在其他的情况下它将是不可访问的属性变量。除非这个类已经被初始化了。

##### 四、解释和测试代码
一个强引用的闭包：

	class Parent {
    	var title = "Dad"
    	lazy var parentOf:(String)->String = {
    		(childname:String) in return childname+"'s "+self.title
    	} // DON'T DO THIS UNLESS YOU WANT THE CLOSURE TO KEEP THE INSTANCE ALIVE!!!
	 }
	 
因此，我们可以添加一个捕获列表来解决这个问题：

	class Parent {
    	var title = "Dad"
    	lazy var parentOf:(String)->String = {
   	 [unowned self] (childname:String) in return childname+"'s "+self.title
    	} // OK
 	}
 	
**Note:** 这个捕获列表是属于**[unowned self]**部分

#### 五、 unowned和weak的区别

即使**title**变量是可选类型的，但是我们仍然可以使用**owned**

	class Parent {
   	 var title:String? = "Dad"
   	 lazy var parentOf:(String)->String = {
   	 	[unowned self](childname:String) in return childname+"'s "+self.title!
    		}
	}
	
因为**title**是一个可选类型的字符串变量，而不是一个可选类型的类实例。然而，让我们假设一下创建一个**child**类并包含一个可选类型的**parent**属性变量：

	class Child {
    var parent:Parent?
    init (parent:Parent) {
        self.parent = parent
    }
    lazy var myParent:()->() = { [weak parent = self.parent] in print(parent!.title) }
	}
 
现在，我们这里使用的是**weak**，因为**parent**是一个可选类型的类实例变量。我们同样使用了这个语法来引用了**self**并允许去指定属性。需要记住的是在闭包中如何使用**parent**而不是**self.parent**.(Tip:尝试将**weak**用**unowned**替换，编译器会提示错误的)

#### 六、同时使用**weak**和**unowned**
让我们继续添加一个不是可选类型的**grandparent**变量。我们可以简单地添加它到捕获列表中（此时，我们使用的是**unowned**因为它是一个非可选类型的变量）

	class Child {
    var parent:Parent?
    var grandparent:GrandParent
    init (parent:Parent, grandparent:GrandParent) {
        self.parent = parent
        self.grandparent = grandparent
    }
    lazy var myParent:()->() = { [weak parent = self.parent, unowned grandparent = self.grandparent] in print("\(parent!.title) is \(grandparent.title)'s son"); }
	}
	
另外，这个是grandpaent的类的定义

	class GrandParent {
    var title:String = "Gramps"
	}
	
接下来就是一些基本的使用方式

	let kid = Child(parent: Parent(), grandparent:GrandParent())
	kid.myParent()
	
#### 总结
