# ==需要常看==

21集，关于栈帧，栈堆，计数器的关系

设计模式，两阶段终止37-40

p56，对象锁的底层

（p60-62）线程八锁习题

==p65==变量的安全测评一定要多看

线程安全习题P69-74

==P76集多看一定要==（Monitor）

P77多看（字节码文件的）

P85集多看（批量重偏向）

==应为这两个方法只有重量级锁有，等待唤醒机制只有重量级锁才有，偏向锁和轻量级锁没有==

# 属于自己的JUC

# 第三章

## 3.4 * 原理之线程运行

### 栈与栈帧

我们都知道 JVM 中由堆、栈、方法区所组成，其中栈内存是给谁用的呢？其实就是线程，每个线程启动后，虚拟 机就会为其分配一块栈内存。

**每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占用的内存**

**每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法**



### 线程上下文切换

因为以下一些原因导致 cpu 不再执行当前的线程，转而执行另一个线程的代码

因为以下一些原因导致cpu 不再执行当前的线程，转而执行另一个线程的代码
·线程的 cpu 时间片用完
·垃圾回收
·有更高优先级的线程需要运行
·线程自已调用了 sleep、yield、wait、join、park、synchronized、lock 等方法

当Context Switch 发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，java 中对应的概念就是程序计数器(Program Counter Register)，它的作用是记住下一条jvm 指令的执行地址，是线程私有的。

**状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等
**Context Switch 频繁发生会影响性能



## 3.5常用方法

![image-20241127165651301](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127165651301.png)

![image-20241127165821241](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127165821241.png)java中状态有6种

小结
直接调用run 是在主线程中执行了run，没有启动新的线程
使用 start 是启动新的线程，通过新的线程间接执行run 中的代码

![image-20241129105930805](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129105930805.png)

#### 线程优先级（要看cpu忙不忙）

### join方法

等某个线程结束运行，才会结束，会暂时阻塞）

##### 同步异步概念

需要等待结果返回，才能继续运行就是同步 

不需要等待结果返回，就能继续运行就是异步



### ==interrupt 方法详解==

#### 打断 sleep,wait,join 的线程（会清空标记的阻塞线程）

*可以*打断 sleep,wait,join 的线程（也就是阻塞状态的线程）**

这几个方法都会让线程进入阻塞状态

<u>打断阻塞线程会发出一个打断异常，一般会捕获。然后打断标记变成true再变成false</u>



####  **打断正常运行的线程**

打断正常运行的线程，不会出现异常，也不会清空打断状态（true）



<u>打断所有线程（正常+阻塞）会出现一个打断标记，布尔值，就是说你这个线程以前有没有被打断过，干扰过。</u>

但是很奇怪，当你打断的是一个阻塞线程，打断标记会先设置为true（interrupt概念），然后再里面恢复成false（阻塞线程性质）

![image-20241129112153390](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129112153390.png)

一般会得到打断状态，来判断是否停止线程，更加优雅







#### ==设计模式之两阶段终止==

就是教你如何优雅的终止线程，给他料理后事的机会

![image-20241129112357678](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129112357678.png)

共享资源没有机会释放就直接结束

一般实现方式过程（有代码实例）

![image-20241129112619412](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129112619412.png)

这个要注意，我们常用的

![image-20241129114244469](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129114244469.png)

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129114301531.png" alt="image-20241129114301531" style="zoom: 33%;" />

#### 打断park线程（不会清空标记的阻塞线程）

park方法是第三方的，是所得支持类



## 3.6主线程和守护线程

==主线程就是main，虚拟机线程==

<u>主线程和java进程是两个概念。当你有两个线程，一个主线程一个附属线程</u>

<u>主线程结束，附属线程继续运行，那么你的java进程就不会终止</u>



在finally中一定会执行，保证了即使是出现其它异常也能够执行响应的善后操作，所以这里建议写在finally语句块中

守护线程的使用场景，GC（垃圾回收）当对象不被其他对象引用，就会被回收

垃圾回收线程都是守护线程，当其他线程都停止，垃圾回收线程不回收垃圾了，直接停止

![image-20241127193341886](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127193341886.png)

非守护线程结束，守护线程必然立刻结束

## 3.7线程状态

java线程状态有两种说法

5种状态（从操作系统层来讲）

![image-20241127194221032](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127194221032.png)

6种状态（从java的API来讲，枚举）

![image-20241127194338384](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127194338384.png)

3+3（阻塞状态被细分为三种）

## 3.8第三章小结

![image-20241129115056872](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129115056872.png)

状态，5-6种。









## 重点！

![image-20241127165257493](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127165257493.png)

真正意义上的上下文切换。

线程过多导致核心数不足以继续完当前线程，要去进行别的线程

此时当前线程必须保存程序计数器和栈空间，堆空间数据。以便后面cpu核心有空闲继续执行。==也就是cpu的<u>时间片</u>给到当前线程==

而保存这些数据会浪费大量资源

==这！  就是系统并发量上来的时候，性能下降的根本原因！！！==





## --------------------------------



![image-20241129135648439](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129135648439.png)

==我把java并发呢后续的这些知识分成了两大模型，共享模型和非共享模型==

那第四章呢是共享模型中一个章节叫做管程。==通过悲观锁的思想来解决并发问题==

之后还会讲（JMM）共享模型中的java内存模型

# 第四章（就是管程）

![image-20241129115222759](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129115222759.png)



## 4.1共享问题

小故事（多线程下会出现的问题）

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129140724082.png" alt="image-20241129140724082" style="zoom:33%;" />

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129140740115.png" alt="image-20241129140740115" style="zoom:33%;" />

原因是由于，java的代码命令不是原子性的。字节码文件有很多语句

==指令交错和上下文交换=线程问题==

==我们使用多线程的目的就是为了提高效率，不然还不如不用==





#### 临界区 Critical Section

·一个程序运行多个线程本身是没有问题的
      ·问题出在多个线程访问共享资源
  。多个线程读共享资源其实也没有问题
   。在多个线程对共享资源读写操作时发生指令交错，就会出现问题
==·一段代码块内如果存在对共享资源的多线程读写操作，称这段代码块为临界区
例如==

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129141633410.png" alt="image-20241129141633410" style="zoom:25%;" />

#### 竞态条件 Race Condition

多个线程在临界区内执行，由于代码的执行序列不同而导致结果无法预测，称之为发生了竞态条件



## ==4.2 synchronized（对象锁） 解决方案==



![image-20241127201602266](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127201602266.png)

解决方案分为阻塞式和非阻塞式

synchronized必须和对象合作使用

synchronized，来解决上述问题，即俗称的【对象锁】，它采用互斥的方式让同一 时刻至多只有一个线程能持有【对象锁

#### synchronized语法

![image-20241129141949282](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129141949282.png)

**这个图不咋重要**

![image-20241127201919784](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127201919784.png)

你指定的锁对象就如同一个房间，你拥有这个锁对象，就相当于有对应的房间的钥匙

但是一个房间只有一个专属钥匙（锁对象）

**<u>这个图重要</u>**

CPU多核和单核最大的不同是是否支持并行操作，多核CPU支持多个线程同时进行获取锁的操作，但是最终CPU调度器只会将锁给其中一个线程，此时便将并行操作变成了串行操作，所以加锁是会有性能损耗的。

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127202715070.png" alt="image-20241127202715070" style="zoom:50%;" />

这个图，一定要一步一步看，从中体现出锁的底层原理。



1.持有锁，是指持有锁对象（也就是任意对象，你把他作为锁，放到关键字里，他就是锁对象）

2.当你拿到锁的线程，时间片没了之后，依旧会停止。这时候其他线程可能会抢到执行权。但是他没有锁

3.==判断自己没有锁后会阻塞==，而不会继续抢夺执行权。所以比一个阻塞一个。防止一直抢夺。转化上下文消耗性能

#### 思考

![image-20241127203300111](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127203300111.png)

**思考**



之前我们都是面向过程来加锁，编辑。但是思路不清晰。

#### 面向对象思路来加工一下对象锁

![image-20241127204426440](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127204426440.png)

将处理的数据放入对象。每一个堆数据处理的方法加锁

## 4.3方法上的synchronized

![image-20241127204600444](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241127204600444.png)

synchronized加载成员方法上，锁住的其实是this对象（this）

synchronized加载成员方法上，锁的住其实是类对象（A.class）

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129143039430.png" alt="image-20241129143039430" style="zoom:67%;" />

无法保证原子性

#### 线程八锁（方法上的锁例题）

（p60-62）



## ==4.4变量的线程安全分析==

### ==变量的线程安全分析==

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129143333067.png" alt="image-20241129143333067" style="zoom:33%;" />

1.成员变量和静态变量看共享没有，共享之后看读写操作。写一般有线程安全问题

2.局部变量一般安全，但他引用的对象未必。如果是堆中的对象，可能被共享

==如果该对象逃离方法的作用范围，需要考虑线程安全==



这里有几个例题P65，66可以看看

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129144857583.png" alt="image-20241129144857583" style="zoom:50%;" />

方法访问修饰符带来的思考，如果把 method2 和 method3 的方法修改为 public 不会代理线程安全问题？
情况1 :有其它线程调用 method2 和 method3

情况2：在情况1的基础上，为 ThreadSafe 类添加子类，子类覆盖 method2或method3方法，即

情况1:

将两个私有方法定义为public公共的，并用两个不同的线程去调用方法不会有线程安全问题。因为两个list对象不是一个对象。

情况2:有问题，

list的对象是同一个。这时候在方法内部调用method3，创建一个新线程并使用list的局部变量，这个list是同一个。会有问题



==<u>**解决方法**</u>==（及其过于重要）

==1.对于可以私有的方法，直接杜绝使用public，使用private。这样子类就无法覆盖他了。不能重写==

==2.对于必须要公开的public方法，最好用final防止子类重写。导致安全问题==



从这个例子可以看出 private 或 ﬁnal 提供【安全】的意义所在，来提高线程安全性。

所以我们就知道平时说的private和final可以提供安全，这个安全就是线程安全

这也是我们啊这个面向对象编程原则中啊一个很重要的原则，开闭原则中的close



### 常见的线程安全类

![image-20241128104732908](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241128104732908.png)

用idea给我们包装的线程安全类

JUC= java.util.concurrent 包 JDK5之后出现

线程安全类能线程安全，一定是加了锁的，但效率不如其他的

#### 组合调用

==虽然每一个方法都可以保证原子性，但是组合不一定有原子性，外部要加锁==

如下

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129151233865.png" alt="image-20241129151233865" style="zoom:25%;" />

#### 不可变类线程安全性

![image-20241129151356052](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129151356052.png)

只是创建了新的，而不是线程安全。



## 4.5习题

## -----------------------------

接下来更深入的学习对象锁的底层synchronized

## ==4.6Monitor概念==

### java对象头

对象体里存成员变量，头就是上面两个

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129160842412.png" alt="image-20241129160842412" style="zoom:33%;" />

Mark Word (32 bits)     里面是详细的信息，如下

正常状态下（哈希code），（分代年龄 垃圾CG）  （加锁状态）

Klass Word(32 bits)，指向了从属的类（本质是一个4字节的指针），指定是什么类

![image-20241128112718992](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241128112718992.png)

一个int类型     是基本数据类型   占四个字节

一个Interger    是对象                   占12个字节    8（对象头）+4（包含的int类型基本变量）

接近大三倍，一般为了节省可以int

![image-20241128112857769](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241128112857769.png)

一个对象占8个字节



好那接下来呢我们来看一个非常重要的概念

### ==Monitor(synchronized底层的原理，也就是java里真正的锁！)==

synchronized是对象锁，是锁的一种，那真正的锁就是Monitor

synchronized是基于Monitor实现的

Monitor是操作系统提供的

==P76集多看一定要==

==你给对象加锁，就相当于和Monitor进行关联==

#### 原理之synchronized

![image-20241128224830262](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241128224830262.png)

任何一个对象都可以和锁关联，关联就是在任意对象里加一个锁（Monitor）的地址，指向Monitor对象

Monitor是操作系统给的对象（C++或者C写的）

里面有几个属性，Owner是锁主人

entryList是阻塞队列，别的线程尝试获取锁回到这里，==阻塞==

![image-20241128113655718](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241128113655718.png)

![image-20241128113946318](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241128113946318.png)

![image-20241128114106888](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241128114106888.png)



##### ==字节码文件==

P77多看

![image-20241129153844918](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129153844918.png)

1.这时候将对象和锁对象进行关联（加锁），加锁对象的对象头本来会存哈希code和分代年龄，加了锁之后会变成锁的地址值。对象头的哈希code和分代年龄被存在monitor锁里了

2.解锁的时候会从锁对象里还回来，把锁monitor的地址删除，换回原本的数据

3.异常表，字节码文件里的，jvm讲了。监视从哪里到哪里等等

4.这一部分是我们代码外的，是处理异常的，可看到出现异常就会执行

#### ==原理之synchronized进阶==

synchronized（重量级锁）

让每个对象都关联个monitor，它是由操作系统提供的。使用成本比较高

每次进入synchronized都要获取monitor。那对我们的程序运行的性能是有影响的

所以JDK6之后，就对他获取锁的方式进行了优化

==从直接使用monitor锁进行优化，开始使用一种叫轻量级锁，一种叫偏向锁来进行优化==

==接下来我们就讲解，轻量级锁和偏向锁的原理==

##### 一个小故事

房间门上-防盗锁-Monitor
书包-轻量级锁
刻上名－偏向锁
批量重刻名－一个类的偏向锁次数撤销到达20

超过40次改名字，不可用偏向锁，用轻量级锁挂书包



偏向锁-》轻量级锁—》重量锁

![image-20241129155311700](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129155311700.png)

没有竞争的时候轻量级锁，当有竞争，把锁升级为重量级锁

如果只有一个线程的时候，基本另一个不用，就写下自己的名字。连书包都不用挂了。这就是偏向锁（专属于某个线程使用）。假如出现竞争，就会升级为轻量级锁





##### ==1.轻量级锁==

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129160533498.png" alt="image-20241129160533498" style="zoom:33%;" />



轻量级锁是应对少量碰撞的，重量级锁是为了应对多次碰撞的。如果一个资源在同一时间要被多个线程使用，就应该用重量级锁

==交替无竞争是偏向锁，竞争小轻量级，竞争大重量级===

首先他们每次调用异常方法，里面都会出现一个栈帧

在栈帧里会出现一个叫锁记录的对象

他不是java层面的，是jvm的对象。

里面包含两部分，一部分是里面包含了一个对象指针（就是加锁对象的堆地址），另一部分存储我们未来要加锁的对象的对象头（markdown）

###### 加锁步骤1

![image-20241129161808024](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129161808024.png)

步骤一，先用锁记录对象里的属性关联Object对象的地址

###### 加锁步骤2

把锁记录里的数据（上面一层的）和mark里的数据做个交换

这里左右分别是00和01，00代表轻量级锁的状态，01代表不加锁状态

![image-20241129161613808](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129161613808.png)

![image-20241129161527068](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129161527068.png)

同时，mark还会变成记录锁记录地址



###### 结果

![image-20241129162154587](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129162154587.png)

==这里我们提到一个cas操作，未来会讲==

目前我们只需要指定他就是完成这两个数据的替换操作，是一个原子性的就行了啊



###### 加锁失败的两种情况

![image-20241129162648072](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129162648072.png)

1.这是成功情况。如果mark已经被别的改成00了，也就是说别人已经加锁了，这是说明有竞争了，要升级重量级锁。就不会加锁成功

2.假如你是重入锁，一个线程中连续加了两次一样的锁，就会加锁失败，但是可以从地址值做判断知道是有一次给同一个对象加锁了

这个锁记录还是会被创建出来，因为他也是有意义的

只不过它的这个数据这不是在存储那个more word了，而是null。这样你就知道加了几次重入锁了。解锁一次去掉一个。



###### 解锁步骤1

![image-20241129163204843](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129163204843.png)

###### 解锁步骤2

![image-20241129163225522](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129163225522.png)

当不是null，说明是最后一个。这时候要用cas来恢复两个的值



## --------------------------------

##### 2.锁膨胀

这是成功情况。如果mark已经被别的改成00了，也就是说别人已经枷锁了，这是说明有竞争了，要升级重量级锁。就不会加锁成功

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129163431440.png" alt="image-20241129163431440" style="zoom:33%;" />

0已经加锁，1不能成功

1就会进入锁膨胀的流程

所谓的锁膨胀就是，让接下来的解锁操作变成重量级锁的解锁

###### 步骤1

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129163936119.png" alt="image-20241129163936119" style="zoom:33%;" />

由于不是重量级锁，不会阻塞，所以1就去==为Object申请重量级锁==

重要的是对象和线程之间的指向线条。

先将指向地址变成重量级锁，然后后面变成10（重量级锁）

然后1先阻塞



###### 步骤2

1进入阻塞，等0结束同步代码块会准备解锁

这回它解锁的时候就跟之前有些不样了啊

![image-20241129164341377](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129164341377.png)

0结束后还原mark的值给对象头的时候肯定会失败，因为锁膨胀了，现在object的mark地址已经变成重量级锁地址了。后两位也变成10

所以肯定会恢复失败

==这时候就进入重量级锁的解锁流程==





##### 3.自旋优化

就是先不要进行阻塞，先循环几次，看看能不能拿到锁。==因为阻塞要进入上下文切换。比较耗费性能==

但是首先自旋肯定要用cpu（毕竟多个线程同时运行，执行一些功能）

所以自旋优化适合多核cpu

单核cpu基本就不用了

自旋优化成功

![image-20241129165358039](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129165358039.png)

自旋优化失败

![image-20241129170006543](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129170006543.png)

注意点

![image-20241129170028391](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129170028391.png)

在Java6之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会高，就多自旋几次；反之，就少自旋甚至不自旋，总之，比较智能。
==自旋会占用CPU时间，单核CPU自旋就是浪费，多核CPU自旋才能发挥优势。==
Java7之后不能控制是否开启自旋功能

## --------------------------------



##### ==4.偏向锁（对轻量级锁的优化）==

###### 

从JDK15起，偏向锁已被废弃， JDK20被移除， 可以在JDK8中将其关闭以提高性能

==原来线程栈中的锁记录（轻量级锁）来代替monitor（重量级锁）==

轻量级锁也有一些缺点

轻量级锁在没有竞争时（就自己这个线程），每次重入仍然需要执行CAS操作。

如下操作

![image-20241129171027935](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129171027935.png)

![image-20241129171107867](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129171107867.png)

所以引入偏向锁对其进行优化

就是重入的时候做判断，而不需要cas过程了

偏向锁就是无锁，不存在重入，下次来发现是自己的名字，代表无竞争

![image-20241129171314365](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129171314365.png)

###### 偏向状态

![image-20241129171549625](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129171549625.png)

1.是否启用偏向状态，就是说现在是不是可以加偏向锁的状态

如果是偏向状态，就是线程id和epoch，而不是hash

==接下来我们会去查看对象头是不是101结尾的==

那我怎么去查看这个对象的对象头呢？这个要借助一个第三方的jar

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129171953723.png" alt="image-20241129171953723" style="zoom: 33%;" />

查看不是001,为什么，因为他是延迟执行的。所以几个办法

1.可以加入虚拟机参数才可以

2.可以睡眠几秒钟



![image-20241129172843698](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129172843698.png)

==这时候知道dog对象可以加偏向锁，但此时还没有加。所以等他延迟加载后，之间给这个dog对象加锁就可以了。默认加的是偏向锁先==

第一行：前面没有，后面101说明现在支持偏向锁

第二行：前面多了一些东西，就是101情况下mark里的数据，如线程id。这里的就是主线程的线程id

第三行：明明解锁，为什么和第二行一样，正如故事里说的。他默认把这个锁交给主线程了。这个对象已经偏向于主线程了，以后就给主线程用了，里面就是主线程线程id

###### 

==当我们多线程情况下，可以直接禁用掉偏向锁。不然更浪费资源==

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129173323520.png" alt="image-20241129173323520" style="zoom:33%;" />

###### ==Hashcode的问题==

==有一个很奇怪的地方，当你调用这个对象的hashcode的时候会禁用这个对象的偏向锁==

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129173413277.png" alt="image-20241129173413277" style="zoom:33%;" />

思考为什么呢？

![image-20241129173629639](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129173629639.png)

1.hashcode默认为空，当你需要用的时候，调用函数才会产生，并且保存

2.因为你已经没有空间存刚刚产生的32位的hashcode了。

3.所以他会撤销你的偏向状态，去存hashcode，去掉线程id那些。

也要注意





###### 撤销

偏向状态除了Hashcode问题之外，还有一些方法可以撤销

1.比如，其他线程使用，这样就不是偏向了，还会变成轻量级锁

本质就是把偏向状态变成不可偏向

![image-20241129174725711](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129174725711.png)

2.调用wait/notify方法

为什么呢？==应为这两个方法只有重量级锁有，等待唤醒机制只有重量级锁才有==





###### ==批量重偏向（P85集多看）（‘20’）==

就是将20个名字印记全部抹除

那撤销偏向呢其实对性能的损耗也是有些大啊，其实之前的情况没有竞争。

==他就是一个线程来了，等他用完了之后，另一个线程才用。这把偏向状态解除了，但是两个发生时间不在一块。不会发生竞争关系==

==批量重偏向就是对他的优化==

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129175524321.png" alt="image-20241129175524321" style="zoom:33%;" />

不是降级，默认就是偏向级的，只不过达到20次后直接转为轻量级的（个人理解）



1.==循环30次，分别给30个对象加锁，这里都是偏向锁==，然后这些对象都会偏向于t1线程

==2.等线程1执行完了，再让线程二分别加锁（防止竞争让锁升级）==

3.这时候又让t2线程，分别一个个给对象加锁，这时候是线程二调用的。所以30个对象会从偏向于线程1到撤销偏向状态

4.这时候，==前20个==对象从偏向锁变成轻量级锁。变成不可偏向的。然而，连续20个对象都偏向新的==，这时候jvm就思考会不会之前偏向错线程了==

5.所以从第20个开始就不会撤销偏向状态了（毕竟他很消耗性能），而是直接将后面10个对象的偏向状态从偏向1线程转换到偏向2线程



==P85集多看==

###### 批量撤销（‘40’）

![image-20241129181328425](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129181328425.png)

==撤销到40个的时候，jvm会觉得自己又偏向错了。因为他发现。根本就不应该偏向！==

新建出来的类也会变成不可偏向



###### ==锁消除（JIT的即时编译）==

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129182106745.png" alt="image-20241129182106745" style="zoom:33%;" />

定义两个方法，都是对静态变量做++操作。不同的是一个加锁一个不加

然后看他们运行速度对比，jvm加热运行

先打jar包。target目录下运行

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129182144107.png" alt="image-20241129182144107" style="zoom: 50%;" />

第一个是求运行的平均时间，第四个是纳秒为单位，速度比较快，毫秒不够

第二个是预热次数，让代码充分优化，得到最真实的结果

第三个就是说我要测试几轮



最后的到结果，几乎两个方法速度一样，为什么呢，明明加锁有一定的性能优化，即便我们这里进行了轻量级锁和偏向锁的优化，也会有损耗

<img src="C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129182357903.png" alt="image-20241129182357903" style="zoom: 50%;" />

为什么这么近呢？

==原因==

java运行时候，他有一个叫JIT的一个即时编译器，会对我们的java字节码文件进行一个进一步的优化。

java是一个解释加编译的语言

虚拟机对我们的字节码文件是解释的方式来运行的。但是他会对一些热点代码

==也就这个反复执行啊的循环反复调用的方法，超过一定阈值就会用JIT即时编译器进行优化==

优化有几个方向：首先，看你是不是局部变量。发现你是一个局部变量，那么不可能被线程共享。所以JIT就会把synchronized这个关键字直接优化去掉



当然这个JIT可以手动关闭，关闭前后，效率对比

![image-20241129182858059](C:\Users\20520\AppData\Roaming\Typora\typora-user-images\image-20241129182858059.png)







###### 锁粗化（了解，不一定存在）

就是，当多个方法重复调用锁synchronized ，比如在for 循环中，就可以相当于在synchronized中进行for循环，进行粗化
