#GC与内存泄漏


日期：2017年8月13日</br>
作者：王宇滔

危害：内存泄漏不仅会造成vm运行时可用内存变小，还可能造成gc触发频繁（内存抖动，页面卡顿），甚至造成oom。

##对象的回收

* **c的世界（c/c++/objective c）**
	* **手动控制**</br>
	  	手动创建，回收对象
	
	* **RC**(reference counting)</br>
	  	RC技术利用引用计数在**编译期间**插入回收代码；但是引用计数有个缺点，它没办法很好的解决循环引用的问题。ARC是通过week引用的方式解决的。有兴趣的同学可以参考[这篇](http://www.jianshu.com/p/e7b1651df746)文章和[这篇](http://www.jianshu.com/p/492be28d63c4)文章。

* **java的世界**
	* **rc gc**(reference counting garbage collector) </br>
	  	使用RC实现的垃圾回收器。
	  
	* **tracking gc** (tracing garbage collector)</br>
	  	使用垃圾追踪技术实现的垃圾回收器，android的gc就是这种。
	  
##内存怎么分配
android app程序运行时的内存分配策略有四种：

* 静态分配</br>
	静态分配:主要存放静态数据、全局 static 数据和常量。这块内存在类被加载的时候就分配好了，并且在app运行期间都存在（但极限情况有可能被回收）。
	
* 栈分配</br>
	栈:当方法被执行时，方法体内的局部变量（其中包括基础数据类型、对象的引用）都在栈上创建，并在方法执行结束时这些局部变量所持有的内存将会自动被释放。</br>
	[相关知识](https://www.zhihu.com/question/22444939)

* 堆分配</br>
	堆区:app运行时直接 new 出来的内存，也就是对象的实例。这部分内存在不使用时将会由gc来负责回收。

* native分配</br>
	jni调用:进行本地代码调用时候分配的内存，这部分内存是不受gc管理的，需要手动释放。
	
知识拓展：	

* MemoryFile:匿名共享内存Ashmem，这部分内存类似于Native内存区。
fresco在android4.4的版本就是使用Ashmem来缓存位图（三级缓存），以此来降低gc造成的内存抖动从而来获取更高的内存性能。fresco相关类：GingerbreadPurgeableDecoder和KitKatPurgeableDecoder。

##何时回收内存
* 堆内存不够用时
* 被建议回收内存时（System.gc）
* 超过vm配置阀值时

##如何回收内存
垃圾回收简单的流程如下

* step 1 : stop the world</br>
	线程暂停工作，lock住heap；

* step 2 : 标记出垃圾</br>
	运用tracing技术追踪 直接／间接 与GC ROOT关联的对象；

* step 3 : 清理垃圾</br>
	清理掉与GC ROOT无关联对象（不同的gc会进行不同的操作，与使用的回收算法有关[参考这篇文章](https://www.zhihu.com/question/19912197)）；

* step 4 : resume the world</br>
	线程恢复工作

* GC ROOT 包括:
	1. Class:由系统的类加载器加载的类对象
	2. Static Fields
	3. Thread:活着的线程
	4. Stack Local: java方法的局部变量或参数
	5. JNI Local／Global: jni中对应java的 全局变量／局部变量
	7. Monitor used: 用于同步的监控对象

MS：对android gc具体过程有兴趣的同学可以参考[这篇](http://blog.csdn.net/luoshengyang/article/details/41822747)文章。

##内存泄漏如何发生的
内存的泄漏是发生在堆内存上的。如果长生命周期的对象持有短生命周期的引用，就很可能会出现内存泄露。或者说一个时刻，应该被回收的对象却被GC ROOT集合引用着，就可以认为这个对象泄漏了。



##Android内存泄漏如何解决
* Handler内存泄漏；
	* activity销毁时候，消息队列清理掉这个Handler的消息；
	
			mHandler.removeCallbacksAndMessages(null);

* 非静态匿名内部类的内存泄漏；
	* 改为静态内部类；
	* 改为非内部类，使用weekReference包裹对象；

			private static class MyHandler extends Handler {
		    private final WeakReference<Activity> mActivity;
		
			    public MyHandler(SampleActivity activity) {
			      mActivity = new WeakReference<Activity>(activity);
			    }
			
			    @Override
			    public void handleMessage(Message msg) {
			      Activity activity = mActivity.get();
			      if (activity != null) {
			        // ...
			      }
			    }
		    }

* 静态对象系列的内存泄漏；
	* 对象生命周期结束时，手动断开与静态对象的引用； 

* 多线程操作系列的内存泄漏；
	* 设法中断线程，或者用弱引用包裹对象； 

* 其他： 
	* cursor io 等用完记得关闭；	
	* 需要使用context时，如果与ui无关，尽量使用applicationContext;
	* 对于泄漏要有直觉，感觉会漏自己check一下（特别是用内部类，多线程等情况）；
	* 用leakcanry辅助判断；http://www.jianshu.com/p/5ee6b471970e
	 