# Get Tree (of Runnables)

> Get a tree of runnables, their details and/or arguments

In DevAssistant, the script implementing a workflow is called an *assistant*.
There are also *actions*, which take care about other activities like
installing packages, displaying help etc. In the API, these two types are equal
and are called *runnables*. They are ordered in a tree-like structure that
corresponds to their role, and which can be obtained from the server by
executing a `get_tree` request.

This request is useful when constructing a user interface or an argument parser
in the client that shows the user what they can run in DevAssistant with their
current configuration.

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

The options for this call affect how many details will be sent in the response,
allowing different clients to only request information they need (the
command-line client has no use for icons).

Example:
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

## Options

### Arguments

> Provide information about the runnables' arguments

|**Key**      | `"arguments"`|
| :---------- | :--- |
|**Values**   | `true` or `false`
|**Default**  | `false`
|**Mandatory**| no

To get information about the arguments of the runnables, set the `"arguments"`
option to `true`. The information will be present with every runnable in the
tree under the `"arguments"` key.

### Depth

> `"depth"`: Depth of the returned tree

|**Key**      | `"depth"`|
| :---------- | :--- |
|**Values**   | Positive integer, 0 meaning infinity
|**Default**  | `0`
|**Mandatory**| no

This option limits the depth of the returned tree. A limited tree depth may be
useful for GUI clients.

### Icons

> `"icons"`: Fetch icons or their checksum

|**Key**      | `"icons"`|
| :---------- | :--- |
|**Values**   | `null`, `"data"` or `"checksum"`
|**Default**  | `null`
|**Mandatory**| no

With this option (selecting `"data"`), base64-encoded icon files can be
obtained for displaying in the client. At the moment, only SVG and PNG formats
are supported in DevAssistant.

To save data, the client can fetch only checksums of the icons it stores in a
cache, and if they differ, perform another call to refresh them.

### Root

> `"root"`: Return children of this runnable

|**Key**      | `"root"`|
| :---------- | :--- |
|**Values**   | Slash-delimited path to runnable (with the leading slash), `"/"`, or empty string
|**Default**  | `"/"`
|**Mandatory**| no

**Example**: `"root": "/crt/python"`

If you want to retrieve only the children of a particular runnable, you may
select so with the `"root"` key. In the example above, that would be
subassistants to the Python *creator* assistant.


# Reply

The reply contains a list of details on runnables and their details, including
a list of their children (forming a tree-like structure).

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

## Fields

### Name

| **Present** | **yes** |
| :---------- | :------ |

A short name representation of the runnable. Last element of the field
`"path"`.

### Fullname

| **Present** | **yes** |
| :---------- | :------ |

A human-readable name of the runnable.

### Description

| **Present** | **yes** |
| :---------- | :------ |

A detailed, multi-line description of the runnable.

### Path

| **Present** | **yes** |
| :---------- | :------ |

**Example**: `"/crt/python"`

Path to the runnable. In the example, leading to the Python *creator*
assistant.

### Icon

| **Present** | maybe |
| :---------- | :------ |

An object describing the icon's data. Based on the query from the client,
either the field `"checksum"` with the checksum will be present, or `"data"`
and `"mimetype"` will.

### Arguments

| **Present** | maybe |
| :---------- | :------ |

A list of objects, each describing an argument the runnable takes. The fields
in the argument object closely correspond to the arguments to the
`ArgumentParser.add_argument` method in Python.

### Children

| **Present** | **yes** |
| :---------- | :------ |

A list of child runnables (same format as this level, suited for recursive
traversal).
