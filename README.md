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
JSON objects delimited by `\n`. The client may disconnect after receiving a
reply to a call; however, it is necessary that the client stay
connected all the time a [run](api/run.md) is underway until a message
indicating [the run has finished](api/run.md#reply-run-finished) is
received—reconnecting to an existing run is not possible.

# Client

At the start of a communication, the client must make a request to the server.
The types of requests are described [below](#request-types). Any other messages
(unless they are [answers](api/run.md#answer-client--server) to server's
queries during a run of a runnable) result in an error message.

All messages with requests from the client must follow the format described in
the example below.

**Example**

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

## Message properties

### Version

|**Property** | `"version"`|
| :---------- | :--- |
|**Values**   | String |
|**Mandatory**| yes |

This property denotes the version of the API. At the moment, the only supported
version is `1.0a`. The server will only support the last version of the API,
potentially allowing a client with a slightly older API version to connect if
the compatibility is not affected.

This property must be present in each and every message sent between the client
and the server.

### Query

|**Property** | `"query"`|
| :---------- | :--- |
|**Values**   | Object |
|**Mandatory**| yes |

All client requests must have this property, where the details of the query are
specified.

The request type (property `"request"`) and its options (property `"options"`)
are mandatory properties of the `"query"` object.

[Answers to the server's questions](api/run.md#answer-client--server) during
the run of a runnable are exempt from this requirement as they are not queries.

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

# Server

Once the server is started, it waits for connections. When the connection is
opened, no messages are emitted unless the client sends one first. The replies
the server may send to the client's requests are described on the respective
request's page.

## Error message

In case the server encounters an error, an error message is emitted.

**Example**

```
{
    "version": "1.0a",
    "error": {
        "id": "0123456789abcdef",
        "reason": "Can not create file './foo': The file already exists."
    }
}
```

The property `"id"` denoting a run ID is optional, and is only included if the
error was encountered during [a run](api/run.md) of a runnable.

The property `"reason"` containing a human-readable description of the error
is always present.
