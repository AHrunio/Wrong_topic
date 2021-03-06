# 面试重点题、错题集

⚠本错题集回答为个人见解仅供参考

+ epoll 过程

> epoll使用epoll_create()在内核创建epoll对象，内核中使用红黑树来跟踪所有待检测文件描述符。把监控的socket通过epoll_ctl()加入到内核红黑树，不用每次传一个集合，只传入一个待检测socket。
> 内核中epoll使用事件驱动系统，维护一个链表记录就绪事件。当某socket有事件时通过回调函数将其加入就绪事件链表，当某用户调用epoll_wait()，只返回有网络事件的socket的个数，而不是轮询扫描fd集合

+ LT和ET触发模式

> 水平触发（LT，Level Trigger）模式下，只要一个文件描述符就绪，就会触发通知，如果用户程序没有一次性把数据读写完，下次还会通知；
> 边缘触发（ET，Edge Trigger）模式下，当描述符从未就绪变为就绪时通知一次，之后不会再通知，直到再次从未就绪变为就绪（缓冲区从不可读/写变为可读/写）。
> 区别：边缘触发效率更高，减少了被重复触发的次数，函数不会返回大量用户程序可能不需要的文件描述符。
> 为什么边缘触发一定要用非阻塞（non-block）IO：避免由于一个描述符的阻塞读/阻塞写操作让处理其它描述符的任务出现饥饿状态。

+ MySQL了解过吗？ 讲一下索引的底层

> 索引底层用b+树实现，MyISAM引擎使用B+Tree作为索引结构，叶节点的data域存放的是数据记录的地址。MyISAM的索引文件仅仅保存数据记录的地址。在MyISAM中，主索引和辅助索引（Secondary key）在结构上没有任何区别，只是主索引要求key是唯一的，而辅助索引的key可以重复。
> InnoDB也使用B+Tree作为索引结构，但具体实现方式却与MyISAM截然不同。
>
> 第一个重大区别是InnoDB的数据文件本身就是索引文件。MyISAM索引文件和数据文件是分离的，索引文件仅保存数据记录的地址。而在InnoDB中，表数据文件本身就是按B+Tree组织的一个索引结构，这棵树的叶节点data域保存了完整的数据记录。这个索引的key是数据表的主键，因此InnoDB表数据文件本身就是主索引。InnoDB的辅助索引data域存储相应记录主键的值而不是地址。换句话说，InnoDB的所有辅助索引都引用主键作为data域。聚集索引这种实现方式使得按主键的搜索十分高效，但是辅助索引搜索需要检索两遍索引：首先检索辅助索引获得主键，然后用主键到主索引中检索获得记录。

+ b+树 时间复杂度

> 层数log(m)n，每一层平均m/2（每一层平均都要遍历m/2个节点才能确定往下面哪一个分支走），所以一共是(m/2) * log(m)n，由于m是常数，所以复杂度O(log n)，即以2为底n的对数。

+ 哈希索引和b+树索引的区别

> 哈希索引能以 O(1) 时间进行查找，但是只支持精确查找，无法用于部分查找和范围查找，无法用于排序与分组；B树索引支持大于小于等于查找，范围查找。哈希索引遇到大量哈希值相等的情况后查找效率会降低。哈希索引不支持数据的排序

+ shared_ptr缺点

> 智能指针可能出现的问题：循环引用，在两个类中分别定义另一个类的对象的共享指针，由于在程序结束后，两个指针相互指向对方的内存空间，导致内存无法释放。

+ string底层分配空间

> 第一次申请的时候就申请一块大一些的内存，比如初始化的字符串长度为10，就申请一块大小为20的内存，如果这块内存用完了而字符串还需要扩展，那就去找一块更大的内存，能够同时容纳需要的内存空间，而且还有一些余量，直接将字符串整体迁移到新内存空间中，放弃原来那部分内存空间，这样就实现了字符串的内存连续。

+ SSL握手流程

> 客户端向服务器发送请求，同时发送客户端支持的一套加密规则（包括对称加密、非对称加密、摘要算法）；
> 服务器从中选出一组加密算法与HASH算法，并将自己的身份信息以证书的形式发回给浏览器。证书里面包含了网站地址，加密公钥（用于非对称加密），以及证书的颁发机构等信息（证书中的私钥只能用于服务器端进行解密）；
> 客户端验证服务器的合法性，包括：证书是否过期，CA（证书颁发机构） 是否可靠，发行者证书的公钥能否正确解开服务器证书的“发行者的数字签名”，服务器证书上的域名是否和服务器的实际域名相匹配；
> 如果证书受信任，或者用户接收了不受信任的证书，浏览器会生成一个随机密钥（用于对称算法），并用服务器提供的公钥加密（采用非对称算法对密钥加密）；使用Hash算法对握手消息进行摘要计算，并对摘要使用之前产生的密钥加密（对称算法）；将加密后的随机密钥和摘要一起发送给服务器；
> 服务器使用自己的私钥解密，得到对称加密的密钥，用这个密钥解密出Hash摘要值，并验证握手消息是否一致；如果一致，服务器使用对称加密的密钥加密握手消息发给浏览器；
> 浏览器解密并验证摘要，若一致，则握手结束。之后的数据传送都使用对称加密的密钥进行加密

+ 利用二分法求一个数的平方根,精度要求 e < 10^-6

> ```cc
> #include <math.h> 
> 
> double sqrt(double x )
> {
>  	double l=0,r=x;
>      while(1){
>          auto m=l+(r-l)/2;
>          if(fabs(m-x/m )<=1e-6)return m;
>           else if(m<x/m)l=m+1;
>           else r=m-1;
>       }
>  }
>  ```

+ 如何只用2GB内存从20/40/80亿个整数中找到出现次数最多的数?

> 可以采用哈希表来统计，把这个数作为 key，把这个数出现的次数作为 value，之后我再遍历哈希表哪个数出现最多的次数。
> key 和 value 都是 int 型整数，一个 int 型占用 4B 的内存，所以哈希表的一条记录需要占用 8B，最坏的情况下，这 20 亿个数都是不同的数，大概会占用 16GB 的内存.
> 可以把这 20 亿个数映射到不同的文件中去，例如，数值在 0 至 2亿之间的存放在文件1中，数值在2亿至4亿之间的存放在文件2中….，由于 int 型整数大概有 42 亿个不同的数，所以我可以把他们映射到 21 个文件中去.

+ 如果我给的这 20 亿个数数值比较集中的话，例如都处于 1~20000000 之间，那么你都会把他们全部映射到同一个文件中，你有优化思路吗？

> 可以先把每个数先做哈希函数映射，根据哈希函数得到的哈希值，再把他们存放到对应的文件中，如果哈希函数设计到好的话，那么这些数就会分布的比较平均。

+ 那如果我把 20 亿个数加到 40 亿个数呢？

> 可以加大文件的数量

+ 如果我给的这 40 亿个数中数值都是一样的，那么你的哈希表中，某个 key 的 value 存放的数值就会是 40 亿，然而 int 的最大数值是 21 亿左右，那么就会出现溢出，你该怎么办？

> 我可以把 value 初始值赋值为 负21亿，这样，如果 value 的数值是 21 亿的话，就代表某个 key 出现了 42 亿次了。

+ 那我如果把 40 亿增加到 80 亿呢？

> 一边遍历一遍判断啊，如果我在统计的过 程中，发现某个 key 出现的次数超过了 40 亿次，那么，就不可能再有另外一个 key 出现的次数比它多了，那我直接把这个 key 返回。

+ free和delete的区别

> 1. delete 用于释放 new 分配的空间，free 有用释放 malloc 分配的空间
>
> 2. delete [] 用于释放 new [] 分配的空间
>
> 3. delete 释放空间的时候会调用 相应对象的析构函数
>
>     顺便说一下new在分配空间的时候同时会调用对象的构造函数，对对象进行初始化，使用malloc则只是分配内存
>
> 4. 调用free 之前需要检查 需要释放的指针是否为空，使用delete 释放内存则不需要检查指针是否为NULL
>
> 5. malloc、free 是库函数，而new、delete 是关键字。 new 申请空间时，无需指定分配空间的大小，编译器会根据类型自行计算；malloc 在申请空间时，需要确定所申请空间的大小。

+ GET与POST的区别？

> GET是幂等的，即读取同一个资源，总是得到相同的数据，POST不是幂等的；
>
> 幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同
>
> GET一般用于从服务器获取资源，而POST有可能改变服务器上的资源；
>
> 请求形式上：GET请求的数据附在URL之后，在HTTP请求头中；POST请求的数据在请求体中；
>
> 安全性：GET请求可被缓存、收藏、保留到历史记录，且其请求数据明文出现在URL中。POST的参数不会被保存，安全性相对较高；
>
> GET只允许ASCII字符，POST对数据类型没有要求，也允许二进制数据；		
>
> GET的长度有限制（操作系统或者浏览器），而POST数据大小无限制

+ post的数据很大怎么办

> 原则上post是默认无限制的，多大的数据都可以请求，
> 但是springboot内置的tomcat服务器，对这post请求做了默认限制，
> maxPostsize为2m；
>
> Tomcat的配置文件里取消POST大小限制，在conf目录下，server.xml文件，修改：
>
> ```
> <Connector port="8080" protocol="HTTP/1.1"  
>       connectionTimeout="20000"  
>       redirectPort="8443" maxPostSize="0"/>  
> ```
>
> maxPostSize=”0”,即取消POST的大小限制；

+ 如何传输大文件

> 数据压缩
> 浏览器在发送请求时都会带着 Accept-Encoding 头字段，里面是浏览器支持的压缩格式列表，例如 gzip、deflate、br 等，这样服务器就可以从中选择一种压缩算法，放进 Content-Encoding 响应头里，再把原数据压缩后发给浏览器。
>
> 分块传输
> 除了压缩文件之外，另一种办法就是分块传输。它们的原理差不多，都是把大文件变小传输。分块传输会把一个大文件切成很多小块，把这些小块依次发给浏览器，浏览器收到之后再组装复原。这样浏览器和服务器都不用在内存中保存全部文件，每次只收发一小部分，网络也不会被大文件长时间占用，内存、带宽等资源也就节省下来了。
>
> 具体实现是在 response 响应报文里用头字段 Transfer-Encoding: chunked 来表示，表示报文里的 body 部分不是一次性发过来的，而是分成了许多的块（chunk）逐个发送。当 chunk 为 0 时说明是最后一个，传输结束。
>
> 范围请求
> 为什么会有范围请求？
>
> 你看电影时，想跳过开头直接看正片，这实际上是想获取一个大文件其中的片段数据，而分块传输没有这个能力。
>
> HTTP 协议为了满足这种需求，提出了「范围请求」的概念，允许客户端在请求头里使用专用字段来表示只获取文件的一部分。
>
> 范围请求不是 Web 服务器必须实现的功能，所以服务器必须在响应头里使用字段 「Accept-Ranges: bytes 」明确告知客户端自己支持范围请求。如果不支持的话，服务器就会发送「Accept-Ranges：none」或者不发送此字段。这样客户端就只能收发整块文件了。

+ 堆是平衡二叉树吗

> 平衡二叉树（Balanced Binary Tree）又被称为AVL树（有别于AVL算法），且具有以下性质：它是一 棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。这个方案很好的解决了二叉查找树退化成链表的问题，把插入，查找，删除的时间复杂度最好情况和最坏情况都维持在O(logN)
>
> 1 堆是一种完全二叉树（不是平衡二叉树，也不是二分搜索树哦）
> 2 堆要求孩子节点要小于等于父亲节点（如果是最小堆则大于等于其父亲节点）

+ 图的存储结构

> 1. 邻接矩阵
>
>    ![image-20210809233547688](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210809233547688.png)
>
>    2. 邻接表
>
>       ![image-20210809233622872](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210809233622872.png)
>
>       3. 十字链表
>       4. 多重邻接表

+ MySQL 的Join及底层实现原理

> MySQL是只支持一种JOIN算法Nested-Loop Join（嵌套循环链接），不过MySQL的Nested-Loop Join（嵌套循环链接）也是有很多变种，能够帮助MySQL更高效的执行JOIN操作：
>
> （1）Simple Nested-Loop Join
>
> ![image-20210809233857623](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210809233857623.png)
>
> 从驱动表中取出R1匹配S表所有列，然后R2，R3,直到将R表中的所有数据匹配完，然后合并数据，可以看到这种算法要对S表进行RN次访问，虽然简单，但是相对来说开销还是太大了
>
> （2）Index Nested-Loop Join
>
> ![image-20210809234051786](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210809234051786.png)
>
> 索引嵌套联系由于非驱动表上有索引，所以比较的时候不再需要一条条记录进行比较，而可以通过索引来减少比较，从而加速查询。
>
> 这种算法在链接查询的时候，驱动表会根据关联字段的索引进行查找，当在索引上找到了符合的值，再回表进行查询，也就是只有当匹配到索引以后才会进行回表。至于驱动表的选择，MySQL优化器一般情况下是会选择记录数少的作为驱动表，但是当SQL特别复杂的时候不排除会出现错误选择。（ON后面写索引的字段就完事了）
>
> 在索引嵌套链接的方式下，如果非驱动表的关联键是主键的话，这样来说性能就会非常的高，如果不是主键的话，关联起来如果返回的行数很多的话，效率就会特别的低，因为要多次的回表操作。
>
> （3）Block Nested-Loop Join
>
> ![image-20210809234240285](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210809234240285.png)
>
> 在有索引的情况下，MySQL会尝试去使用Index Nested-Loop Join算法，在有些情况下，可能Join的列就是没有索引，那么这时MySQL的选择绝对不会是最先介绍的Simple Nested-Loop Join算法，而是会优先使用Block Nested-Loop Join的算法。
>
> Block Nested-Loop Join对比Simple Nested-Loop Join多了一个中间处理的过程，也就是join buffer，使用join buffer将驱动表的查询JOIN相关列都给缓冲到了JOIN BUFFER当中，然后批量与非驱动表进行比较，这也来实现的话，可以将多次比较合并到一次，降低了非驱动表的访问频率。也就是只需要访问一次S表。这样来说的话，就不会出现多次访问非驱动表的情况了，也只有这种情况下才会访问join buffer。
>
> 在MySQL当中，我们可以通过参数join_buffer_size来设置join buffer的值，然后再进行操作。默认情况下join_buffer_size=256K，在查找的时候MySQL会将所有的需要的列缓存到join buffer当中，包括select的列，而不是仅仅只缓存关联列。

+ 说说awk 命令

> ## 1、awk 的基本用法
>
> awk 是以文件的一行为处理单位的，awk每接收文件的一行，就执行相应的命令。
>
> **1、基本命令格式：**
>
> ```bash
> awk '{pattern + action}' <file>
> ```
>
> 其中，pattern表示在数据中要查找的内容，action表示要执行的一系列命令。
>
> awk 通过指定分隔符，将一行分为多个字段，依次用 $1、$2 ... $n 表示第一个字段、第二个字段... 第n个字段。比如有一log文件，若只想获取 vel、acc、steer 的值，则可以通过下面的命令：
>
> ```bash
> awk '{print $2, $4, $6}' log
> ```
>
> ![image-20210809234432764](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210809234432764.png)
>
> **2、awk 的分隔符**
>
> awk的默认分隔符是空格和制表符，上面的例子中，若希望把逗号去掉，则可以使用 -F 参数来指定分隔符，命令如下：
>
> ```bash
> awk -F ':|,' '{print $2, $4, $6}' log
> ```
>
> 这里指定冒号（:）和逗号（,）同时作为分隔符。
>
> ![image-20210809234518071](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210809234518071.png)
>
> **3、awk 的内置变量**
>
> 除了 $1、$2 ... $n，awk 还有一些内置变量，常用的如下：
>
> - $0：表示当前整行，$1表示第一个字段，$2表示第二个字段，$n 表示第n个字段；
> - NR：表示当前已读的行数；
> - NF：表示当前行被分割的列数，NF表示最后一个字段，NF-1 表示倒数第二个字段；
> - FILENAME：表示当前文件的名称
>
> 如下图所示，在每一行前加上文件名、行号、每行列数，命令如下：
>
> ```bash
> awk '{print FILENAME, NR, NF, ":", $0}' log
> ```
>
> ![image-20210809234625631](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210809234625631.png)
>
> **4、条件判断**
>
> awk 的 pattern 也支持使用条件判断，比如只打印 vel 小于 5.0 的行，命令如下：
>
> ```bash
> awk '$2 < 5.0 {print $0}' log
> ```
>
> ![image-20210809234711426](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210809234711426.png)

+ 同一进程中的线程可以共享哪些数据？

>
> 进程代码段
>
> 进程的公有数据（全局变量、静态变量...）
>
> 进程打开的文件描述符
>
> 文件描述符是一个简单的整数，用以 标明每一个被进程所打开的文件和socket。第一个打开的文件是0，第二个是1，依此类推。
>
> 进程的当前目录
>
> 信号处理器/信号处理函数：对收到的信号的处理方式
>
> 进程ID与进程组ID

+ 线程独占哪些资源？

> 线程ID
>
> 一组寄存器的值
>
> 线程的栈（堆是共享的）
>
> 错误返回码：线程可能会产生不同的错误返回码，一个线程的错误返回码不应该被其它线程修改；
>
> 信号掩码/信号屏蔽字(Signal mask)：表示是否屏蔽/阻塞相应的信号（SIGKILL,SIGSTOP除外）

+ 什么是协程？

> 协程是一种用户态的轻量级线程，协程的调度完全由用户控制。协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量，所以上下文的切换非常快。
>
> 协程多与线程进行比较：
> 一个线程可以拥有多个协程，一个进程也可以单独拥有多个协程
> 线程进程都是同步机制，而协程则是异步
> 协程能保留上一次调用时的状态，每次过程重入时，就相当于进入上一次调用的状态

+ 进程的内存分布

>  自底向上的方式进行讲解：
>
>   1. 代码段：主要是程序的代码以及编译时静态链接进来的库。这段内存大小在程序运行之前就已经确定，而且是只读，可能存在一些常量，比如字符串常量。
>
>   2. 数据段：分为data和bss两个段，表现为静态内存段，data段存放已初始化的全局变量（静态内存分配的变量和初始化全局变量）。bss段存放未初始化的全局变量，在内存中bss段被清零。
>
>   3. 堆  段：用于程序动态内存分配和管理，如何分配和管理由程序的开发者决定，大小不固定（跟您的机器内存有关系），可以动态伸缩。
>
>   4. 映射段：该内存区域存放链接其它动态程序库的向量，共享内存映射向量等等。
>
>   5. 栈  段：栈是一种先进后出的数据结构，该段内存区域由程序在运行中自行管理，如：局部变量保存和撤除、函数调用相关等。
>
>   6. 输入的环境变量和参数段：主要内存程序执行时的环境变量，输入参数等等。
>
>   7. 就是系统区域。
>
>      <img src="https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210809234932908.png" alt="image-20210809234932908" style="zoom:50%;" />

+ 异步日志如何实现？

> 异步读写
> 异步是利用队列来做
>
> 使用队列的好处：
> 解耦，这样每个模块独立，互补影响
> 提高性能；每个模块都没有了写文件的损耗，所有写文件的损耗都由日志模块来承担。
> 每个模块都将日志写入队列，不关心写入成功还是失败；创建线程专门用于读取队列中的日志信息，进行写日志文件
> 实现使用的队列使STL中的queue为底层，使用单例模式确保全局只有唯一的一个对象，保证使用相同的队列；
> 使用开源库log4cplus作为读写日志库

+ 如何解决多并发的问题？


> 1、负载均衡
> 负载均衡将是大型网站解决高负荷访问和大量并发请求采用的终极解决办法。
> （1）单个重负载的运算分担到多台节点设备上做并行处理，每个节点设备处理结束后，将结果汇总，返回给用户，系统处理能力得到大幅度提高.
> （2）大量的并发访问或数据流量分担到多台节点设备上分别处理，减少用户等待响应的时间
> 2、数据库集群
> 就是利用至少两台或者多台数据库服务器，构成一个虚拟单一数据库逻辑映像，像单数据库系统那样，向客户端提供透明的数据服务。
>
> 3、库表散列
> 采用Hash算法把数据分散到各个分表中, 这样IO更加均衡。
> 4、图片服务器分离
> 大家知道，对于Web服务器来说，不管是Apache、IIS还是其他容器，图片是最消耗资源的，于是我们有必要将图片与页面进行分离，这是基本上大型网站都会采用的策略，他们都有独立的图片服务器，甚至很多台图片服务器。这样的架构可以降低提供页面访问请求的服务器系统压力
> 5、镜像
> 镜像是大型网站常采用的提高性能和数据安全性的方式，镜像的技术可以解决不同网络接入商和地域带来的用户访问速度差异，数据进行定时更新或者实时更新。每当主数据库更新时，DBMS会自动把更新后的数据复制过去，即DBMS自动保证镜像数据与主数据的一致性。
>
> 6、缓存
> Apache提供了自己的缓存模块
>
> 7、HTML静态化
> 静态化的html页面效率最高、消耗最小，所以我们可以尽可能使我们的网站上的页面采用静态页面。
>
> 8、CDN加速技术
> CDN的全称是内容分发网络。其是通过在现有的Internet中增加一层新的网络架构，将网站的内容发布到最接近用户的网络“边缘”，使用户可以就近取得所需的内容，提高用户访问网站的响应速度。

+ 多进程如何实现，资源如何回收

> 实现多进程，就是用fork函数来创建子进程。

+ fork继承了什么？又没继承什么？


> 继承：uid，gid，进程组id，会话id，当前工作目录和根目录，文件描述符fd，信号屏蔽和处理，存储映射，资源限制 等。
> 没继承：pid，进程时间，文件锁，未处理信号等

+ 什么是僵尸进程？

> 如果一个子进程结束，但是它的父进程没有调用wait，那么这个子进程就会变成僵尸进程。
>
> 如果子进程先退出，父进程还未退出，此时子进程必须等到父进程捕获到子进程的退出状态后，子进程才能真正结束，否则，此时的子进程就会变成僵尸进程。
> 如果父进程先退出，子进程还未退出，此时子进程的父进程会变为init（pid=1）进程，init进程在子进程结束时候，会默认执行wait操作，所以该子进程一定不会变成将僵尸进程。（备注：任何一个进程都必须有父进程）

+ 如何避免僵尸进程？

> 第一种方法：就是在父进程用wait或者waitpid来回收子进程资源，即可避免僵尸进程。
> 第二种方法：子进程退出会发送SIGCHLD信号给父进程，在父进程上，可以选择忽略或者使用信号处理函数接收处理它，就可以避免僵尸进程。

+ wait和waitpid

> wait和waitpid的作用是为了获得子进程状态改变的信息，更为重要地，是为了回收子进程资源，避免僵尸进程。
> 子进程状态改变包含：
> 子进程终止（the child terminated）
> 子进程由于信号停止（the child was stopped by a signal）
> 子进程由于信号继续（the child was resumed by a signal）
>
> 当子进程结束时，父进程调用wait或者waitpid，会回收子进程的相关资源（pid，子进程结束状态status，资源使用信息）。如果没有回收，子进程就变成僵尸子进程会占用系统进程表的一个槽位，如果该表满了，就无法继续创建新的进程了。

+ wait函数表现

> 1、当子进程还未结束，那么父进程调用wait会阻塞，直到子进程结束 或 信号处理函数中断。
> 2、当子进程已经结束，那么父进程调用wait会立刻返回。

+ wait和waitpid区别

> waitpid比wait功能强大很多，wait只是waitpid的一个特例，如下：
> wait(&wstatus); 等价于 waitpid(-1, &wstatus, 0);
> 看上面waitpid的pid参数就知道，waitpid不但可以回收子进程（pid=-1）资源，也可以回收指定子进程pid的进程资源（pid>0，比如fork很多次，就有很多子进程，这时候只回收其中一个子进程的情况）。

+ 多线程如何实现，资源如何回收

> Linux系统中程序的线程资源是有限的，表现为对于一个程序其能同时运行的线程数是有限的。而默认的条件下，一个线程结束后，其对应的资源不会被释放，于是，如果在一个程序中，反复建立线程，而线程又默认的退出，则最终线程资源耗尽，进程将不再能建立新的线程。
> pthread_create( );建立进程
>
> 如果想在线程结束时，由系统释放线程资源，则需要设置线程属性为detach。
>
> pthread_join( t)等待线程t退出，并释放t线程所占用的资源。
> pthread_join函数会阻塞等待指定线程退出，然后回收资源，这样就有同步的功能，使一个线程等待另一个线程退出，然后才继续运行，但是对于服务器程序如果主线程在新创建的线程工作时还需要做别的事情，这种方法不是很好，就需要使用方法一。
>
> linux线程pthread有两种状态joinable状态和unjoinable状态，如果线程是joinable状态，当线程函数自己返回退出时或pthread_exit时都不会释放线程所占用堆栈和线程描述符（总计8K多）。只有当你调用了pthread_join之后这些资源才会被释放。
> 若是unjoinable状态的线程，这些资源在线程函数退出时或pthread_exit时自动会被释放。
>
> unjoinable属性可以在pthread_create时指定，或在线程创建后在线程中pthread_detach自己,如：pthread_detach(pthread_self())，将状态改为unjoinable状态，确保资源的释放。
>
> pthread_detach(threadid)函数的功能是使线程ID为threadid的线程处于分离状态，一旦线程处于分离状态，该线程终止时底层资源立即被回收；否则终止子线程的状态会一直保存（占用系统资源）直到主线程调用pthread_join(threadid,NULL)获取线程的退出状态。
>
> 主线程使用pthread_create()创建子线程以后，一般可以调用pthread_detach(threadid)分离刚刚创建的子线程，这里的threadid是指子线程的threadid；如此以来，该子线程止时底层资源立即被回收；
> 被创建的子线程也可以自己分离自己，子线程调用pthread_detach(pthread_self())就是分离自己，因为pthread_self()这个函数返回的就是自己本身的线程ID

+ 线程同步的方式和机制

> 临界区（Critical Section）、互斥对象（Mutex）、信号量（Semaphore）、事件对象（Event）
>
> 1、临界区：通过对多线程的串行化来访问公共资源或一段代码，速度快，适合控制数据访问。在任意时刻只允许一个线程对共享资源进行访问，如果有多个线程试图访问公共资源，那么在有一个线程进入后，其他试图访问公共资源的线程将被挂起，并一直等到进入临界区的线程离开，临界区在被释放后，其他线程才可以抢占。
>
> 2、互斥对象：互斥对象和临界区很像，采用互斥对象机制，只有拥有互斥对象的线程才有访问公共资源的权限。因为互斥对象只有一个，所以能保证公共资源不会同时被多个线程同时访问。当前拥有互斥对象的线程处理完任务后必须将线程交出，以便其他线程访问该资源。
>
> 3、信号量：信号量也是内核对象。它允许多个线程在同一时刻访问同一资源，但是需要限制在同一时刻访问此资源的最大线程数目
> 用CreateSemaphore()创建信号量时即要同时指出允许的最大资源计数和当前可用资源计数。一般是将当前可用资源计数设置为最 大资源计数，每增加一个线程对共享资源的访问，当前可用资源计数就会减1 ，只要当前可用资源计数是大于0 的，就可以发出信号量信号。但是当前可用计数减小 到0 时则说明当前占用资源的线程数已经达到了所允许的最大数目，不能在允许其他线程的进入，此时的信号量信号将无法发出。线程在处理完共享资源后，应在离 开的同时通过ReleaseSemaphore（）函数将当前可用资源计数加1
>
> 4、事件对象： 通过通知操作的方式来保持线程的同步，还可以方便实现对多个线程的优先级比较的操作
>
> Event
> 1）事件是内核对象，事件分为手动置位事件和自动置位事件。事件Event内部它包含一个使用计数（所有内核对象都有），一个布尔值表示是手动置位事件还是自动置位事件，另一个布尔值用来表示事件有无触发。
> 2）事件可以由SetEvent()来触发，由ResetEvent()来设成未触发。还可以由PulseEvent()来发出一个事件脉冲。

+ 什么是回调函数？

> 编程分为两类：系统编程（system programming）和应用编程（application programming）。所谓系统编程，简单来说，就是编写库；而应用编程就是利用写好的各种库来编写具某种功用的程序，也就是应用。系统程序员会给自己写的库留下一些接口，即API（application programming interface，应用编程接口），以供应用程序员使用。所以在抽象层的图示里，库位于应用的底下。
>
> 当程序跑起来时，一般情况下，应用程序（application program）会时常通过API调用库里所预先备好的函数。但是有些库函数（library function）却要求应用先传给它一个函数，好在合适的时候调用，以完成目标任务。这个被传入的、后又被调用的函数就称为回调函数（callback function）。
>
> 打个比方，有一家旅馆提供叫醒服务，但是要求旅客自己决定叫醒的方法。可以是打客房电话，也可以是派服务员去敲门，睡得死怕耽误事的，还可以要求往自己头上浇盆水。这里，“叫醒”这个行为是旅馆提供的，相当于库函数，但是叫醒的方式是由旅客决定并告诉旅馆的，也就是回调函数。而旅客告诉旅馆怎么叫醒自己的动作，也就是把回调函数传入库函数的动作，称为登记回调函数（to register a callback function）
>
> 在传入一个回调函数之前，中间函数是不完整的。换句话说，程序可以在运行时，通过登记不同的回调函数，来决定、改变中间函数的行为。这就比简单的函数调用要灵活太多了。

+ 一个日志系统需要具备哪些功能？

> 日志配置读取：方便不同项目部署，通过更改配置文件即可
> 日志级别：为了减少线上日志大小，开发环境和线上环境记录错我的级别一般是不一样的，比如一般线上只记fatal、error和info，开发环境则需要记录rpc、warning、notice等
> 自动捕获错误：需要注册error和shutdown时的回调方法
> 记录错误时的调用栈：当出现fatal和error级别的错误时，有时只靠错误信息时很难准确定位到错误代码的，所以需要记录函数的调用栈，方便排查错误
> 动态改变日志级别：记录日志时需要检测当前配置的日志级别，只记录级别大于等于配置级别的日志
> 基本日志字段：log_id、时间、耗时、产品线、模块名称、请求uri、分布式调用xhop、错误信息、请求返回信息、客户端IP
> 分日期和小时记录，方便定期归档

+ 知道哪些大型的http服务器?

> IIS nginx apache

+ 谈谈Nginx有哪些特点 

> 1、热部署
>
>        我个人觉得这个很不错。在master管理进程与worker工作进程的分离设计，使的Nginx具有热部署的功能，那么在7×24小时不间断服务的前提下，升级Nginx的可执行文件。也可以在不停止服务的情况下修改配置文件，更换日志文件等功能。
>
> 2、可以高并发连接
>
>       这是一个很重要的一个特性！在这一个互联网快速发展，互联网用户数量不断增加，一些大公司、网站都需要面对高并发请求，如果有一个能够在峰值顶住10万以上并发请求的Server，肯定会得到大家的青睐。理论上，Nginx支持的并发连接上限取决于你的内存，10万远未封顶。
>
> 3、低的内存消耗
>
>        在一般的情况下，10000个非活跃的HTTP Keep-Alive 连接在Nginx中仅消耗2.5M的内存，这也是Nginx支持高并发连接的基础。
>
> 4、处理响应请求很快
>
>        在正常的情况下，单次请求会得到更快的响应。在高峰期，Nginx可以比其他的Web服务器更快的响应请求。
>
> 5、具有很高的可靠性
>
>       Nginx是一个高可靠性的Web服务器，这也是我们为什么选择Nginx的基本条件，现在很多的网站都在使用Nginx，足以说明Nginx的可靠性。高可靠性来自其核心框架代码的优秀设计、模块设计的简单性；并且这些模块都非常的稳定。

+ vector的内存增长方式？

> vector其中一个特点：内存空间只会增长，不会减小，C++ Primer：为了支持快速的随机访问，vector容器的元素以连续方式存放，每一个元素都紧挨着前一个元素存储。设想一下，当vector添加一个元素时，为了满足连续存放这个特性，都需要重新分配空间、拷贝元素、撤销旧空间，这样性能难以接受。因此STL实现者在对vector进行内存分配时，其实际分配的容量要比当前所需的空间多一些。就是说，vector容器预留了一些额外的存储区，用于存放新添加的元素，这样就不必为每个新元素重新分配整个容器的内存空间。

+ vector的内存释放

> 由于vector的内存占用空间只增不减，比如你首先分配了10,000个字节，然后erase掉后面9,999个，留下一个有效元素，但是内存占用仍为10,000个。所有内存空间是在vector析构时候才能被系统回收。empty()用来检测容器是否为空的，clear()可以清空所有元素。但是即使clear()，vector所占用的内存空间依然如故，无法保证内存的回收。
>
> vector，可以用swap()来帮助你释放内存

+ C++ vector的reserve和resize？

> vector 的reserve增加了vector的capacity，但是它的size没有改变！而resize改变了vector的capacity同时也增加了它的size

+ 智能指针

> auto_ptr
> （C++98的方案，C++11已经抛弃）采用所有权模式。
>
> unique_ptr
> 替换auto_ptr）unique_ptr实现独占式拥有或严格拥有概念，保证同一时间内只有一个智能指针可以指向该对象。它对于避免资源泄露(例如“以new创建对象后因为发生异常而忘记调用delete”)特别有用。
>
> 采用所有权模式，还是上面那个例子C++有一个标准库函数std::move()，让你能够将一个unique_ptr赋给另一个。尽管转移所有权后 还是有可能出现原有指针调用（调用就崩溃）的情况。但是这个语法能强调你是在转移所有权，让你清晰的知道自己在做什么，从而不乱调用原有指针。
>
> shared_ptr
> shared_ptr实现共享式拥有概念。多个智能指针可以指向相同对象，该对象和其相关资源会在“最后一个引用被销毁”时候释放。从名字share就可以看出了资源可以被多个指针共享，它使用计数机制来表明资源被几个指针共享。可以通过成员函数use_count()来查看资源的所有者个数。除了可以通过new来构造，还可以通过传入auto_ptr, unique_ptr,weak_ptr来构造。当我们调用release()时，当前指针会释放资源所有权，计数减一。当计数等于0时，资源会被释放。
>
> 成员函数：
>
> use_count 返回引用计数的个数
>
> unique 返回是否是独占所有权( use_count 为 1)
>
> swap 交换两个 shared_ptr 对象(即交换所拥有的对象)
>
> reset 放弃内部对象的所有权或拥有对象的变更, 会引起原有对象的引用计数的减少
>
> get 返回内部对象(指针), 由于已经重载了()方法, 因此和直接使用对象是一样的
>
> weak_ptr
> share_ptr虽然已经很好用了，但是有一点share_ptr智能指针还是有内存泄露的情况，当两个对象相互使用一个shared_ptr成员变量指向对方，会造成循环引用，使引用计数失效，从而导致内存泄漏。
>
> 
>
> ```cc
> class B; //声明
> class A
> {
> public:
>  shared_ptr<B> pb_;
>  ~A()
>  {
>   cout << "A delete\n";
>  }
> };
> 
> class B
> {
> public:
>  shared_ptr<A> pa_;
>  ~B()
>  {
>   cout << "B delete\n";
>  }
> };
> 
> void fun()
> {
>  shared_ptr<B> pb(new B());
>  shared_ptr<A> pa(new A());
>  cout << pb.use_count() << endl; //1
>  cout << pa.use_count() << endl; //1
>  pb->pa_ = pa;
>  pa->pb_ = pb;
>  cout << pb.use_count() << endl; //2
>  cout << pa.use_count() << endl; //2
> }
> ```
>
> 如果把其中一个改为weak_ptr就可以了，我们把类A里面的shared_ptr pb_，改为weak_ptr pb_ 
>
> 我们不能通过weak_ptr直接访问对象的方法，比如B对象中有一个方法print()，我们不能这样访问，pa->pb_->print()，因为pb_是一个weak_ptr，应该先把它转化为shared_ptr，如：
>
> cpp
> shared_ptr<B> p = pa->pb_.lock();
> p->print();
>
>
> weak_ptr 没有重载*和->但可以使用 lock 获得一个可用的 shared_ptr 对象.

+ 如何理解redo和undo的作用？

> <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210810220945057.png" alt="image-20210810220945057" style="zoom:50%;" />
>
> 是undo log 记录数据的过程，当执行数据变化出现异常时候，可以使用undo log日志回滚数据。
>
> 异常场景：如果在执行第7步断电了，B没有写入磁盘，会怎么样？
>
> 此时Undo log日志是这样的：
>
>     <T1,开始>
>     <T1,A,200>
>     <T1,B,100>
>
> 当系统恢复重启，需要检查这个undo log日志，发现事务T1并未提交，即Undo log中没有**<T1,提交>**这条记录，系统知道这个事务此次执行过程中出现异常，失败了，就做恢复操作，恢复A,B的初始值，然后再日志中再加上一条**<T1,回滚>**，这样下次就不在恢复相关的数据。
>
> <img src="https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210810221101228.png" alt="image-20210810221101228" style="zoom:50%;" />
>
> Undo里是怎么样去撤销一个改变；redo里是怎么样去重做一个改变；
>
> Undo用于回滚、一致性读（readconsistency）和闪回（flashback）；redo用于数据库前滚（rolling forward）、恢复和数据的改变；
>
> Undo放在undo表空间中；redo是放在redo日志文件中；
>
> Undo是来保护一致性读；redo来保证数据不丢失。

+ 假设有下面代码

  ```cc
  class A{}；
  A a1，a2；
  a1 = a2 + 100;
  ```

  如何让上面两句通过编译？

> 设置单参、无参构造函数的列表初始化、有参构造函数成员列表初始化、重载加法运算符：
>
> ```cc
> class A{
> public:
>  int val;
>  A():val(0){};
>  A(int a) :val(a){};
>  A operator+(const A & a1){
>   A a0;
>   a0.val = this->val + a1.val;
>   return a0;
>  }
> 
> };
> 
> int main()
> {
> 
>  A a1 ,a2;
>  a1 = a2 + 100;
>  cout << a1.val;
>  getchar();
>  return 0;
> }
> ```

+ const成员变量能否在成员函数里面初始化？

> 不行，const的成员变量只能在类的构造函数通过成员列表初始化，或者声明的时候初始化。
> 但是能够用成员函数通过指针改变他的值
>
> ```cc
> const int a=99;
> int  seta(int n){
>   int* p = (int*)&a;
>   *p = n;
>   return a;
>  } 
> ```

+ 空类提供了哪些构造函数？

> 缺省拷贝构造、缺省构造（补充：析构、赋值运算符、取地址运算符、cosnt取地址运算符）
>
> ```cc
> const class1*operator&()const{}//取址运算符 const
> ```

+ 什么时候需要自己定义构造、析构函数？

> 需要在实例化类的时候就初始化成员，你就需要自己定义构造函数。
> 比如深拷贝的时候也需要自己定义拷贝构造函数。
>
> ```cc
> int *p;
> Test(const Test &tmp) // 定义拷贝构造函数
>  {
>   p = new int(*tmp.p);
>   cout << "Test(const Test &tmp)" << endl;
>  }
> ```

+ 构造函数、析构函数是否需要定义成虚函数？为什么？

> 构造函数一般不定义为虚函数，原因：
>
> 从存储空间的角度考虑：构造函数是在实例化对象的时候进行调用，如果此时将构造函数定义成虚函数，需要通过访问该对象所在的内存空间才能进行虚函数的调用（因为需要通过指向虚函数表的指针调用虚函数表，虽然虚函数表在编译时就有了，但是没有虚函数的指针，虚函数的指针只有在创建了对象才有），但是此时该对象还未创建，便无法进行虚函数的调用。所以构造函数不能定义成虚函数。
> 从使用的角度考虑：虚函数是基类的指针指向派生类的对象时，通过该指针实现对派生类的虚函数的调用，构造函数是在创建对象时自动调用的。
> 从实现上考虑：虚函数表是在创建对象之后才有的，因此不能定义成虚函数。
> 从类型上考虑：在创建对象时需要明确其类型
>
>
> 析构函数一般定义成虚函数，原因：
>
> 析构函数定义成虚函数是为了防止内存泄漏，因为当基类的指针或者引用指向或绑定到派生类的对象时，如果未将基类的析构函数定义成虚函数，会调用基类的析构函数，那么只能将基类的成员所占的空间释放掉，派生类中特有的就会无法释放内存空间导致内存泄漏。

+ vector底层，时间复杂度

> vector 是动态数组，vector 动态增加大小，并不是在原空间之后持续新空间（因为无法保证原空间之后尚有可供配置的空间），而是以原大小的两倍另外配置一块较大的空间，然后将原内容拷贝过来，然后才开始在原内容之后构造新元素，并释放原空间。
>
> 对任何元素的访问都是 O(1)，也就是常数时间的。 对最后元素操作最快（在后面添加删除最快）是 O(1)，此时一般不需要移动内存。对中间和开始处进行添加删除元素操作需要移动内存，是 O(n)。
>
> 支持[]操作符

+ list底层，时间复杂度

> list 底层是一个双向链表，而且是一个环状双向链表。好处是每次插入或删除一个元素，就配置或释放一个元素空间，元素也是在堆中。在添加、删除 O(1)，访问 O(n)。
>
> 不支持[]操作符

+ deque底层，时间复杂度

> 是一种双向开口的线性空间，动态地以分段连续空间组合而成，可以在队尾两端分别做元素的插入和删除操作。没有所谓容量观念。
>
> 头尾O(1)
>
> 支持[]操作符

+ vector、list区别

> 底层结构、访问、增删复杂度

+ vector、deque区别

> 在标准库中 vector 和 deque 提供几乎相同的接口，在结构上区别主要在于在组织内存上不一样，deque 是按页或块来分配存储器的，每页包含固定数目的元素；相反 vector 分配一段连续的内存，vector 只是在序列的尾段插入元素时才有效率，而 deque 的分页组织方式即使在容器的前端也可以提供常数时间的 insert 和 erase 操作，而且在体积增长方面也比 vector 更具有效率。

+ stack底层，时间复杂度

> stack 是一种先进后出的数据结构。它只有一个出口，stack 允许新增元素，移除元素，取得最顶端元素。
>
> 一般用deque 实现，封闭头部即可，不用 vector 的原因应该是容量大小有限制，扩容耗时

+ queue底层，时间复杂度

> queue 是一种先进先出的数据结构。
>
> 一般用 list 或 deque 实现，封闭头部即可 

> stack 和 queue 其实是适配器，而不叫容器，因为是对容器的再封装。

+ unorderer_map 底层，时间复杂度

> 底层数据结构为 hash 表，无序，不重复。

+ unorderer_set 底层，时间复杂度

> 底层数据结构为 hash 表，无序，不重复。

+ map 底层，时间复杂度

> map的特性是，所有元素都会根据元素的键值自动被排序。
>
> map 的所有元素都是 pair，同时拥有实值（value）和键值（key）。pair 的第一元素被视为键值，第二元素被视为实值。map**不允许两个元素拥有相同的键值**。由于 RB-tree 是一种平衡二叉搜索树，自动排序的效果很不错，所以标准的STL map 即**以 RB-tree 为底层机制**。
>
> multimap 的特性以及用法与 map 完全相同，唯一的差别在于它允许键值重复，因此它的插入操作采用的是底层机制 RB-tree 的 insert_equal() 而非 insert_unique。 

+ set 底层，时间复杂度

> set 的特性是，所有元素都会根据元素的键值自动被排序。set 的元素不像 map 那样可以同时拥有实值(value)和键值(key)，set 元素的键值就是实值，实值就是键值，set不允许两个元素有相同的值。
>
> set 底层是通过红黑树（RB-tree）来实现的，由于红黑树是一种平衡二叉搜索树，自动排序的效果很不错
>
> multiset的特性以及用法和 set 完全相同，唯一的差别在于它允许键值重复。底层机制是 RB-tree 的 insert_equal() 而非 insert_unique()。

+ heap 底层，时间复杂度

> heap 并不归属于 STL 容器组件，它是个幕后英雄，扮演 priority queue（优先队列）的助手。priority queue 允许用户以任何次序将任何元素推入容器中，但取出时一定按从优先权最高的元素开始取。
>
> heap 可分为 max-heap 和 min-heap 两种，前者每个节点的键值(key)都大于或等于其子节点键值，后者的每个节点键值(key)都小于或等于其子节点键值。
>
> max-heap 的最大值在根节点，并总是位于底层vector的起头处；min-heap 的最小值在根节点，亦总是位于底层vector起头处。

+ priority_queue  底层，时间复杂度

> 一般为vector为底层容器，再加上 heap 处理规则
>
> 以vector 表现的完全二叉树max-heap 可以满足 priority_queue 所需要的“依权值高低自动递减排序”的特性。

+ 模板底层

> 模板函数只有存在调用的时才会编译。
>
> 模板中的代码只有在用到时才会被实例化。也就是说，当遇到template<class T> T max(T a, T b) 时，编译器并不会完全展开整个模板类。只有当访问了模板上的T max(T a, T b)函数时，才会将成员函数的代码展开作语义检查，此时编译会报错。
>
> 编译器会对函数模板进行两次编译
>
> （1）在声明的位置对模板代码进行编译
>
> （2）在调用的位置对参数替换后的代码进行编译
>
> 函数模板不是说只 **一个函数** 就可以实现对任意数据类型的操作，而是通过两次编译生成了满足我们调用需求所需要的所有代码。

+ 模板和宏定义的区别

> 1.宏是在预处理阶段处理，模板是在编译阶段处理
>
> 2.宏不会进行类型检查，只会单纯的进行文本替换，模板会进行类型检查。比如下面代码模板就会出错，而宏不会
>
> 3.宏直接就可以产生代码，而编译器遇到模板定义时，并不产生代码，只有当模板实例化后时才会产生代码。

+ move函数

> std::move 可以将一个左值强制转化为右值，继而可以通过右值引用使用该值，以用于移动语义。
>
> ```cc
> template <typename T>typename remove_reference<T>::type&& move(T&& t){    
>     return static_cast<typename remove_reference<T>::type &&>(t);
> }
> ```
>
> 说明：引用折叠原理
>
> + 右值传递给上述函数的形参 T&& 依然是右值，即 T&& && 相当于 T&&。
> + 左值传递给上述函数的形参 T&& 依然是左值，即 T&& & 相当于 T&。
>
> 小结：通过引用折叠原理可以知道，`move()` 函数的形参既可以是左值也可以是右值。
>
> 使用方法：
>
> ```cc
> template <class T> struct remove_reference<T&> //左值引用
> { typedef T type; }
>  
> template <class T> struct remove_reference<T&&> //右值引用
> { typedef T type; } 
> 
> int i;
> remove_refrence<decltype(42)>::type a;             //使用原版本，
> remove_refrence<decltype(i)>::type  b;             //左值引用特例版本
> remove_refrence<decltype(std::move(i))>::type  b;  //左值变为右值引用特例版本 
> ```
>
> 左值引用不能绑定到要转换的表达式、字面常量或返回右值的表达式。
>
> 右值引用必须绑定到右值的引用，通过 && 获得。右值引用只能绑定到一个将要销毁的对象上，因此可以自由地移动其资源。
>
> ```
> #include <iostream>
> using namespace std;
> 
> void fun1(int& tmp) 
> { 
>   cout << "fun1(int& tmp):" << tmp << endl; 
> } 
> 
> void fun2(int&& tmp) 
> { 
>   cout << "fun2(int&& tmp)" << tmp << endl; 
> } 
> 
> int main() 
> { 
>   int var = 11; 
>   fun1(12); // error: cannot bind non-const lvalue reference of type 'int&' to an rvalue of type 'int'
>   fun1(var);
>   fun2(1); 
> }
> ```

+ 长连接 `Connection： keep-alive `的底层实现

> Client收到Web Server的response中包含"Connection： keep-alive"，就认为是一个长连接，不close tcp连接。并用该tcp连接再发送request。

+ 什么时候不用索引

> 1. 表记录太少；
> 2. 数据重复且分布平均的字段（只有很少数据值的列）；
> 3. 经常插入、删除、修改的表要减少索引；
> 4. text，image等类型不应该建立索引，这些列的数据量大（假如text前10个字符唯一，也可以对text前10个字符建立索引）；
> 5. MySQL能估计出全表扫描比使用索引更快时，不使用索引；

+ 订阅了解吗 

> Redis 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。
>
> Redis 客户端可以订阅任意数量的频道。
>
> 1、 打开一个客户端订阅channel1
>
> SUBSCRIBE channel01
>
>  ![image-20210717183337866](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210717183337866.png)
>
> 2、打开另一个客户端，给channel1发布消息hello
>
> publish channel01 sb
>
>  ![image-20210717183405015](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210717183405015.png)
>
> 返回的1是订阅者数量
>
> 3、打开第一个客户端可以看到发送的消息
>
>  ![image-20210717183426918](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210717183426918.png)

+ 读写锁

> 一个读写锁同时只能有一个写者或多个读者（与CPU数相关），但不能同时既有读者又有写者。在读写锁保持期间也是抢占失效的。
>
> 如果读写锁当前没有读者，也没有写者，那么写者可以立刻获得读写锁，否则它必须自旋在那里，直到没有任何写者或读者。如果读写锁没有写者，那么读者可以立即获得该读写锁，否则读者必须自旋在那里，直到写者释放该读写锁。

+ 左值和右值的概念

> 左值是可以放在赋值号左边可以被赋值的值；左值必须要在内存中有实体；
>
> 右值当在赋值号右边取出值赋给其他变量的值；右值可以在内存也可以在CPU寄存器。
>
> 一个对象被用作右值时，使用的是它的内容(值)，被当作左值时，使用的是它的地址**。**

+ 引用和指针的区别

> + 指针所指向的内存空间在程序运行过程中可以改变，而引用所绑定的对象一旦绑定就不能改变。（是否可变）
> + 指针本身在内存中占有内存空间，引用相当于变量的别名，在内存中不占内存空间。（是否占内存）
> + 指针可以为空，但是引用必须绑定对象。（是否可为空）
> + 指针可以有多级，但是引用只能一级。（是否能为多级）

+ 事务原子性怎么实现

> 想要保证事务的原子性，就需要在异常发生时，对已经执行的操作进行回滚。恢复机制是通过回滚日志(undo log)实现的。

+ 可串行化

> 强制事务串行执行，使之不可能相互冲突，从而解决幻读问题。可能导致大量的超时现象和锁竞争，实际很少使用。

+ 用户级线程和内核级线程的区别

> **（1）**内核支持线程是OS内核可感知的，而用户级线程是OS内核不可感知的。
>
> **（2）**用户级线程的创建、撤消和调度不需要OS内核的支持，是在语言这一级处理的；而内核支持线程的创建、撤消和调度都需OS内核提供支持，而且与进程的创建、撤消和调度大体是相同的。
>
> **（3）**用户级线程执行系统调用指令时将导致其所属进程被中断，而内核支持线程执行系统调用指令时，只导致该线程被中断。
>
> **（4）**在只有用户级线程的系统内，CPU调度还是以进程为单位，处于运行状态的进程中的多个线程，由用户程序控制线程的轮换运行；在有内核支持线程的系统内，CPU调度则以线程为单位，由OS的线程调度程序负责线程的调度。
>
> **（5）**用户级线程的程序实体是运行在用户态下的程序，而内核支持线程的程序实体则是可以运行在任何状态下的程序。
>
> **内核线程的优点：**
>
> （1）当有多个处理机时，一个进程的多个线程可以同时执行。
>
> **缺点：**
>
> （1）由内核进行调度。
>
> **用户进程的优点：**
>
> （1） 线程的调度不需要内核直接参与，控制简单。
>
> （2） 可以在不支持线程的操作系统中实现。
>
> （3） 创建和销毁线程、线程切换代价等线程管理的代价比内核线程少得多。
>
> （4） 允许每个进程定制自己的调度算法，线程管理比较灵活。
>
> （5） 线程能够利用的表空间和堆栈空间比内核级线程多。
>
> （6） 同一进程中只能同时有一个线程在运行，如果有一个线程使用了系统调用而阻塞，那么整个进程都会被挂起。另外，页面失效也会产生同样的问题。
>
> **缺点：**
>
> （1）资源调度按照进程进行，多个处理机下，同一个进程中的线程只能在同一个处理机下分时复用

+ linux文件操作 内核态和用户态发生了什么

> 以open为例，用户态到内核态的切换
>
> 1. 保存上下文
>
> 2. 将系统调用号保存到寄存器
>
> 3. 触发中断
>
>    
>
> 系统调用函数：open  read  write  close   lseek  stat
>
> ![image-20210827223336600](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210827223336600.png)



+ map, set, hash_map, hash_set时间复杂度

> map, set, multimap, and multiset
> 上述四种容器采用红黑树实现，红黑树是平衡二叉树的一种。不同操作的时间复杂度近似为:
> 插入: O(logN)
>
> 查看:O(logN)
>
> 删除:O(logN)
>
> hash_map, hash_set, hash_multimap, and hash_multiset
> 上述四种容器采用哈希表实现，不同操作的时间复杂度为：
> 插入:O(1)，最坏情况O(N)。
>
> 查看:O(1)，最坏情况O(N)。
>
> 删除:O(1)，最坏情况O(N)。

+ 场景题：如果有持续不断的url输入，怎么统计数量最多的前10个url

> 维护10size的小顶堆，只插访问次数比根大的值进去，pop了根，相当于是哈希map+小顶堆，维护一个map存放url和url的freq）

+  管道底层是怎么通信的，为什么只能父子通信

> **匿名管道**
>
> 父进程在fork出子进程后，子进程拥有了和父进程一样的代码，当然也拥有和父进程一样的文件描述符，在fork出子进程后，父进程关闭管道的读端fd[0]，子进程关闭管道的写端fd[1]，从而来实现父子进程之间的管道通信，即从子进程向管道内写数据，父进程从管道内读数据。
>
> **命名管道FIFO**
>
> 命名管道相较于匿名管道的不同，就是能够适用于任意进程间的通信。不同的是，匿名管道在用pipe函数创建的同时也打开了，而命名管道在mkfifo创建了之后，还需用open函数打开该命名管道。
>
> **对于操作系统，进程就是一个数据结构**，我们直接来看 Linux 的源码：
>
> ```
> struct task_struct {
>     // 进程状态
>     long              state;
>     // 虚拟内存结构体
>     struct mm_struct  *mm;
>     // 进程号
>     pid_t             pid;
>     // 指向父进程的指针
>     struct task_struct   *parent;
>     // 子进程列表
>     struct list_head      children;
>     // 存放文件系统信息的指针
>     struct fs_struct      *fs;
>     // 一个数组，包含该进程打开的文件指针
>     struct files_struct   *files;
> };
> ```
>
> `task_struct`就是 Linux 内核对于一个进程的描述，也可以称为「进程描述符」。源码比较复杂，我这里就截取了一小部分比较常见的。
>
> 我们主要聊聊`mm`指针和`files`指针。**`mm`指向的是进程的虚拟内存**，也就是**载入资源和可执行文件**的地方；**`files`指针指向一个数组**，这个数组里**装着所有该进程打开的文件的指针**。
>
> **文件描述符是什么**
>
> 先说`files`，它是一个**文件指针数组**。一般来说，一个进程会从`files[0]`**读取输入**，将**输出写入**`files[1]`，将**错误信息写入**`files[2]`。
>
> 举个例子，以我们的角度 C 语言的`printf`函数是向命令行打印字符，但是从进程的角度来看，就是向`files[1]`写入数据；同理，`scanf`函数就是进程试图从`files[0]`这个文件中读取数据。
>
> **每个进程被创建时，`files`的前三位被填入默认值，分别指向标准输入流、标准输出流、标准错误流。我们常说的「文件描述符」就是指这个文件指针数组的索引**，所以程序的文件描述符默认情况下 0 是输入，1 是输出，2 是错误。
>
> 我们可以重新画一幅图：
>
> ![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gibkIz0MVqdEZbbic0diawibWHE9EoMFmX8qGdHmurRkzVeMrXvIYXveQQkA3ZCQe7gCKDglcMZo6OiaMQYy1NickJng/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
>
> 对于一般的计算机，输入流是键盘，输出流是显示器，错误流也是显示器，所以现在这个进程和内核连了三根线。因为硬件都是由内核管理的，我们的**进程**需要**通过「系统调用」**让内核进程**访问硬件资源**。
>
> PS：不要忘了，Linux 中一切都被抽象成文件，设备也是文件，可以进行读和写。
>
> 如果我们写的程序需要其他资源，比如**打开一个文件**进行读写，这也很简单，进行系统调用，让内核把文件打开，这个文件就会被放到`files`的第 4 个位置，对应文件描述符 3：
>
> ![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gibkIz0MVqdEZbbic0diawibWHE9EoMFmX8qj7mQ2bDPdIMZIA7aNrWwbWMlbdiaYPn8EXVsjLY2uckc4IgUzric5tvA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
>
> 明白了这个原理，**输入重定向**就很好理解了，程序想读取数据的时候就会去`files[0]`读取，所以我们只要把`files[0]`指向一个文件，那么程序就会从这个文件中读取数据，而不是从键盘：
>
> ![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gibkIz0MVqdEZbbic0diawibWHE9EoMFmX8qUGlh8sIut3YFIPFIEA6H8oahMlWpjjiak2KM9K7HX24zK7GLzknYAPQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
>
> 同理，**输出重定向**就是把`files[1]`指向一个文件，那么程序的输出就不会写入到显示器，而是写入到这个文件中：
>
> ![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gibkIz0MVqdEZbbic0diawibWHE9EoMFmX8qPcRT8weuAZoENbribFJbhbpMkwDfez6QxVE2H8hDRhPMHWmAcBs0rcg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
>
> 错误重定向也是一样的，就不再赘述。
>
> **管道符**其实也是异曲同工，把一个进程的输出流和另一个进程的输入流接起一条「管道」，数据就在其中传递，不得不说这种设计思想真的很巧妙：
>
> ![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gibkIz0MVqdEZbbic0diawibWHE9EoMFmX8qBRhdjExXRmiccwQ37ZXZ4645LlAdYY4VOUUibUDNrLjNLUGXWjGPibOgw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

+ http为什么要设立一个头部来存放相关信息不直接放在body里

> （自己回答的时候从协议栈工作的角度回答了，结论是主要是为了解耦）
>
> HTTP包括message-header和message-body两部分。
>
> `header`主要来存放cookie，token等信息的 `body`主要用来存放post的一些数据

+   把默认的三级隔离级别换成二级隔离级别会有什么影响

> （自己回答的时候从x锁、s锁的角度、行级锁和表级锁、快照读和当前读的角度进行了回答，认为程序员层面这样的改动没有影响，因为只是出现了不可重复读的问题，不知道对不对）
>
> A事务需要重复读两次时，第一次读到事务B未提交之前的数据第二次读到了B已提交的数据，就造成了两次读到的数据不一致，从而有可能影响到A的业务。

+ 为何需要3个随机数协商密钥

> 客户端和服务端在握手hello消息中明文交换了client_random和server_random，使用RSA公钥加密传输pre master secret，最后通过算法，客户端和服务端分别计算master secret。其中，不直接使用premaster secret的原因是：保证secret的随机性不受任意一方的影响。

+ git rebase和merge区别

> merge和rebase都是用来合并分支的。
>
> 1. 采用merge和rebase后，git log的区别，merge命令不会保留merge的分支的commit
>
> 2. 处理冲突的方式：
>
>    + 使用`merge`命令合并分支，解决完冲突，执行`git add .`和`git commit -m'fix conflict'`。这个时候会产生一个commit。
>
>    + 使用`rebase`命令合并分支，解决完冲突，执行`git add .`和`git rebase --continue`，不会产生额外的commit。好处是干净，分支上不会有无意义的解决分支的commit；坏处，如果合并的分支中存在多个`commit`，需要重复处理多次冲突

+ `git merge` 和 `git merge --no-ff`的区别

> 如果想在没有冲突的情况下也自动生成一个commit，记录此次合并就可以用：`git merge --no-ff`命令，下面用一张图来表示两者的区别：\
>
> ![image-20210911220528314](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210911220528314.png)
>
> 如果不加 --no-ff 则被合并的分支之前的commit都会被抹去，只会保留一个解决冲突后的 merge commit。

+ 结合二者的优势用数组和链表设计数据结构（哈希表...）

> ![image-20210917104543696](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210917104543696.png)

+ void占几字节，void*占几字节

> 不能造void型的変量...
>
> void*占4字节

+ 有多个基类的派生类有几个虚函数表

> （有几个基类就有几个虚函数表）
>
> ![image-20210917105351703](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210917105351703.png)

+ 了解哪些哈希散列函数（只知道取模）

  > 几种常见的哈希函数（散列函数）构造方法
  >
  > - 直接定址法 
  >   - 取关键字或关键字的某个线性函数值为散列地址。
  >   - 即 H(key) = key 或 H(key) = a*key + b，其中a和b为常数。
  >   - 比如![这里写图片描述](https://img-blog.csdn.net/20161026171706654)
  > - 除留余数法 
  >   - 取关键字被某个不大于散列表长度 m 的数 p 求余，得到的作为散列地址。
  >   - 即 H(key) = key % p, p < m。 
  >   - 比如![这里写图片描述](https://img-blog.csdn.net/20161026171807417)
  > - 数字分析法 
  >   - 当关键字的位数大于地址的位数，对关键字的各位分布进行分析，选出分布均匀的任意几位作为散列地址。
  >   - 仅适用于所有关键字都已知的情况下，根据实际应用确定要选取的部分，尽量避免发生冲突。
  >   - 比如 ![这里写图片描述](https://img-blog.csdn.net/20161026172017748)
  > - 平方取中法 
  >   - 先计算出关键字值的平方，然后取平方值中间几位作为散列地址。
  >   - 随机分布的关键字，得到的散列地址也是随机分布的。
  >   - 比如 ![这里写图片描述](https://img-blog.csdn.net/20161026171618181)
  > - 折叠法（叠加法） 
  >   - 将关键字分为位数相同的几部分，然后取这几部分的叠加和（舍去进位）作为散列地址。
  >   - 用于关键字位数较多，并且关键字中每一位上数字分布大致均匀。 
  >   - 比如 ![这里写图片描述](https://img-blog.csdn.net/20161026173032699)
  > - 随机数法 
  >   - 选择一个随机函数，把关键字的随机函数值作为它的哈希值。

+ 哈希冲突的解决

  > + **链接法（拉链法）**链表
  > + **开放定址法**
  >   + **双重散列法**

+ 实现memcpy

```c++
void *memcpy(void *dst, const void *src, unsigned int n)
{
     if((src==nullptr)||(dst==nullptr)){
       return ;
     }
     while(n--){
       *dst++=*src++;
     }
   	 return dst;
 }
```

+ 二维码的原理

> 待整理：https://www.cnblogs.com/jamaler/p/12610349.html
