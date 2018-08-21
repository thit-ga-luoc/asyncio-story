a simple event loop
===================

a simple event loop

How to implement an event loop ?

- coroutine/generator
- asyncio task
- event loop
- I/O
- sleep/timers


-------------------------
Generators and Coroutines
-------------------------

A ``generator`` is a function that produces a sequence of values. It generates a series of values through the ``yield`` statement.

- Generator doesn't produce all values at one time, it only generate value when you "need" it (calling ``.next()`` in python2 or ``next(generator)`` in python3). it doesn't waste memory.
- Calling a generator function creates an generator object. However, it does not start running the function, until the ``.next()`` call.
- The function only executes on ``next()``.
- ``yield`` produces a value, but suspends the function.
- Function resumes on next call to `next()`.
- If a generator function calls return or reaches the end its definition, a ``StopIteration`` exception is raised.

let's go through simple code snippet.

.. code-block:: python

    def generate_numbers(n):
        print("starting generating numbers...")
        for i in range(n):
            yield i

    # python2.7
    >>> g = generate_numbers(3)
    >>> g
    <generator object generate_numbers at 0x108808a50>
    # generator has not been started, it didn't print anything
    >>> g.next()
    starting generating numbers...
    0
    >>> g.next()
    1
    >>> g.next()
    2
    >>> g.next()
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    StopIteration


In `python 2.5`, `PEP342 <https://www.python.org/dev/peps/pep-0342/>`_ introduced a feature of sending stuff into a paused ``generator``, python has the concept of ``coroutines`` then. Among other things, ``PEP342`` introduced the ``send()`` method on generators. This allowed one to not only pause generators, but to send a value back into a generator where it paused.

- ``coroutine`` do more than pause function and generate value, it can also consume values sent to it
- Sent values are returned by ``(yield)``
- Execution is the same as for a generator
- When you call a coroutine, nothing happens, they only run in response to ``next()`` and ``send()`` methods

.. code-block:: python

    def compare_numbers(n):
        print("Does input number less than {} ?".format(n))
        while True:
            i = (yield)
            if i < n:
                print("yes!")
            else:
                print("no!")

    >>> c = compare_numbers(5)
    # did not print anything
    >>> c.next()
    # coroutine starts running after first .next() call
    Does input number less than 5 ?
    >>> c.send(3)
    yes!
    >>> c.send(10)
    no!

------------
asyncio task
------------

What do we actually do when defining ``async def`` function in ``> Python 3.5`` ? Let's see,

.. code-block:: python

    async def hello_world():
        print("hey, hello world!")

    # create coroutine
    coro = hello_world()

    >>> type(coro)
    <class 'coroutine'>
    >>> type(coro.send)
    <class 'builtin_function_or_method'>
    >>> type(coro.throw)
    <class 'builtin_function_or_method'>

Note that the code above never printed our "hey, hello world!" message. That's because nothing happened, we never actually executed statements inside the coroutine function, we simply created the coroutine object.


But, how can we execute the coroutine ? We can schedule it by calling ``.send(None)``.

.. code-block:: python

    >>> coro.send(None)
    hey, hello world!
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    StopIteration


As we can see, a couroutine object's ``.send(None)`` method raises ``StopIteration``. We can try schedule the coroutine and catch
``StopIteration``.

.. code-block:: python

    >>> try:
    >>>     coro.send(None)
    >>> except StopIteration:
    >>>     pass
    hey, hello world!


We've seen how task runs, but what happens if task raises ``Exception`` ?

.. code-block:: python

    async def hello_world():
        raise ValueError("hey, hello world!")

    # create coroutine
    coro = hello_world()

    >>> try:
    >>>     coro.send(None)
    >>> except Exception as e:
    >>>     print("Catch coroutine's exception {}".format(str(e)))



----------
event loop
----------


------------
Handling I/O
------------


----------------
sleep and timers
----------------
