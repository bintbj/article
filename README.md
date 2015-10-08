# article
some article about programm.
***

##2015-10-08

###UIWebView与JS的深度交互

>  博主一直以来是我的学习榜样，很多很好的解决方法。

 * 原文地址[f*ck GFW](http://kittenyang.com/webview-javascript-bridge/)

记得以前刚开始的CRM系统也是使用JS来开发的。当时自己也是尝试解决原生与JS之间的交互问题，但是效果还不是很理想。思路其实也是跟当前博主的思路一样的， 可惜当时自己的能力与任务繁重问题导致没有更好地去解决。
 曾记得，之前微博也是转发了一个第三方库用来解决原生与JS之间的深度交互，需要找找。**最后也是暴露了自己需要找个时间来学习前段才可以了。**



##2015-09-12
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
