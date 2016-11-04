---
title: python的queue模块的问题
date: 2016-07-03 02:45:46
tags:
  - queue
  - block
---

### 官网例子
`python` 的 `queue` 是线程安全的, 所有用起来比较爽.    
官方文档或者网上的例子都是比较简单的, 例如    

```python
def worker():
    while True:
        item = q.get()
        if item is None:
            break
        do_work(item)
        q.task_done()

q = queue.Queue()
threads = []
for i in range(num_worker_threads):
    t = threading.Thread(target=worker)
    t.start()
    threads.append(t)

for item in source():
    q.put(item)

# block until all tasks are done
q.join()

# stop workers
for i in range(num_worker_threads):
    q.put(None)
for t in threads:
    t.join()
```

### 实际问题
例子比较简单明了. 它们都有个特点, 就任务是有限的, 已知的, 先把所有任务放入队列, 然后在执行.        
而在实际使用中还有其他情况, 比如, 任务数量未知, 一边消耗, 一边添加. 这样也没有问题,直到给
`Queue` 设置了 `maxsize`, 问题就来了, `queue` 是满的, 所有的线程都卡住了, 包括添加任务
还是执行任务. 原因是默认是 `block`, 执行的任务还没完成, 添加任务就把 `queue` 添加满了，这时
在添加任务就不行, 卡在这里, 而执行任务的线程执行完任务后, 来取任务就取不了, 就这样双双卡死.

#### block
先来看看什么是 `block`     
```python
q = queue.Queue(3)
i = 0
while i < 10:
    r = q.get()
    print('get', r)
    print('put', i)
    q.put(i)
    i += 1
```
上面这个例子就卡住了, 什么都不打印, 因为 `q` 是空的, 它没取到任务就 `block` 在那里.
在来看看我的问题   

#### 实际问题
```python
q = queue.Queue(3)
i = 0
while i < 10:
    print('put', i)
    q.put(i)

    if i % 3 == 0:
        r = q.get()
        print('get', r)

    i += 1
```
这个例子就是我碰到的问题, 任务添加不进去, 任务也取不到, 原因就是 ***队列满了***.

#### 解决方案
如何解决这个问题就看[官网文档](https://docs.python.org/3/library/queue.html)   
```python
Queue.get(block=True, timeout=None)
Queue.put(item, block=True, timeout=None)
```

一个方法是把 `block` 设置为 `False`, 另外一个就是设置 `timeout`.

```python
q = queue.Queue(3)
i = 0
while i < 10:
    try:
        print('put', i)
        q.put(i, False)
    except queue.Full:
        print('put error', i)

    if i % 3 == 0:
        r = q.get(False)
        print('get', r)

    i += 1
```
这样才可以把 `i` 全部执行完.
