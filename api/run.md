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

## Workflow

The workflow could be formalized as follows:

1. **Client → Server**: Run request
1. **Server → Client**: Acknowledgment
1. Any number of times one of:
    * **Server → Client**: 1 or more logging messages
    * An interactive query
        1. **Server → Client**: Query
        1. **Client → Server**: Reply to the query
    * **Server → Client**: Error message
1. **Server → Client**: Run finished

## Query Options

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

# Reply (Acknowledgment)

When the run is initiated, the server send the client a message that looks like
this:

```
{
    "version": "1.0a",
    "run": {
        "id": "0123456789abcdef"
    }
}
```

The run ID is a UUID identifying the particular run. The same ID will be
present in all logging messages and queries that occur during that run. The
client should save it for the time of the run as it is necessary for the
replies to interactive querying.

# Reply (Logging)

**Example**

```
{
    "version": "1.0a",
    "log": {
        "level": "info",
        "message": "File created: ./foo",
        "id": "0123456789abcdef"
    }
}
```

The log message always contains all properties shown in the example. The
logging messages fall into the standard four levels of logging: `debug`,
`info`, `warning`, and `error`, and each message is accompanied by its run ID.

# Reply (Run Finished)

**Example**

```
{
    "version": "1.0a",
    "finished": {
        "id": "0123456789abcdef",
        "status": "ok"
    }
}
```

This message is emitted by the server when the run of the specified ID is
finished. The `"status"` can be either `"ok"`, or `"error"`. The server will
always emit this message when the run is finished regardless of errors.

# Interactive Querying

## Question (Server → Client)

**Example**

```
{
    "version": "1.0a",
    "question": {
        "id": "0123456789abcdef"
        "prompt": "Username:",
        "message": "Please select a GitHub username.",
        "type": "password"
    }
}
```

### ID

| **Property** | `"id"` |
| :---------- | :------ |
| **Type** | String |
| **Present** | Always |

The ID of the run.

### Prompt

| **Property** | `"prompt"` |
| :---------- | :------ |
| **Type** | String |
| **Present** | Always |

The short prompt text of the question (without a space at the end).

### Message

| **Property** | `"message"` |
| :---------- | :------ |
| **Type** | String |
| **Present** | Always |

The long ushering text of the question.

### Type

| **Property** | `"type"` |
| :---------- | :------ |
| **Type** | `null` or `"password"` |
| **Present** | Always |

The `type` property is currently used for indicating a password, i. e. the text
should be hidden.

In the future, multiple-choice and similar question types should be supported.

## Answer (Client → Server)

**Example**

```
{
    "version": "1.0a",
    "answer": {
        "id": "0123456789abcdef"
        "value": "foo",
    }
}
```

The value of the answer is always a string.
