# 基本元素（Atom、Task、Retry）

## Atom

Atom是一个抽象类，是Task和Retry类型的基类，是TaskFlow中的最小单位。

## Task

Task类是一个抽象类，继承了Atom类，表示任务流中的一个任务。一个Task对象包含了任务运行`execute()`与任务回滚`revert()`两个方法。

示例：

```python
class Writer(task.Task):
    default_provides = "article"

    def execute(self, idea):
        article = "article about {}".format(idea)
        return article

    def revert(self, idea, result, flow_failures):
        if result:
            print("Can't write an article about {}".format(idea))


class Reader(task.Task):
    def execute(self, article):
        print("Read the article: {}".format(article))

```

示例中定义了两个任务类（`Writer`和`Reader`），其中`execute()`方法用于任务的运行，`Writer`任务会接收一个参数名为`idea`的输入参数，经过处理后返回参数名为`article`的输出结果，而`Reader`任务将接收来自`Writer`的返回值`article`并运行任务。而`Writer`类中的`revert()`方法将负责处理任务的回滚。

## Retry

Retry类是一个抽象类，继承了Atom类，用于处理异常与控制任务流执行，能够根据配置的策略与模式对其他任务进行重试操作。

Retry类包含一个`on_failure()`方法，返回一个枚举类型值，用于决定处理失败时采用的策略。`on_failure()`方法返回值：

- `REVERT` *= 'REVERT'*：只回滚其包含或关联的子Flow。

- `REVERT_ALL` *= 'REVERT_ALL'*：回滚整个Flow，忽略上层Retry的策略。

- `RETRY` *= 'RETRY'*：重试其包含或关联的子Flow。

TaskFlow为不同的Retry模式提供了若干实现类。Retry类的具体实现类：

- `AlwaysRevert`：发生错误时，回滚子Flow
- `AlwaysRevertAll`：发生错误时，回滚所有Flow
- `Times`：发生错误时，重试指定次数的子Flow
- `ForEach`：发生错误时，根据提供的参数值列表，将参数传递给子Flow依次重试，直到成果或列表遍历结束。
- `ParameterizedForEach`：同ForEach，但是是从存储空间中获取参数。

示例：

```python
class EchoTask(task.Task):
    def execute(self, *args, **kwargs):
        print(self.name)
        print(args)
        print(kwargs)
        
flow = linear_flow.Flow("f1").add(
    EchoTask("t1"),
    linear_flow.Flow("f2", retry=retry.ForEach(values=['a', 'b', 'c'], 
                                               name="r1", 
                                               provides="value"))
    .add(EchoTask("t2"),
         EchoTask("t3", requires="value")),
    EchoTask("t4")
)
```

任务流f2中包含一个`ForEach`类型Retry实现类r1。当任务t2或t3失败时，任务流f2将会回滚，同时Retry对象r1依次将`['a', 'b', 'c']`列表中各元素传递给任务流f2并重试。当任务t1或t4失败时，Retry对象r1不会进行重试操作。

```python
class SendMessage(task.Task):
    def execute(self, message):
        print("Sending message: %s" % message)


flow = linear_flow.Flow('send_message', retry=retry.Times(5)).add(SendMessage('sender'))
```

当任务流send_message中SendMessage任务失败时，任务流将重试最多5次，之后任务将完全回滚，任务流运行失败。

## 继承关系

![Inheritance diagram of taskflow.atom, taskflow.task, taskflow.retry.Retry, taskflow.retry.AlwaysRevert, taskflow.retry.AlwaysRevertAll, taskflow.retry.Times, taskflow.retry.ForEach, taskflow.retry.ParameterizedForEach](https://docs.openstack.org/taskflow/latest/_images/inheritance-132045c46dd04436997cbb2633233e2fad13b700.png)
