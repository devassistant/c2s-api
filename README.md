# DevAssistant Client/Server API

In this document, you can find examples and the description of usage of the
DevAssistant client/server API available in DevAssistant version 1.0 upwards.
This API is available while communicating through a UNIX domain socket and a
TCP socket without difference.

The API is a mix of synchronous and asynchronous calls between the client and
the server. All calls are blocking as the server is a single-thread,
single-serve application (that was a design decision tied to the internal
workings of DevAssistant).

The connection stays open as long as the client wishes, and all messages are
JSON objects delimited by `\n`. It does not matter if the client makes an API
call, disconnects, and re-connects to initiate a run of a runnable, or stays
connected the whole time.

# Client Requests

The client may call any one of these three requests to initiate an action on
the server: `get_tree`, `get_detail`, `run`. Any other messages sent from the
client are replies to server's calls during a run of a runnable.

All requests follow the format described below:
```
{
    "version": "1.0a"
    "query":
    {
        "request": "get_tree" or "get_detail" or "run"
        OPTION1: VALUE
        OPTION2: VALUE
        ...
    }
}
```

## Required arguments

### Version

> The version of the API

Currently, the only supported version is `1.0a`. The server will only support
the last version of the API, potentially allowing a client with an older API
version if the compatibility is not affected.

### Query

> A key whose value is a dictionary with the parameters of the call

All client requests must have this keyword.

## Request types

### Get Tree (of runnables)

In DevAssistant, the script implementing a workflow is called an *assistant*.
There are also *actions*, which take care about other activities like
installing packages, displaying help etc. In the API, these two types are equal
and are called *runnables*. They are ordered in a tree-like structure that
corresponds to their role, and which can be obtained from the server by
executing a `get_tree` request.

Example of a tree of runnables:
```
- crt
    - python
      - django
      - library
    - ruby
      - rails
- pkg
    - install
    - info
    - remove
- help
```

### Format

The request
