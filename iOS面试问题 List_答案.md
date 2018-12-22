# iOS面试问题 List

# 一、项目概况
项目是最简单的，不算考察，聊天式的拉近距离，让候选人放松，气氛融洽。
## 1、项目：
- **项目概况：**多久？多人？结果如何？上线没？多少人用？你感觉上个项目总体做的如何？（自我，同事，结果）
- **项目难点：**哪方面？技术、配合、沟通？如何解决？
- **最好的地方：**架构、分层？
- **复盘：**如果重新做，有没有什么可以改进的地方？做得不太好的地方


# 二、计算机技术：
考察基础
## 2、多线程  
### 2.1 任务队列
**任务：** 即操作，你想要干什么，说白了就是一段代码。 任务有两种执行方式：`同步`、`异步`

- 同步：只能在当前线程执行，会阻塞当前线程。不能开辟新的线程。而且是必须、立即执行。
- 异步：另开线程，在新的线程执行。当前线程会直接往下执行，它不会阻塞当前线程。

**队列：** 用于存放任务。一共有两种队列， `串行队列` 和 `并行队列`
并发队列可以让多个线程同时执行（必须是异步），
串行队列则是让任务一个接一个的执行。
![](https://github.com/Qiaomumer/Images/blob/master/Multithreading001.png?raw=true)

#### 同异步 串行并行队列 组合
###### 1.串行队列同步执行
Serial Dispatch Queue把任务一个个拿出来，线程一个个的执行拿出来的任务。
![](https://github.com/Qiaomumer/Images/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B_%E4%B8%B2%E8%A1%8C%E9%98%9F%E5%88%97%E5%90%8C%E6%AD%A5%E6%89%A7%E8%A1%8C.png?raw=true)
###### 2.并行队列同步执行
Concurrent Dispatch Queue会快速的把任务都拿出来，但是线程就只有一个，还是只能一个个得执行任务，所以并没有多线程执行。
###### 3.串行队列异步执行
Serial Dispatch Queue会把任务一个个拿出来，但拿出下一个前提是上一个任务已经被线程执行完毕了。所以这里会有多个线程来等待执行任务，但是任务却一个个出来，真是浪费了这么多线程资源啊。
###### 4.并行队列异步执行
这里任务快速拿出，线程快速执行，就实现了多线程操作，提高效率。
![](https://github.com/Qiaomumer/Images/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B_%E5%B9%B6%E8%A1%8C%E9%98%9F%E5%88%97%E5%BC%82%E6%AD%A5%E6%89%A7%E8%A1%8C.png?raw=true)


### 2.2 死锁
**main\_queue + 同步**

    - (void)viewDidLoad {
        [super viewDidLoad];
        NSLog(@"----------1");
        dispatch_sync(dispatch_get_main_queue(), ^{
            NSLog(@"----------2");
        });
        NSLog(@"----------3");
        }
        
        结果：1
        
dispatch\_get\_main\_queue： 主队列，是是典型的串行 Queue
主队列正在执行任务，然后执行block，所以block加入任务后边，（即3、2）前边的任务执行完毕，才能执行block。
但是同步执行，任务需要等block返回才能继续执行下去，（即2、3）。但是。所以都在等待，死锁。

**解1：global\_queue + 同步**

    - (void)viewDidLoad {
        [super viewDidLoad];
        NSLog(@"----------1");
        dispatch_sync(dispatch_get_global_queue(0, 0), ^{
            NSLog(@"----------2");
        });
        NSLog(@"----------3");
    }
    结果：1、2、3
    
类似于上次，不同之处在于，block在新的线程中，执行完毕了，所以可以执行3。结果为：123

**解2：**

    dispatch_queue_t queue = dispatch_queue_create("test.queue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queue, ^{
        [self syncMain];
    });
    
    -(void)syncMain{
        NSLog(@"----------1");
        dispatch_sync(dispatch_get_main_queue(), ^{
            NSLog(@"----------2");
        });
        NSLog(@"----------3");
    }

### 2.3 如何用GCD同步若干个异步调用？
（如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）

    使用Dispatch Group追加block到Global Group Queue,这些block如果全部执行完毕，就会执行Main Dispatch Queue中的结束处理的block。
    
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, );
    dispatch_group_t group = dispatch_group_create();
    dispatch_group_async(group, queue, ^{ /*加载图片1 */ });
    dispatch_group_async(group, queue, ^{ /*加载图片2 */ });
    dispatch_group_async(group, queue, ^{ /*加载图片3 */ });
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
         // 合并图片
    });




### 2.4 问题：如果要保证某个属性可以线程安全的读写，如何操作？
访问数据库或者文件的时候，我们可以使用Serial Dispatch Queue可避免数据竞争问题

###### 1、采用内置的synchronization [ˈsɪŋkrənaɪz]  block

    - (void)setName:(NSString *)name{
        @synchronized(self) {
            _name = [name copy];
        }
    }
 
    - (NSString *)name
    {
        @synchronized(self) {
            return _name;
        }
    }
**缺点：**

- **频繁执行，效率下降**。因为共用同一个锁的那些同步块，都必须按顺序执行。
- 事实上，上面的写法，就是**atomic**，也就是原子性属性xcode自动生成的代码，这种方法，在访问属性时，必定可以从中得到有效值。
- atomic的意思就是setter/getter这个函数，是一个原语操作。如果有多个线程同时调用setter的话，不会出现某一个线程执行完setter全部语句之前，另一个线程开始执行setter情况，相当于函数头尾加了锁一样，可以保证数据的完整性。nonatomic不保证setter/getter的原语行，所以你可能会取到不完整的东西。因此，在多线程的环境下原子操作是非常必要的，否则有可能会引起错误的结果。
- 然而如果在**一个线程上多次调用getter**方法，每次得到的**结果却未必相同**，在两次读操作之间，其**他线程可能会写入新的属性值**。

###### 2、GCD可以简单高效的代替同步块或者锁对象。
使用，串行同步队列，将读操作以及写操作都安排在同一个队列里，即可保证数据同步。

    static dispatch_queue_t _queue;
    
    - (instancetype)init{
        if (self = [super init]) {
        _queue = dispatch_queue_create("com.person.syncQueue", DISPATCH_QUEUE_SERIAL);
        }
        return self;
    }
 
    - (void)setName:(NSString *)name
    {
        dispatch_sync(_queue, ^{
            _name = [name copy];
        });
    }
 
    - (NSString *)name
    {
        __block NSString *tempName;
        dispatch_sync(_queue, ^{
            tempName = _name;
        });
        return tempName;
    }

###### 3、并行队列 +  dispatch\_barrier\_async
但2还不是最优的，它只可以实现单读、单写。整体来看，我们最终要解决的问题是，在写的过程中不能被读，以免数据不对，但是读与读之间并没有任何的冲突！
多个getter方法（也就是读取）是可以并发执行的，而getter(读)与setter（写）方法是不能并发执行的，利用这个特点，还能写出更快的代码来，这次注意，不用串行队列，而改用并行队列：

    - (instancetype)init{
        if (self = [super init]) {
        _concurrentQueue = dispatch_queue_create("com.person.syncQueue", DISPATCH_QUEUE_CONCURRENT);
        }
        return self;
    }
    - (void)setName:(NSString *)name{
        dispatch_barrier_async(_concurrentQueue, ^{
            _name = [name copy];
        });
    }
    - (NSString *)name{
        __block NSString *tempName;
        dispatch_sync(_concurrentQueue, ^{
            tempName = _name;
        });
        return tempName;
    }   
### dispatch\_barrier\_async 栅栏函数
在队列中，barrier块必须单独执行，不能与其他block并行。
这只对并发队列有意义，并发队列如果发现接下来要执行的block是个barrier block，那么就一直要等到当前所有并发的block(barrier函数之前)都执行完毕，才会单独执行这个barrier block代码块，等到这个barrier block执行完毕，再继续正常处理其他并发block。
该函数需要同dispatch\_queue\_create函数生成的concurrent Dispatch Queue队列一起使用


## 3、网络tcp 

### 3.1 TCP 概览
![](https://github.com/Qiaomumer/Images/blob/master/tcp_client_server.jpg?raw=true)

#### 3.1.1 TCP 为什么安全？
三次握手 + 滑动窗口协议

#### 3.1.2 TCP 三次握手
    
    第一次握手：建立连接时，客户端发送同步序列编号到服务器，并进入发送状态，等待服务器确认
    第二次：服务器收到同步序列编号，确认并同时自己也发送一个同步序列编号+确认标志，此时服务器进入接收状态
    第三次：客户端收到服务器发送的包，并向服务器发送确认标志，随后链接成功。
    注意：是在链接成功后在进行数据传输。

###### 为什么三次握手？
三次握手这个说法不好，其实是**双方各一次握手，各一次确认，其中一次握手和确认合并在一起**
双方都需要确认自己的发信和收信功能正常，收信功能通过接收对方信息得到确认，发信功能需要发出信息—>对方回复信息得到确认。
###### 不是2次？
1、需要第三次握手的原因在于Server端在第二次握手（发出信息）后并不知道对方是否能够接收、己方的发送功能是否正常。
但此时数据的单向通道已经建立，对于Client来说，已经确认了Server端可以接收信号，因此可以单向给Server发送数据了。
2、“已失效的连接请求报文段”的产生在这样一种情况下：client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送数据。但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。采用“三次握手”的办法可以防止上述现象发生。


###### 不是4次？
高效 至于为何没有第四次，因为通过三次握手双方已经互相确保了己方、对方都可以正常接收、发送信息了，无需第四次。

###### 最后一次握手丢失了怎么办？
对面没有收到ACK，就会重新发送SYN+ACK，如果这次收到了并且回复的ACK对面也收到了，那就可以继续。
如果超过设定的时间或者设定的重发次数，再进入CLOSED状态。

#### 3.1.3 TCP 四次挥手：

    第一次： 客户端向服务器发送一个带有结束标记的报文。
    第二次： 服务器收到报文后，向客户端发送一个确认序号，同时通知自己相应的应用程序：对方要求关闭连接
    第三次： 服务器向客户端发送一个带有结束标记的报文。
    第四次： 客户端收到报文后，向服务器发送一个确认序号。链接关闭。

#### 为什么要4次挥手？
TCP是全双工模式，这就意味着，当主机1发出FIN报文段时，只是表示主机1已经没有数据要发送了，主机1告诉主机2，它的数据已经全部发送完毕了；但是，这个时候主机1还是可以接受来自主机2的数据；当主机2返回ACK报文段时，表示它已经知道主机1没有数据发送了，但是主机2还是可以发送数据到主机1的；当主机2也发送了FIN报文段时，这个时候就表示主机2也没有数据要发送了，就会告诉主机1，我也没有数据要发送了，之后彼此就会愉快的中断这次TCP连接。如果要正确的理解四次分手的原理，就需要了解四次分手过程中的状态变化。

#### 握手挥手本质是差不多的，为什么握手将23合起来了，但是挥手没有？
握手时候，1说建立1->2连接。2收到后回复：同意+建立2->1连接,1收到后，同意。连接就建立了。

挥手时候，1对2说，我发完了，要断开。2说同意你断。但是此时，2还可能给1发信息，所以，不是同时给1说我发完了。而是真的等到2发完了，才向1说，我要断开了。1回复：同意断开。



### https 原理 如何加密？什么方式加密？什么是对称加密？
![HTTPS对应的通信时序图](https://github.com/Qiaomumer/Images/blob/master/HTTPS_001.png?raw=true)

HTTP 每次请求都要带什么参数？放哪儿？做什么作用？（签名）
![](https://github.com/Qiaomumer/Images/blob/master/http_token.png?raw=true)

#### 签名的设计 
原理：用户登录后向服务器提供用户认证信息（如账户和密码），服务器认证完后给客户端返回一个Token令牌，用户再次获取信息时，带上此令牌，如果令牌正取，则返回数据。对于获取Token信息后，访问用户相关接口，客户端请求的url需要带上如下参数：`时间戳：timestamp`、`Token令牌：token`。然后将所有用户请求的参数按照字母排序（包括timestamp，token），然后更具MD5加密（可以加点盐），全部大写，生成sign签名，这就是所说的url签名算法。然后登陆后每次调用用户信息时，带上sign，timestamp，token参数。
例如：
    
    原请求
    https://www.andy.cn/api/user/update/info.shtml?city=北京 （post和get都一样，对所有参数排序加密）
    
    加上时间戳和token
    https://www.andy.cn/api/user/update/info.shtml?city=北京×tamp=12445323134&token=wefkfjdskfjewfjkjfdfnc
    
    然后更具url参数生成sign
    https://www.andy.cn/api/user/update/info.shtml?city=北京×tamp=12445323134&token=wefkfjdskfjewfjkjfdfnc&sign=FDK2434JKJFD334FDF2
其最终的原理是减小明文的暴露次数；保证数据安全的访问。

#### 防止token被截获
之前可以用http的时候，用http签名+token现在只能用https，就简单了，ssl+tokenssl返回的token无法被监听和截获篡改（https相关安全不做讨论），那么token是安全的，与服务器交互用token，而不是uid，uid可以被猜测或离职人员可以通过这个渠道拉到其他用户信息，token就不行了，随机生成的，且有有效时间控制。
获取 token 之后，每次请求的所有参数根据根据一个算法然后 MD5 加密生成一个签名 sign，同样这个生成签名的算法 client secret 会参与进去，但是在请求的时候不传输。
RESTful 服务器端使用同样的算法计算签名，如果签名与请求的参数签名一致，则执行请求，否则返回错误信息。

[公钥、私钥 VS  数字签名  数字证书](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)


## 4、数据结构
[链表简单算法](http://blog.csdn.net/luckyxiaoqiang/article/details/7393134#topic1)
#### 4.1 链表和数组区别:

#### 4.2  将单链表反转
从头到尾遍历原链表，每遍历一个结点，将其摘下放在新链表的最前端。
#### 4.3 给定单链表头结点，删除链表中倒数第k个结点
**思路1：**先统计单链表中结点的个数，然后再找到第（n-k）个结点。注意链表为空，k为0，k为1，k大于链表中节点个数时的情况。时间复杂度为O（n）。

**思路2：**主要思路就是使用两个指针，先让前面的指针走到正向第k个结点，这样前后两个指针的距离差是k-1，之后前后两个指针一起向前走，前面的指针走到最后一个结点时，后面指针所指结点就是倒数第k个结点。

#### 4.4 查找单链表的中间结点
此题可应用于上一题类似的思想。也是设置两个指针，只不过这里是，两个指针同时向前走，前面的指针每次走两步，后面的指针每次走一步，前面的指针走到最后一个结点时，后面的指针所指结点就是中间结点，即第（n/2+1）个结点。注意链表为空，链表结点个数为1和2的情况。时间复杂度O（n）。

#### 4.5 给定单链表，检测是否有环。如果有环，则求出进入环的第一个节点。
如果一个链表中有环，也就是说用一个指针去遍历，是永远走不到头的。即定义一个快指针fast和一个慢指针slow，使得fast每次跳跃两个节点，slow每次跳跃一个节点。如果链表没有环的话，则slow与fast永远不会相遇（这里链表至少有两个节点）；如果有环，则fast与slow将会在环中相遇。

#### 4.6 判断两个单链表是否相交
如果两个链表相交于某一节点，那么在这个相交节点之后的所有节点都是两个链表所共有的。也就是说，如果两个链表相交，那么最后一个节点肯定是共有的。先遍历第一个链表，记住最后一个节点，然后遍历第二个链表，到最后一个节点时和第一个链表的最后一个节点做比较，如果相同，则相交，否则不相交。

# 三、iOS知识
目的：实际开发中遇到的问题
## 5、block
[参考文章 1](https://halfrost.com/ios_block/)
[参考文章 2](https://www.zybuluo.com/MicroCai/note/57603)

### 1、是什么
从定义描述上来说，block是**带有自动变量的匿名函数**。
从数据结构上来说，block是**Objective-C的对象**。

### 2、省略

一个标注的Block应该被定义为这样： 

    ^ 返回值类型 参数列表 表达式
    
    如： 
    ^ int (int a) { return a; };

但是我们有时候会省略一部分，比如

    省略掉返回值类型：
    ^ (int a) { a++; }

    省略掉返回值和参数列表：
     ^ { printf(“hello block”);}

我们知道Objective-C里面的方法都会具体在执行的时候转化为C语言的代码，比如：objc_msgSend

而其实Block也是如此。

### 3、源码

我们编写一段简单的Block代码，使用如下命令来转换：

`clang -rewrite-objc 源代码文件名`

源文件中代码如下：

    int main(int argc, const char * argv[]) {
   	    @autoreleasepool {
            void (^block)(void) = ^void(void){printf("Block");}; 
            block(); 
	    }
        return 0;
    }

转化之后得到代码深入分析，得知：
一个block实际上由两部分组成，一部分是struct `__block_impl` impl，一部分是struct `__main_block_desc_0`* Desc。


    struct __block_impl {
        void *isa;//所有对象都有isa指针，用于实现对象相关功能
        int Flags;//按bit位表示一些block的附加信息
        int Reserved;//今后版本升级所需的区域
        void *FuncPtr;//函数指针，指向具体block函数调用地址
    };__main_block_desc_0定义：


    static struct __main_block_desc_0 {
        size_t reserved;//今后版本升级所需的区域
        size_t Block_size;//Block的大小
    } __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};

### 4、block 的内存管理001

可以看到一个block, ***有一个isa的指针，而isa指针实际上就是id类型，所以我们说block是对象，block能支持copy能添加到数组或者字典中***。

![](https://ob6mci30g.qnssl.com/Blog/ArticleImage/21_4.png)
静态全局变量，全局变量由于作用域的原因，于是可以直接在Block里面被改变。他们也都存储在全局区。

静态变量传递给Block是内存地址值，所以能在Block里面直接改变值。
总结一下在Block中改变变量值有2种方式，一是传递内存地址指针到Block中，二是改变存储区方式(__block)。
block的类型:

    _NSConcreteGlobalBlock  //保存在数据区域（.data区），全局静态block，不访问任何外部变量
    _NSConcreteStackBlock   //保存在栈中的block，当函数返回时会被销毁
    _NSConcreteMallocBlock  //保存在堆中的block，当引用计数为0时被销毁

如果**不访问外部变量**，那么就是Global类型的，保存在数据区域（.data区），全局静态block，不访问任何外部变量
如果**访问外部变量**那么就是Stack的，保存在栈中，当函数返回时会被销毁

在以下情况会变成Malloc类型的，被复制到堆上，当引用计数为0时被销毁

    1、调用Block的copy实例方法
    2、被作为函数返回值返回
    3、被赋值给strong或者block类型的变量
    4、在方法名中含有usingBlock的Cocoa框架方法或者GCD的API中传递Block
### ##5、block 的内存管理002

在前文中，已经提到了 block 的三种类型 `_NSConcreteGlobalBlock`、`_NSConcreteStackBlock`、`_NSConcreteMallocBlock`，见名知意，可以看出三种 block 在内存中的分布
![](http://blogofzuoyebuluo.qiniudn.com/image_note57603_2.png)
### _NSConcreteGlobalBlock

    1、当 block 字面量写在全局作用域时，即为 global block；
    2、当 block 字面量不获取任何外部变量时，即为 global block；

除了上述描述的两种情况，其他形式创建的 block 均为 stack block。

    // 下面 block 虽然定义在 for 循环内，但符合第二种情况，所以也是 global block
    typedef int (^blk_t)(int);
    for (int rate = 0; rate < 10; ++rate) 
    {
        blk_t blk = ^(int count){return rate * count;}; 
    }

`_NSConcreteGlobalBlock` 类型的 block 处于内存的 ROData 段，此处没有局部变量的骚扰，运行不依赖上下文，内存管理也简单的多。
### _NSConcreteStackBlock

`_NSConcreteStackBlock` 类型的 block 处于内存的栈区。global block 由于处在 data 段，可以通过指针安全访问，但 stack block 处在内存栈区，如果其变量作用域结束，这个 block 就被废弃，block 上的 \_\_block 变量也同样会被废弃。
![](http://blogofzuoyebuluo.qiniudn.com/image_note57603_3.png)
为了解决这个问题，block 提供了 copy 的功能，将 block 和 \_\_block 变量从栈拷贝到堆，就是下面要说的 _NSConcreteMallocBlock。
### _NSConcreteMallocBlock

当 block 从栈拷贝到堆后，当栈上变量作用域结束时，仍然可以继续使用 block
![](http://blogofzuoyebuluo.qiniudn.com/image_note57603_4.png)
此时，堆上的 block 类型为 `_NSConcreteMallocBlock`，所以会将 `_NSConcreteMallocBlock` 写入 isa

    impl.isa = &_NSConcreteMallocBlock;

如果你细心的观察上面的转换后的代码，会发现访问结构体`__Block_byref_intValue_0` 内部的成员变量都是通过访问 `__forwarding` 指针完成的。为了保证能正确访问栈上的 __block 变量，进行 copy 操作时，会将栈上的 `__forwarding` 指针指向了堆上的 block 结构体实例。

### 6、对象复制指针，基本数据类型复制值
block 在调用 外部局部变量的时候 其实是将外部局部变量 copy了一份 使用的 所以在没有任何修饰符的时候是不可以修改外部局部变量的。

我们一个开始就说block是带有自动变量的匿名函数，其中***带有自动变量的意思是block会自动复制变量的值到block中***。

这样就是block中能访问到block外的值，但是不能改变，如果我们想要改变则需要为外部变量添加__block修饰符。

则：

    对于OC中的对象类型，因为变量是指针，在block内访问对象变量的时候实际上是
    复制了指针的值也就是地址到block中，所以对象类型不需要添加__block修饰符。

    对于基本数据类型，比如int，float，double，bool等,不使用__block的时候，
    block只会复制这些类型的值，如果需要改变这些变量是需要添加__block修饰符的。

### 7、基本数据类型（局部变量）复制值，Apple为什么要这样做？
    int main(int argc, char * argv[]) {
        int val = 1;
        void (^blk) (void) = ^{
            NSLog(@"%d",val);
        };
        blk();
        return 0;
    }
执行Block的时候怎么去访问val？ 
Block执行的时候，其实是执行 `__main_block_func_0`，参数是struct `__main_block_impl_0 *`\_\_cself，这个cself相当于C++中的this、OC中的self，然后通过\_\_cself->val访问。
其实理论上说可以通过指针来修改val这个变量，但是**main函数中的val和`__main_block_func_0`其实不在同一个作用域**，`__main_block_func_0`执行时，变量还在栈中，但是在很多情况下，Block是作为参数传递以供后续回调的，如果在这种情况下，**Block执行时，局部变量已经被释放了，在用指针指向的时候肯定就出错了， 所以局部变量不允许修改是合理的**。

### 8、为什么__block修饰符修饰的基本数据类型可以在Block中修改
block说明符（block storage-class-specifier）类似于static、auto说明符，它们用于指定变量值设置到哪个存储域，比如auto标识作为自动变量存储在栈区，static标识作为静态变量存储在.data区。
在main函数中将val的定义前加上__block
    
    int main(int argc, char * argv[]) {
    __block int val = 1;
    void (^blk) (void) = ^{
        val = 2;
    };
    blk();
    return 0;
    }
    
编译后看代码发现，block修饰的变量其实是一个`Block_byref_val_0`的实例，而且这个结构体持有相当于自动变量的成员变量。而且这个结构体也包含了forwarding持有指向该实例自身的指针。同时Block的main\_block\_impl\_0结构体实例持有指向block变量的Block\_byref\_val\_0的指针，通过forwarding访问修改成员变量val。


### 9、捕获外部变量
关于Block捕获外部变量有很多用途，用途也很广，只有弄清了捕获变量和持有的变量的概念以后，之后才能清楚的解决Block循环引用的问题。
使用了__block关键字的变量会将变量从栈上复制到堆上。栈上那个变量会指向复制到堆上的变量。
再次回到文章开头，5种变量，`自动变量，函数参数 ，静态变量，静态全局变量，全局变量`，如果严格的来说，捕获是必须在Block结构体`__main_block_impl_0` 里面有成员变量的话，Block能捕获的变量就只有带有自动变量和静态变量了。捕获进Block的对象会被Block持有。

对于非对象的变量来说，

自动变量的值，被copy进了Block，不带__block的自动变量只能在里面被访问，并不能改变值。

带__block的自动变量 和 静态变量 就是直接地址访问。所以在Block里面可以直接改变变量的值。

而剩下的静态全局变量，全局变量，函数参数，也是可以在直接在Block中改变变量值的，但是他们并没有变成Block结构体`__main_block_impl_0` 的成员变量，因为他们的作用域大，所以可以直接更改他们的值。

值得注意的是，静态全局变量，全局变量，函数参数他们并不会被Block持有，也就是说不会增加retainCount值。

对于对象来说，

在MRC环境下，__block根本不会对指针所指向的对象执行copy操作，而只是把指针进行的复制。 而在ARC环境下，对于声明为__block的外部对象，在block内部会进行retain，以至于在block环境内能安全的引用外部对象。对于没有声明__block的外部对象，在block中也会被retain。


### 10、循环引用
而在ARC环境下，对于声明为__block的外部对象，在block内部会进行retain，以至于在block环境内能安全的引用外部对象，所以才会产生循环引用的问题！

在ARC环境下，对于没有声明为__block的外部对象，也会被retain。

在ARC环境下，不仅仅是声明了__block的外部对象，没有加__block的对象，在block内部也会被retain。因为加了__block，只是对一个自动变量有影响，它们是指针， 相当于延长了指针变量的声明周期，只要访问对象的话还是会retain


当***block对象 作为类的 属性或者成员变量***，并且在block初始化的时候，调用了***self or self类的成员变量***。都会引起循环引用。

在ARC下，我们会将变量添加__weak修饰符来避免循环引用。

### 11、 __weak __strong

在上述使用 block中，虽说使用__weak，但是此处会有一个隐患，你不知道 self 什么时候会被释放，为了保证在block内不会被释放，我们添加__strong。

    __weak __typeof(self) weakSelf = self; 
    self.testBlock =  ^{
       __strong __typeof(weakSelf) strongSelf = weakSelf;
       [strongSelf test]; 
    });

PS： __strong 表示强引用，对应定义 property 时用到的 strong。当对象没有任何一个强引用指向它时，它才会被释放。

weakSelf是为了block不持有self，避免循环引用，而再声明一个strongSelf是因为一旦进入block执行，就不允许self在这个执行过程中释放。block执行完后这个strongSelf会自动释放，没有循环引用问题。



### 12、为什么 Objective-C 对象存储在堆上而不是栈上

#### 1、什么是栈对象和堆对象

在 Objective-C 中，对象通常是指一块有特定布局的连续内存区域。我们通常这样创建一个对象：

    NSObject *obj = [[NSObject alloc] init];

这行代码创建了一个 NSObject 类型的指针 obj 和一个 NSObject 类型的对象，obj 指针存储在栈上，而其指向的对象则存储在堆上（简称为堆对象）。

目前 Objective-C 并不支持直接在栈上创建对象（简称为堆对象），但可以通过如下方式间接地创建：

    struct {
    Class isa;
    } fakeNSObject;
    fakeNSObject.isa = [NSObject class];

    NSObject *obj = (NSObject *)&fakeNSObject;
    NSLog(@"%@", [obj description]);

栈对象 obj 也能正常工作，由此可见栈对象和堆对象都是可行的，但为什么 Objective-C 不默认使用栈对象呢？

#### 2、栈对象优缺点
###### 优点：

    1. 速度
    在栈上创建对象是非常快的，因为很多东西在编译时就确定了，运行时分配空间几乎不耗时；相对而言在堆上创建对象就非常耗时。

    2. 简单
    栈对象的生命周期是确定的，对象出栈以后就会被释放，不会存在内存泄漏，但这同时也是栈对象的最大缺点。
###### 缺点：

    1. 生命周期固定

    Objective-C 变量有效范围是由 “{}” 包含的块来决定的，也就是说栈对象的生命周期仅限于其所在的块里，出了块立马会被释放。
    一个对象被创建以后有可能会通过方法调用传递到别的方法，当栈对象的创建方法返回时，栈对象会被一起 pop 出栈而释放，导致其没法在别处被继续持有。
    此时 retain 操作会失效，除非用 copy 方法在想持有该栈对象的地方重新拷贝一份属于自己的栈对象。
    因此，栈对象回给对象的内存管理造成相当大的麻烦。

    2. 空间
    现代操作系统的栈和线程绑定，而栈空间是有限的，具体如下：

        512 KB (secondary threads)
        8   MB (OS X main thread)
        1   MB (iOS main thread)
    
    因此对象如果都在栈上创建不太现实，而堆只要物理内存不告警可以无限制使用。

#### 简略来说：
栈块存储在栈区，超出作用域则马上被销毁。
堆块存储在堆区中，是一个带引用计数的对象，需要自行管理其内存。

综合以上优缺点，Objective-C 选择用堆存储对象。

#### 3、程序的内存分配

一个由C/C++编译的程序占用的内存分为以下几个部分

- 栈区（stack）：由编译器自动分配释放 ，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。

- 堆区（heap）： 一般由程序员分配释放， 若程序员不释放，程序结束时可能由OS回收 。注意它与数据结构中的堆是两回事，分配方式倒是类似于链表。

- 全局区（静态区）（static）：全局变量和静态变量的存储是放在一块的，初始化的全局变量和静态变量在一块区域， 未初始化的全局变量和未初始化的静态变量在相邻的另一块区域。 程序结束后有系统释放。

- 文字常量区 ：常量字符串就是放在这里的。 程序结束后由系统释放。

- 程序代码区：存放函数体的二进制代码。

#### 4、block默认在栈上分配
Block对象与一般的类实例对象有所不同，一个主要的区别就是分配的位置不同，block默认在栈上分配，一般类的实例对象在堆上分配。

#### 5、为什么默认在栈上分配block？
### 答案：我不知道。

## 6、KVO  (key value obeserveing)

### 6.1 KVO 本质
(键值监听)
**定义::Key-Value Observing,它提供一种机制,当指定的对象的属性被修改后,则对象就会接受到通知。**

### 6.2 使用代码：

    # 把视图控制器注册为观察者来接收 KVO 的通知：
    - (void)addObserver:(NSObject *)anObserver
             forKeyPath:(NSString *)keyPath
                options:(NSKeyValueObservingOptions)options
                context:(void *)context

    # 在当 keyPath 的值改变的时候在观察者 anObserver（视图控制器） 上面被调用：
    - (void)observeValueForKeyPath:(NSString *)keyPath
                          ofObject:(id)object
                            change:(NSDictionary *)change
                           context:(void *)context

    # 必须记得调用以下的方法来移除观察者，否则我们我们的 app 会因为某些奇怪的原因崩溃：
    - (void)removeObserver:(NSObject *)anObserver
                forKeyPath:(NSString *)keyPath


### 6.3 KVO 原理：
1. 当你观察一个对象（称该对象为被观察对象）时，一个**继承**自被观察对象所对应类的子类会动态被创建。
2. 子类重写该被观察属性的setter方法：**在赋值语句前后加上相应的通知**（或曰方法调用）；
3. **把被观察对象的isa指针（isa指针告诉Runtime系统这个对象的类是什么）指向这个新创建的中间类，对象就神奇变成了新创建类的实例**。

### 6.4 KVO 实现
使用  `willChangeValueForKey`、`didChangeValueForKey`来通知变化

    - (void)setLComponent:(double)lComponent;
    {
        if (_lComponent == lComponent) {
            return;
        }
        [self willChangeValueForKey:@"lComponent"];
        _lComponent = lComponent;
        [self didChangeValueForKey:@"lComponent"];
    }

### 6.5 _A=5
**对属性A监听，则若用`self.A=5` 来赋值会收到通知；那么用`_A=5`赋值会改变吗？
不会，因为监听是重写setter方法，用`_A=5`直接操作实例变量，不走setter，则不会发通知。**

### 6.6 KVO 槽点
#### 1. 所有的observe处理都放在一个方法里
很容易写出一个特别长的方法。
#### 2. keyPath的类型是NSString，有可能写错keyPath。
KVO中的`keyPath`必须是`NSString`这个事实使得编译器没办法在编译阶段将错误的keyPath给找出来；
**解决：**使用`NSStringFromSelector(SEL aSelector)`方法，即改`@"contentSize"`为`NSStringFromSelector(@selector(contentSize))`。
#### 3. 多次相同的remove observer会导致crash
一般在dealloc中进行remove observer操作。
譬如，假设上述的`ZWTableViewController`的父类`UITableViewController`也对`tableView.contentSize`注册了相同的监听；那么`UITableViewController # dealloc`中常常会写出如下这样的代码：

    [_tableView removeObserver:self forKeyPath:@"contentSize" context:NULL];
按照一般习惯，`ZWTableViewController # dealloc`也会有相同的处理；那么当`ZWTableViewController`对象被释放时，`ZWTableViewController的dealloc`和其父类`UITableViewController`的`dealloc`都被调用，这样会导致相同的`remove observer`被执行两次，自然会导致crash。

**解决：**在`add observer`时为`context`参数设置一个独一无二的值即可，在`responding`处理时对这个`context`值进行检验。
解决了问题，但这需要靠用户（各个层级类的程序员用户）自觉遵守。 
**具体示例：**
在`addObserver`时候传入这个类唯一的 context。
    
    # 唯一的 context 写在此类 .m 文件的顶端
    static int const PrivateKVOContext;
    
    # addObserver时候传入：
    [otherObject addObserver:self forKeyPath:@"someKey" options:someOptions context:&PrivateKVOContext];
    
    # 然后在 observeValueForKeyPath:... 的方法中判断区分：
    - (void)observeValueForKeyPath:(NSString *)keyPath
                          ofObject:(id)object
                            change:(NSDictionary *)change
                           context:(void *)context
    {
        if (context == &PrivateKVOContext) {
            // 这里写相关的观察代码
        } else {
            [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
        }
    }


### 6.7 方法解析

#### 1) NSKeyValueObservingOptionInitial
我们常常需要当一个值改变的时候更新 UI，但是我们也要在第一次运行代码的时候更新一次 UI。
KVO 通知在调用 -addObserver:forKeyPath:... 到时候也被触发。
### 2) NSKeyValueObservingOptionPrior
我们在键值改变之前被通知。这和-willChangeValueForKey:被触发的时间相对应。
我们将会收到两个通知：一个在值变更前，另一个在变更之后。变更前的通知将会在 change 字典中有不同的键。我们可以像以下这样区分通知是在改变之前还是之后被触发的：
    
    if ([change[NSKeyValueChangeNotificationIsPriorKey] boolValue]) {
        // 改变之前
    } else {
        // 改变之后
    }
### 3) NSKeyValueObservingOptionNew、NSKeyValueObservingOptionOld
可以获得改变前后的值。
    
    id oldValue = change[NSKeyValueChangeOldKey];
    id newValue = change[NSKeyValueChangeNewKey];
通常来说 KVO 会在 -willChangeValueForKey: 和 -didChangeValueForKey: 被调用的时候存储相应键的值。

## 7、KVC (key value coding)
通过字符串的名字（key）来访问类属性的机制。而不是通过调用Setter、Getter方法访问。

### 7.1 KVC使用代码：    

    @property (nonatomic, copy) NSString *name;

    # 取值：
    NSString *n = [object valueForKey:@"name"]
    # 设定：
    [object setValue:@"Daniel" forKey:@"name"]

不仅可以访问对象属性，而且也能访问一些标量（例如 int 和 CGFloat）和 struct（例如 CGRect）。
    
    @property (nonatomic) CGFloat height;
    [object setValue:@(20) forKey:@"height"]

KVC 允许我们用属性的字符串名称来访问属性，字符串在这儿叫做键。

### 7.2 KVC 具体使用--简化列表 UI

假设我们有这样一个对象：

    @interface Contact : NSObject

    @property (nonatomic, copy) NSString *name;
    @property (nonatomic, copy) NSString *nickname;
    @property (nonatomic, copy) NSString *email;
    @property (nonatomic, copy) NSString *city;

    @end

还有一个 detail 视图控制器，含有四个对应的 UITextField 属性：

    @interface DetailViewController ()

    @property (weak, nonatomic) IBOutlet UITextField *nameField;
    @property (weak, nonatomic) IBOutlet UITextField *nicknameField;
    @property (weak, nonatomic) IBOutlet UITextField *emailField;
    @property (weak, nonatomic) IBOutlet UITextField *cityField;

    @end

我们可以简化更新 UI 的逻辑。

    # 返回 model 里我们用到的所有键的方法
    - (NSArray *)contactStringKeys;
    {
        return @[@"name", @"nickname", @"email", @"city"];
    }
    
    # 把键映射到对应的文本框的方法
    - (UITextField *)textFieldForModelKey:(NSString *)key;
    {
        return [self valueForKey:[key stringByAppendingString:@"Field"]];
    }

有了这个，我们可以从 model 里更新文本框，如下所示：

    - (void)updateTextFields;
    {
        for (NSString *key in self.contactStringKeys) {
            [self textFieldForModelKey:key].text = [self.contact valueForKey:key];
        }
    }

我们也可以用一个 action 方法让四个文本框都能实时更新 model：

    - (IBAction)fieldEditingDidEnd:(UITextField *)sender
    {
        for (NSString *key in self.contactStringKeys) {
            UITextField *field = [self textFieldForModelKey:key];
            if (field == sender) {
                [self.contact setValue:sender.text forKey:key];
                break;
            }
        }
    }
注意：我们之后会添加验证输入的部分，在键值验证里会提到。

最后，我们需要确认文本框在需要的时候被更新：

    - (void)viewWillAppear:(BOOL)animated;
    {
        [super viewWillAppear:animated];
        [self updateTextFields];
    }

    - (void)setContact:(Contact *)contact
    {
        _contact = contact;
        [self updateTextFields];
    }


### 7.3 键值验证 (KVV)
KVV 也是 KVC API 的一部分。这是一个用来验证属性值的 API，只是它光靠自己很难提供逻辑和功能。
如果我们写能够验证值的 model 类的话，我们就应该实现 KVV 的 API 来保证一致性。
**KVC 不会做任何的验证，也不会调用任何 KVV 的方法。
那是你的控制器需要做的事情。通过 KVV 实现你自己的验证方法会保证它们的一致性。**

#### 例子：
    
    - (IBAction)nameFieldEditingDidEnd:(UITextField *)sender;
    {
        NSString *name = [sender text];
        NSError *error = nil;
        if ([self.contact validateName:&name error:&error]) {
            self.contact.name = name;
        } else {
            // Present the error to the user
        }
        sender.text = self.contact.name;
    }

**它强大之处在于，当 model 类（Contact）验证 name 的时候，会有机会去处理名字。**

如果我们想让名字不要有前后的空白字符，我们应该把这些逻辑放在 model 对象里面。Contact 类可以像这样实现 KVV：
    
    - (BOOL)validateName:(NSString **)nameP error:(NSError * __autoreleasing *)error
    {
        if (*nameP == nil) {
            *nameP = @"";
            return YES;
        } else {
            *nameP = [*nameP stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]];
        return YES;
        }
    }
    
## 8、runtime  
### 8.1 消息发送
Objective-C是基于C语言加入了`面向对象`特性和`消息转发机制`的`动态语言`，这意味着它不仅需要一个编译器，还**需要Runtime系统来动态创建类和对象，进行消息发送和转发。**

使用  `[receiver message]`方法，编译器转化为:

    id objc_msgSend ( id self, SEL op, ... );
并不会马上执行`receiver`对象的`message`方法的代码，而是向`receiver`发送一条`message`消息；这条消息可能由`receiver`来处理，也可能由转发给其他对象来处理，也有可能假装没有接收到这条消息而没有处理。

[Runtime数据结构详情：isa、sel、id、class、Meta Class、Method、Ivar、IMP、Cache](http://www.jianshu.com/p/25a319aee33d)

#### 消息发送具体实现
- 首先根据`receiver`对象的`isa`指针获取它对应的`class`
- 优先在`class`的`cache`查找`message`方法，如果找不到，再到`methodLists`查找
- 如果没有在`class`找到，再到`super_class`查找
- 一旦找到`message`这个方法，就执行它实现的`IMP`。

![](https://github.com/Qiaomumer/Images/blob/master/runtime_objc_message.jpg?raw=true)


### 8.2 方法解析与消息转发

`[receiver message]`调用方法时，如果在`message`方法在`receiver`对象的类继承体系中没有找到方法，那怎么办？
一般情况下，程序在运行时就会Crash掉，抛出 `unrecognized selector sent to …`类似这样的异常信息。
但在抛出异常之前，还有三次机会按以下顺序让你拯救程序。
> Method Resolution
Fast Forwarding
Normal Forwarding

![](https://github.com/Qiaomumer/Images/blob/master/runtime_message_forward.jpg?raw=true)


#### `Method Resolution`
首先OC在运行时调用`+ resolveInstanceMethod:`或`+ resolveClassMethod:`方法，让你添加方法的实现。
如果你添加方法并返回YES，那系统在运行时就会重新启动一次消息发送的过程。
#### `Fast Forwarding`
如果目标对象实现`- forwardingTargetForSelector:`方法，系统就会在运行时调用这个方法，只要这个方法返回的不是nil或self，也会重启消息发送的过程，把这消息转发给其他对象来处理。否则，就会继续Normal Fowarding。
#### `Normal Forwarding`
如果没有使用`Fast Forwarding`来消息转发，最后只有使用`Normal Forwarding`来进行消息转发。它首先调用`methodSignatureForSelector:`方法来获取函数的参数和返回值，如果返回为nil，程序会Crash掉，并抛出`unrecognized selector sent to instance`异常信息。如果返回一个函数签名，系统就会创建一个`NSInvocation`对象并调用`-forwardInvocation:`方法。

#### 三种方法的选择

Runtime提供三种方式来将原来的方法实现代替掉，那该怎样选择它们呢？

- `Method Resolution`：由于`Method Resolution`不能像消息转发那样可以交给其他对象来处理，所以只适用于在原来的类中代替掉。
- `Fast Forwarding`：它可以将消息处理转发给其他对象，使用范围更广，不只是限于原来的对象。
- `Normal Forwarding`：它跟`Fast Forwarding`一样可以消息转发，但它能通过`NSInvocation`对象获取更多消息发送的信息，例如：`target、selector、arguments`和返回值等信息。

### 8.3 `Method Swizzling`
`Method Swizzling`就是在运行时将一个方法的实现代替为另一个方法的实现。如果能够利用好这个技巧，可以写出简洁、有效且维护性更好的代码。
[Method Swizzling 和 AOP 实践](http://tech.glowing.com/cn/method-swizzling-aop/)

### 8.4 动态语言 静态语言

#### 静态语言（强类型语言）
静态语言是在编译时变量的数据类型即可确定的语言，多数静态类型语言要求在使用变量之前必须声明数据类型。  例如：C++、Java、Delphi、C#等。

#### 动态语言（弱类型语言）
动态语言是在运行时确定数据类型的语言。变量使用之前不需要类型声明，通常变量的类型是被赋值的那个值的类型。  例如PHP/ASP/Ruby/Python/Perl/ABAP/SQL/JavaScript/Unix Shell等等。

#### 强类型定义语言
强制数据类型定义的语言。也就是说，一旦一个变量被指定了某个数据类型，如果不经过强制转换，那么它就永远是这个数据类型了。举个例子：如果你定义了一个整型变量a,那么程序根本不可能将a当作字符串类型处理。强类型定义语言是类型安全的语言。

#### 弱类型定义语言
数据类型可以被忽略的语言。它与强类型定义语言相反, 一个变量可以赋不同数据类型的值。强类型定义语言在速度上可能略逊色于弱类型定义语言，但是强类型定义语言带来的严谨性能够有效的避免许多错误。

### 动态 静态 区别

#### 特性
强类型语言是一旦变量的类型被确定，就不能转化的语言。  弱类型语言则反之，一个变量的类型是由其应用上下文确定的。

#### 静态语言的优势
1. 由于类型的强制声明，使得IDE有很强的代码感知能力，故，在实现复杂的业务逻辑、开发大型商业系统、以及那些生命周期很长的应用中，依托IDE对系统的开发很有保障；
2. 由于静态语言相对比较封闭，使得第三方开发包对代码的侵害性可以降到最低；

#### 动态语言的优势
1. 思维不受束缚，可以任意发挥，把更多的精力放在产品本身上；
2. 集中思考业务逻辑实现，思考过程即实现过程；


### 8.5 Swift是静态语言还是动态语言？
简单的看，Swift通过var和let声明变量和常量，不需要指定数据类型，非常像JavaScript等动态语言。
但是仔细学习可以发现，其实Swift是静态语言，而且是类型安全的静态语言，即使是Int和Double也需要显示转换。
那么不需要数据类型的声明其实就是语法糖了，是编译器做的类型推断，一旦类型确定就无法再改变了。所以Swift应该还是静态语言。




## 9、runloop

## 10、三方库  SDWebImage  缓存
### SDWebImage内部实现过程

1. 入口 `setImageWithURL:placeholderImage:options: `会先把 `placeholderImage` 显示，然后 `SDWebImageManager` 根据 `URL` 开始处理图片。
2. 进入 `SDWebImageManager # -downloadWithURL:delegate:options:userInfo:`，交给 `SDImageCache` 从缓存查找图片是否已经下载 `queryDiskCacheForKey:delegate:userInfo:`.
3. 先从内存图片缓存查找是否有图片，如果内存中已经有图片缓存，`SDImageCacheDelegate` 回调 `imageCache:didFindImage:forKey:userInfo:` 到 `SDWebImageManager`。
4. `SDWebImageManagerDelegate` 回调 `webImageManager:didFinishWithImage:` 到 `UIImageView+WebCache` 等前端展示图片。
5. 如果内存缓存中没有，生成 `NSInvocationOperation` 添加到队列开始从硬盘查找图片是否已经缓存。
6. 根据 `URLKey` 在硬盘缓存目录下尝试读取图片文件。这一步是在 `NSOperation` 进行的操作，所以回主线程进行结果回调 `notifyDelegate`:。
7. 如果上一操作从硬盘读取到了图片，将图片添加到内存缓存中（如果空闲内存过小，会先清空内存缓存）。`SDImageCacheDelegate` 回调 `imageCache:didFindImage:forKey:userInfo:`。进而回调展示图片。
8. 如果从硬盘缓存目录读取不到图片，说明所有缓存都不存在该图片，需要下载图片，回调 `imageCache:didNotFindImageForKey:userInfo:`。
9. 共享或重新生成一个下载器 `SDWebImageDownloader` 开始下载图片。
10. 图片下载由 `NSURLConnection` 来做，实现相关 `delegate` 来判断图片下载中、下载完成和下载失败。
11. `connection:didReceiveData:` 中利用 `ImageIO` 做了按图片下载进度加载效果。
12. `connectionDidFinishLoading:` 数据下载完成后交给 `SDWebImageDecoder` 做图片解码处理。
13. 图片解码处理在一个 `NSOperationQueue` 完成，不会拖慢主线程 UI。如果有需要对下载的图片进行二次处理，最好也在这里完成，效率会好很多。
14. 在主线程 `notifyDelegateOnMainThreadWithInfo:` 宣告解码完成，`imageDecoder:didFinishDecodingImage:userInfo:` 回调给 `SDWebImageDownloader`。
15. `imageDownloader:didFinishWithImage:` 回调给 `SDWebImageManager` 告知图片下载完成。
16. 通知所有的 `downloadDelegates` 下载完成，回调给需要的地方展示图片。
17. 将图片保存到 `SDImageCache` 中，内存缓存和硬盘缓存同时保存。写文件到硬盘也在以单独 `NSInvocationOperation` 完成，避免拖慢主线程。
18. `SDImageCache` 在初始化的时候会注册一些消息通知，在内存警告或退到后台的时候清理内存图片缓存，应用结束的时候清理过期图片。
19. `SDWI` 也提供了 `UIButton+WebCache` 和 `MKAnnotationView+WebCache`，方便使用。
20. `SDWebImagePrefetcher` 可以预先下载图片，方便后续使用。


## 11、优化app



# 四、综合技术
目的：考察知识储备，将各个知识点融会贯通，相互联系起来解决问题。
## 12、点击登录按钮，都发生了什么？

- 响应链
- 事件分发
- 动态语言
- 密码验证
- http-s tcp 参数、body、签名、加密
- dns
- 数据解析 Json-Model
- 数据显示VM？

# 五、发散思维
目的：逻辑力，压力下思维灵敏性，随机应变能力，解决问题能力。
## 13、构造一个字典
#### 13.1 可变数组
key、value放在两个数组中，index相同。查询到key，则可以获取到value。
#### 13.2 内存地址对应（类似于数组）
最简单是将 key，value 放在一起，线性排。
    
    k1, k2, k3, k4, k4 ....
    v1, v2, v3, v4, v5 ....
当需要从 key 找到对应的 value 时，就从头到尾遍历过去。依次判断 k1, k2, k3, k4 是不是等于key, 当等于的时候，就找到 key 的具体位置，从而也就找到了value。但这样从头到尾遍历，速度就太慢了，时间复杂度为 O(N)。N为数据的大小。为了快速从key找到value。dictionary(或者说map)的通常有两种实现方式。

#### 13.3 链表 
两个指针，一个指value，一个指next

#### 13.4 哈希 
**map.put：**我们将K/V传给put方法时，它调用hashCode计算hash从而得到bucket位置。若冲突，链表解决。
**map.get：**我们将K传给get，它调用hashCode计算hash从而得到bucket位置，若链表，进一步调用equals(k)方法确定结果。
**如果两个键的hashcode相同，因为存储的是键值对，遍历链表+对比键值对来找到结果。**
![HashMap结构](https://github.com/Qiaomumer/Images/blob/master/HashMap%E7%BB%93%E6%9E%84.png?raw=true)

#### 13.5 二叉树
二叉树查找的时间复杂度为 O(logN)，
#### 13.6 字典树



## 14、桥问题：


## 15、音视频
推流，把采集阶段封包好的内容传输到服务器的过程。
拉流：服务器已有直播内容，用指定地址进行拉取的过程。

目前的移动直播架构
![目前的移动直播架构](https://github.com/Qiaomumer/Images/blob/master/liveStreaming.png?raw=true)

选择协议
推流端的协议有RTMP, WebRTC和基于UDP的私有协议。
#### RTMP
RTMP是基于TCP的标准协议，CDN网络普遍支持，也能做到相对比较低的延迟。
即构科技的互动直播技术在推流端使用RTMP协议，拉流端兼容三种协议：RTMP，HLS和FLV。
HLS协议的延迟比较大，在需要进行连麦互动的场景下，不应该使用HLS协议。

#### HLS
HLSHttp Live Streaming是由Apple公司定义的基于HTTP的流媒体实时传输协议。
它的原理是将整个流分为多个小的文件来下载，每次只下载若干个。服务器端会将最新的直播数据生成新的小文件，客户端只要不停的按顺序播放从服务器获取到的文件，就实现了直播。基本上，HLS是以点播的技术实现了直播的体验。因为每个小文件的时长很短，客户端可以很快地切换码率，以适应不同带宽条件下的播放。分段推送的技术特点，决定了HLS的延迟一般会高于普通的流媒体直播协议。传输内容包括两部分：一是M3U8描述文件，二是TS媒体文件。TS媒体文件中的视频必须是H264编码，音频必须是AAC或MP3编码。由于数据通过HTTP协议传输，所以完全不用考虑防火墙或者代理的问题，而且分段文件的时长很短，不过HLS的

#### WebRTC
WebRTC的好处在于用户体验好，不需要安装东西，分享一个链接就可以看。
但是它有一个缺点，就是WebRTC是Google推的一项技术，除了Google Chrome和Opera支持WebRTC，其他浏览器大部分不支持WebRTC。换一句话说，40%的浏览器支持WebRTC，剩下60%浏览器不支持，所以适用范围就比较局限。
然后，在中国国内，WebRTC在Google Chrome上的表现也大打折扣。最后，因为浏览器没有开放核心的能力，所以在浏览器上运行的协议比较难以做到比较低的延迟。

#### UDP
基于UDP的私有协议十分适合做实时音视频系统，它是面向无连接的，避免了TCP做网络质量控制所需要的开销，能够做到比较低的延迟。但是它也有一个缺点，那就是**私有协议的兼容性不好**。
CDN支持标准的RTMP协议，但是不支持基于UDP的私有协议。为了吸纳UDP的优点，而避免UDP的缺点，即构科技的互动直播技术采用了基于UDP的私有协议作为补充，在有必要的时候用来弥补RTMP协议的不足。比如说，只有在网络环境比较恶劣或者在跨国互通的情况下，才使用基于UDP的私有协议；比如说，只在推流端到媒体服务器这一段才使用基于UDP的私有协议，而从媒体服务器转推流到CDN网络这一段采用RTMP协议，在这两段之间通过把UDP私有协议转换成RTMP协议来进行适配和衔接。这样一来，即构科技的直播方案既拥有超低延迟的优势，又保留了标准协议普遍被CDN网络支持的好处。



####     缓冲自适应

由于有**网络抖动的存在，数据包的到达不是匀速的。**最直接的降低延迟的方法就是把缓冲队列的长度设置为零，接收到什么数据包就直接渲染什么数据包，然而这样做的后果就是播放不流畅，会出现卡顿。

`延迟`和`流畅`两者本身就是一对矛盾的因素。
**平衡 低延迟、流畅 --> 建立缓冲队列。**

1. 在拉流端和混流服务器都需要建立缓冲队列。
对于一个实时系统来说，缓冲队列的长度必须不是固定的，而是**自适应**：
**当网络很好的时候，缓冲队列的长度就会变得比较短，接近零，甚至为零；
当网络不好的情况下，缓冲队列的长度会变得比较长，但是不能超过能接受的上限，毕竟缓冲队列的长度本质上就是延迟的时间。**

2. `缓冲自适应` + `快播` + `慢播` 
当网络由差转好的情况下，可以适当的播得快一点，尽快缩短缓冲队列的长度。
当网络由好转差的情况下，可以适当的播得慢一点，让缓冲队列适当变长，保持流畅性。
快播和慢播是结合观众的心理学模型，在适合快播和慢播的条件下采用，让观众没有觉察出播放速度的变化，同时整体感觉也显得既流畅又低延迟。

### 14、weak 的作用
[weak底层实现](http://www.cocoachina.com/ios/20170328/18962.html)
weak 关键字的作用弱引用，所引用对象的计数器不会加一，并在引用对象被释放的时候自动被设置为 nil。

#### 现在我们将 weak 的思路整理一下：

整个系统中存在很多个对象，这些对象都可能会被弱引用，那么我们需要一个容器来容纳这些被弱引用的对象，比如数组，在此将这个容器的数据结构标识为 objectContainerDataStructure；

一个对象可能会被多次弱引用，当这个对象被销毁时，我们需要找到这个对象的所有弱引用，所以我们需要将这些弱引用的地址（即指针）放在一个容器里，比如数组，在此将这些弱引用的地址的数据结构标识为 pointerContainerDataStructure；

当对象不再被强引用时需要销毁的时候，我们需要通过这个对象在 objectContainerDataStructure 找到其对应的 pointerContainerDataStructure，进而找到这个对象的所有弱引用，将其置为 nil，

####     通过上面的步骤，我们大概可以得出这么一个数据结构：

pointerContainerDataStructure 仅仅只是容纳一个对象的所有弱引用的地址，所以用数组即可；

objectContainerDataStructure 是一个 key-value 数据结构，将对象作为 key，对象的内存地址是最好的选择；

在 iOS 中常用的 key-value 数据结构就是字典 Dictionary ，在这里我们的 key 是一个数值对象，value 则是一个数值数组对象，可以用哈希表实现；

####     总结

为了实现 weak，我们需要这样的一张弱引用表：

表的数据结构是哈希表；

表的 key 是对象的内存地址；

value 是指向该对象的所有弱引用的指针；


### 15、常用的底层的实现，而不是简单的应用