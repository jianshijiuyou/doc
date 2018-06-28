
> [原文链接](https://www.jianshu.com/p/b5e347b3a17c)

# 基本用法

## 定义协程

使用 `async` 关键字将一个函数定义为协程

``` python
import asyncio

async def do_some_work(x):
    print("Waiting: ", x)

loop = asyncio.get_event_loop()
loop.run_until_complete(do_some_work(2))
```

`asyncio.get_event_loop` 方法可以创建一个事件循环，然后使用 `run_until_complete` 将协程注册到事件循环，并启动事件循环。

## 创建一个 task

协程对象不能直接运行，在注册事件循环的时候，其实是 `run_until_complete` 方法将协程包装成为了一个任务（`task`）对象。所谓 `task` 对象是 `Future` 类的子类。保存了协程运行后的状态，用于未来获取协程的结果。

``` python
import asyncio

async def do_some_work(x):
    print("Waiting: ", x)

loop = asyncio.get_event_loop()

coro = do_some_work(2)
task = loop.create_task(coro)

print(task) # 挂起状态
loop.run_until_complete(task)
print(task) # 完成状态

# <Task pending coro=<do_some_work() running at ...
# Waiting:  2
# <Task finished coro=<do_some_work() done, defined at ...
```

创建 `task` 后，`task` 在加入事件循环之前是 `pending` 状态，因为 `do_some_work` 中没有耗时的阻塞操作，`task` 很快就执行完毕了。后面打印的 `finished` 状态。

`asyncio.ensure_future(coroutine)` 和 `loop.create_task(coroutine)` 都可以创建一个 `task`，`run_until_complete` 的参数是一个 `futrue` 对象。当传入一个协程，其内部会自动封装成 `task`，`task` 是 `Future` 的子类。`isinstance(task, asyncio.Future)` 将会输出 `True`。

## 绑定回调

绑定回调，在 `task` 执行完毕的时候可以获取执行的结果，回调的最后一个参数是 `future` 对象，通过该对象可以获取协程返回值。如果回调需要多个参数，可以通过偏函数（`functools.partial`）导入。

``` python
import asyncio

async def do_some_work(x):
    print("Waiting: ", x)
    return 'this is result {}'.format(x)

def callback(future):
    print('callback: ', future.result())

loop = asyncio.get_event_loop()

coro = do_some_work(2)
task = loop.create_task(coro)
task.add_done_callback(callback)

loop.run_until_complete(task)

# Waiting:  2
# callback:  this is result 2
```

用 `partial` 传递多个参数

``` python
def callback(t, future):
    print('Callback:', t, future.result())

task.add_done_callback(functools.partial(callback, 2))
```

可以看到，`coroutine` 执行结束时候会调用回调函数。并通过参数 `future` 获取协程执行的结果。我们创建的 `task` 和回调里的 `future` 对象，实际上是同一个对象。

## future 与 result

回调中我们使用了 `future` 对象的 `result` 方法。前面不绑定回调的例子中，我们可以看到 `task` 有 `fiinished` 状态。在那个时候，可以直接读取 `task` 的 `result` 方法。

``` python
import asyncio

async def to_some_work(x):
    print('waiting {}'.format(x))
    return "this is result"

loop = asyncio.get_event_loop()
coro = to_some_work(2)
task = asyncio.ensure_future(coro)
loop.run_until_complete(task)
print(task.result())

# waiting 2
# this is result
```

## 阻塞和 await

使用 `async` 可以定义协程对象，使用 `await` 可以针对耗时的操作进行挂起，就像生成器里的 `yield` 一样，函数让出控制权。协程遇到 `await`，事件循环将会挂起该协程，执行别的协程，直到其他的协程也挂起或者执行完毕，再进行下一个协程的执行。

耗时的操作一般是一些 `IO` 操作，例如网络请求，文件读取等。我们使用 `asyncio.sleep` 函数来模拟 `IO` 操作。协程的目的也是让这些 `IO` 操作异步化。

``` python
import asyncio
import time
async def to_some_work(x):
    print('waiting {}'.format(x))
    await asyncio.sleep(2)
    return "this is result"

loop = asyncio.get_event_loop()
coro = to_some_work(2)
task = asyncio.ensure_future(coro)
start_time = time.time()
loop.run_until_complete(task)
end_time = time.time()
print(task.result())
print('total time: {}'.format(end_time-start_time))

# waiting 2
# this is result
# total time: 2.000361442565918
```

在 `sleep` 的时候，使用 `await` 让出控制权。即当遇到阻塞调用的函数的时候，使用 `await` 方法将协程的控制权让出，以便 `loop` 调用其他的协程。现在我们的例子就用耗时的阻塞操作了。

## 并发和并行

> 并发和并行一直是容易混淆的概念。并发通常指有多个任务需要同时进行，并行则是同一时刻有多个任务执行。用上课来举例就是，并发情况下是一个老师在同一时间段辅助不同的人功课。并行则是好几个老师分别同时辅助多个学生功课。简而言之就是一个人同时吃三个馒头还是三个人同时分别吃一个的情况，吃一个馒头算一个任务。

`asyncio` 实现并发，就需要多个协程来完成任务，每当有任务阻塞的时候就 `await`，然后其他协程继续工作。

``` python
import asyncio
import time


async def to_some_work(x):
    print('waiting {}'.format(x))
    await asyncio.sleep(x)
    return "this is result: {}".format(x)

loop = asyncio.get_event_loop()
coro1 = to_some_work(1)
coro2 = to_some_work(3)
coro3 = to_some_work(2)
tasks = [
    asyncio.ensure_future(coro1),
    asyncio.ensure_future(coro2),
    asyncio.ensure_future(coro3),
]
start_time = time.time()
loop.run_until_complete(asyncio.wait(tasks))
end_time = time.time()

for task in tasks:
    print(task.result())

print('total time: {}'.format(end_time-start_time))

# waiting 1
# waiting 3
# waiting 2
# this is result: 1
# this is result: 3
# this is result: 2
# total time: 2.9870269298553467
```

> `asyncio.wait(tasks)` 也可以使用 `asyncio.gather(*tasks)` 替代，前者接受一个 `task` 列表，后者接收一堆 `task`。

总共需要 `6` 秒的事情通过并发只需要 `3` 秒就可以完成了

## 协程嵌套

使用 `async` 可以定义协程，可以再协程中 `await` 另一个协程。

``` python
import asyncio
import time


async def to_some_work(x):
    print('waiting {}'.format(x))
    await asyncio.sleep(x)
    return "this is result: {}".format(x)

async def main():
    coro1 = to_some_work(1)
    coro2 = to_some_work(3)
    coro3 = to_some_work(2)
    tasks = [
        asyncio.ensure_future(coro1),
        asyncio.ensure_future(coro2),
        asyncio.ensure_future(coro3),
    ]
    dones, pendings = await asyncio.wait(tasks)
    for task in dones:
        print(task.result())

start_time = time.time()

loop = asyncio.get_event_loop()
loop.run_until_complete(main())

end_time = time.time()
print('total time: {}'.format(end_time-start_time))
```

如果使用的是 `asyncio.gather` 创建协程对象，那么 `await` 的返回值就是协程运行的结果。

``` python
    results = await asyncio.gather(*tasks)
    for result in results:
        print(result)
```

不在 `main` 协程函数里处理结果，直接返回 `await` 的内容，那么最外层的 `run_until_complete` 将会返回 `main` 协程的结果。

``` python
import asyncio
import time


async def to_some_work(x):
    print('waiting {}'.format(x))
    await asyncio.sleep(x)
    return "this is result: {}".format(x)

async def main():
    coro1 = to_some_work(1)
    coro2 = to_some_work(3)
    coro3 = to_some_work(2)
    tasks = [
        asyncio.ensure_future(coro1),
        asyncio.ensure_future(coro2),
        asyncio.ensure_future(coro3),
    ]
    return await asyncio.gather(*tasks) # *************


start_time = time.time()

loop = asyncio.get_event_loop()
results = loop.run_until_complete(main()) # *************
for result in results:
    print(result)

end_time = time.time()
print('total time: {}'.format(end_time-start_time))
```

或者返回使用 `asyncio.wait` 方式挂起协程。

``` python
import asyncio
import time


async def to_some_work(x):
    print('waiting {}'.format(x))
    await asyncio.sleep(x)
    return "this is result: {}".format(x)

async def main():
    coro1 = to_some_work(1)
    coro2 = to_some_work(3)
    coro3 = to_some_work(2)
    tasks = [
        asyncio.ensure_future(coro1),
        asyncio.ensure_future(coro2),
        asyncio.ensure_future(coro3),
    ]
    return await asyncio.wait(tasks) # *************


start_time = time.time()

loop = asyncio.get_event_loop()
dones, pending = loop.run_until_complete(main()) # *************
for task in dones:
    print(task.result())

end_time = time.time()
print('total time: {}'.format(end_time-start_time))
```

也可以使用 `asyncio` 的 `as_completed` 方法

``` python
import asyncio
import time


async def to_some_work(x):
    print('waiting {}'.format(x))
    await asyncio.sleep(x)
    return "this is result: {}".format(x)

async def main():
    coro1 = to_some_work(1)
    coro2 = to_some_work(3)
    coro3 = to_some_work(2)
    tasks = [
        asyncio.ensure_future(coro1),
        asyncio.ensure_future(coro2),
        asyncio.ensure_future(coro3),
    ]
    for task in asyncio.as_completed(tasks):
        result = await task
        print(result)


start_time = time.time()

loop = asyncio.get_event_loop()
loop.run_until_complete(main())

end_time = time.time()
print('total time: {}'.format(end_time-start_time))

# waiting 1
# waiting 3
# waiting 2
# this is result: 1
# this is result: 2
# this is result: 3
# total time: 3.0048904418945312
```

!> 注意：使用 `as_completed` 的时候，每执行完一个任务就直接返回了，所以结果是按照时间顺序打印的，和之前的都不一样。

## 协程停止

`future` 对象有几个状态：

* Pending
* Running
* Done
* Cancelled

创建 `future` 的时候，`task` 为 `pending`，事件循环调用执行的时候当然就是 `running`，调用完毕自然就是 `done`，如果需要停止事件循环，就需要先把 `task` 取消。可以使用 `asyncio.Task` 获取事件循环的 `task`

# 结合 aiohttp

一个下载图片的例子

``` python
import re
import asyncio
import aiohttp


async def download_img(url):
    filename_pattern = re.compile(r".*filename=\"(.*?)\";.*?")
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            cd = resp.headers['Content-Disposition']
            filename_match = filename_pattern.match(cd)
            imgname = filename_match.groups()[0]
            with open("images/{}".format(imgname),'wb') as fp:
                img = await resp.read()
                fp.write(img)

def main():
    pattern = re.compile(r".*(https.*file\?h=\d+&i=\d+.*?)")
    img_urls = []
    with open('imagesurl.txt', 'r') as fp:
        for line in fp:
            match = pattern.match(line)
            if match:
                url = match.groups()[0]
                img_urls.append(url)
    
    loop = asyncio.get_event_loop()
    to_do = [download_img(url) for url in img_urls]
    wait_coro = asyncio.wait(to_do)
    loop.run_until_complete(wait_coro)
    loop.close()

if __name__ == '__main__':
    main()
```

# 多线程多进程

多线程应该使用 `threading` 模块  
多进程应该使用 `multiprocessing` 模块

`concurrent.futures` 模块是对上面两个的封装，使用起来更加简单

## 多线程


``` python
from concurrent import futures
import threading

def task(args):
    print(threading.current_thread().getName())
    return args

MAX_WORKERS = 5

with futures.ThreadPoolExecutor(MAX_WORKERS) as executor:
    res = executor.map(task, range(10))
    print(list(res))

# ThreadPoolExecutor-0_0
# ThreadPoolExecutor-0_0
# ThreadPoolExecutor-0_1
# ThreadPoolExecutor-0_1
# ThreadPoolExecutor-0_2
# ThreadPoolExecutor-0_2
# ThreadPoolExecutor-0_1
# ThreadPoolExecutor-0_4
# ThreadPoolExecutor-0_2
# ThreadPoolExecutor-0_0
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

方法二

``` python
from concurrent import futures
import threading

def task(args):
    print(threading.current_thread().getName())
    return args

with futures.ThreadPoolExecutor(max_workers=3) as executor:
    to_do = []
    for item in range(10):
        future = executor.submit(task, item) # 添加任务
        to_do.append(future)

    for future in futures.as_completed(to_do): # 当任务完成就返回结果
        result = future.result()
        print(result)

# ThreadPoolExecutor-0_0
# ThreadPoolExecutor-0_0
# ThreadPoolExecutor-0_1
# 1
# ThreadPoolExecutor-0_0
# ThreadPoolExecutor-0_1
# ThreadPoolExecutor-0_2
# 0
# ThreadPoolExecutor-0_0
# ThreadPoolExecutor-0_1
# ThreadPoolExecutor-0_2
# 2
# ThreadPoolExecutor-0_0
# 4
# 3
# 5
# 6
# 7
# 8
# 9
```

!> 如果发现任务函数提前结束，加上返回值试试

## 多进程

``` python
from concurrent import futures
import multiprocessing

def task(args):
    print(multiprocessing.current_process().name)
    return args

with futures.ProcessPoolExecutor() as executor: # 默认为 os.cpu_count() 数量的进程
    res = executor.map(task, range(10))
    print(list(res))
```

或者

``` python
from concurrent import futures
import multiprocessing

def task(args):
    print(multiprocessing.current_process().name)
    return args

with futures.ProcessPoolExecutor() as executor:
    to_do = []
    for item in range(10):
        future = executor.submit(task, item) # 添加任务
        to_do.append(future)

    for future in futures.as_completed(to_do): # 当任务完成就返回结果
        result = future.result()
        print(result)
```