.. highlight:: python
.. module:: aioredis.commands

Getting started
===============


Commands Pipelining
-------------------

Commands pipelining is built-in.

Every command is sent to transport at-once
(ofcourse if no TypeErrors/ValueErrors were raised)

When you making a call with ``yield from`` you will be waiting result,
but if you want to make several calls simply collect futures of those calls
and then gather results.

Simple example show both cases (:download:`get source code<../examples/pipeline.py>`):

.. literalinclude:: ../examples/pipeline.py
   :language: python
   :lines: 10-24
   :dedent: 4

.. note::

   As as convenience :mod:`aioredis` provides
   :meth:`~TransactionsCommandsMixin.pipeline`
   method allowing to execute bulk of commands at once
   (:download:`get source code<../examples/pipeline.py>`):

      .. literalinclude:: ../examples/pipeline.py
         :language: python
         :lines: 26-35
         :dedent: 4


Multi/Exec transactions
-----------------------

:mod:`aioredis` provides several ways for executing transactions:

* when using raw connection you can issue 'Multi'/'Exec' commands
  manually;

* when using :class:`aioredis.Redis` instance you can either use
  :meth:`~TransactionsCommandsMixin.multi`/
  :meth:`~TransactionsCommandsMixin.exec` methods

* or use :meth:`~TransactionsCommandsMixin.multi_exec` transaction pipeline.

The later one is described in more details.

:meth:`~TransactionsCommandsMixin.multi_exec` method creates and returns new
:class:`~aioredis.commands.MultiExec` object which is used for buffering commands and
then executing them inside MULTI/EXEC block.

Here is simple example
(:download:`get source code<../examples/transaction2.py>`):

.. literalinclude:: ../examples/transaction2.py
   :language: python
   :lines: 10-17
   :linenos:
   :emphasize-lines: 6
   :dedent: 4

As you can notice ``yield from`` is **only** used at line 6 with ``tr.execute``
and **not with** ``tr.set(...)`` calls.

.. warning::

   It is very important not to ``yield from`` buffered command
   (ie ``tr.set('foo', '123')``) as it will block forever.

   The following code will block forever::

      tr = redis.multi_exec()
      yield from tr.incr('foo')   # that's all. we've stuck!


Pub/Sub mode
------------

:mod:`aioredis` provides support for Redis Publish/Subscribe messaging.

To switch connection to subcribe mode you must execute ``subscribe`` command
by yield'ing from :meth:`~PubSubCommandsMixin.subscribe` it returns a list of
:class:`~aioredis.Channel` objects representing subscribed channels.

As soon as connection is switched to subscribed mode the channel will receive
and store messages
(the ``Channel`` object is basically a wrapper around :class:`asyncio.Queue`).
To read messages from channel you need to use :meth:`~aioredis.Channel.get`
or :meth:`~aioredis.Channel.get_json` coroutines.

.. note::
   In Pub/Sub mode redis connection can only receive messages or issue
   (P)SUBSCRIBE / (P)UNSUBSCRIBE commands.

Pub/Sub example (:download:`get source code<../examples/pubsub2.py>`):

.. literalinclude:: ../examples/pubsub2.py
   :language: python
   :lines: 7-34
   :dedent: 4

.. warning::
   Using Pub/Sub mode with :class:`~aioredis.Pool` is possible but
   only within ``with`` block or by explicitly ``acquiring/releasing``
   connection. See example below.

Pub/Sub example (:download:`get source code<../examples/pool_pubsub.py>`):

.. literalinclude:: ../examples/pool_pubsub.py
   :language: python
   :lines: 14-36
   :dedent: 4


Python 3.5 async/await support
------------------------------

:mod:`aioredis` is compatible with :pep:`492`.

:class:`~aioredis.Pool` can be used with :ref:`async with<async with>`
(:download:`get source code<../examples/python_3.5_pool.py>`):

.. literalinclude:: ../examples/python_3.5_pool.py
   :language: python
   :lines: 6-8,20-22
   :dedent: 4


It also can be used with ``await``:

.. literalinclude:: ../examples/python_3.5_pool.py
   :language: python
   :lines: 6-8,26-28
   :dedent: 4


New ``scan``-family commands added with support of :ref:`async for<async for>`
(:download:`get source code<../examples/python_3.5_iscan.py>`):

.. literalinclude:: ../examples/python_3.5_iscan.py
   :language: python
   :lines: 7-9,29-31,34-36,39-41,44-45
   :dedent: 4


SSL/TLS support
---------------

Though Redis server `does not support data encryption <data_encryption_>`_
it is still possible to setup Redis server behind SSL proxy. For such cases
:mod:`aioredis` library support secure connections through :mod:`asyncio`
SSL support. See `BaseEventLoop.create_connection`_ for details.

.. _data_encryption: http://redis.io/topics/security#data-encryption-support
.. _BaseEventLoop.create_connection: https://docs.python.org/3/library/asyncio-eventloop.html#creating-connections
