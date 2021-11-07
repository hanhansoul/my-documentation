# `greenthread`

*class* eventlet.greenthread.**GreenThread**(*parent*)

The GreenThread class is a type of Greenlet which has the additional property of being able to retrieve the return value of the main function. Do not construct GreenThread objects directly; call [`spawn()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.spawn) to get one.

GreenThread类是greenlet的衍生类，GreenThread增加了额外的属性用于获取main函数的返回值。使用`spawn()`等方法创建GreenThread对象。

- **cancel**(**throw_args*)

  通过`greenthread.kill()`方法杀死未启动的绿色线程。

  Kills the greenthread using [`kill()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.kill), but only if it hasn’t already started running. After being canceled, all calls to [`wait()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread.wait) will raise *throw_args* (which default to `greenlet.GreenletExit`).

- **kill**(**throw_args*)

  通过`greenthread.kill()`方法杀死绿色线程。

  Kills the greenthread using [`kill()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.kill). After being killed all calls to [`wait()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread.wait) will raise *throw_args* (which default to `greenlet.GreenletExit`).

- **link**(*func*, **curried_args*, ***curried_kwargs*)

  Set up a function to be called with the results of the GreenThread.

  The function must have the following signature:

  ```python
  def func(gt, [curried args/kwargs]):
  ```

  When the GreenThread finishes its run, it calls *func* with itself and with the [curried arguments](http://en.wikipedia.org/wiki/Currying) supplied at link-time. If the function wants to retrieve the result of the GreenThread, it should call wait() on its first argument.

  Note that *func* is called within execution context of the GreenThread, so it is possible to interfere with other linked functions by doing things like switching explicitly to another greenthread.

- **unlink**(*func*, **curried_args*, ***curried_kwargs*)

  remove linked function set by [`link()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread.link)

  Remove successfully return True, otherwise False

- **wait**()

  返回GreenThread的main函数的结果或异常。

  Returns the result of the main function of this GreenThread. If the result is a normal return value, [`wait()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread.wait) returns it. If it raised an exception, [`wait()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread.wait) will raise the same exception (though the stack trace will unavoidably contain some frames from within the greenthread module).



---

- eventlet.greenthread.**getcurrent**() → greenlet

  Returns the current greenlet (i.e. the one which called this function).

- eventlet.greenthread.**kill**(*g*, **throw_args*)

  Terminates the target greenthread by raising an exception into it. Whatever that greenthread might be doing; be it waiting for I/O or another primitive, it sees an exception right away.

  By default, this exception is GreenletExit, but a specific exception may be specified.*throw_args* should be the same as the arguments to raise; either an exception instance or an exc_info tuple.

  Calling [`kill()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.kill) causes the calling greenthread to cooperatively yield.

- eventlet.greenthread.**sleep**(*seconds=0*)

  Yield control to another eligible coroutine until at least *seconds* have elapsed.

  *seconds* may be specified as an integer, or a float if fractional seconds are desired. Calling `sleep()` with *seconds* of 0 is the canonical way of expressing a cooperative yield. For example, if one is looping over a large list performing an expensive calculation without calling any socket methods, it’s a good idea to call `sleep(0)`occasionally; otherwise nothing else will run.

- eventlet.greenthread.**spawn**(*func*, **args*, ***kwargs*)

  Create a greenthread to run `func(*args, **kwargs)`. Returns a [`GreenThread`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread) object which you can use to get the results of the call.Execution control returns immediately to the caller; the created greenthread is merely scheduled to be run at the next available opportunity. Use [`spawn_after()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.spawn_after) to arrange for greenthreads to be spawned after a finite delay.

- eventlet.greenthread.**spawn_after**(*seconds*, *func*, **args*, ***kwargs*)

  Spawns *func* after *seconds* have elapsed. It runs as scheduled even if the current greenthread has completed.*seconds* may be specified as an integer, or a float if fractional seconds are desired. The *func* will be called with the given *args* and keyword arguments *kwargs*, and will be executed within its own greenthread.The return value of [`spawn_after()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.spawn_after) is a [`GreenThread`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread) object, which can be used to retrieve the results of the call.To cancel the spawn and prevent *func* from being called, call [`GreenThread.cancel()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread.cancel) on the return value of [`spawn_after()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.spawn_after). This will not abort the function if it’s already started running, which is generally the desired behavior. If terminating *func*regardless of whether it’s started or not is the desired behavior, call [`GreenThread.kill()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread.kill).

- eventlet.greenthread.**spawn_after_local**(*seconds*, *func*, **args*, ***kwargs*)

  Spawns *func* after *seconds* have elapsed. The function will NOT be called if the current greenthread has exited.*seconds* may be specified as an integer, or a float if fractional seconds are desired. The *func* will be called with the given *args* and keyword arguments *kwargs*, and will be executed within its own greenthread.The return value of [`spawn_after()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.spawn_after) is a [`GreenThread`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread) object, which can be used to retrieve the results of the call.To cancel the spawn and prevent *func* from being called, call [`GreenThread.cancel()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread.cancel) on the return value. This will not abort the function if it’s already started running. If terminating *func* regardless of whether it’s started or not is the desired behavior, call [`GreenThread.kill()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.GreenThread.kill).

- eventlet.greenthread.**spawn_n**(*func*, **args*, ***kwargs*)

  Same as [`spawn()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.spawn), but returns a `greenlet` object from which it is not possible to retrieve either a return value or whether it raised any exceptions. This is faster than [`spawn()`](http://eventlet.net/doc/modules/greenthread.html#eventlet.greenthread.spawn); it is fastest if there are no keyword arguments.If an exception is raised in the function, spawn_n prints a stack trace; the print can be disabled by calling [`eventlet.debug.hub_exceptions()`](http://eventlet.net/doc/modules/debug.html#eventlet.debug.hub_exceptions) with False.



---



```python
eventlet.sleep = greenthread.sleep
eventlet.spawn = greenthread.spawn
eventlet.spawn_n = greenthread.spawn_n
eventlet.spawn_after = greenthread.spawn_after
eventlet.kill = greenthread.kill
```

