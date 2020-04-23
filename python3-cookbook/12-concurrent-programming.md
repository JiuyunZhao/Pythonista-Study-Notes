### 12.1 启动与停止线程

Python中的线程除了使用is\_alive\(\)查询它是否存活和使用join\(\)将它加入到当前线程并等待它终止之外，并没有提供多少可以对线程操作的方法，例如不能主动终止线程，不能给线程发送信号等，如果想要对线程进行别的查询和操作，可以参考如下方案。

```py
import time
from threading import Thread


class CountdownTask:
    def __init__(self):
        self._running = True

    def terminate(self):
        self._running = False

    def run(self, n):
        while self._running and n > 0:
            print('T-minus', n)
            n -= 1
            time.sleep(5)


c = CountdownTask()
t = Thread(target=c.run, args=(10,))
t.start()
# 主动终止线程
c.terminate()
# 等待线程终止
t.join()
```



### 12.3 线程间通信

当你需要在线程间交换数据时，可以考虑使用queue库中的队列了，它的优势在于其本身就是线程安全的，如果你使用的是其他的数据结构，就需要在代码中手动添加线程锁的相关操作了。需要注意的是，队列的qsize\(\)、full\(\)和empty\(\)等方法并不是线程安全的，例如当qsize\(\)获取结果为0时，可能另一个线程马上就往队列中添加了一个数据，此时qsize\(\)的获取结果就是1了。

对于队列终止的判断，可以通过在队列中添加结束标志或者异常捕获来判断，当然，队列的操作，还是要根据具体的场景来做。

```py
from queue import Queue
from threading import Thread

# 队列终止标志
_sentinel = object()


def producer(out_q):
    while True:
        # 数据处理
        ...

        # 向队列中添加数据
        out_q.put(data)

    out_q.put(_sentinel)


def consumer(in_q):
    while True:
        # 从队列获取数据
        data = in_q.get()

        # 判断队列是否结束
        if data is _sentinel:
            in_q.put(_sentinel)
            break

        # 数据处理
        ...


q = Queue()
t1 = Thread(target=consumer, args=(q,))
t2 = Thread(target=producer, args=(q,))
t1.start()
t2.start()
```

```py
import queue

q = queue.Queue()

try:
    data = q.get(block=False)
except queue.Empty:
    ...

try:
    data = q.get(timeout=5.0)
except queue.Empty:
    ...

try:
    q.put(item, block=False)
except queue.Full:
    ...
```



### 12.4 给关键部分加锁

当你需要给可变对象添加锁时，应该考虑使用with语句，而不是手动调用acquire方法和release方法，在进入with语句时会自动获取锁，离开with语句时则自动释放锁。



### 12.5 防止死锁的加锁机制

此小节主要记录一个“哲学家就餐问题”的避免死锁的解决方案，有兴趣的可以看下。

哲学家就餐问题：五位哲学家围坐在一张桌子前，每个人 面前有一个碗饭和一只筷子。在这里每个哲学家可以看做是一个独立的线程，而每只筷子可以看做是一个锁。每个哲学家可以处在静坐、 思考、吃饭三种状态中的一个。需要注意的是，每个哲学家吃饭是需要两只筷子的，这样问题就来了：如果每个哲学家都拿起自己左边的筷子， 那么他们五个都只能拿着一只筷子坐在那儿，直到饿死。此时他们就进入了死锁状态。

```py
import threading
from contextlib import contextmanager

# 线程运行时，local()返回的实例会为每个线程创建一个属于它自己的本地存储，不同线程的本地存储互不影响，且互不可见
_local = threading.local()


# 利用上下文管理器和锁的id值进行排序来控制锁的分配
@contextmanager
def acquire(*locks):
    # Sort locks by object identifier
    locks = sorted(locks, key=lambda x: id(x))

    # 每个线程第一次运行到这儿时，结果都是空列表
    acquired = getattr(_local, 'acquired', [])
    if acquired and max(id(lock) for lock in acquired) >= id(locks[0]):
        raise RuntimeError('Lock Order Violation')

    # 为线程的本地存储添加一个列表，存储所有锁的id值
    acquired.extend(locks)
    _local.acquired = acquired

    try:
        for lock in locks:
            lock.acquire()
        yield
    finally:
        # Release locks in reverse order of acquisition
        for lock in reversed(locks):
            lock.release()
        del acquired[-len(locks):]


# 5个哲学家就餐问题实现
# The philosopher thread
def philosopher(left, right):
    while True:
        with acquire(left, right):
            print(threading.currentThread(), 'eating')


# The chopsticks (represented by locks)
NSTICKS = 5
chopsticks = [threading.Lock() for n in range(NSTICKS)]

# Create all of the philosophers
for n in range(NSTICKS):
    t = threading.Thread(target=philosopher,
                         args=(chopsticks[n], chopsticks[(n + 1) % NSTICKS]))
    t.start()
```



### 12.6 保存线程的状态信息

threading.local\(\)返回的实例可以为每个线程创建一个本地存储，即一个底层字典，不同线程之间的字典是不可见的。以下示例中，每个线程都有自己的专属套接字连接，所以多线程运行时它们是互不影响的。

```py
import threading
from functools import partial
from socket import socket, AF_INET, SOCK_STREAM


class LazyConnection:
    def __init__(self, address, family=AF_INET, type=SOCK_STREAM):
        self.address = address
        self.family = AF_INET
        self.type = SOCK_STREAM
        self.local = threading.local()

    def __enter__(self):
        if hasattr(self.local, 'sock'):
            raise RuntimeError('Already connected')
        self.local.sock = socket(self.family, self.type)
        self.local.sock.connect(self.address)
        return self.local.sock

    def __exit__(self, exc_ty, exc_val, tb):
        self.local.sock.close()
        del self.local.sock


def test(conn):
    with conn as s:
        s.send(b'GET /index.html HTTP/1.0\r\n')
        s.send(b'Host: www.python.org\r\n')

        s.send(b'\r\n')
        resp = b''.join(iter(partial(s.recv, 8192), b''))

    print('Got {} bytes'.format(len(resp)))


if __name__ == '__main__':
    conn = LazyConnection(('www.python.org', 80))

    t1 = threading.Thread(target=test, args=(conn,))
    t2 = threading.Thread(target=test, args=(conn,))
    t1.start()
    t2.start()
    t1.join()
    t2.join()
```



### 12.7 创建一个线程池

如果程序中需要使用到线程池，或者需要线程所执行函数的返回结果时，可以考虑使用from concurrent.futures import ThreadPoolExecutor。

```py
import urllib.request
from concurrent.futures import ThreadPoolExecutor


def fetch_url(url):
    u = urllib.request.urlopen(url)
    data = u.read()
    return data


# 创建线程池对象，并允许同时运行10个线程
pool = ThreadPoolExecutor(10)
# 传入线程要执行的函数，以及它的参数
a = pool.submit(fetch_url, 'http://www.python.org')
b = pool.submit(fetch_url, 'http://www.pypy.org')

# 获取线程执行结果时，会阻塞当前线程，直到该线程执行完毕并返回结果
x = a.result()
y = b.result()
```



### 12.8 简单的并行编程

如果想要进行CPU密集型运算，并且想利用CPU多核的特性，可以考虑使用concurrent.futures的ProcessPoolExecutor类，但是正如此小节标题所言，只能是执行一些简单的函数形式，其他的类方法、闭包等形式并不支持，并且函数的参数和返回结果也必须兼容pickle。

它的原理是创建N个独立的Python解释器来执行，N取决于系统CPU核心数，当然，在实例化时也可以指定ProcessPoolExecutor\(N\)。

可以使用对应map来批量执行函数，也可以使用submit来单独执行某个函数，具体使用见示例。

```py
# 使用map批量执行
from concurrent.futures import ProcessPoolExecutor


def work(x):
    ...
    return result

# 普通做法
# results = map(work, data)

# 利用CPU多核特点
with ProcessPoolExecutor() as pool:
    results = pool.map(work, data)
```

```py
from concurrent.futures import ProcessPoolExecutor


def work(x):
    ...
    return result

def when_done(r):
    print('Got:', r.result())


with ProcessPoolExecutor() as pool:
    ...
    # 单独执行某个函数
    future_result = pool.submit(work, arg)

    # 在使用result()获取结果时，当前程序会被阻塞，直到产生结果
    r = future_result.result()
    ...

    # 单独执行如果不想被阻塞，可以使用add_done_callback指定一个回调函数
    # 这个函数接受一个Future实例参数，可以在回调函数中获取执行结果
    future_result.add_done_callback(when_done)
```



