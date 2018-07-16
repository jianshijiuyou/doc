# 基本用法

``` python
import argparse
import socket

def recvall(sock, length):
    data = b''
    while len(data) < length:
        more = sock.recv(length - len(data)) # 会阻塞等待数据
        if not more:
            raise EOFError('没有更多数据了')
        data += more
    return data

def server(interface, port):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    sock.bind((interface, port))
    sock.listen(1) # 设置连接最大数
    print('listening at', sock.getsockname()) # ('0.0.0.0', 1060)
    while True:
        sc, sockname = sock.accept()
        print('接收到一个连接', sockname) # ('127.0.0.1', 36632)
        print('Socket name:', sc.getsockname()) # ('127.0.0.1', 1060)
        print('Socket peer:', sc.getpeername()) # ('127.0.0.1', 36632)
        message = recvall(sc, 16)
        print('收到的消息:', repr(message))
        sc.sendall(b'Farewell, client')
        sc.close()
        print(' Reply sent, socket closed')

def client(host, port):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect((host, port))
    print('已经连接到服务器:' ,sock.getsockname()) # ('127.0.0.1', 36632)
    sock.sendall(b'Hi there, server')
    reply = recvall(sock, 16)
    print('The server said', repr(reply))
    sock.close()

if __name__ == '__main__':
    choices = {'client': client, 'server': server}
    parser = argparse.ArgumentParser('Send and receive over TCP')
    parser.add_argument('role', choices=choices, help='which role to play')
    parser.add_argument('host', help='interface the server listens at;'
                        ' host the client sends to')
    parser.add_argument('-p', metavar = 'PORT', type=int, default=1060,
                        help='TCP port (default 1060)')
    args = parser.parse_args()
    function = choices[args.role]
    function(args.host, args.p)
```

构造一个服务端，将所有字符转化成大写

``` python
import socket, sys

def server(host, port):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    sock.bind((host, port))
    sock.listen(1)
    print('listening at', sock.getsockname())
    while True:
        sc, sockname = sock.accept()
        print('接收到 client:', sockname)
        while True:
            data = sc.recv(1024)
            if not data:
                break
            output = data.decode('ascii').upper().encode('ascii')
            sc.sendall(output)
            sys.stdout.flush()
        print()
        sc.close()

if __name__ == '__main__':
    server('', 1060)
```

## 死锁

如果服务端一直在向客户端发送数据，而客户端没有接收，那么就会填满客户端的缓冲区，导致死锁发生。

## 半开连接

双向套接字的任何一方都可以调用 `shutdown()` 方法关闭指定方向上的通信连接

``` python
sock.shutdown(socket.SHUT_WR)
```

参数有三个选择：

* `SHUT_WR`: 最长用的参数值，表示调用方不再向套接字写入数据，通信对方也不会再读取任何数据并认为遇到了文件结束符。
* `SHUT_RD`: 表示不再从套接字读取数据，当对方尝试发送更多数据时，就会引发文件结束错误。
* `SHUT_RDWR`: 表示不再写入也不再读取。关闭双向通信。

# 高并发 Demo

使用 Tornado 实现高并发

server

``` python
from tornado.tcpserver import TCPServer
from tornado.iostream import StreamClosedError
from tornado.ioloop import IOLoop

class EchoServer(TCPServer):
    async def handle_stream(self, stream, address):
        print('address:{}'.format(address))
        while True:
            try:
                data = await stream.read_until(b"\n")
                await stream.write(data)
            except StreamClosedError:
                print('客户端断开连接 {}'.format(address))
                break

server = EchoServer()
server.listen(8888)
IOLoop.current().start()
```

测试 client

``` python
import socket
import asyncio

async def task(index):
    print('创建第 {} 个连接'.format(index))
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect(('192.168.31.31', 8888))
    address = sock.getsockname()
    try:
        while True:
            sock.sendall(b'this is test text\n')
            data = sock.recv(1024)
            print(address)
            await asyncio.sleep(1)
    except Exception:
        print('关闭连接')
        sock.close()

loop = asyncio.get_event_loop()

tasks = [task(i) for i in range(2000)]

loop.run_until_complete(asyncio.wait(tasks))

```

linux 下默认只能同时保持 1024 个文件句柄，需要修改这个设置

https://blog.csdn.net/fdipzone/article/details/34588803

同时连接数也受内存，带宽等的限制