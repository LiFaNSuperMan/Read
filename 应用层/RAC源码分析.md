# RAC源码阅读

###	整体结构
-	信号源
	-	RACStream及其子类			
		-	RACSignal
			-	冷信号
				-	简单理解就是现在无人使用只是创建了的信号就是冷信号
			-	热信号
				-	冷信号通过订阅者订阅信号变成了热信号
			-	实现原理
				-	创建信号->实际创建的是RACDynamicSignal信号，最终disSubscribe block函数被保存下来
				-	订阅信号->实际创建的是RACSubscriber Block函数被保存下来，只有在订阅之后,创建信号的block函数中的subscriber才通过订阅信号函数时创建的RACSubscriber对象通过block返回回来
				-	执行信号->实际上就是执行block的过程
			-	总结
				-	实际上就是对block函数的保存使用 在整个过程中真正的执行者是RACSubscriber
	-	RACSubject
		-	实现原理
			-	实现比较简单，在内部有个数据保存RACSubscriber，发送信号的时候遍历数组，执行nextBlock
-	订阅者
	-	RACSubscriber实现类及其子类
		-	在RAC中，RACSubscriber实际上是一个协议，所有实现了这个协议的类都可以订阅信号
-	调度器
	-	RACScheduler及其子类
		-	只是对GCD的简单封装，保证任务按顺序执行
-	清洁工
	-	RACDisposable及其子类				
		-	它封装了取消和清理一次订阅所必需的工作		
###	常用方法的源码分析

-	RACObserve
	-	常见用法
		-	RACObserve(self.viewModel , isNeedRefresh)		
			-	RACObserve最终实现的是NSObject(RACPropertySubscribing) Category中的方法在内部调用NSObject(RACKVOWrapper)中的方法，最终上的实现依赖RACKVOTrampoline，在这里实现了系统级别的KVO方法，将代理转为Block作为输出。简而言之就是在内部实现KVO,然后将KVO的代理回调变成了Block的回调 
		-	Map	映射	FlattenMap
			-	重新定义返回值，两者的区别在于Map只是修改了返回值，而FlattenMap则直接修改了信号
		-	Filter
			-				