
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

可以看到，`coroutine` 执行结束时候会调用回调函数。并通过参数 `future` 获取协程执行的结果。我们创建的 `task` 和回调里的future对象，实际上是同一个对象。