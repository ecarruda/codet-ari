**ORIGINAL REP :** [asterisk/ari-py](https://github.com/asterisk/ari-py.git) 

About
-----

This package contains the Python client library for the Asterisk REST
Interface. It builds upon the
`Swagger.py <https://github.com/Thulebona/codet-swaggerpy.git>`__ library, providing an
improved, Asterisk-specific API over the API generated by Swagger.py

Usage
-----

Install from source using the ``setup.py`` script.

::

    $ pip install codet-ari


API
===

An ARI client can be created simply by the ``ari.connect`` method. This will
create a client based on the Swagger API downloaded from Asterisk.

The API is modeled into the Repository Pattern, as you would find in Domain
Driven Design. Each Swagger Resource (a.k.a. API declaration) is mapped into a
Repository object, which is provided as a field on the client
(``client.channels``, ``client.bridges``).

Responses from Asterisk are mapped into first-class objects, akin to Domain
Objects in the Repository Pattern. These are provided both on the responses
to RESTful API calls, and for fields from events received over the WebSocket.

Making REST calls
=================

Each Repository Object provides methods which invoke the non-instance specific
operations of the associated Swagger resource (``bridges.list()``,
``channels.get()``). Instance specific methods are also provided, which require
identity parameters to be passed along (``channels.get(channelId=id)``).

Instance specific methods are also provided on the Domain Objects
(``some_channel.hangup()``).

Registering event callbacks
===========================

Asterisk may send asyncronous messages over a WebSocket to indicate events of
interest to the application.

The ``Client`` object has an ``on_event`` method, which can be used to
subscribe for specific events from Asterisk.

The first-class objects also have 'on_event' methods, which can subscribe to
Stasis events relating to that object.

Object lifetime
===============

The Repository Objects exist for the lifetime of the client that owns them.

Domain Objects are ephemeral, and not tied to the lifetime of the underlying
object in Asterisk. Pratically, this means that if you call
``channels.get('1234')`` several times, you may get a different object back
every time.

You may hold onto an instance of a Domain Object, but you should consider it
to be stale. The data contained in the object may be out of date, but the
methods on the object should still behave properly.

If you invoke a method on a stale Domain Object that no longer exists in
Asterisk, you will get a HTTPError exception (404 Not Found).

Caveats
=======

The dynamic methods exposed by Repository and Domain objects are, effectively,
remote procedure calls. The current implementation is synchronous, which means
that if anything were to happen to slow responses (slow network, packet loss,
system load, etc.), then the entire application could be affected.

Examples
========

.. code:: Python

    import ari

    client = ari.connect('http://localhost:8088/', 'hey', 'peekaboo')

    def on_dtmf(channel, event):
        digit = event['digit']
        if digit == '#':
            channel.play(media='sound:goodbye')
            channel.continueInDialplan()
        elif digit == '*':
            channel.play(media='sound:asterisk-friend')
        else:
            channel.play(media='sound:digits/%s' % digit)


    def on_start(channel, event):
        channel.on_event('ChannelDtmfReceived', on_dtmf)
        channel.answer()
        channel.play(media='sound:hello-world')


    client.on_channel_event('StasisStart', on_start)
    client.run(apps="hello")



Development
-----------

The code is documented using `Sphinx <http://sphinx-doc.org/>`__, which
allows `IntelliJ IDEA <http://confluence.jetbrains.net/display/PYH/>`__
to do a better job at inferring types for autocompletion.

To keep things isolated, I also recommend installing (and using)
`virtualenv <http://www.virtualenv.org/>`__.

::

    $ sudo pip install virtualenv
    $ mkdir -p ~/virtualenv
    $ virtualenv ~/virtualenv/ari
    $ . ~/virtualenv/ari/bin/activate

`Setuptools <http://pypi.python.org/pypi/setuptools>`__ is used for
building. `Nose <http://nose.readthedocs.org/en/latest/>`__ is used
for unit testing, with the `coverage
<http://nedbatchelder.com/code/coverage/>`__ plugin installed to
generated code coverage reports. Pass ``--with-coverage`` to generate
the code coverage report. HTML versions of the reports are put in
``cover/index.html``.

::

    $ ./setup.py develop   # prep for development (install deps, launchers, etc.)
    $ ./setup.py test # run unit tests
    $ ./setup.py bdist_egg # build distributable

TODO
====

 * Create asynchronous bindings that can be used with Twisted, Tornado, etc.
 * Add support for Python 3

License
-------

Copyright (c) 2013-2014, Digium, Inc. All rights reserved.

Swagger.py is licensed with a `BSD 3-Clause
License <http://opensource.org/licenses/BSD-3-Clause>`__.
