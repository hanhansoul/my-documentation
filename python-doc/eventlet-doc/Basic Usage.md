## Greenthread Spawn

- eventlet.**spawn**(*func*, **args*, ***kw*)

  **spawn**方法启动一个绿色线程用于执行函数*func*。 `spawn`方法返回一个[`greenthread.GreenThread`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread)对象，可用于获取函数*func*的返回值。

- eventlet.**spawn_n**(*func*, **args*, ***kw*)

  **spawn_n**方法与[`spawn()`](http://eventlet.net/doc/basic_usage.html#eventlet.spawn)类似，但无法获取函数*func*的返回值，但执行速断更快。

- eventlet.**spawn_after**(*seconds*, *func*, **args*, ***kw*)

  **spawn_after**方法与[`spawn()`](http://eventlet.net/doc/basic_usage.html#eventlet.spawn)类似，是在给定时间之后执行函数*func*。调用 [`spawn_after()`](http://eventlet.net/doc/basic_usage.html#eventlet.spawn_after)方法的返回对象的`GreenThread.cancel()`方法可以中止函数执行。

## Greenthread Control

- eventlet.**sleep**(*seconds=0*)

  暂停当前绿色线程给定时间，并允许其他绿色线程有机会执行。

- *class* eventlet.**GreenPool**

  用于控制并发的线程池。

  Pools control concurrency. It’s very common in applications to want to consume only a finite amount of memory, or to restrict the amount of connections that one part of the code holds open so as to leave more for the rest, or to behave consistently in the face of unpredictable input data. GreenPools provide this control. See [`GreenPool`](http://eventlet.net/doc/modules/greenpool.html#eventlet.greenpool.GreenPool) for more on how to use these.

- *class* eventlet.**GreenPile**

  GreenPile对象表示一系列任务。GreenPile对象本质上是一个保存任务的迭代器。

  GreenPile objects represent chunks of work. In essence a GreenPile is an iterator that can be stuffed with work, and the results read out later. See [`GreenPile`](http://eventlet.net/doc/modules/greenpool.html#eventlet.greenpool.GreenPile) for more details.

- *class* eventlet.**Queue**

  Queue用于在绿色线程之间传递数据。

  Queues are a fundamental construct for communicating data between execution units. Eventlet’s Queue class is used to communicate between greenthreads, and provides a bunch of useful features for doing that. See [`Queue`](http://eventlet.net/doc/modules/queue.html#eventlet.queue.Queue) for more details.

- *class* eventlet.**Timeout**

  This class is a way to add timeouts to anything. It raises *exception* in the current greenthread after *timeout* seconds. When *exception* is omitted or `None`, the Timeout instance itself is raised.Timeout objects are context managers, and so can be used in with statements. See [`Timeout`](http://eventlet.net/doc/modules/timeout.html#eventlet.timeout.Timeout) for more details.

## Patching Functions

- eventlet.**import_patched**(*modulename*, **additional_modules*, ***kw_additional_modules*)

  Imports a module in a way that ensures that the module uses “green” versions of the standard library modules, so that everything works nonblockingly. The only required argument is the name of the module to be imported. For more information see [Import Green](http://eventlet.net/doc/patching.html#import-green).

- eventlet.**monkey_patch**(*all=True*, *os=False*, *select=False*, *socket=False*, *thread=False*, *time=False*)

  Globally patches certain system modules to be greenthread-friendly. The keyword arguments afford some control over which modules are patched. If *all* is True, then all modules are patched regardless of the other arguments. If it’s False, then the rest of the keyword arguments control patching of specific subsections of the standard library. Most patch the single module of the same name (os, time, select). The exceptions are socket, which also patches the ssl module if present; and thread, which patches thread, threading, and Queue. It’s safe to call monkey_patch multiple times. For more information see [Monkeypatching the Standard Library](http://eventlet.net/doc/patching.html#monkey-patch).

## Network Convenience Functions

- eventlet.**connect**(*addr*, *family=AddressFamily.AF_INET*, *bind=None*)

  Convenience function for opening client sockets.Parameters**addr** – Address of the server to connect to. For TCP sockets, this is a (host, port) tuple.**family** – Socket family, optional. See [`socket`](https://docs.python.org/3/library/socket.html#module-socket) documentation for available families.**bind** – Local address to bind to, optional.ReturnsThe connected green socket object.

- eventlet.**listen**(*addr*, *family=AddressFamily.AF_INET*, *backlog=50*, *reuse_addr=True*, *reuse_port=None*)

  Convenience function for opening server sockets. This socket can be used in [`serve()`](http://eventlet.net/doc/basic_usage.html#eventlet.serve)or a custom `accept()` loop.Sets SO_REUSEADDR on the socket to save on annoyance.Parameters**addr** – Address to listen on. For TCP sockets, this is a (host, port) tuple.**family** – Socket family, optional. See [`socket`](https://docs.python.org/3/library/socket.html#module-socket) documentation for available families.**backlog** – The maximum number of queued connections. Should be at least 1; the maximum value is system-dependent.ReturnsThe listening green socket object.

- eventlet.**wrap_ssl**(*sock*, **a*, ***kw*)

  Convenience function for converting a regular socket into an SSL socket. Has the same interface as [`ssl.wrap_socket()`](https://docs.python.org/3/library/ssl.html#ssl.wrap_socket), but can also use PyOpenSSL. Though, note that it ignores the cert_reqs, ssl_version, ca_certs, do_handshake_on_connect, and suppress_ragged_eofs arguments when using PyOpenSSL.The preferred idiom is to call wrap_ssl directly on the creation method, e.g., `wrap_ssl(connect(addr))` or `wrap_ssl(listen(addr), server_side=True)`. This way there is no “naked” socket sitting around to accidentally corrupt the SSL session.:return Green SSL object.

- eventlet.**serve**(*sock*, *handle*, *concurrency=1000*)

  Runs a server on the supplied socket. Calls the function *handle* in a separate greenthread for every incoming client connection. *handle* takes two arguments: the client socket object, and the client address:`def myhandle(client_sock, client_addr):    print("client connected", client_addr) eventlet.serve(eventlet.listen(('127.0.0.1', 9999)), myhandle) `Returning from *handle* closes the client socket.[`serve()`](http://eventlet.net/doc/basic_usage.html#eventlet.serve) blocks the calling greenthread; it won’t return until the server completes. If you desire an immediate return, spawn a new greenthread for [`serve()`](http://eventlet.net/doc/basic_usage.html#eventlet.serve).Any uncaught exceptions raised in *handle* are raised as exceptions from [`serve()`](http://eventlet.net/doc/basic_usage.html#eventlet.serve), terminating the server, so be sure to be aware of the exceptions your application can raise. The return value of *handle* is ignored.Raise a [`StopServe`](http://eventlet.net/doc/basic_usage.html#eventlet.StopServe) exception to gracefully terminate the server – that’s the only way to get the server() function to return rather than raise.The value in *concurrency* controls the maximum number of greenthreads that will be open at any time handling requests. When the server hits the concurrency limit, it stops accepting new connections until the existing ones complete.

- *class* eventlet.**StopServe**

  Exception class used for quitting [`serve()`](http://eventlet.net/doc/basic_usage.html#eventlet.serve) gracefully.

These are the basic primitives of Eventlet; there are a lot more out there in the other Eventlet modules; check out the [Module Reference](http://eventlet.net/doc/modules.html).