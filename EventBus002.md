## EventBus

#### 线程模式ThreadMode
**PostThread：**事件的处理在和事件的发送在相同的进程，所以事件处理时间不应太长，不然影响事件的发送线程。

**MainThread: **事件的处理会在UI线程中执行。事件处理时间不能太长，这个不用说的，长了会ANR的。

**BackgroundThread：**如果事件是在UI线程中发布出来的，那么事件处理就会在子线程中运行，如果事件本来就是子线程中发布出来的，那么事件处理直接在该子线程中执行。所有待处理事件会被加到一个队列中，由对应线程依次处理这些事件，如果某个事件处理时间太长，会阻塞后面的事件的派发或处理。

**Async：**事件处理会在单独的线程中执行，主要用于在后台线程中执行耗时操作，每个事件会开启一个线程。

#### priority事件优先级 EventBus黏性事件  终止对事件继续传递的功能
优先级高（priority 设置的值越大，默认为0），则会先接收事件进行处理；优先级低（priority 设置的值越小），则会后接收事件进行处理。

    @Subscribe(priority = 100,sticky = true)
    public void onToastEvent(ToastEvent event){
        Toast.makeText(MainActivity.this,event.getContent(),Toast.LENGTH_SHORT).show();
        EventBus.getDefault().cancelEventDelivery(event);
    }
    
    EventBus.getDefault().postSticky(new ToastEvent("Toast,。。"));

#### Register
1. 通过subscriptionsByEventType得到该事件类型所有订阅者信息队列，根据优先级将当前订阅者信息插入到订阅者队列subscriptionsByEventType中；
2. 在typesBySubscriber中得到当前订阅者订阅的所有事件队列，将此事件保存到队列typesBySubscriber中，用于后续取消订阅； 
3. 检查这个事件是否是 Sticky 事件，如果是则从stickyEvents事件保存队列中取出该事件类型最后一个事件发送给当前订阅者。

#### Post
1. post 函数会首先得到当前线程的 post 信息PostingThreadState，其中包含事件队列，将当前事件添加到其事件队列中，然后循环调用 postSingleEvent 函数发布队列中的每个事件。
2. 调用 postSingleEventForEventType 函数发布每个事件到每个订阅者。
3. 调用 postToSubscription 函数向每个订阅者发布事件。 postToSubscription 函数中会判断订阅者的 ThreadMode，从而决定在什么 Mode 下执行事件响应函数。
    
## Window、Activity、DecorView、ViewRoot
#### Activity
Activity并不负责视图控制，它只是控制生命周期和处理事件。真正控制视图的是Window。一个Activity包含了一个Window，Window才是真正代表一个窗口。Activity就像一个控制器，统筹视图的添加与显示，以及通过其他回调方法，来与Window、以及View进行交互。

#### Window
Window是视图的承载器，内部持有一个 DecorView，而这个DecorView才是 view 的根布局。Window是一个抽象类，实际在Activity中持有的是其子类PhoneWindow。PhoneWindow中有个内部类DecorView，通过创建DecorView来加载Activity中设置的布局R.layout.activity_main。Window 通过WindowManager将DecorView加载其中，并将DecorView交给ViewRoot，进行视图绘制以及其他交互。

#### DecorView
DecorView是FrameLayout的子类，它可以被认为是Android视图树的根节点视图。DecorView作为顶级View，一般情况下它内部包含一个竖直方向的LinearLayout，在这个LinearLayout里面有上下三个部分，上面是个ViewStub，延迟加载的视图（应该是设置ActionBar，根据Theme设置），中间的是标题栏(根据Theme设置，有的布局没有)，下面的是内容栏。

#### ViewRoot
ViewRoot可能比较陌生，但是其作用非常重大。所有View的绘制以及事件分发等交互都是通过它来执行或传递的。
ViewRoot对应ViewRootImpl类，它是连接WindowManagerService和DecorView的纽带，View的三大流程（测量（measure），布局（layout），绘制（draw））均通过ViewRoot来完成。
ViewRoot并不属于View树的一份子。从源码实现上来看，它既非View的子类，也非View的父类，但是，它实现了ViewParent接口，这让它可以作为View的名义上的父视图。RootView继承了Handler类，可以接收事件并分发，Android的所有触屏事件、按键事件、界面刷新等事件都是通过ViewRoot进行分发的。
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
## Binder
**Binder基于Client-Server通信模式，传输过程只需一次拷贝，为发送发添加UID/PID身份，既支持实名Binder也支持匿名Binder，安全性高。**
    
1. socket作为一款通用接口，其传输效率低，开销大，主要用在跨网络的进程间通信和本机上进程间的低速通信。
2. 消息队列和管道采用存储-转发方式，即数据先从发送方缓存区拷贝到内核开辟的缓存区中，然后再从内核缓存区拷贝到接收方缓存区，至少有两次拷贝过程。
3. 共享内存虽然无需拷贝，但控制复杂，难以使用。
    
    

**Client进程**：使用服务的进程。
**Server进程**：提供服务的进程。
**ServiceManager进程**：ServiceManager的作用是将字符形式的Binder名字转化成Client中对该Binder的引用，使得Client能够通过Binder名字获得对Server中Binder实体的引用。
**Binder驱动**：驱动负责进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持。

#### Binder运行机制
**注册服务(addService)**：Server进程要先注册Service到ServiceManager。该过程：Server是客户端，ServiceManager是服务端。
**获取服务(getService)**：Client进程使用某个Service前，须先向ServiceManager中获取相应的Service。该过程：Client是客户端，ServiceManager是服务端。
**使用服务**：Client根据得到的Service信息建立与Service所在的Server进程通信的通路，然后就可以直接与Service交互。该过程：Client是客户端，Server是服务端。

图中的Client，Server，Service Manager之间交互都是虚线表示，是由于它们彼此之间不是直接交互的，而是都通过与Binder驱动进行交互的，从而实现IPC通信（Interprocess Communication）方式。

其中Binder驱动位于内核空间，Client，Server，Service Manager位于用户空间。Binder驱动和Service Manager可以看做是Android平台的基础架构，而Client和Server是Android的应用层，开发人员只需自定义实现Client、Server端，借助Android的基本平台架构便可以直接进行IPC通信。

#### Binder运行的实例解释

首先我们看看我们的程序跨进程调用系统服务的简单示例，实现浮动窗口部分代码：
    
    //获取WindowManager服务引用
    WindowManager wm = (WindowManager)  getSystemService(getApplication().WINDOW_SERVICE);
    //布局参数layoutParams相关设置略...
    View view = LayoutInflater.from(getApplication()).inflate(R.layout.float_layout, null);
    //添加view
    wm.addView(view, layoutParams);
注册服务(addService)： 在Android开机启动过程中，Android会初始化系统的各种Service，并将这些Service向ServiceManager注册（即让ServiceManager管理）。这一步是系统自动完成的。
获取服务(getService)： 客户端想要得到具体的Service直接向ServiceManager要即可。客户端首先向ServiceManager查询得到具体的Service引用，通常是Service引用的代理对象，对数据进行一些处理操作。即第2行代码中，得到的wm是WindowManager对象的引用。
使用服务： 通过这个引用向具体的服务端发送请求，服务端执行完成后就返回。即第6行调用WindowManager的addView函数，将触发远程调用，调用的是运行在systemServer进程中的WindowManager的addView函数。

使用服务的具体执行过程

1. Client通过获得一个Server的代理接口，对Server进行调用。
2. 代理接口中定义的方法与Server中定义的方法是一一对应的。
3. Client调用某个代理接口中的方法时，代理接口的方法会将Client传递的参数打包成Parcel对象。
4. 代理接口将Parcel发送给内核中的Binder Driver。
5. Server会读取Binder Driver中的请求数据，如果是发送给自己的，解包Parcel对象，处理并将结果返回。
6. 整个的调用过程是一个同步过程，在Server处理的时候，Client会Block住。因此Client调用过程不应在主线程。
    
    
## 热修复    
一个ClassLoader可以包含多个dex文件，每个dex文件是一个Element，多个dex文件排列成一个有序的数组dexElements，当找类的时候，会按顺序遍历dex文件，然后从当前遍历的dex文件中找类，如果找类则返回，如果找不到从下一个dex文件继续查找。(来自：安卓App热补丁动态修复技术介绍)

那么这样的话，我们可以在这个dexElements中去做一些事情，比如，在这个数组的第一个元素放置我们的patch.jar，里面包含修复过的类，这样的话，当遍历findClass的时候，我们修复的类就会被查找到，从而替代有bug的类。
    
    
## 内存 
**静态存储区（方法区）**：主要存放静态数据、全局 static 数据和常量。这块内存在程序编译时就已经分配好，并且在程序整个运行期间都存在。
**栈区** ：当方法被执行时，方法体内的局部变量都在栈上创建，并在方法执行结束时这些局部变量所持有的内存将会自动被释放。因为栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限。
**堆区** ： 又称动态内存分配，通常就是指在程序运行时直接 new 出来的内存。这部分内存在不使用时将会由 Java 垃圾回收器来负责回收。
    
集合类泄漏 final
单例造成的内存泄漏
非静态内部类和异步线程 相互引用
匿名内部类  MainActivity实例会被ref2持有，如果将这个引用再传入一个异步线程，此线程和此Acitivity生命周期不一致的时候，就造成了Activity的泄露。
Handler 造成的内存泄漏    由于 Handler 属于 TLS(Thread Local Storage) 变量, 生命周期和 Activity 是不一致的。因此这种实现方式一般很难保证跟 View 或者 Activity 的生命周期保持一致，故很容易导致无法正确释放。
修复方法：在 Activity 中避免使用非静态内部类，比如上面我们将 Handler 声明为静态的，则其存活期跟 Activity 的生命周期就无关了。同时通过弱引用的方式引入 Activity，避免直接将 Activity 作为 context 传进去，
尽量避免使用 static 成员变量
资源未关闭造成的内存泄漏
对于使用了BraodcastReceiver，ContentObserver，File，游标 Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销，否则这些资源将不会被回收，造成内存泄漏。


## LeakCanary方式
0. 监听Activity.onDestroy() 之后泄露的 activity。
1. RefWatcher.watch() 创建一个 KeyedWeakReference 到要被监控的对象。
2. 然后在后台线程检查引用是否被清除，如果没有，调用GC。
3. 如果引用还是未被清除，把 heap 内存 dump 到 APP 对应的文件系统中的一个 .hprof 文件中。
4. 在另外一个进程中的 HeapAnalyzerService 有一个 HeapAnalyzer 使用HAHA 解析这个文件。
5. 得益于唯一的 reference key, HeapAnalyzer 找到 KeyedWeakReference，定位内存泄露。
6. HeapAnalyzer 计算 到 GC roots 的最短强引用路径，并确定是否是泄露。如果是的话，建立导致泄露的引用链。
7. 引用链传递到 APP 进程中的 DisplayLeakService， 并以通知的形式展示出来。
