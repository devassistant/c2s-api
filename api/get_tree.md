# Get Tree (of Runnables)

> Get a tree of runnables, their details and/or arguments

In DevAssistant, the script implementing a workflow is called an *assistant*.
There are also *actions*, which take care about other activities like
installing packages, displaying help etc. In the API, these two types are equal
and are called *runnables*. They are ordered in a tree-like structure (the top
level is not a single item, but an array) that corresponds to their role, and
which can be obtained from the server by executing a `get_tree` request.

This request is useful when constructing a user interface or an argument parser
in the client that shows the user what they can run in DevAssistant with their
current configuration.

Example of a tree of runnables:
```
├── crt
│   ├── python
│   │   ├── django
│   │   └── library
│   └── ruby
│       └── rails
├── pkg
│   ├── info
│   ├── install
│   └── remove
└── help
```

The options for this call affect how many details will be sent in the response,
allowing different clients to only request information they need (e. g. the
command-line client has no use for icons).

## Workflow

The workflow for the `get_tree` request is purely synchronous:

1. **Client → Server**: `get_tree` request
1. One of:
    * **Server → Client**: Reply
    * **Server → Client**: [Error message](../README.md#error-message)


## Query Options

The `"version"` property is mandatory.

**Example**

```
{
    "version": "1.0a",
    "query": {
        "request": "get_tree",
        "options": {
            "arguments": true,
            "icons": "checksum",
            "depth": 0
        }
    }
}
```

The query may (or *must* if specified) include one of the following options:

### Arguments

|**Property** | `"arguments"`|
| :---------- | :--- |
|**Values** | `true` or `false` |
|**Default** | `false` |
|**Mandatory**| no |

To get information about the arguments of the runnables, set the `"arguments"`
option to `true`. The information will be present with every runnable in the
tree under the `"arguments"` property.

### Depth

|**Property** | `"depth"`|
| :---------- | :--- |
|**Values** | Positive integer, 0 meaning infinity |
|**Default** | `0` |
|**Mandatory**| no |

This option limits the depth of the returned tree. A limited tree depth may be
useful for GUI clients.

### Icons

|**Property** | `"icons"`|
| :---------- | :--- |
|**Values** | `null`, `"data"` or `"checksum"` |
|**Default** | `null` |
|**Mandatory**| no |

With this option (selecting `"data"`), base64-encoded icon files can be
obtained for displaying in the client. At the moment, only SVG and PNG formats
are supported in DevAssistant.

To save data, the client can fetch only checksums of the icons it stores in a
cache, and if they differ, perform another call to refresh them.

### Root

|**Property** | `"root"`|
| :---------- | :--- |
|**Values** | Slash-delimited path to runnable (with the leading slash), `"/"`, or empty string |
|**Default** | `"/"` |
|**Mandatory**| no |

**Example**: `"root": "/crt/python"`

If you want to retrieve only the children of a particular runnable, you may
select so with the `"root"` property. In the example above, that would be
subassistants to the Python *creator* assistant.


# Reply

The reply contains an array of details on runnables and their details,
including an array of their children (forming a tree-like structure).

**Example**
```
{
  "version": "1.0a",
  "tree": [
    {
      "name": "crt",
      "fullname": "Create assistants",
      "description": "Kickstart new projects easily with DevAssistant.",
      "path": "/crt",
      "icon": {
        "checksum": "0123456789abcdef",
        "data": "base64...",
        "mimetype": "image/svg+xml; charset=us-ascii"
      },
      "arguments": [
        { },
        ...
      ],
      "children": [...]
    },
    ...
  ]
}
```

## Properties

Each item in the array of runnables has these properties (some may be omitted
as specified):

### Name

| **Property** | `"name"` |
| :---------- | :------ |
| **Type** | String |
| **Present** | Always |

A short name representation of the runnable. Identical to the last element of
the property [`"path"`](#path).

### Full Name

| **Property** | `"fullname"` |
| :---------- | :------ |
| **Type** | String |
| **Present** | Always |

A human-readable name of the runnable.

### Description

| **Property** | `"description"` |
| :---------- | :------ |
| **Type** | String (with newlines) |
| **Present** | Always |

A detailed, multi-line description of the runnable.

### Path

| **Property** | `"path"` |
| :---------- | :------ |
| **Type** | String |
| **Present** | Always |

**Example**: `"/crt/python"`

Path to the runnable. In the example, leading to the Python *creator*
assistant.

### Icon

| **Property** | `"icon"` |
| :---------- | :------ |
| **Type** | Object |
| **Present** | If requested (see [`"icons"`](#icons)) |

**Example**:

```
"icon": { "checksum": "0123456789abcdef" }
```

```
"icon": { "data": "0123456789abcdef", "mimetype": "image/svg+xml; charset=us-ascii"}
```

An object describing the icon's data. Based on the [`"icons"` property](#icons)
from the query, either the property `"checksum"` with the icon's MD5 checksum
will be present, or `"data"` and `"mimetype"` will. The data are
base64-encoded.

The client may either calculate the checksums of the icons every time to verify
if they have changed, or (which may be more practical) store the checksums as
provided by the server, and check against those.

### Arguments

| **Property** | `"arguments"` |
| :---------- | :------ |
| **Type** | Array |
| **Present** | If requested (see [`"arguments"`](#arguments) |

**Example**:

```
"arguments" : [
    {
        "name": "name",
        "flags": ["-n", "--name"],
        "kwargs": {
            "dest": "name",
            "help": "Name of the project to create",
            "required": true
        },
        "positional": false
    },
    {
        "name": "venv",
        "flags": ["--venv"],
        "kwargs": {
            "action": "store_const",
            "const": "venv",
            "default": "",
            "dest": "venv",
            "help": "Use virtualenv to set up project and install dependencies."
        },
        "positional": false
    },
    ...
]

```

An array of objects, each describing an argument the runnable takes. The
properties within `"kwargs"` closely mirror the arguments of [the
`ArgumentParser.add_argument`
method](https://docs.python.org/3/library/argparse.html?highlight=argumentparser.add_argument#argparse.ArgumentParser.add_argument)
in Python. The `"kwargs"` object may be empty.

### Children

| **Property** | `"children"` |
| :---------- | :------ |
| **Type** | Array |
| **Present** | Always |

An array of child runnables (same format as this runnable, suited for recursive
traversal).
