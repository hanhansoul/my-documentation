# Job

## 概述

Job与JobBoard对象为任务流的运行提供高可用性、可扩展性与容错性。

JobBoard类似于一个队列，消费者从队列中获取需要执行的任务，只有执行成功才会将其从队列中移除，否则任务会保留在队列中直到成功运行。

## 定义

1. Job：包含一个唯一标识、名称和一个包含任务相关信息的LogBook对象。

2. JobBoard：负责管理一个或多个Job，提供了Job的发布、获取与搜索功能。

![../_images/jobboard.png](https://docs.openstack.org/taskflow/latest/_images/jobboard.png)

1. 高可用
2. 原子性
3. 任务流创建与运行的解耦合
4. 异步执行

```python
from taskflow.persistence import backends as persistence_backends
from taskflow.jobs import backends as job_backends

...
persistence = persistence_backends.fetch({
    "connection': "mysql",
    "user": ...,
    "password": ...,
})
book = make_and_save_logbook(persistence)
board = job_backends.fetch('my-board', {
    "board": "zookeeper",
}, persistence=persistence)
job = board.post("my-first-job", book)
...
```



```python
import time

from taskflow import exceptions as exc
from taskflow.persistence import backends as persistence_backends
from taskflow.jobs import backends as job_backends

...
my_name = 'worker-1'
coffee_break_time = 60
persistence = persistence_backends.fetch({
    "connection': "mysql",
    "user": ...,
    "password": ...,
})
board = job_backends.fetch('my-board', {
    "board": "zookeeper",
}, persistence=persistence)
while True:
    my_job = None
    for job in board.iterjobs(only_unclaimed=True):
        try:
            board.claim(job, my_name)
        except exc.UnclaimableJob:
            pass
        else:
            my_job = job
            break
    if my_job is not None:
        try:
            perform_job(my_job)
        except Exception:
            LOG.exception("I failed performing job: %s", my_job)
            board.abandon(my_job, my_name)
        else:
            # I finished it, now cleanup.
            board.consume(my_job)
            persistence.get_connection().destroy_logbook(my_job.book.uuid)
    time.sleep(coffee_break_time)
...
```



```python
from oslo_utils import uuidutils

from taskflow import engines
from taskflow.persistence import backends as persistence_backends
from taskflow.persistence import models
from taskflow.jobs import backends as job_backends


...
persistence = persistence_backends.fetch({
    "connection': "mysql",
    "user": ...,
    "password": ...,
})
board = job_backends.fetch('my-board', {
    "board": "zookeeper",
}, persistence=persistence)

book = models.LogBook('my-book', uuidutils.generate_uuid())

flow_detail = models.FlowDetail('my-job', uuidutils.generate_uuid())
book.add(flow_detail)

connection = persistence.get_connection()
connection.save_logbook(book)

flow_detail.meta['store'] = {'a': 1, 'c': 3}

job_details = {
    "flow_uuid": flow_detail.uuid,
    "store": {'a': 2, 'b': 1}
}

engines.save_factory_details(flow_detail, flow_factory,
                             factory_args=[],
                             factory_kwargs={},
                             backend=persistence)

jobboard = get_jobboard(zk_client)
jobboard.connect()
job = jobboard.post('my-job', book=book, details=job_details)

# the flow global parameters are now the combined store values
# {'a': 2, 'b': 1', 'c': 3}
...
```



## 类型

1. ZooKeeper：ZookeeperJobBoard
2. Redis：RedisJobBoard



## 继承关系

![Inheritance diagram of taskflow.jobs.base, taskflow.jobs.backends.impl_redis, taskflow.jobs.backends.impl_zookeeper](https://docs.openstack.org/taskflow/latest/_images/inheritance-e896f0e266651aa414dca3d7d8cc50224c0edad6.png)