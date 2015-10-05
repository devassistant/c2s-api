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

## Options

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
            "icons": checksum,
            "depth": 0
        }
    }
}
```

### Depth

> `"depth"`: Depth of the returned tree

**Values**: Positive integer, 0 meaning infinity

This option limits the depth of the returned tree. A limited tree depth may be
useful for GUI clients.

### Icons

> `"icons"`: Fetch icons or their checksum

**Values**: `null`, `"data"`, or `"checksum"`
