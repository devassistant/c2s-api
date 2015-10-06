# DevAssistant Client/Server API

In this document, you can find examples and the description of usage of the
DevAssistant client/server API available in DevAssistant version 1.0 upwards.
This API is available while communicating through a UNIX domain socket or a
TCP socket without difference.

The API is a mix of synchronous and asynchronous calls between the client and
the server. All calls are blocking as the server is a single-thread,
single-connection application (that is an intentional design decision tied to
the internal workings of DevAssistant).

The connection stays open as long as the client wishes, and all messages are
JSON objects delimited by `\n`. However, it is necessary that the client stay
connected all the time a [run](api/run.md) is underway until a message
indicating [the run has finished](api/run.md#reply-run-finished) is
received—reconnecting to an existing run is not possible.

# Client Requests

The client may call any one of these three requests to initiate an action on
the server:

`get_tree`, `get_detail`, `run`.

Any other messages sent from the client are replies to server's calls during a
run of a runnable.

All requests follow the format described below:

```
{
    "version": "1.0a"
    "query":
    {
        "request": "get_tree",
        "options": {
            ...
        }
    }
}
```

## Request Types

### Get Tree

A request of this type is used to obtain a tree-like structure with the
runnables that can be invoked on the server using the [Run](#run) request, with
various level of detail.

View the [full description](api/get_tree.md).

### Get Detail

The same as the [Get Tree](#get-tree) request, only pertaining to a single
runnable (useful e. g. for GUI clients).

View the [full description](api/get_detail.md).

### Run

This request is used for invoking a runnable (an *assistant* or *action* in
DevAssistant terminology).

View the [full description](api/run.md).

## Message properties

### Version

This property denotes the version of the API. At the moment, the only supported
version is `1.0a`. The server will only support the last version of the API,
potentially allowing a client with a slightly older API version to connect if
the compatibility is not affected.

### Query

All client requests must have this property, where the details of the query are
specified.

# Server Replies

The replies the server may send to the client's requests are described on the
respective request's page.
