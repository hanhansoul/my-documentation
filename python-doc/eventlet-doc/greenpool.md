# `greenpool`

*class* eventlet.greenpool.**GreenPile**(*size_or_pool=1000*)

GreenPile is an abstraction representing a bunch of I/O-related tasks.

Construct a GreenPile with an existing GreenPool object. The GreenPile will then use that pool’s concurrency as it processes its jobs. There can be many GreenPiles associated with a single GreenPool.

A GreenPile can also be constructed standalone, not associated with any GreenPool. To do this, construct it with an integer size parameter instead of a GreenPool.

It is not advisable to iterate over a GreenPile in a different greenthread than the one which is calling spawn. The iterator will exit early in that situation.

- **next**()

  Wait for the next result, suspending the current greenthread until it is available. Raises StopIteration when there are no more results.

- **spawn**(*func*, **args*, ***kw*)

  Runs *func* in its own green thread, with the result available by iterating over the GreenPile object.
  
  

---

*class* eventlet.greenpool.**GreenPool**(*size=1000*)

The GreenPool class is a pool of green threads.

- **free**()

  Returns the number of greenthreads available for use.

  If zero or less, the next call to [`spawn()`](http://eventlet.net/doc/modules/greenpool.html#eventlet.greenpool.GreenPool.spawn) or [`spawn_n()`](http://eventlet.net/doc/modules/greenpool.html#eventlet.greenpool.GreenPool.spawn_n) will block the calling greenthread until a slot becomes available.

- **imap**(*function*, **iterables*)

  This is the same as `itertools.imap()`, and has the same concurrency and memory behavior as [`starmap()`](http://eventlet.net/doc/modules/greenpool.html#eventlet.greenpool.GreenPool.starmap).

  It’s quite convenient for, e.g., farming out jobs from a file:

```python
def worker(line):
    return do_something(line)
pool = GreenPool()
for result in pool.imap(worker, open("filename", 'r')):
    print(result)
```

- **resize**(*new_size*)

  Change the max number of greenthreads doing work at any given time.

  If resize is called when there are more than *new_size* greenthreads already working on tasks, they will be allowed to complete but no new tasks will be allowed to get launched until enough greenthreads finish their tasks to drop the overall quantity below *new_size*. Until then, the return value of free() will be negative.

- **running**()

  Returns the number of greenthreads that are currently executing functions in the GreenPool.

- **spawn**(*function*, **args*, ***kwargs*)

  Run the *function* with its arguments in its own green thread. Returns the `GreenThread` object that is running the function, which can be used to retrieve the results.If the pool is currently at capacity, `spawn` will block until one of the running greenthreads completes its task and frees up a slot.This function is reentrant; *function* can call `spawn` on the same pool without risk of deadlocking the whole thing.

- **spawn_n**(*function*, **args*, ***kwargs*)

  Create a greenthread to run the *function*, the same as [`spawn()`](http://eventlet.net/doc/modules/greenpool.html#eventlet.greenpool.GreenPool.spawn). The difference is that [`spawn_n()`](http://eventlet.net/doc/modules/greenpool.html#eventlet.greenpool.GreenPool.spawn_n) returns None; the results of *function* are not retrievable.

- **starmap**(*function*, *iterable*)

  This is the same as [`itertools.starmap()`](https://docs.python.org/3/library/itertools.html#itertools.starmap), except that *func* is executed in a separate green thread for each item, with the concurrency limited by the pool’s size. In operation, starmap consumes a constant amount of memory, proportional to the size of the pool, and is thus suited for iterating over extremely long input lists.

- **waitall**()

  Waits until all greenthreads in the pool are finished working.

- **waiting**()

  Return the number of greenthreads waiting to spawn.

