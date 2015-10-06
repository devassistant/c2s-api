# Run (a runnable)

By performing the `run` request, a runnable will be loaded and invoked by the
server. After receiving the request, the server will reply with an
acknowledgment message, and initiate the run. During the run, the server will
send logging messages to the client to display, and may issue interactive
queries for the user to answer.

Multiple logging messages can (and likely will) be sent to the client as
logging events occur on the server, meaning this part of the communication is
not of the request-response type.

The interactive querying has the form of the server asking a question while
halting the operation of the runnable. Before the client sends an answer, no
logging messages will be emitted.

**Example**

```
{
    "version": "1.0a",
    "query": {
        "request": "run",
        "options": {
            "path": "/crt/python/django",
            "pwd": "/home/user"
            "arguments": {
                "name": "foo",
                "virtualenv": true
            }
        }
    }
}
```

## Workflow

1. **Client -> Server**: Run request
1. **Server -> Client**: Acknowledgment
1. Any number of times one of:
    * **Server -> Client**: 0+ Logging messages
    * An interactive query
        1. **Server -> Client**: Query
        1. **Client -> Server**: Reply to the query
1. **Server -> Client**: Run finished

## Options

### Path

|**Property** | `"path"`|
| :---------- | :--- |
|**Values**   | Slash-delimited path to runnable (with the leading slash) |
|**Mandatory**| yes |

**Example**: `"path": "/crt/python/django"`

The path to the runnable as provided in the `"path"` property in the `get_tree`
or `get_detail` requests.

### PWD

|**Property** | `"pwd"`|
| :---------- | :--- |
|**Values**   | Slash-delimited path to current directory |
|**Mandatory**| yes |

The path to the directory where the server should run the runnable, usually the
client's `$PWD`.

### Arguments

|**Property** | `"arguments"`|
| :---------- | :--- |
|**Values**   | Object with `"name": value` pairs for each argument |
|**Mandatory**| yes |

**Example**

```
"arguments": {
    "name": "foo",
    "virtualenv": true
}
```

The arguments for the assistant to run, as provided by the `get_tree` and
`get_detail` requests. The property name is the argument name (as opposed to a
flag or metavar).
