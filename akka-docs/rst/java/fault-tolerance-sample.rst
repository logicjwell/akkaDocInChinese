.. _fault-tolerance-sample-java:

容错示例图解
----------------------------------------------

.. image:: ../images/faulttolerancesample-normal-flow.png

*上图阐明了正常的消息流*

**正常流程:**

======= ==================================================================================
步骤    描述
======= ==================================================================================
1       ``Listener`` 开始运转。
2       ``Worker`` 通过周期性地向自己发送 ``Do`` 消息来调度工作。
3, 4, 5 当收到 ``Do`` 时 ``Worker`` 让 ``CounterService`` 自增计数器三次。 ``Increment`` 消息被转发到 ``Counter``, 它会更新计数器变量并且将当前值发送到 ``Storage``.
6, 7    ``Worker`` 向 ``CounterService``询问计数器的当前值，并且用管道将此结果连接回``Listener``.
======= ==================================================================================


.. image:: ../images/faulttolerancesample-failure-flow.png

*上图阐明了存储错误的情况下所发生的事情。*

**错误流程:**

=========== ==================================================================================
步骤        描述
=========== ==================================================================================
1           ``Storage`` 抛出 ``StorageException``.
2           ``CounterService`` 是 ``Storage`` 的监管者，在 ``StorageException`` 被抛出时重启了  ``Storage`` 。
3, 4, 5, 6  ``Storage`` 继续出错，并被重启。
7           在发生3次错误并在5秒后重启之后， ``Storage`` 被其监管者停止，也就是 ``CounterService``.
8           ``CounterService`` 也在监控 ``Storage`` 的终止，并且当 ``Storage`` 被停止后接收到了 ``Terminated`` 消息 ...
9, 10, 11   并告诉 ``Counter`` 不存在 ``Storage``.
12          ``CounterService`` 调度一个 ``Reconnect`` 消息发给自己。
13, 14      当接收到 ``Reconnect`` 消息时它创建一个新的 ``Storage`` ...
15, 16      并且让 ``Counter`` 使用新的 ``Storage``
=========== ==================================================================================

容错示例的完整源代码
------------------------------------------------------

.. includecode:: code/docs/actor/japi/FaultHandlingDocSample.java#all

