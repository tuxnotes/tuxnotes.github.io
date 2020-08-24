# 先看serverfault解释

file-max is the maximum File Descriptors (FD) enforced on a kernel level, which cannot be surpassed by all processes without increasing. The ulimit is enforced on a 
process level, which can be less than the file-max.

There is no performance impact risk by increasing file-max. Modern distributions have the maximum FD set pretty high, whereas in the past it required kernel recompilation 
and modification to increase past 1024. I wouldn't increase system-wide unless you have a technical need.

The per-process configuration often needs tuned for serving a particular daemon be it either a database or a Web server. If you remove the limit entirely, that daemon 
could potentially exhaust all available system resources; meaning you would be unable to fix the problem except by pressing the reset button or power cycling. 
Of course, either of those is likely to result in corruption of any open files.

# linux最大文件句柄数量之（file-max ulimit -n limit.conf）

到底最大文件数被什么限制了？too many open files错误到底可以通过什么参数控制？网上的很多文章说的大致步骤是没有错的，大致如下：

shell级限制
通过ulimit -n修改，如执行命令ulimit -n 1000，则表示将当前shell的当前用户所有进程能打开的最大文件数量设置为1000.

用户级限制  
ulimit -n是设置当前shell的当前用户所有进程能打开的最大文件数量，但是一个用户可能会同时通过多个shell连接到系统，所以还有一个针对用户的限制，通过修改 /etc/security/limits.conf实现，例如，往limits.conf输入以下内容：
root soft nofile 1000
root hard nofile 1200
soft nofile表示软限制，hard nofile表示硬限制，软限制要小于等于硬限制。上面两行语句表示，root用户的软限制为1000，硬限制为1200，即表示root用户能打开的最大文件数量为1000，不管它开启多少个shell。

系统级限制
修改cat /proc/sys/fs/file-max 

但是呢，有很多很重要的细节，有很多错误的描述，一塌糊涂，因此特的在这里做一个说明。

一 、ulimit -n

     网上很多人说，ulimit -n限制用户单个进程的问价打开最大数量。严格来说，这个说法其实是错误的。看看ulimit官方描述：
Provides control over the resources available to the shell and to processes started by  it,  on  systems that allow such control. 
The -H and -S options specify that the hard or soft limit is set for the given resource. A hard limit cannot be increased once it is set; 
a soft limit may  be  increased  up  to  the value of the hard limit. If neither -H nor -S is specified, both the soft and hard limits are set.
The value of limit can be a number in the unit specified for the resource or one of the special values hard, soft,  or  unlimited,  which  stand 
for  the  current hard limit, the current soft limit, and no limit,  respectively.
If limit is omitted, the current value of the soft limit  of  the  resource  is  printed,  unless  the  -H  option is given. 
When more than one resource is specified, the limit name and unit are  printed before the value.
 
人家从来就没说过是限制用户的单个进程的最大文件打开数量，看看红色部分，是限制当前shell以及该shell启动的进程打开的文件数量。为什么会给人限制单个线程的最大文件数量的错觉，
因为很多情况下，在一个shell环境里，虽然可能会有多个进程，但是非常耗费文件句柄的进程不会很多，只是其中某个进程非常耗费文件句柄，比如服务器上运行着一个tomcat，
那么就是java进程要占用大多数文件句柄。此时ulimit设置的最大文件数和java进程耗费的最大文件数基本是对应的，所以会给人这样的一个错觉。

   
还有，很多文章称ulimit -n 只允许设置得越来越小，比如先执行了ulimit -n 1000，在执行ulimit -n 1001，就会报"cannot modify limit: Operation not permitted"错误。
这个其实也是不准确的说法。首先要搞清楚，任何用户都可以执行ulimit，但root用户和非root用户是非常不一样的。

非root用户只能越设置越小，不能越设置越大

我在机器上以非root先执行：

[wxx@br162 etc]$ ulimit -n 900
执行成功，再增大：
[wxx@br162 etc]$ ulimit -n 901
-bash: ulimit: open files: cannot modify limit: Operation not permitted
增加失败，如果减少则是OK的：

[wxx@br162 etc]$ ulimit -n 899
如果再增加到900是不行的：
[wxx@br162 etc]$ ulimit -n 900
-bash: ulimit: open files: cannot modify limit: Operation not permitted

root用户不受限制

首先切换到root：

[wxx@br162 etc]$ sudo su -
查看下当前限制：
[root@br162 ~]# ulimit -n
1000000

增大：
[root@br162 ~]# ulimit -n 1000001
可以成功增大，再减小：
[root@br162 ~]# ulimit -n 999999
减小也是成功的，再增大：
[root@br162 ~]# ulimit -n 1000002
也是ok的，可见root是不受限制的。 

ulimit里的最大文件打开数量的默认值
如果在limits.conf里没有设置，则默认值是1024，如果limits.con有设置，则默认值以limits.conf为准。例如我换了一台机器，登录进去，ulimit -n显示如下：

[root@zk203 ~]# ulimit -n
2000

这是因为我的limits.conf里的文件打开数是2000，如下：

[root@zk203 ~]# cat /etc/security/limits.conf
root soft nofile 2000
root hard nofile 2001

如果limits.conf里不做任何限制，则重新登录进来后，ulimit -n显示为1024。

[root@zk203 ~]# ulimit -n
1024

ulimit修改后生效周期
修改后立即生效，重新登录进来后失效，因为被重置为limits.conf里的设定值
 
二、/etc/security/limits.conf

网上还有缪传，ulimit -n设定的值不能超过limits.conf里设定的文件打开数（即soft nofile）
好吧，其实这要分两种情况，root用户是可以超过的，比如当前limits.conf设定如下：
root soft nofile 2000
root hard nofile 2001

但是我用root将最大文件数设定到5000是成功的：
[root@zk203 ~]# ulimit -n 5000
[root@zk203 ~]# ulimit -n
5000
但是非root用户是不能超出limits.conf的设定，我切换到wxx，执行命令如下：

[wxx@zk203 ~]# ulimit -n 5000
-bash: ulimit: open files: cannot modify limit: Operation not permitted

所以网上的说法是错误的，即使非root用户的最大文件数设置不能超过limits.conf的设置，这也只是一个表象，实际上是因为，每个用户登录进来，ulimit -n的默认值是
limits.conf的 soft nofile指定的，但是对于非root用户，ulimit -n只能越来越小，不能越来越大，其实这个才是真正的原因，但是结果是一样的。

修改了limits.conf需要重启系统？

这个说法非常搞笑，修改了limits.conf，重新登录进来就生效了。在机器上试试就知道了，好多人真的很懒，宁愿到处问也不愿意花一分钟时间实际操作一下。
 
三、/proc/sys/fs/file-max

网上说，ulimit -n 和limits.conf里最大文件数设定不能超过/proc/sys/fs/file-max的值，这也是搞笑了，/proc/sys/fs/file-max是系统给出的建议值，
系统会计算资源给出一个和合理值，一般跟内存有关系，内存越大，改值越大，但是仅仅是一个建议值，limits.conf的设定完全可以超过/proc/sys/fs/file-max。

[root@zk203 ~]# cat /proc/sys/fs/file-max
1610495

我将limits.conf里文件最大数量设定为1610496，保存后，打印出来：

[root@zk203 ~]# cat /etc/security/limits.conf
root soft nofile1610496
root hard nofile1610496 
 
四、总结一下

    /proc/sys/fs/file-max限制不了/etc/security/limits.conf
    只有root用户才有权限修改/etc/security/limits.conf

    对于非root用户， /etc/security/limits.conf会限制ulimit -n，但是限制不了root用户
    对于非root用户，ulimit -n只能越设置越小，root用户则无限制

    任何用户对ulimit -n的修改只在当前环境有效，退出后失效，重新登录新来后，ulimit -n由limits.conf决定

    如果limits.conf没有做设定，则默认值是1024
    当前环境的用户所有进程能打开的最大问价数量由ulimit -n决定
    
# /proc/sys/fs/file-max VS ulimit -n

简单的说， max-file表示系统级别的能够打开的文件句柄的数量， 而ulimit -n控制进程级别能够打开的文件句柄的数量。 

man 5 proc， 找到file-max的解释：
    file-max中指定了系统范围内所有进程可打开的文件句柄的数量限制(系统级别， kernel-level)。 
    （The value in file-max denotes the maximum number of file handles that the Linux kernel will allocate）。
    当收到"Too many open files in system"这样的错误消息时， 就应该增加这个值了。
    
    # cat /proc/sys/fs/file-max
    4096
    # echo 100000 > /proc/sys/fs/file-max  --立即生效，重启后失效（其值恢复为sysctl.conf值或默认参数）

    或者
    # echo ""fs.file-max=65535" >> /etc/sysctl.conf  --修改后需要重新加载参数文件 sysctl -p
    # sysctl -p
    
    file-nr 可以查看系统中当前打开的文件句柄的数量。 他里面包括3个数字： 第一个表示已经分配了的文件描述符数量， 第二个表示空闲的文件句柄数量， 
    第三个表示能够打开文件句柄的最大值（跟file-max一致）。  内核会动态的分配文件句柄， 但是不会再次释放他们（这个可能不适应最新的内核了， 在我的file-nr中看到第二列一直为0， 第一列有增有减）


man bash， 找到说明ulimit的那一节：
    提供对shell及其启动的进程的可用资源（包括文件句柄， 进程数量， core文件大小等）的控制。 这是进程级别的， 也就是说系统中某个session及其启动的每个进程能打开多少个文件描述符，
    能fork出多少个子进程等... 
    当达到上限时， 会报错"Too many open files"或者遇上Socket/File: Can’t open so many files等
    
    另外需要注意的是， 每种资源都有相关的软硬限制， 软限制是内核强加给相应资源的限制值，硬限制是软限制的最大值。 非授权调用进程只可以将其软限制指定为0~硬限制范围中的某个值，
    同时能不可逆转地降低其硬限制。授权进程可以任意改变其软硬限制。RLIM_INFINITY的值表示不对资源限制。
    分别使用-H和-S选项来指定需要对资源是做硬限制/软限制的设置。 如果都不指定， 硬限制和软限制同时设置。 
    打印资源的限制值， 如果不明确指定-H， 打印的是-S
   
    要改apache的ulimit， 可以在 /usr/sbin/apachectl 这个脚本中修改 ULIMIT_MAX_FILES 这个值


    可打开文件句柄数设置的太大， 有那些危害：
    If the file descriptors are tcp sockets, etc, then you risk using up a large amount of memory for the socket buffers and other kernel objects; this memory is 
    not going to be swappable.
    另外要记得的是socket connection也是文件。 
