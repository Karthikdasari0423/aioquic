QUIC API
========

The QUIC API performs no I/O on its own, leaving this to the API user.
This allows you to integrate QUIC in any Python application, regardless of
the concurrency model you are using.

Connection
----------

.. automodule:: aioquic.quic.connection

    .. autoclass:: QuicConnection
        :members:


Configuration
-------------

.. automodule:: aioquic.quic.configuration

    .. autoclass:: QuicConfiguration
        :members:

.. automodule:: aioquic.quic.logger

    .. autoclass:: QuicLogger
        :members:

Understanding Stream Limits
---------------------------

QUIC employs flow control mechanisms at both the stream and connection levels,
and also limits the number of concurrent streams a peer can initiate. These
limits are communicated during the TLS handshake via transport parameters.

By default, an `aioquic` endpoint (acting as a server) will advertise to its
peer (the client) that the client can initiate up to 128 concurrent
bidirectional streams and 128 concurrent unidirectional streams. These are
fixed default values in the library.

The peer must not open more streams than advertised by this endpoint. If a peer
attempts to do so, `aioquic` will treat this as a connection error.

Conversely, when `aioquic` is acting as a client, it respects the
`initial_max_streams_bidi` and `initial_max_streams_uni` values *received from*
the server. If an application using `aioquic` as a client attempts to create
more streams than the server allows (e.g., by calling
:meth:`~aioquic.quic.connection.QuicConnection.get_next_available_stream_id`
and sending data on that new stream ID), the stream creation might appear to
succeed locally, but the underlying `QuicConnection` will buffer the request
to open a new stream until the server increases its limit. This is because the
`QuicConnection` cannot send data for new streams if it would exceed the
peer-advertised limit.

Both endpoints can dynamically update their stream limits by sending
``MAX_STREAMS`` frames. `aioquic` automatically sends these frames when
appropriate, for instance, when the local application consumes data from
streams, effectively freeing up capacity and allowing the peer to initiate more.

Applications that need to open a large number of streams (especially on the
client-side) should be aware of these limits. While `aioquic` handles the
underlying mechanism of respecting and updating stream limits, an application
trying to open streams aggressively might see requests queue up if the peer's
advertised limit is reached and not yet updated. The `http3_client.py` example
includes a warning mechanism if many client-side requests are pending, which can
be a symptom of approaching or hitting such server-imposed limits.

Events
------

.. automodule:: aioquic.quic.events

    .. autoclass:: QuicEvent
        :members:

    .. autoclass:: ConnectionTerminated
        :members:

    .. autoclass:: HandshakeCompleted
        :members:

    .. autoclass:: PingAcknowledged
        :members:

    .. autoclass:: StopSendingReceived
        :members:

    .. autoclass:: StreamDataReceived
        :members:

    .. autoclass:: StreamReset
        :members:
