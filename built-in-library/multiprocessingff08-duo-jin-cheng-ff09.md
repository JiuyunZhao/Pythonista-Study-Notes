# multiprocessing（多进程）

Python的多进程因为可以充分利用CPU多核的特点，所以通常用于计算密集型的场景或者需要大量数据操作的场景，而对于多线程，在某些语言中因为可以充分利用CPU，所以可能多线程的场景使用得多一点，但是在Python中，多线程只能在CPU的单核中运行，不能充分利用CPU多核的特点，所以Python多线程通常用于IO密集型的场景或者少量数据的并发操作场景。总而言之，Python的多线程只是并发执行，而不是真正的并行执行，而且只能在CPU单核上进行，所以如果需要进行大量的数据操作或者比较耗时的并行操作，那么就可以考虑使用多进程了。

本文只是根据官方文档简单记了一下multiprocessing模块中进程的基本操作，包括创建进程、进程启动方式、进程间通信、进程间同步、进程池，如果需要其他更多操作，可以参考此模块的官方中文文档[https://docs.python.org/zh-cn/3.7/library/multiprocessing.html?highlight=multiprocessing\#module-multiprocessing](https://docs.python.org/zh-cn/3.7/library/multiprocessing.html?highlight=multiprocessing#module-multiprocessing)

### 创建进程

实例化Process类创建一个进程对象，然后调用它的start方法即可生成一个新的进程（子进程）。Process进程对象的使用其实和多线程模块threading中的Thread线程对象非常相似，可以参考着来使用。

```py
"""
简单示例：创建一个子进程
"""
import os
from multiprocessing import Process


def func(s):
    # 输出传入的参数，当前子进程的进程ID，当前进程的父进程ID
    print(s, os.getpid(), os.getppid())


# 注意：此处的if __name__ == '__main__'语句不能少
if __name__ == '__main__':
    # 打印当前进程的进程ID
    print(os.getpid())
    print('main process start...')
    # 创建进程对象
    p = Process(target=func, args=('hello', ))
    # 生成一个进程，并开始运行新的进程
    p.start()
    # 等待子进程运行完毕
    p.join()
    print('main process end!')
```

打印输出

```py
13888
main process start...
hello 12484 13888
main process end!
```

#### Process类

**Process\(group=None, target=None, name=None, args=\(\), kwargs={}, \*, daemon=None\)：**group不用特别指定，使用默认就行；target表示需要调用的对象；name表示新进程的名称；args和kwargs表示传给target对象的元组参数和字典参数；daemon是一个关键字参数，使用时必须指定参数名，表示是否为守护进程，如果不指定则默认继承自调用者进程。

**注：**需要注意的是如果重写了Process的\_\_init\_\_方法，那么在做任何操作之前需要先调用Process.\_\_init\_\_\(\)方法。

常用的方法和属性：

* **run：**表示进程活动的方法，即此方法的运行是在新开启的进程中，如果在子类中重写了此方法，应该在此方法中调用target对象。
* **start\(\)：**用于启动进程活动（注意此方法是在调用者进程中，而不是在新的进程中），并用于保证run方法在一个新的进程中被调用。
* **join\(\[timeout\]\)：**如果timeout参数没有指定（默认），则会阻塞当前进程直到调用join方法的进程（子进程）运行结束，如果指定了timeout参数，则会阻塞指定的秒数。注意，join方法不能在start方法之前调用，但join方法可以调用多次。如果想要知道进程的状态（包括是否结束），可以查看进程对象的exitcode值来进行判断。
* **name：**进程名称，没什么实际意义，只是用来表示进程，多个进程可能有相同的名称。如果没有特别指定，则默认命名格式为Process-N1:N2:N3...。
* **is\_alive\(\)：**此进程是否存活。
* **damemon：**表示进程是否为守护进程，这个标识必须在start\(\)方法调用之前进行设置，如果不设置，默认继承创建者进程。当一个进程终止时，会尝试终止它的所有守护子进程，需要注意的是，守护进程是不允许创建子进程的。
* **pid：**进程ID。
* **exitcode：**进程退出状态，当进程还未结束时，值为None，如果进程结束了，会用一个负值-N表示结束信号。
* **authkey：**进程的身份验证密钥（字节字符串），当multiprocessing被初始化时，主进程会使用os.urandom\(\)分配一个随机的字符串，当创建Process子进程时，子进程会继承其父进程的身份密钥，当然，你也可以修改子进程的身份密钥。
* **sentinel：**系统对象的数字句柄，当进程结束时将变为“ready”。如果想要使用multiprocessing.connection.wait\(\)一次等待多个事件，那可以使用这个值，否则调用join\(\)方法会更简单。
* **terminate\(\)：**终止进程，在Unix上使用的是SIGTERM信号，在Windows上使用的是TerminateProcess\(\)。注意，进程的后代进程不会被终止（会变成“孤儿”进程）。另外，如果被终止的进程在使用Pipe或Queue时，它们有可能会被损害，并无法被其他进程使用；如果被终止的进程已获得锁或信号量等，则有可能导致其他进程死锁。所以请谨慎使用此方法。
* **kill\(\)：**也是终止进程，但是在Unix上使用的是SIGKILL信号。
* **close\(\)：**关闭Process对象，并释放与之关联的所有资源，如果底层进程仍在运行，则会引发ValueError。而且，一旦close\(\)方法成功返回，Process对象的大多数方法和属性也可能会引发ValueError。

### 进程启动方式

multiprocessing模块中进程的启动方式有三种spawn、fork和forkserver，在不同的系统平台上它们的使用和默认设置也会有所不同：

* **spawn：**由父进程启动一个新的Python解释器Process子进程，子进程只会继承run\(\)方法中所必需的资源，而父进程中那些非必需的文件描述符和句柄是不会被继承的。而且，相对于使用fork和forkserver来启动进程，spawn方法启动是非常慢的。spawn启动方式可以在Unix和Windows上使用，且Windows上默认使用此方法启动。
* **fork：**父进程使用os.fork\(\)来产生一个新的Python解释器分叉（fork）子进程，子进程在开始时与父进程是相同的，即子进程会继承父进程拥有的所有资源。这种方式的问题在于当父进程中存在多线程时，启动的新的子进程的安全性需要自己留意。fort启动方式只能在Unix中使用，且也是Unix中默认的启动方式。
* **forkserver：**程序会先使用forkserver启动一个服务器进程，然后当需要运行一个新的进程时，父进程会先连接到服务器并请求其分叉（fork）一个新的进程。 相比于fork启动方式，由于forkserver启动的服务器进程是单线程的进程，所以由它通过os.fork\(\)启动的进程是安全的（此服务器进程没有多线程的情况）。forkserver启动方式可以在Unix平台使用，并支持通过Unix管道传递文件描述符。

**设置统一的启动方式：**可以在程序运行开始时，即if \_\_name\_\_ == "\_\_main\_\_"中使用multiprocessing.set\_start\_method\(method\)函数来设置启动方式，设置时传入对应启动方式的字符串即可（"spawn"/"fork"/"forkserver"）。但是需要注意两点，一是需要在if \_\_name\_\_ == "\_\_main\_\_"子句中指定，二是只能指定一次，指定之后就不能在其他地方再次指定。

**设置特定的启动方式：**可以使用multiprocessing.get\_context\(method\)函数来设置上下文中的启动方式，需要注意的是在此上下文中创建的对象可能与其他上下文中的对象不兼容，比如，使用fork方式的上下文中的锁不能传递给spawn或forkserver中使用，另外，如果你不想采用默认的方式或者全局统一的方式，就可以考虑使用get\_context\(method\)方法来指定自己的启动方式。

**注：**在Unix上，spawn和forkserver启动方法不能和“冻结的”可执行内容一同使用（例如，PyInstaller和cx\_Freeze包产生的二进制文件），但是fork启动方法可以。

### 进程间通信

使用多进程时，一般使用消息机制（Pipe\(\)管道和Queue\(\)队列）实现进程间的通信，而且应该尽可能地避免同步操作，例如锁。（如果这两种方式不能满足你的要求，可以参考下官方文档中关于multiprocessing.connection的描述，它提供了如监听器对象Listener和客户端对象Client等通信方式，感兴趣的话也可以去看下）

#### Pipe类

**Pipe\(\[duplex\]\)：**返回一对连接对象（conn1，conn2），它们代表了管道的两端。参数duplex默认True，表示双向的（双工通信），表示管道每一端都可以进行发送和接收数据；如果设置False，则表示单向的（单工通信），此时conn1只能接受数据，conn2只能发送数据。

```py
"""
简单示例：使用管道Pipe进行进程间通信
"""
from multiprocessing import Process, Pipe


def func(conn):
    print('send a list object ot other side...')
    # 从管道对象的一端发送数据对象
    conn.send(['33', 44, None])
    conn.close()


if __name__ == '__main__':
    # 默认创建一个双工管道对象，返回的两个对象代表管道的两端，
    # 双工表示两端的对象都可以发送和接收数据，但是需要注意，
    # 需要避免多个进程或线程从一端同时读或写数据
    parent_conn, child_conn = Pipe()
    p = Process(target=func, args=(child_conn, ))
    p.start()
    # 从管道的另一端接收数据对象
    print(parent_conn.recv())
    p.join()
```

#### Connection类

**multiprocessing.connection.Connection：**Connection对象允许收发可以序列化的对象或字符串，Connection对象通常使用Pipe来创建。

常用的方法:

* **send\(obj\)：**将一个对象发送到连接的另一端，另一端可以使用recv\(\)方法来读取。注意，发送的对象必须是可以序列化的，对象如果过大可能会引发ValueError异常。
* **recv\(\)：**返回一个对端使用send\(\)方法发送的对象，该方法会一直阻塞直到接收到对象为止。如果对端关闭了连接或者没有东西可以接收时，将会抛出EOFError异常。
* **fileno\(\)：**返回由连接对象使用的描述符或句柄。
* **close\(\)：**关闭连接。当连接对象被垃圾回收时，这个方法会被自动调用。
* **poll\(\[timeout\]\)：**返回连接对象中是否有可以读取的数据，如果未指定参数timeout（默认），此方法会立刻返回结果，如果指定了timeout，则会阻塞对应timeout秒数，如果timeout为None，则会一直阻塞，不会发生超时。
* **send\_bytes\(buffer\[, offset\[, size\]\]\)：**从一个bytes-like object（字节类对象）中取出字节数组作为一条完整消息发送。offset参数表示偏移量或者buffer中数据的位置，size表示从offset开始读取多少数据。如果buffer过大，可能会引发ValueError异常。
* **recv\_bytes\(\[maxlength\]\)：**以字符串的形式返回一条对端发送过来的字节数据，此方法会一直阻塞直到接收到消息，如果对端关闭了连接或者没有数据可以接收时，将会抛出EOFError异常。如果接收的数据长度大于了maxlength指定的长度，那么也会抛出EOFError异常，并且此时此连接对象不再可读。
* **recv\_bytes\_into\(buffer\[, offset\]\)：**将一条完整的字节数据读入buffer，并返回数据的字节数。此方法会一直阻塞直到接收到数据，如果对端关闭或者没有数据可以读取，则会抛出EOFError异常。buffer必须是一个可写入的字节类对象，如果指定了offset参数，将会从offset指定的位置开始写入buffer，如果buffer过小，也会引发BufferTooShort异常。

#### Queue类

Queue队列采用的是FIFO（先进先出）的通信方式。（另外还有SimpleQueue和JoinableQueue，感兴趣的可以参考下官方文档）

当一个对象被放入队列中时，这个对象首先会被一个后台线程序列化，然后会将序列化的数据通过一个底层管道传递到队列中，从队列中将数据取出来时也会进行反序列化的操作。

注意一点，在一个空队列中放入对象后，它的empty\(\)方法会在一个极小的延迟后才会返回False。

**注：**如果一个子进程将一些对象放入队列中，那么这个进程在所有缓冲区的对象被刷新进管道之前，是不会终止的，所以，通常在终止这类进程时，应该保证队列中的数据都已被使用了。（见示例中的注释）

```py
"""
简单示例：使用队列Queue进行进程间通信
"""
from multiprocessing import Process, Queue


def func(q):
    print('put a list object to queue...')
    # 向Queue对象中添加一个对象
    q.put(['33', 44, None])
    # q.put('X' * 1000000)


if __name__ == '__main__':
    # 创建一个队列
    q = Queue()
    p = Process(target=func, args=(q, ))
    p.start()
    # 从Queue对象中获取一个对象
    print(q.get())
    # 这里需要注意，当向队列中放入的数据较大时，比如将['33', 44, None]替换为'X' * 1000000时，
    # 就会在join()处卡死，为了避免这种情况，
    # 通常的做法是先使用get()将数据取出来，再使用join()方法
    p.join()
```

**Queue\(\[maxsize\]\)：**返回一个使用Pipe管道和少量锁和信号量实现的共享队列实例，当一个进程将一个对象放入队列时，一个写入线程将会启动并将对象从缓冲区写入管道中。

**注：**multiprocessing.Queue实现了标准库queue.Queue中除了task\_done\(\)和join\(\)的所有方法。

常用的方法和属性：

* **qsize\(\)：**返回队列的大致长度，但这个数字在多进程或多线程的环境中通常是不可靠的。注意，在Unix平台上，例如Mac OS X，这个方法可能会抛出NotImplementedError，因为该平台没有实现sem\_getvalue\(\)。
* **empty\(\)：**队列为空则返回True，否则返回False，在多进程或多线程的环境中，此方法是不可靠的。
* **full\(\)：**队列满则返回True，否则返回False，在多进程或多线程的环境中，此方法是不可靠的。
* **put\(obj\[, block\[, timeout\]\]\)：**将对象obj放入队列。如果参数block为True（默认）且timeout为None（默认），则会阻塞当前进程，直到有空的缓冲槽。如果设置了timeout，则会阻塞指定的timeout秒数，如果阻塞timeout指定秒数后还是没有可用的缓冲槽，则会抛出queue.Full异常。如果block为False，此时会忽略timeout参数，并且当前有空的缓冲槽可用时才能放入对象，否则会抛出queue.Full异常。
* **put\_nowait\(obj\)：**相当于put\(obj, False\)。
* **get\(\[block\[, timeout\]\]\)：**从队列中获取一个对象。如果参数block为True（默认）且timeout为None（默认），则会阻塞当前进程，直到获取到对象。如果设置了timeout，则会阻塞指定的timeout秒数，如果阻塞timeout指定秒数后还是没有获取到对象，则会抛出queue.Empty异常。如果block设置为False，此时会忽略timeout参数，并且当前有对象可以获取时才能获取，否则会抛出queue.Empty异常。
* **get\_nowait\(\)：**相当于get\(False\)。
* **close\(\)：**表示当前进程将不会再往队列中放入对象了。一旦缓冲区的所有数据被写入管道后，对应的后台线程就会退出。而且这个方法在队列被gc回收时会自动调用。
* **join\_thread\(\)：**等待后台线程，这个方法仅在调用了close\(\)方法之后可以被调用，并且会阻塞当前进程，当变为非阻塞状态之后，队列的后台线程会退出，以此确保缓冲区中的所有数据都被写入管道中。默认情况下，如果一个进程不是此队列的创建者进程，当它退出时，默认会尝试等待这个队列的后台线程，当然这个进程也可以使用cancel\_join\_thread\(\)方法使join\_thread\(\)方法什么都不做直接跳过。
* **cancel\_join\_thread\(\)：**用于防止join\_thread\(\)方法阻塞当前进程，即防止进程退出时自动等待队列后线程的情况。使用这个方法有可能会导致队列中的数据丢失，因此大多情况下这个方法并不需要用到，当然，如果你只是想要进程马上退出，也不在意数据的丢失，那么可以使用这个方法。

**注：**multiprocessing使用了queue.Empty和queue.Full异常去表示超时，需要从内置的queue模块中导入它们，而不是从multiprocessing中导入。

### 进程间同步

通常来说同步原语在多进程环境中并不像在多线程环境中那么必要，但是也可以参考下。注意，也可以使用Manager\(\)对象创建同步原语。

**multiprocessing.Barrier\(parties\[, action\[, timeout\]\]\)：**类似threading.Barrier的栅栏对象。

**multiprocessing.Semaphore\(\[value\]\)：**信号量对象，类似于threading.Semaphore。

**multiprocessing.BoundedSemaphore\(\[value\]\)：**类似threading.BoundedSemaphore的有界信号量对象。

**multiprocessing.Condition\(\[lock\]\)：**是threading.Condition的别名，参数lock应该是multiprocessing中的Lock或者RLock对象。

**multiprocessing.Event：**类似threading.Event的事件对象。

**multiprocessing.Lock：**原始锁，除非特别说明，否则用法与threading.Lock是一致的。

* **acquire\(block=True, timeout=None\)：**获取锁，需要注意一下参数block和timeout与threading.Lock中的名称和用法的区别。如果block设置为True（默认值），此方法会阻塞进程直到获取锁；如果block参数设置为False，进程将不会阻塞，且会忽略timeout参数；如果设置了timeout参数且为正数，则会阻塞指定秒数，如果设置为负数，则等效于值为0的情况，如果timeout为None（默认值），则会一直阻塞，需要注意timeout设置为负数和0时，其作用和threading.Lock是不一致的。此方法的返回值，在获取到锁并将锁的状态设置为“锁住”时返回True，超时或者没有获取到锁时返回False。
* **release\(\)：**释放锁，注意，任何进程或线程都可释放这种锁，并不是只有获取锁的进程或线程才可以释放锁。当试图释放一个未“锁住”的锁时会引发ValueError异常。其他用法与threading.Lock是一致的。

**multiprocessing.RLock：**递归锁，类似于threading.RLock，只能由获取锁的进程或线程来进行释放，并且可以获取多次，注意，释放次数必须要与获取次数一致。

* **acquire\(block=True, timeout=None\)：**当block设置为True时（默认值），会阻塞进程直到获取锁，如果当前进程已经获取到了锁（递归锁可以多次获取），那么不会阻塞，并且锁内的递归等级加1，并返回True。如果block设置为False，则不会阻塞，此时如果没有获取到锁，则锁内的递归等级不会变，并返回False。timeout的使用与multiprocessing.Lock.acquire是一样的，但是注意，此参数与threading.RLock中的使用是有区别的。
* **release\(\)：**释放锁，使锁内的递归等级减1。如果释放后锁的递归等级降低为0，则会重置锁的状态为“释放”状态，表名此时锁没有被任何进程或线程持有；如果释放后锁的递归等级不为0，则锁定状态还是“未释放”的状态，当前进程或线程仍然是锁的持有者。如果锁已经处于“释放”状态，或者是非锁的持有者调用了此方法，则会抛出AssertionError异常，注意这个异常与threading.RLock.release\(\)中抛出的异常是不同的。

```py
"""
简单示例：使用锁保证进程间的同步操作
"""
from multiprocessing import Process, Lock


def func(lc, num):
    # 使用锁保证以下代码同一时间只有一个进程在执行
    lc.acquire()
    print('process num: ', num)
    lc.release()


if __name__ == '__main__':
    lock = Lock()
    for i in range(5):
        Process(target=func, args=(lock, i)).start()
```

打印输出

```py
process num:  0
process num:  1
process num:  3
process num:  2
process num:  4
```

### 进程间共享数据

在多进程的并发编程中应当尽量避免使用共享状态，但是如果必须要使用的话，multiprocessing模块提供了两种方式来使用：共享内存和服务进程管理器（Manager\(\)管理器对象会开启一个服务进程，允许不同机器上的进程通过网络共享数据，本文就不写了，感兴趣的可以去官方文档了解下（有对应的中文文档））。

#### 共享内存

可以在共享内存中创建可被子进程继承的共享ctypes对象，特点是快捷方便。

**multiprocessing.Value\(typecode\_or\_type, \*args, lock=True\)：**返回一个在共享内存上创建的ctypes对象，可以通过它的value属性来访问它的值。

* **typecode\_or\_type：**指定返回的对象类型，可以是一个ctypes类型，也可以是array模块中每个类型对应的单字符的字符串。
* **args：**这个参数会传给这个类的构造函数。
* **lock：**默认为True，则会新建一个递归锁用于对这个值的同步访问操作；如果lock指定为一个Lock或RLock，则会使用这个锁来控制这个变量的同步操作；如果lock为False，那么这个值将没有锁进行保护，也就是说这个变量不是进程安全的。
* **注：**如果想要进行+=这种操作时，因为这种操作并不是原子性的，它是分开的读和写操作，所以可以考虑使用如下方式进行这种操作：with my\_value\_obj.get\_lock\(\): my\_value\_obj.value += 1。

**multiprocessing.Array\(typecode\_or\_type, size\_or\_initializer, \*, lock=True\)：**返回一个在共享内存中创建的ctypes类型的数组。

* **typecode\_or\_type：**指定数组中元素的类型，可以是一个ctypes类型，也可以是array模块中每个类型对应的单字符的字符串。
* **size\_or\_initializer：**如果是一个整数，则用于指定数组的长度，否则，应该传入一个序列用于初始化这个数组对象，这个序列的长度就是这个数组对象的长度。
* **lock：**默认为True，则会新建一个锁用于对这个值的同步访问操作；如果lock指定为一个Lock或RLock，则会使用这个锁来控制这个变量的同步操作；如果lock为False，那么这个值将没有锁进行保护，也就是说这个变量不是进程安全的。
* **注：**ctypes.c\_char类型的数组具有 value和raw属性，可以用来保存和提取字符串。

```py
"""
简单示例：使用共享内存的方式，共享值Value对象和数据Array对象
"""
from multiprocessing import Process, Value, Array


def func(n, a):
    n.value = 3.333
    for i in range(len(a)):
        a[i] = -a[i]


if __name__ == '__main__':
    num = Value('d', 0.0)  # 第一个参数d表示数据类型“double”双精度浮点类型
    arr = Array('i', range(6))  # 第一个参数i表示数据类型“integer”整型
    p = Process(target=func, args=(num, arr))
    p.start()
    p.join()
    print(num.value)
    print(arr[:])
```

打印输出

```py
3.333
[0, -1, -2, -3, -4, -5]
```

### 进程池

创建一个Pool进程池对象，并执行提交给它的任务，进程池对象允许其中的进程以不同的方式运行，但是需要注意，Pool对象的方法只能是创建它的进程才能调用。

#### Pool类

**multiprocessing.pool.Pool\(\[processes\[, initializer\[, initargs\[, maxtasksperchild\[, context\]\]\]\]\]\)：**创建一个进程池对象，支持带有超时和回调的异步结果，以及一个并行的map实现。

* **processes：**指定进程池中的工作进程数量，如果为None，则使用os.cpu\_count\(\)的返回值。
* **initializer和initargs：**如果initializer不为None，则每个工作进程将会在启动时调用initializer\(\*initargs\)。
* **maxtasksperchild：**指定一个工作进程在退出或者被一个新的进程替代之前能完成的任务数量，以便保证资源的释放。默认为None，表示工作进程的寿命与进程池是相同的。
* **context：**指定启动的工作进程的上下文。通常一个进程池是通过multiprocessing.Pool\(\)或者上下文对象的Pool\(\)来创建的，而这两种创建进程池的方式都是可以的。
* **注：**使用进程池对象时，应该正确终结该对象，应该将进程池对象当做上下文管理器来使用（with语句），或者手动调用close\(\)和terminate\(\)方法，而依赖于垃圾回收器来销毁进程池对象是不正确的做法。

进程池对象的常用方法：

* **apply\(func\[, args\[, kwds\]\]\)：**在进程池中开启一个新的进程并执行func函数，另外两个参数则是函数的参数，在这个函数执行完之前，当前进程会一直阻塞。
* **apply\_async\(func\[, args\[, kwds\[, callback\[, error\_callback\]\]\]\]\)：**这是apply方法的一个变体，会返回一个AsyncResult结果对象。如果指定了callback参数，则会在func函数执行成功后将返回结果当做参数传入callback指定的可调用对象，执行失败则会调用error\_callback指定的可调用对象。
* **map\(func, iterable\[, chunksize\]\)：**内置map\(\)函数的一个并行版本，会一直阻塞当前进程直到运行完可迭代对象中的所有元素，并返回结果。此方法会将可迭代对象分割为许多块，chunksize参数用于指定每个块的大小，并行的进程，每个进程会对应一个块，每次会运行块中的一个元素。注意，对于比较大的迭代对象，可能会很耗时，此时可以考虑使用imap\(\)或者imap\_unordered\(\)，并且使用时指定chunksize参数可能会得到更好的效率。
* **map\_async\(func, iterable\[, chunksize\[, callback\[, error\_callback\]\]\]\)：**map方法的一个变体，可以返回一个处理后的AsyncResult结果对象。其他参数的使用与appply\_async方法是一致的。
* **imap\(func, iterable\[, chunksize\]\)：**map\(\)方法的延迟执行版本，对于较大的迭代，chunksize设置一个较大的值会比默认值1会有更高的执行效率，同样，对于比较消耗内存的迭代，建议使用这个方法，而不是使用map\(\)方法。如果chunksize为1，则imap\(\)方法返回的迭代器的next\(\)方法拥有一个可选的参数timeout，如果在指定的timeout时间内未得到执行结果，next\(timeout\)就会抛出multiprocessing.TimeoutError异常。
* **imap\_unordered\(func, iterable\[, chunksize\]\)：**和imap\(\)类似，只不过返回的结果是无序的，当然只有一个进程的时候，返回的结果就是有序的。
* **starmap\(func, iterable\[, chunksize\]\)：**和map\(\)类似，不过iterable中的每个元素都会被再次解包作为func的参数传入进去，如\[\(1, 2\), \(3, 4\)\]会转化为类似\[func\(1, 2\), func\(3, 4\)\]。
* **starmap\_async\(func, iterable\[, chunksize\[, callback\[, error\_callback\]\]\]\)：**和starmap类似，会返回一个结果对象。
* **close\(\)：**会阻止后续任务提交到进程池，当所有任务都执行完成后，工作进程就会退出。
* **terminate\(\)：**不用等待未完成的任务，立即停止工作进程，当进程池被垃圾回收时，此方法会被立即调用。
* **join\(\)：**等待工作进程结束，注意，调用此方法前必须先调用close\(\)方法或terminate\(\)方法。

#### AsyncResult类

**multiprocessing.pool.AsyncResult：**apply\_async\(\)和map\_async\(\)这两个方法返回的结果对象对应的类。

常用的方法：

* **get\(\[timeout\]\)：**用于获取执行结果。如果timeout参数不是None，并且在指定时间内没有得到执行结果，则会抛出multiprocessing.TimeoutError异常。
* **wait\(\[timeout\]\)：**阻塞当前进程，直到返回结果，或者timeout超时。
* **ready\(\)：**判断执行是否完成。
* **successful\(\)：**判断调用是否已经完成，并且未引发异常，如果未执行完成，则会引发ValueError异常。

```py
"""
这是官方文档上给出的示例，我就直接贴在这儿了
"""
from multiprocessing import Pool
import time


def f(x):
    return x * x


if __name__ == '__main__':
    with Pool(processes=4) as pool:  # start 4 worker processes
        result = pool.apply_async(f, (10,))  # evaluate "f(10)" asynchronously in a single process
        print(result.get(timeout=1))  # prints "100" unless your computer is *very* slow

        print(pool.map(f, range(10)))  # prints "[0, 1, 4,..., 81]"

        it = pool.imap(f, range(10))
        print(next(it))  # prints "0"
        print(next(it))  # prints "1"
        print(it.next(timeout=1))  # prints "4" unless your computer is *very* slow

        result = pool.apply_async(time.sleep, (10,))
        print(result.get(timeout=1))  # raises multiprocessing.TimeoutError
```

### 编程指导

这是官方文档中对于multiprocessing模块给出的一些编程建议，我放在这里了，可以参考下。

#### 对于所有启动方法

* **避免共享状态：**应该避免在进程间传递大量数据，传递的数据应该越少越好。最好使用队列或者管道进行进程间的通信，而不是使用底层的同步原语。
* **可序列化：**保证代理的方法的参数是可序列化的。
* **代理的线程安全：**不要在多线程之间同时使用一个代理对象，除非你用锁保护它，但是在多进程之间使用相同的代理对象是不会有问题的。
* **使用join避免僵尸进程：**在Unix上，如果一个进程执行完成但是没有被join，就会变成僵尸进程。一般来说，僵尸进程不会很多，因为每次启动新进程或者active\_children\(\)被调用时，所有已执行完成且没有被join的进程都会被自动被join，而且对一个执行完成的进程调用Process.is\_alive也会join这个进程。尽管如此，对自己启动的进程显式调用join依然是最佳的实践。
* **继承优于序列化、反序列化：**当使用spawn或者forkserver的启动方式时，multiprocessing模块中的许多类型都必须是可序列化的，这样子进程才能使用它们。但是，通常我们都应该避免使用管道和队列来发送共享对象到另一个进程，而是应该优先采用让子进程通过继承的方式从父进程中访问这些共享对象。
* **避免手动杀死进程：**通过Process.terminate终止一个进程很容易导致这个进程正在使用的资源（如锁、信号量、管道和队列）损坏或者变得不可用，导致其他需要使用这些资源的进程无法使用。所以，最好是那些从来不使用这些共享资源的进程才调用Process.terminate。
* **使用队列的进程的join：**如果一个进程使用了队列，并往队列中放入数据，那么这个进程会一直阻塞，直到所有的缓存项都被feeder线程传递给底层管道，这意味着，在这个进程使用join方法之前，需要保证放入队列的全部数据都已经被其他的线程或进程消费完，否则不能保证这个队列的进程可以正常终止（注意，非守护进程都会自动join）。如下是一个死锁的示例：解决办法是交换最后两行或者删除p.join\(\)这一行。

```py
from multiprocessing import Process, Queue

def f(q):
    q.put('X' * 1000000)

if __name__ == '__main__':
    queue = Queue()
    p = Process(target=f, args=(queue,))
    p.start()
    p.join()                    # this deadlocks
    obj = queue.get()
```

* **显式传递资源给子进程：**在Unix上，使用fork方式启动的子进程可以使用父进程中全局创建的共享资源，但是还是建议显式的传递资源给子进程，这样保证了子进程结束后，这个资源也不会被回收，如果直接使用，有可能会导致子进程结束时这个资源被释放掉。如下，示例1为错误示范，应该为示例2的方式。

```py
"""示例1"""
from multiprocessing import Process, Lock


def f():
    ... do something using "lock" ...


if __name__ == '__main__':
    lock = Lock()
    for i in range(10):
        Process(target=f).start()
```

```py
"""示例2"""
from multiprocessing import Process, Lock


def f(l):
    ... do something using "l" ...


if __name__ == '__main__':
    lock = Lock()
    for i in range(10):
        Process(target=f, args=(lock,)).start()
```

#### spawn和forkserver启动方式

spawn和forkserver的以下一些特点，相对于另外一种fork启动方式，会有一些区别和限制。

* **更依赖序列化：**Process.\_\_init\_\_\(\)的所有参数都必须是可序列化的，同样的，Process的子类实例在调用start方法时也必须保证是可以被序列化的。
* **全局变量：**如果子进程在代码中尝试访问一个全局变量时，需要小心，它此时的值可能与父进程中执行Process.start方法时的值不一样了，当然，如果它是模块级别的常量时，是没问题的。
* **安全导入主模块：**需要确保主模块可以被新启动的Python解释器（比如启动了一个子进程）安全导入而不会引发其他问题。见示例1和示例2。

**示例1：**以下代码会引发RuntimeError。

```py
from multiprocessing import Process


def foo():
    print('hello')


p = Process(target=foo)
p.start()
```

**示例2：**对于以上代码，应该使用if \_\_name\_\_ == '\_\_main\_\_'来保护程序入口点。

```py
from multiprocessing import Process, freeze_support, set_start_method


def foo():
    print('hello')


# 这个入口点可以允许子进程安全导入此模块并使用此模块中的foo函数
if __name__ == '__main__':
    freeze_support()  # 如果正常运行程序而不是需要打包“冻结”，则可以忽略此句。
    set_start_method('spawn')
    p = Process(target=foo)
    p.start()
```



