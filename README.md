Tasq
====

Very simple broker-less distributed Task queue that allow the scheduling of job functions to be
executed on local or remote workers. Can be seen as a Proof of Concept leveraging ZMQ sockets and
cloudpickle serialization capabilities as well as a very basic actor system to handle different
loads of work from connecting clients.


## Quickstart

Starting a worker on a node, with debug flag set to true on configuration file

```
$ tq --worker
DEBUG - Push channel set to 127.0.0.1:9000
DEBUG - Pull channel set to 127.0.0.1:9001
DEBUG - MainThread - Response actor started
```

In a python shell

```
Python 3.6.5 (default, Apr 12 2018, 22:45:43)
[GCC 7.3.1 20180312] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from tasq import TasqClient
>>> tc = TasqClient('127.0.0.1', 9000)
>>> tc.connect()
>>>
>>> def foo(num):
>>>     import time
>>>     import random
>>>     r = random.randint(0, 2)
>>>     time.sleep(r)
>>>     return f'Foo - {random.randint(0, num)}'
>>>
>>> tc.schedule(foo, 5, name='Task-1')
>>> tc.results
>>> {'Task-1': 'Foo - 3'}
>>>
>>> tc.schedule_blocking(foo, 5, name='Task-2')
>>> ('Task-2', 'Foo - 4')
>>>
>>> tc.results
>>> {'Task-1': 'Foo - 3', 'Task-2': 'Foo - 4'}
```

Tasq also supports an optional static configuration file, in the `tasq.settings.py` module is
defined a configuration class with some default fields. By setting the environment variable
`TASQ_CONF` it is possible to configure the location of the json configuration file on the
filesystem.

By setting the `-f` flag it is possible to also set a location of a configuration to follow on the
filesystem

```
$ tq --worker -f path/to/conf/conf.json
```

## Behind the scenes

Essentially it is possible to start workers across the nodes of a network without forming a cluster
and every single node can host multiple workers by setting differents ports for the communication.
Each worker, once started, support multiple connections from clients and is ready to accept tasks.

Once a worker receive a job from a client, it demand its execution to dedicated actor, usually
selected from a pool according to a defined routing strategy (e.g. Round robin, Random routing or
Smallest mailbox which should give a trivial indication of the workload of each actor and select the
one with minimum pending tasks to execute).

![Tasq master-workers arch](static/worker_model_2.png)

Another (pool of) actor(s) is dedicated to answering the clients with the result once it is ready,
this way it is possible to make the worker listening part unblocking and as fast as possible.

The reception of jobs from clients is handled by `ZMQ.PULL` socket while the response transmission
handled by `ResponseActor` is served by `ZMQ.PUSH` socket, effectively forming a dual channel of
communication, separating ingoing from outgoing traffic.

## Installation

Being a didactical project it is not released on Pypi yet, just clone the repository and install it
locally or play with it using `python -i` or `ipython`.

```
$ git clone https://github.com/codepr/tasq.git
$ cd tasq
$ pip install .
```

or, to skip cloning part

```
$ pip install -e git+https://github.com/codepr/tasq.git@master
```

## Changelog

See the [CHANGES](CHANGES.md) file.

## TODO:

- A meaningful client pool
- Debugging multiprocessing start for more workers on the same node
- Refactor of existing code and corner case handling (Still very basic implementation of even simple
  heuristics)
- Delayed tasks and scheduled cron tasks
- Configuration handling throughout the code
- Better explanation of the implementation and actors defined
- Improve CLI options
- Dockerfile
