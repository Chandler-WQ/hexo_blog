---
title: Golang
date: 2022-10-30 16:33:44
tags:
---
#  一、一些常识
命名类型：例如 int、int64、float32、string、bool 等预先声明类型。
未命名类型：包括 array、struct、pointer、function、interface、slice、Map 和 channel，
值传递：大部分
引用传递：map , slice ,channel
通道：关闭(close)通道后，写：不能写入，写入会直接panic。若通道存在缓冲区则可以读完，读完之后也可以读，但是会为0，可以用ok-idom进行判断是否关闭
变量：a := 100，这种简短模式不能定义在函数外部
常量 :

```go
const (
x int64 = 120
y //和x相同 
s = "abc"
z //和s相同
 
)
```

枚举

```go
const (
h = iota // 一个语法糖 0
i //自动注入1
j //自动注入2
k = 100
l //也为100
o = iota //此时从5开始 
p //自动注入6
)
```

 
类型转换：
String->int32

```go
num,_ := strconv.ParseInt("1000",2,32) //str 进制 位数
```

Int ->string

```go
int, err := strconv.Atoi(string)
```

int转成string：

```go
 string := strconv.Itoa(int)
 
```

int64转成string：

```go
string := strconv.FormatInt(int64,10)
```

 

```go
var num64 int64
var num32 int32
num32 = int32(num64)
num64 = int64(num32)
```

 
引用类型：slice,map,channel， 必须用make进行创建，new创建只是分配了类型本身的内存，并不会分配存的值的类型
命名类型：bool,int string等等
未命名类型：数组，切片，字典，通道等
go语言怎么实现自定义类型的比较，sort函数
go语言中优先队列，linkedlist,栈队列等数据结构
只有后缀自增，并且只能作为单独的语句执行。
零长度的struct{}
var a , b struct{}
 
a和b不为nil,并且相等
 
# 二、函数

* 花括号不能另起一个
* Func只能与nil判等
* 不定项参数本质就是一个切片
* 返回值命名的话可以return空
* 匿名函数可以当作结构体字段，作为函数参数，经过通道传递
* 闭包通过指针引用环境变量，导致环境变量的生命周期延长
* defer是filo的结构，先进后出
* 如果爆出多个panic则只有最后一个会被捕获
# 三、数据

## 3.1 字符串
* 字符串默认是"".
* 字符串是不可变的
* 支持!=,==,<,>,+,+=等操作
* 允许通过索引访问字节数组（不是字符），但是不能获取元素地址
* “`”这个定义的字符串格式不会做处理
* 遍历字符串时如果是用unicode字符则可以用range进行遍历，他会基于rune进行判断，rune可以遍历中文字符
* 修改字符串可以将其转变成rune或者byte，然后修改再转变回来，但是它是基于复制的，每次变化都会复制
* type byte = uint8
 
type rune = int32
 
累加提高性能：1. 弄成数组然后string.Join(s,"") 2.用bytes.Buffer 通过 .Grow()初始化设置大小
## 3.2 数组
* 长度是数组的组成部分
* ...仅在第一维可以使用，len和cap都返回的第一维的长度
* 若元素类型支持== 或者!=，那么数组也支持，当长度/类型不一样的时候直接会编译不通过
* go语言中数组是值类型的 ，赋值和传参都是会复制整个数组数据
## 3.3 切片
* 切片本身底层也是数组，可以理解为它是数组的一个窗户，如果多个切片底层是同一个数组，这个数组改变所有的切片都会改变
* a[x:y:z] cap与z有关系, len与y有关系 x和y 为从下标从0开始的左闭右开关系
* 不支持==，!=操作，只能与nil进行比较
* append追加，最好刚开始的时候a : =make([]int ,0,x)，x设置的范围刚好，这样子就减少底层数组的重新复制次数，减少扩容，每一次扩容都会扩容2cap，当然如果底层slice超级大的时候，会尝试扩容1/4,不够再说
* 复制 copy（a,b），将b复制给a,若a,b指向同一个底层数组则允许目标区间重叠，最终复制长度以较小的为准
* 优化：如果用到了大数组中的小切片，那样最好新建一个数组，这样的就不会因为这个切片让大数组的垃圾回收变缓
## 3.4 字典
* 字典要求key必须支持==，!=zxc78`1
* 访问不存在的key会返回0，当然可以用ok_idiom
* v, ok := m['a']
 
迭代每一次顺序都可能不一样
* value不能访问地址，所以不能直接修改value的值，可以取出然后重新赋值
* 不能对nil的字典进行写操作，但是能进行读操作，
* 字典不允许并发不加锁的读写，会造成崩溃，使用sync.RWMutex
* make(map[T]T ,x)，预先先准备足够的空间，可减少扩容的次数
* New和make的差别，new只是为当前的结构分配了地址，并没有真正的初始化，而make则进行了初始化
结构：

hash冲突：链地址法，溢出桶，
扩容：超过负载因子6.5两倍扩容，溢出桶过多（超过2^B||2^15）单倍扩容，扩容步骤为增量迁移
key的hash：之后，低8位定位哈希桶位置，高8位定位key的位置
迁移完毕才会进行下一次的扩容
溢出桶、hmap ,bmap
超级赞的两个博客：
https://segmentfault.com/a/1190000018632347
https://segmentfault.com/a/1190000018387055
## 3.5 结构体
* 可以用_进行补全（使用场景是什么呢）
* 只有所有字段都支持== 才可以判等
* struct{}长度都为0,这种长度为0的变量都指向runtime.zerobase变量
* 匿名字段，主要就是嵌入其他的sturct（不一定是struct），需要显示的初始化，但是可以直接通过.访问匿名字段成员， 未命名类型没有名字标识别不能作为匿名字段，不能将基础类型和其指针类型同时嵌入，因为其隐式名字相同
* 初始化会分配内存一次性，并且其内存布局是顺序的
# 四、方法

* 接收器可以为指针和非指针，并且编译器会自动帮忙在指针和非指针上进行转换
* 可以访问匿名字段的方法和成员，但是注意其不属于继承
* 方法集中 T方法集包括接收器T的全部方法，*T包括接收器T和接收器*T的所有方法
* 若匿名嵌入S字段，则T方法集包括S的所有方法，若嵌入*S字段则包含*S+S的所有方法
* 方法隶属的类型其实并不局限于结构体类型，但必须是某个自定义的数据类型，并且不能是任何接口类型。
# 五、接口

* 不能含有字段，不能定义自己的方法、只能声明方法不能实现、可嵌入其他的接口类型(要求不能有同名方法)，若含有匿名接口类型则“父”接口的实现能够转换为子接口的实现
* 实现的接口的类型可以==则可以==
* 将实例赋值接口是进行复制，并且无法修改接口存储的接口复制品，因为它是unaddressable
* 只有接口的的tab和data都为nil时接口才为nil，这里很容易遇到坑就是如果自己实现error方法，然后var 声明之后，即使他没有被初始化也不会nil，因为它是一个有类型的接口类型

```go
type iface struct{
 
tab *itab //类型信息
 
data unsafe.Pointer //实际对象指针
 
}
 
type itab struct{
 
inter *interfacetype //接口类型
 
_type *_type //实际对象类型
 
fun [1]uintptr //实际对象方法地址
 
}
```

 
用 实例.（类型）如a.(fmt.Stringer)方式还原接口类型，并且可以利用ok_idiom
* switch中的fallthrough，会直接执行下一个case
* 技巧可以在init（）函数检查是否实现某一个接口

```go
func init(){
 
var _ fmt.Stringer = x
 
}
```

 
# 六、Sync
## 6.1 sync.mutex
type Mutex struct {

```go
 
state int32 //表示锁的状态
 
sema uint32 //sema 是用于控制锁状态的信号量
 
}
```
互斥锁的加锁过程比较复杂，它涉及自旋、信号量以及调度等概念：
* 如果互斥锁处于初始化状态，就会直接通过置位 mutexLocked 加锁；
* 如果互斥锁处于 mutexLocked 并且在普通模式下工作，就会进入自旋，执行 30 次 PAUSE 指令消耗 CPU 时间等待锁的释放；
* 如果当前 Goroutine 等待锁的时间超过了 1ms，互斥锁就会切换到饥饿模式；
* 互斥锁在正常情况下会通过 sync.runtime_SemacquireMutex 函数将尝试获取锁的 Goroutine 切换至休眠状态，等待锁的持有者唤醒当前 Goroutine；
* 如果当前 Goroutine 是互斥锁上的最后一个等待的协程或者等待的时间小于 1ms，当前 Goroutine 会将互斥锁切换回正常模式；
互斥锁的解锁过程与之相比就比较简单，其代码行数不多、逻辑清晰，也比较容易理解：
* 当互斥锁已经被解锁时，那么调用 sync.Mutex.Unlock 会直接抛出异常；
* 当互斥锁处于饥饿模式时，会直接将锁的所有权交给队列中的下一个等待者，等待者会负责设置 mutexLocked 标志位；
* 当互斥锁处于普通模式时，如果没有 Goroutine 等待锁的释放或者已经有被唤醒的 Goroutine 获得了锁，就会直接返回；在其他情况下会通过 sync.runtime_Semrelease 唤醒对应的 Goroutine；
## 6.2 sync.RWmutex

```go
* type RWMutex struct {
 
w Mutex
 
writerSem uint32
 
readerSem uint32
 
readerCount int32
 
readerWait int32
 
}
 
```

复用互斥锁提供的能力；
* writerSem 和 readerSem — 分别用于写等待读和读等待写：
* readerCount 存储了当前正在执行的读操作的数量；
* readerWait 表示当写操作被阻塞时等待的读操作个数；
* 写操作使用 sync.RWMutex.Lock 和 sync.RWMutex.Unlock 方法；
* 读操作使用 sync.RWMutex.RLock 和 sync.RWMutex.RUnlock 方法；
### 6.2.1 写锁 sync.RWMutex.Lock

1. 调用结构体持有的 sync.Mutex 的 sync.Mutex.Lock 方法阻塞后续的写操作；因为互斥锁已经被获取，其他 Goroutine 在获取写锁时就会进入自旋或者休眠；
2. 调用 atomic.AddInt32 方法阻塞后续的读操作，将readerCount置为负数：
3. 如果仍然有其他 Goroutine 持有互斥锁的读锁（r != 0），该 Goroutine 会调用 sync.runtime_SemacquireMutex 进入休眠状态等待所有读锁所有者执行结束后释放 writerSem 信号量将当前协程唤醒。

```go
func (rw *RWMutex) Lock() {
 
rw.w.Lock()
 
r := atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders) + rwmutexMaxReaders
 
if r != 0 && atomic.AddInt32(&rw.readerWait, r) != 0 {
 
runtime_SemacquireMutex(&rw.writerSem, false, 0)
 
}}
 
```

### 6.2.2 写锁sync.RWMutex.Unlock
1. 调用 atomic.AddInt32 函数将变回正数，释放读锁；
2. 通过 for 循环触发所有由于获取读锁而陷入等待的 Goroutine：
3. 调用 sync.Mutex.Unlock 方法释放写锁；

```go
func (rw *RWMutex) Unlock() {
 
r := atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders) //变成正数
 
if r >= rwmutexMaxReaders {
 
throw("sync: Unlock of unlocked RWMutex")
 
}
 
for i := 0; i < int(r); i++ {
 
runtime_Semrelease(&rw.readerSem, false, 0)
 
}
 
rw.w.Unlock()
 
}
 
```

### 6.2.3 读锁sync.RWMutex.RLock
1. 如果该方法返回负数 — 其他 Goroutine 获得了写锁，当前 Goroutine 就会调用 sync.runtime_SemacquireMutex 陷入休眠等待锁的释放；
2. 如果该方法的结果为非负数 — 没有 Goroutine 获得写锁，当前方法就会成功返回；

```go
func (rw *RWMutex) RLock() {
 
if atomic.AddInt32(&rw.readerCount, 1) < 0 {
 
runtime_SemacquireMutex(&rw.readerSem, false, 0)
 
}}
 
```

### 6.2.4 读锁sync.RWMutex.RULock

* 如果返回值大于等于零 — 读锁直接解锁成功；
* 如果返回值小于零 — 有一个正在执行的写操作，在这时会调用sync.RWMutex.rUnlockSlow 方法；
* sync.RWMutex.rUnlockSlow 会减少获取锁的写操作等待的读操作数 readerWait 并在所有读操作都被释放之后触发写操作的信号量 writerSem，该信号量被触发时，调度器就会唤醒尝试获取写锁的 Goroutine。

```go
func (rw *RWMutex) RUnlock() {
 
if r := atomic.AddInt32(&rw.readerCount, -1); r < 0 {
 
rw.rUnlockSlow(r)
 
}
 
}
 
func (rw *RWMutex) rUnlockSlow(r int32) {
 
if r+1 == 0 || r+1 == -rwmutexMaxReaders {
 
throw("sync: RUnlock of unlocked RWMutex")
 
}
 
if atomic.AddInt32(&rw.readerWait, -1) == 0 {
 
runtime_Semrelease(&rw.writerSem, false, 1)
 
}
 
}
 
```

## 6.3 Sync.map
通过引入两个map,将读写分离到不同的map,其中read map只提供读,而dirty map则负责写. 这样read map就可以在不加锁的情况下进行并发读取,当read map中没有读取到值时,再加锁进行后续读取,并累加未命中数,当未命中数到达一定数量后,将dirty map上升为read map.
另外，虽然引入了两个map，但是底层数据存储的是指针，指向的是同一份值．
具体流程： 如插入key 1,2,3时均插入了dirty map中，此时read map没有key值，读取时从dirty map中读取，并记录miss数


https://juejin.im/post/6844903598317371399
Sync.Cond
初始化

```go
var mailbox uint8
 
var lock sync.RWMutex
 
sendCond := sync.NewCond(&lock)
 
recvCond := sync.NewCond(lock.RLocker())
 
```

关键代码

```go
lock.Lock()
 
for mailbox == 1 {
 
sendCond.Wait()
 
}
 
mailbox = 1
 
lock.Unlock()
 
recvCond.Signal()
 
lock.RLock()
 
for mailbox == 0 {
 
recvCond.Wait()
 
}
 
mailbox = 0
 
lock.RUnlock()
 
sendCond.Signal()
```

 
为什么先要锁定条件变量基于的互斥锁，才能调用它的Wait方法？
把调用它的 goroutine（也就是当前的 goroutine）加入到当前条件变量的通知队列中。解锁当前的条件变量基于的那个互斥锁。让当前的 goroutine 处于等待状态，等到通知到来时再决定是否唤醒它。此时，这个 goroutine 就会阻塞在调用这个Wait方法的那行代码上。如果通知到来并且决定唤醒这个 goroutine，那么就在唤醒它之后重新锁定当前条件变量基于的互斥锁。自此之后，当前的 goroutine 就会继续执行后面的代码了。
为什么要用for语句来包裹调用其Wait方法的表达式，用if语句不行吗？
这主要是为了保险起见。如果一个 goroutine 因收到通知而被唤醒，但却发现共享资源的状态，依然不符合它的要求，那么就应该再次调用条件变量的Wait方法，并继续等待下次通知的到来。
条件变量的Signal方法和Broadcast方法有哪些异同？
条件变量的Signal方法和Broadcast方法都是被用来发送通知的，不同的是，前者的通知只会唤醒一个因此而等待的 goroutine，而后者的通知却会唤醒所有为此等待的 goroutine。
源码：

```go
type Cond struct {
 
noCopy noCopy //不允许编译时复制
 
L Locker //保护
 
notify notifyList
 
checker copyChecker
 
}
 
```

* noCopy — 用于保证结构体不会在编译期间拷贝；
* copyChecker — 用于禁止运行期间发生的拷贝；
* L — 用于保护内部的 notify 字段，Locker 接口类型的变量；
* notify — 一个 Goroutine 的链表，它是实现同步机制的核心结构；

```go
type notifyList struct {
 
wait uint32
 
notify uint32
 
lock mutex
 
head *sudog //头指针
 
tail *sudog
 
}
```

 
head 和 tail 分别指向的链表的头和尾，wait 和 notify 分别表示当前正在等待的和已经通知到的 Goroutine，我们通过这两个变量就能确认当前待通知和已通知的 Goroutine。
## 6.4 sync.waitGroup

```go
func coordinateWithWaitGroup() {
 
var wg sync.WaitGroup
 
wg.Add(2)
 
num := int32(0)
 
fmt.Printf("The number: %d [with sync.WaitGroup]\n", num)
 
max := int32(10)
 
go addNum(&num, 3, max, wg.Done)
 
go addNum(&num, 4, max, wg.Done)
 
wg.Wait()
 
}
```

 
sync.WaitGroup类型值中计数器的值可以小于0吗？
不可以。

源码：

```go
type WaitGroup struct {
 
noCopy noCopy //不允许复制
 
state1 [3]uint32}
 


func (wg *WaitGroup) Add(delta int) {
 
statep, semap := wg.state()
 
state := atomic.AddUint64(statep, uint64(delta)<<32)
 
v := int32(state >> 32)
 
w := uint32(state)
 
//为负数
 
if v < 0 {
 
panic("sync: negative WaitGroup counter")
 
}
 
//不为work计数器0
 
if v > 0 || w == 0 {
 
return
 
}
 
//计数为0了
 
*statep = 0
 
for ; w != 0; w-- {
 
//释放信号量
 
runtime_Semrelease(semap, false, 0)
 
}
 
}
 

func (wg *WaitGroup) Wait() {
 
statep, semap := wg.state()
 
for {
 
state := atomic.LoadUint64(statep)
 
v := int32(state >> 32)
 
if v == 0 {
 
return
 
}
 
//为什么这里用比较并交换呢 ，以内
 
if atomic.CompareAndSwapUint64(statep, state, state+1) {
 
runtime_Semacquire(semap)
 
if +statep != 0 {
 
panic("sync: WaitGroup is reused before previous Wait has returned")
 
}
 
return
 
}
 
}}
```

 

## 6.5 Sync.Once
表示只执行一次

```go
one := sync.Once{}
 
a := func() {
 
fmt.Println("ok")
 
}
 
for(int i = 0;i<10;i++){
 
go one.Do(a)
 
}
```

 
原理：
原子操作+快速失败
Do方法在一开始就会通过调用atomic.LoadUint32函数来获取该字段的值，并且一旦发现该值为1，就会直接返回。这也初步保证了“Do方法，只会执行首次被调用时传入的函数”。不过，单凭这样一个判断的保证是不够的。因为，如果有两个 goroutine 都调用了同一个新的Once值的Do方法，并且几乎同时执行到了其中的这个条件判断代码，那么它们就都会因判断结果为false，而继续执行Do方法中剩余的代码。在这个条件判断之后，Do方法会立即锁定其所属值中的那个sync.Mutex类型的字段m。然后，它会在临界区中再次检查done字段的值，并且仅在条件满足时，才会去调用参数函数，以及用原子操作把done的值变为1。
如果你熟悉 GoF 设计模式中的单例模式的话，那么肯定能看出来，这个Do方法的实现方式，与那个单例模式有很多相似之处。它们都会先在临界区之外，判断一次关键条件，若条件不满足则立即返回。这通常被称为 “快路径”，或者叫做“快速失败路径”。
源码：

```go
type Once struct {
 
done uint32
 
m Mutex
 
}
 
func (o *Once) Do(f func()) {
 
if atomic.LoadUint32(&o.done) == 0 {
 
o.doSlow(f)
 
}}
 
//让有返回一定要执行完成了才返回
 
func (o *Once) doSlow(f func()) {
 
o.m.Lock()
 
defer o.m.Unlock()
 
if o.done == 0 {
 
defer atomic.StoreUint32(&o.done, 1)
 
f()
 
}}
```

 
## 6.6 atomic
sync/atomic包中的函数可以做的原子操作有：加法（add）、比较并交换（compare and swap，简称 CAS）、加载（load）、存储（store）和交换（swap）。这些函数针对的数据类型并不多。但是，对这些类型中的每一个，sync/atomic包都会有一套函数给予支持。这些数据类型有：int32、int64、uint32、uint64、uintptr，以及unsafe包中的Pointer。不过，针对unsafe.Pointer类型，该包并未提供进行原子加法操作的函数。
我们都知道，传入这些原子操作函数的第一个参数值对应的都应该是那个被操作的值。比如，atomic.AddInt32函数的第一个参数，对应的一定是那个要被增大的整数。可是，这个参数的类型为什么不是int32而是*int32呢？
因为原子操作函数需要的是被操作值的指针，而不是这个值本身；被传入函数的参数值都会被复制，像这种基本类型的值一旦被传入函数，就已经与函数外的那个值毫无关系了。
用于原子加法操作的函数可以做原子减法吗？比如，atomic.AddInt32函数可以用于减小那个被操作的整数值吗？

```go
package main
 
import (
 
"fmt"
 
"sync/atomic"
 
"time"
 
)
 
func main() {
 
// 第二个衍生问题的示例。
 
num := uint32(18)
 
fmt.Printf("The number: %d\n", num)
 
delta := int32(-3)
 
atomic.AddUint32(&num, uint32(delta))
 
fmt.Printf("The number: %d\n", num)
 
atomic.AddUint32(&num, ^uint32(-(-3)-1))
 
fmt.Printf("The number: %d\n", num)
 
fmt.Printf("The two's complement of %d: %b\n",
 
delta, uint32(delta)) // -3的补码。
 
fmt.Printf("The equivalent: %b\n", ^uint32(-(-3)-1)) // 与-3的补码相同。
 
fmt.Println()
 
// 第三个衍生问题的示例。
 
forAndCAS1()
 
fmt.Println()
 
forAndCAS2()
 
}
 
// forAndCAS1 用于展示简易的自旋锁。
 
func forAndCAS1() {
 
sign := make(chan struct{}, 2)
 
num := int32(0)
 
fmt.Printf("The number: %d\n", num)
 
go func() { // 定时增加num的值。
 
defer func() {
 
sign <- struct{}{}
 
}()
 
for {
 
time.Sleep(time.Millisecond * 500)
 
newNum := atomic.AddInt32(&num, 2)
 
fmt.Printf("The number: %d\n", newNum)
 
if newNum == 10 {
 
break
 
}
 
}
 
}()
 
go func() { // 定时检查num的值，如果等于10就将其归零。
 
defer func() {
 
sign <- struct{}{}
 
}()
 
for {
 
if atomic.CompareAndSwapInt32(&num, 10, 0) {
 
fmt.Println("The number has gone to zero.")
 
break
 
}
 
time.Sleep(time.Millisecond * 500)
 
}
 
}()
 
<-sign
 
<-sign
 
}
```

 

```go
// forAndCAS2 用于展示一种简易的（且更加宽松的）互斥锁的模拟。
 
func forAndCAS2() {
 
sign := make(chan struct{}, 2)
 
num := int32(0)
 
fmt.Printf("The number: %d\n", num)
 
max := int32(20)
 
go func(id int, max int32) { // 定时增加num的值。
 
defer func() {
 
sign <- struct{}{}
 
}()
 
for i := 0; ; i++ {
 
currNum := atomic.LoadInt32(&num)
 
if currNum >= max {
 
break
 
}
 
newNum := currNum + 2
 
time.Sleep(time.Millisecond * 200)
 
if atomic.CompareAndSwapInt32(&num, currNum, newNum) {
 
fmt.Printf("The number: %d [%d-%d]\n", newNum, id, i)
 
} else {
 
fmt.Printf("The CAS operation failed. [%d-%d]\n", id, i)
 
}
 
}
 
}(1, max)
 
go func(id int, max int32) { // 定时增加num的值。
 
defer func() {
 
sign <- struct{}{}
 
}()
 
for j := 0; ; j++ {
 
currNum := atomic.LoadInt32(&num)
 
if currNum >= max {
 
break
 
}
 
newNum := currNum + 2
 
time.Sleep(time.Millisecond * 200)
 
if atomic.CompareAndSwapInt32(&num, currNum, newNum) {
 
fmt.Printf("The number: %d [%d-%d]\n", newNum, id, j)
 
} else {
 
fmt.Printf("The CAS operation failed. [%d-%d]\n", id, j)
 
}
 
}
 
}(2, max)
 
<-sign
 
<-sign
 
}
```

 
## 6.7 Sync.Pool
https://time.geekbang.org/column/article/42160
初始化：

```go
var ppFree = sync.Pool{
 
New: func() interface{} { return new(pp) },//不够就会调用这个函数
 
}
```

 
使用时：
会类似于gmp模型，拉去每一个p对应id本地池中的私有变量，没有再依次去找共享区，实在不行就new

使用完临时对象之后：
都会先抹掉其中已缓冲的内容，然后再把它存放到pool中。这样就为重用这类临时对象做好了准备。
同时垃圾回收：
sync包在被初始化的时候，会向 Go 语言运行时系统注册一个函数，这个函数的功能就是清除所有已创建的临时对象池中的值。我们可以把它称为池清理函数。一旦池清理函数被注册到了 Go 语言运行时系统，后者在每次即将执行垃圾回收时就都会执行前者。另外，在sync包中还有一个包级私有的全局变量。这个变量代表了当前的程序中使用的所有临时对象池的汇总，它是元素类型为*sync.Pool的切片。我们可以称之为池汇总列表。通常，在一个临时对象池的Put方法或Get方法第一次被调用的时候，这个池就会被添加到池汇总列表中。正因为如此，池清理函数总是能访问到所有正在被真正使用的临时对象池。更具体地说，池清理函数会遍历池汇总列表。对于其中的每一个临时对象池，它都会先将池中所有的私有临时对象和共享临时对象列表都置为nil，然后再把这个池中的所有本地池列表都销毁掉。最后，池清理函数会把池汇总列表重置为空的切片。如此一来，这些池中存储的临时对象就全部被清除干净了。

```go
package main
 
import (
 
"bytes"
 
"fmt"
 
"io"
 
"sync"
 
)
 
// bufPool 代表存放数据块缓冲区的临时对象池。
 
var bufPool sync.Pool
 
// Buffer 代表了一个简易的数据块缓冲区的接口。
 
type Buffer interface {
 
// Delimiter 用于获取数据块之间的定界符。
 
Delimiter() byte
 
// Write 用于写一个数据块。
 
Write(contents string) (err error)
 
// Read 用于读一个数据块。
 
Read() (contents string, err error)
 
// Free 用于释放当前的缓冲区。
 
Free()
 
}
 
// myBuffer 代表了数据块缓冲区一种实现。
 
type myBuffer struct {
 
buf bytes.Buffer
 
delimiter byte
 
}
 
func (b *myBuffer) Delimiter() byte {
 
return b.delimiter
 
}
 
func (b *myBuffer) Write(contents string) (err error) {
 
if _, err = b.buf.WriteString(contents); err != nil {
 
return
 
}
 
return b.buf.WriteByte(b.delimiter)
 
}
 
func (b *myBuffer) Read() (contents string, err error) {
 
return b.buf.ReadString(b.delimiter)
 
}
 
func (b *myBuffer) Free() {
 
bufPool.Put(b)
 
}
 
// delimiter 代表预定义的定界符。
 
var delimiter = byte('\n')
 
func init() {
 
bufPool = sync.Pool{
 
New: func() interface{} {
 
return &myBuffer{delimiter: delimiter}
 
},
 
}
 
}
 
// GetBuffer 用于获取一个数据块缓冲区。
 
func GetBuffer() Buffer {
 
return bufPool.Get().(Buffer)
 
}
 
func main() {
 
buf := GetBuffer()
 
defer buf.Free()
 
buf.Write("A Pool is a set of temporary objects that" +
 
"may be individually saved and retrieved.")
 
buf.Write("A Pool is safe for use by multiple goroutines simultaneously.")
 
buf.Write("A Pool must not be copied after first use.")
 
fmt.Println("The data blocks in buffer:")
 
for {
 
block, err := buf.Read()
 
if err != nil {
 
if err == io.EOF {
 
break
 
}
 
panic(fmt.Errorf("unexpected error: %s", err))
 
}
 
fmt.Print(block)
 
}
 
}
```

 
# 七、原码、反码、补码

正数：原码= 反码=补码
负数：原码：符号位为1，其他位与去除负号后一样；反码：原码除符号位外取反 ；补码：原码除符号位外取反 +1
[+1] = [00000001]原 = [00000001]反 = [00000001]补
[-1] = [10000001]原 = [11111110]反 = [11111111]补
机器码中的减法-加法：1-1= 1补码+（-1）补码 。为什么不用反码相加呢？因为反码相加00000000和10000000都表示0和-0，有问题，所以用补码相加10000000可以表示-128.
# 八、反射

t:=reflect.TypeOf(x) 专注类型信息
* t.Name()只是表面类型 t.Kind() 表示真实的底层类型
* 若x为指针*int类型，t.Name()为*int，t.Kind() 为ptr t.Elem()为int
* Elem返回指针，数组，切片，字典（值），通道的基类型
* 结构体指针只有获得他的基类型才可以遍历他的字段
* t.NumField(）获得变量个数 f :=t.Field(x)获取具体的变量 f.Type获取类型 f.offest或者偏移量，f.Anonymous判断当前是否为匿名字段
* name,err := t.FieldByName("str")通过名称查找 ，但是会出现同名遮蔽，匿名字段里面最好应该先取匿名字段二次获取； x :=t.FieldByIndex([]int{1,1})按多级索引查找,这个多级索引一定要存在，否则会panic
* Stuct tag元数据可以通过 f.Tag.Get("tagName")获取原注解
* Implements返回类型是否实现了某个接口,如果参数的类型不是interface类型，则会panic。AssignableTo一个类型的值是否可以赋值给参数指定的类型,下面的例子中Bird类型的指针对象可以赋值给IBird接口。ConvertibleTo 一个类型的值是否可以转换成另一个类型的值。Comparable返回类型是否可以比较：
v:=reflect.ValueOf() 专注对象实例数据的读写
* 不能对非导出字段（首字母不是大写的）进行设置操作 可用v.CanSet()判断
* 需要设置非导出字段可以这样子 *（*int）(unsafe.Pointer(v.UnsafeAddr())) = 100
* v.CanInterface()判断是否能根据接口判断，p,ok:=v.interface().(类型)来进行推断
* 接口有两种nil状态，所以可以用reflect.Value(b).IsNil()进行判断是否值为nil
* v.FieldByName("str").IsValid()判断有无失效
* 调用方法

```go
v := reflect.Value(&a)
 
m := v.MethodByName("Str")
 
in :=[]reflect.Value{
 
reflect.ValueOf(1),
 
reflect.ValueOf(2),
 
}
 
out := m.Call(in)
 
for _,v : range out{
 
fmt.Println(v)
 
}
 
//若是变参数
 
//可以：
 
in :=[]reflect.Value{
 
reflect.ValueOf(1),
 
reflect.ValueOf([]interface{} {"x",100}), //interface根据具体参数而定
 
}
```

 
# 九、测试

## 9.1 测试
* 测试代码必须放在_test结尾的文件中
* 测试函数以Test为名称前缀
* Go test忽略以“_”或“.”开头的测试文件
* Go build和go install回忽略测试文件
## 9.2 单元测试
* 入参 t *testing.T
* T Fail:失败，继续执行； FailNow：失败，终止执行的当前测试函数； SkipNow 跳过当前测试函数
log:打印错误信息；parallel：与有同样设置的并发执行；Error : fail+log ;Fatal failNow +log
* Go test 参数 -args 参数 ；-v输出详细信息；-parallel：并发执行 ；-run：指定测试函数；-timeout：累计超时将panic ；-count :重复测试次数
* 当需要前置条件的时候则可以

```go
func TestMain(m *testing.M){
 
//设置前置条件
 
code := m.Run()
 
//teardown拆除
 
os.Exit(code)
 
}
 
//骚操作
func ExampleAdd(){
 
fmt.Println(add(1,2))
 
//Output:
 
//3
 
}
 
```

利用go test -v可以测试上面函数
 
## 9.3 性能测试
* 必须以Benchmark命名函数，保存在_test.go函数中
* 入参数有b *testing.B
* go test -branch . 开启性能测试
* -cpu 参数设置多个并发的限制来观察结果
* -benchtime 取每次最小测试时间
* 可以运用计时器：b.ResetTimer() //重制时间、b.StopTimer()，停止时间，b.StartTimer()开始时间
* -gcflags "-N -l" 禁止内联和优化
# 十、监控内存
pprof;
# 十一、Context

Context类型的值（以下简称Context值）是可以繁衍的，这意味着我们可以通过一个Context值产生出任意个子值。这些子值可以携带其父值的属性和数据，也可以响应我们通过其父值传达的信号。正因为如此，所有的Context值共同构成了一颗代表了上下文全貌的树形结构。这棵树的树根（或者称上下文根节点）是一个已经在context包中预定义好的Context值，它是全局唯一的。通过调用context.Background函数，我们就可以获取到它（我在coordinateWithContext函数中就是这么做的）。这里注意一下，这个上下文根节点仅仅是一个最基本的支点，它不提供任何额外的功能。也就是说，它既不可以被撤销（cancel），也不能携带任何数据。
除此之外，context包中还包含了四个用于繁衍Context值的函数，即：WithCancel、WithDeadline（什么时间取消）、WithTimeout（几秒后取消）和WithValue。

```go
func coordinateWithContext() {
 
total := 12
 
var num int32
 
fmt.Printf("The number: %d [with context.Context]\n", num)
 
cxt, cancelFunc := context.WithCancel(context.Background())//返回context和撤销函数
 
for i := 1; i <= total; i++ {
 
go addNum(&num, i, func() {
 
if atomic.LoadInt32(&num) == int32(total) {
 
cancelFunc() //在这里进行撤销
 
}
 
})
 
}
 
<-cxt.Done() //阻塞知道运行了cancelFunc()
 
fmt.Println("End.")
 
}
```

 
cotext.backround()和cotext.todo()有什么区别
context.Background()返回的是全局的上下文根（我在文章中多次提到），
context.TODO()返回的是空的上下文（表明应用的不确定性）。
contextwithvalue寻找值的时候会通过找自身和向上寻找

```go
node2 := context.WithValue(node1, keys[0], values[0])
 
node3 := context.WithValue(node1, keys[1], values[1])
 
fmt.Printf("The value of the key %v found in the node3: %v\n",
 
keys[0], node3.Value(keys[0]))
 
fmt.Printf("The value of the key %v found in the node3: %v\n",
 
keys[1], node3.Value(keys[1]))
 
fmt.Printf("The value of the key %v found in the node3: %v\n",
 
keys[2], node3.Value(keys[2]))
 
fmt.Printf("The value of the key %v found in the node3: %v\n",
 
keys[2], node2.Value(keys[2]))
 
fmt.Println()
```

 
源码

```go
type Context interface {
 
Deadline() (deadline time.Time, ok bool)
 
Done() <-chan struct{}
 
Err() error
 
Value(key interface{}) interface{}}
```

 
父子context建立关系后，会单独启一个协程去监听父context有没有被取消，若是被取消了则也把子context取消。

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
 
return WithDeadline(parent, time.Now().Add(timeout))
 
}
 
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
 
if cur, ok := parent.Deadline(); ok && cur.Before(d) {
 
return WithCancel(parent)
 
}
 
c := &timerCtx{
 
cancelCtx: newCancelCtx(parent),
 
deadline: d,
 
}
 
propagateCancel(parent, c)
 
dur := time.Until(d)
 
if dur <= 0 {
 
c.cancel(true, DeadlineExceeded) // 已经过了截止日期
 
return c, func() { c.cancel(false, Canceled) }
 
}
 
c.mu.Lock()
 
defer c.mu.Unlock()
 
if c.err == nil {
 
c.timer = time.AfterFunc(dur, func() {
 
c.cancel(true, DeadlineExceeded)
 
})
 
}
 
return c, func() { c.cancel(true, Canceled) }
 
}
```

 
context.WithDeadline 方法在创建 context.timerCtx 的过程中，判断了父上下文的截止日期与当前日期，并通过 time.AfterFunc 创建定时器，当时间超过了截止日期后会调用 context.timerCtx.cancel 方法同步取消信号。
# 十二、内存相关

用户程序 、内存分配、内存回收

## 12.1 内存分配
区别与java使用线性分配器，go语言更多的是用空闲链表分配器（存在指针所以不能随意改变）

内存管理单元mspan（内存管理单元是线程缓存和中心缓存的组成单位）

存在67个跨度类
66个跨度类是8b-32kb，0号跨度类是存储大于32kb的对象

## 12.2 线程缓存


（处理微对象的分配）
每一个线程缓存都持有 67 * 2 个 runtime.mspan，这些内存管理单元都存储在结构体的 alloc 字段中。
为什么*2呢，因为一个是记录指针类对象，一个记录非指针的
扩容会找中心缓存：
## 12.3 中心缓存


134（（66个跨度类级别+1个大对象）*2（分别包含指针和不含指针））
接到线程缓存的扩容请求后
* 从有空闲对象的 runtime.mspan 链表中查找可以使用的内存管理单元；
* 从没有空闲对象的 runtime.mspan 链表中查找可以使用的内存管理单元；
* 调用 runtime.mcentral.grow 从堆中申请新的内存管理单元；
* 更新内存管理单元的 allocCache 等字段帮助快速分配内存；
中心缓存扩容找页堆
页堆


Heap arena大小：

每一个页是8k
内存分配
* 微对象 (0, 16B) — 先使用微型分配器，再依次尝试线程缓存、中心缓存和堆分配内存；
* 小对象 [16B, 32KB] — 依次尝试使用线程缓存、中心缓存和堆分配内存；
* 大对象 (32KB, +∞) — 直接在堆上分配内存；
# 十三、垃圾回收
## 13.1 三色抽象

为了解决原始标记清除算法带来的长时间 STW，多数现代的追踪式垃圾收集器都会实现三色标记算法的变种以缩短 STW 的时间。三色标记算法将程序中的对象分成白色、黑色和灰色三类4：
* 白色对象 — 潜在的垃圾，其内存可能会被垃圾收集器回收；
* 黑色对象 — 活跃的对象，包括不存在任何引用外部指针的对象以及从根对象可达的对象；
* 灰色对象 — 活跃的对象，因为存在指向白色对象的外部指针，垃圾收集器会扫描这些对象的子对象；
三色标记垃圾收集器的工作原理很简单，我们可以将其归纳成以下几个步骤：
1. 从灰色对象的集合中选择一个灰色对象并将其标记成黑色；
2. 将黑色对象指向的所有对象都标记成灰色，保证该对象和被该对象引用的对象都不会被回收；
3. 重复上述两个步骤直到对象图中不存在灰色对象；
屏障技术

插入写屏障

就是把新指向的变灰

删除写屏障

其实就是把删除的指向的后端变灰

## 13.2 栈内存逃逸
栈区的内存一般由编译器自动进行分配和释放，其中存储着函数的入参以及局部变量，这些参数会随着函数的创建而创建，函数的返回而消亡，一般不会在程序中长期存在。
逃逸分析：
1. 指向栈对象的指针不能存在于堆中；
2. 指向栈对象的指针不能在栈对象回收后存活；
逃逸出现：返回值里面出现；分配的内存过大，栈不够；接口类型，编译期间不清楚是否应该分配到栈中会出现分配到堆中去
# 十四、包结构和常用命令

* GOROOT：Go 语言安装根目录的路径，也就是 GO 语言的安装路径。
* GOPATH：若干工作区目录的路径。是我们自己定义的工作空间。
* GOBIN：GO 程序生成的可执行文件（executable file）的路径。

GOPATH：去放置 Go 语言的源码文件（source file），以及安装（install）后的归档文件（archive file，也就是以“.a”为扩展名的文件）和可执行文件（executable file）。


https://blog.csdn.net/wuya814070935/article/details/50219915
这个博客讲的特别好
go build 编译原码
## 14.1 go build
go build 命令主要是用于测试编译。在包的编译过程中，若有必要，会同时编译与之相关联的包。
1. 如果是普通包，当你执行go build命令后，不会产生任何文件。
2. 如果是main包，当只执行go build命令后，会在当前目录下生成一个可执行文件。如果需要在$GOPATH/bin木下生成相应的exe文件，需要执行go install 或者使用 go build -o 路径/a.exe。
3. 如果某个文件夹下有多个文件，而你只想编译其中某一个文件，可以在 go build 之后加上文件名，例如 go build a.go；go build 命令默认会编译当前目录下的所有go文件。
4. 你也可以指定编译输出的文件名。比如，我们可以指定go build -o myapp.exe，默认情况是你的package名(非main包)，或者是第一个源文件的文件名(main包)。
5. go build 会忽略目录下以”_”或者”.”开头的go文件。
6. 如果你的源代码针对不同的操作系统需要不同的处理，那么你可以根据不同的操作系统后缀来命名文件。例如有一个读取数组的程序，它对于不同的操作系统可能有如下几个源文件：
array_linux.go
 
array_darwin.go
 
array_windows.go
 
array_freebsd.go
 
go build的时候会选择性地编译以系统名结尾的文件（Linux、Darwin、Windows、Freebsd）。例如Linux系统下面编译只会选择array_linux.go文件，其它系统命名后缀文件全部忽略。
## 14.2 go install
go install 命令在内部实际上分成了两步操作：第一步是生成结果文件(可执行文件或者.a包)，第二步会把编译好的结果移到 $GOPATH/pkg 或者 $GOPATH/bin。
.exe文件： 一般是 go install 带main函数的go文件产生的，有函数入口，所有可以直接运行。
.a应用包： 一般是 go install 不包含main函数的go文件产生的，没有函数入口，只能被调用。
## 14.3 go get
go get 命令主要是用来动态获取远程代码包的，目前支持的有BitBucket、GitHub、Google Code和Launchpad。这个命令在内部实际上分成了两步操作：第一步是下载源码包，第二步是执行go install。下载源码包的go工具会自动根据不同的域名调用不同的源码工具，对应关系如下：
BitBucket (Mercurial Git)
GitHub (Git)
Google Code Project Hosting (Git, Mercurial, Subversion)
Launchpad (Bazaar)
所以为了go get 能正常工作，你必须确保安装了合适的源码管理工具，并同时把这些命令加入你的PATH中。其实go get支持自定义域名的功能，具体参见go help remote。
go get 命令本质上可以理解为：首先通过源码工具clone代码到src目录，然后执行go install。
## 14.4 go clean
go clean 命令是用来移除当前源码包里面编译生成的文件，这些文件包括
* _obj/ 旧的object目录，由Makefiles遗留
* _test/ 旧的test目录，由Makefiles遗留
* _testmain.go 旧的gotest文件，由Makefiles遗留
* test.out 旧的test记录，由Makefiles遗留
* build.out 旧的test记录，由Makefiles遗留
* *.[568ao] object文件，由Makefiles遗留
* DIR(.exe) 由 go build 产生
* DIR.test(.exe) 由 go test -c 产生
* MAINFILE(.exe) 由 go build MAINFILE.go产生
## 14.5 go fmt
go fmt 命令主要是用来帮你格式化所写好的代码文件。
比如我们写了一个格式很糟糕的 test.go 文件，我们只需要使用 fmt go test.go 命令，就可以让go帮我们格式化我们的代码文件。但是我们一般很少使用这个命令，因为我们的开发工具一般都带有保存时自动格式化功能，这个功能底层其实就是调用了 go fmt 命令而已。
使用go fmt命令，更多时候是用gofmt，而且需要参数-w，否则格式化结果不会写入文件。gofmt -w src，可以格式化整个项目。
go test
go test 命令，会自动读取源码目录下面名为*_test.go的文件，生成并运行测试用的可执行文件。输出的信息类似
ok archive/tar 0.011s
FAIL archive/zip 0.022s
ok compress/gzip 0.033s...
* 默认的情况下，不需要任何的参数，它会自动把你源码包下面所有test文件测试完毕，当然你也可以带上参数，详情请参考go help testflag
## 14.6 go doc
go doc 命令其实就是一个很强大的文档工具。
如何查看相应package的文档呢？ 例如builtin包，那么执行go doc builtin；如果是http包，那么执行go doc net/http；查看某一个包里面的函数，那么执行godoc fmt Printf；也可以查看相应的代码，执行godoc -src fmt Printf；
通过命令在命令行执行 godoc -http=:端口号 比如godoc -http=:8080。然后在浏览器中打开127.0.0.1:8080，你将会看到一个golang.org的本地copy版本，通过它你可以查询pkg文档等其它内容。如果你设置了GOPATH，在pkg分类下，不但会列出标准包的文档，还会列出你本地GOPATH中所有项目的相关文档，这对于经常被限制访问的用户来说是一个不错的选择。
其他命令
Go语言还提供了其它有用的工具，例如下面的这些工具
go fix 用来修复以前老版本的代码到新版本，例如go1之前老版本的代码转化到go1
go version 查看go当前的版本
go env 查看当前go的环境变量
go list 列出当前全部安装的package
go run 编译并运行Go程序
# 十五、关键字

## 15.1 Defer:
作用域：当前函数返回的时候
预计算参数

```go
func main() {
 
startedAt := time.Now()
 
defer fmt.Println(time.Since(startedAt))
 
time.Sleep(time.Second)}
 
$ go run main.go
 
0s
```

 
原因：调用 defer 关键字会立刻对函数中引用的外部参数进行拷贝，所以 time.Since(startedAt) 的结果不是在 main 函数退出之前计算的，而是在 defer 关键字调用时计算的，最终导致上述代码输出 0s。
放到函数里面就可以

```go
func main() {
 
startedAt := time.Now()
 
defer func() { fmt.Println(time.Since(startedAt)) }()
 
time.Sleep(time.Second)}
 
$ go run main.go
 
1s
 
* type _defer struct {
 
siz int32
 
started bool
 
sp uintptr
 
pc uintptr
 
fn *funcval
 
_panic *_panic
 
link *_defer // 形成一个链表
 
}
```

 
siz 是参数和结果的内存大小；
* sp 和 pc 分别代表栈指针和调用方的程序计数器；
* fn 是 defer 关键字中传入的函数；
* _panic 是触发延迟调用的结构体，可能为空；
link形成链表，使得defer filo 先进后出
* runtime.deferproc 函数负责创建新的延迟调用；
* runtime.deferreturn 函数负责在函数调用结束时执行所有的延迟调用；
## 15.2 Panic recover

```go
func main() {
 
defer fmt.Println("in main")
 
if err := recover(); err != nil {
 
fmt.Println(err)
 
}
 
panic("unknown err")
 
}
```

 
# 十六、原生Rpc流程

不一样：rpc有打包和解码的过程，同时需要经过网络传输，所以会出现会导致比本地方法调用更加容易出错。
通信模式：同步，oneway， future,异步
语义：至少一次，至多一次，幂等语义
rpc流程主要有四个大步骤：
服务端服务注册
* register进行服务注册并且进行语法分析，比如方法接受者和方法都是大写字母开头的，返回值为error，第二个参数是指针类型，因为他是用来接受返回值，一共有三个参数，然后就会生成一个map，key为服务，value为方法
客户端发送请求
* 客户端每次发请求可以是异步，也可以是同步的，发送请求的方式主要有http和tcp两种，不论哪一种都生成对应的client，并且也会生成对应的call对象，同时生成一个map ,key为本次请求的唯一标识seq,value为call对象
服务端处理请求
* 服务端这边一直会开启一个for循环时刻监听有无请求过来，然后另开协程去处理过来的请求，通过解析头部信息（获取服务名，方法名，seq,是否长链接），请求体信息（请求参数，返回格式等），然后调用本地方法进行返回
客户端收到请求
* 在发送请求之后，一般会另起一个协程去监听返回，根据返回的seq就会去定位对应的call，然后把call中的done chan 变成不为空，从而让发送请求的协程知道获取结束了
