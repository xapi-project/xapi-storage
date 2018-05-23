---
title: task

language_tabs:
 - json
 - ocaml
 - python

search: true
---
# task
The Task interface is required if the backend supports long-running  tasks.
## Type definitions
### id
```json
"id"
```
type `id` = `string`
Unique identifier for a task
### task
```json
{
  "state": [ "Pending", 0.0 ],
  "ctime": 0.0,
  "debug_info": "debug_info",
  "id": "id"
}
```
type `task` = `struct { "id": string, "debug_info": string, "ctime": float, "state": variant { Pending, Completed, Failed } }`

#### Members
 Name       | Type                                   | Description 
------------|----------------------------------------|-------------
 id         | string                                 |             
 debug_info | string                                 |             
 ctime      | float                                  |             
 state      | variant { Pending, Completed, Failed } |             
### task_list
```json
[ "task_list" ]
[]
```
type `task_list` = `string list`

## Interface: `Task`
The task interface is for querying the status of asynchronous  tasks. All long-running operations are associated with tasks,  including copying and mirroring of data.
## Method: `stat`
[stat task_id] returns the status of the task

> Client

```json
{
  "method": "Task.stat",
  "params": [ { "id": "id", "dbg": "dbg" } ],
  "id": 41
}
```

```ocaml
try
    let result = Client.stat dbg id in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Task.stat({ dbg: "string", id: "string" })
    print (repr(results))
```

> Server

```json
{
  "state": [ "Pending", 0.0 ],
  "ctime": 0.0,
  "debug_info": "debug_info",
  "id": "id"
}
```

```ocaml
try
    let result = Client.stat dbg id in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Task_myimplementation(Task_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def stat(self, dbg, id):
        """
        [stat task_id] returns the status of the task
        """
        result = {}
        result["result"] = {"id": "string", "debug_info": "string", "ctime": 1.1, "state": None}
        return result
    # ...
```


 Name   | Direction | Type   | Description                   
--------|-----------|--------|-------------------------------
 dbg    | in        | string | Debug context from the caller 
 id     | in        | id     | Unique identifier for a task  
 result | out       | task   |                               
## Method: `cancel`
[cancel task_id] performs a best-effort cancellation of an ongoing  task. The effect of this should leave the system in one of two  states: Either that the task has completed successfully, or that it  had never been made at all. The call should return immediately and  the status of the task can the be queried via the [stat] call.

> Client

```json
{
  "method": "Task.cancel",
  "params": [ { "id": "id", "dbg": "dbg" } ],
  "id": 42
}
```

```ocaml
try
    let () = Client.cancel dbg id in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Task.cancel({ dbg: "string", id: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.cancel dbg id in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Task_myimplementation(Task_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def cancel(self, dbg, id):
        """
        [cancel task_id] performs a best-effort cancellation of an ongoing
        task. The effect of this should leave the system in one of two
        states: Either that the task has completed successfully, or that it
        had never been made at all. The call should return immediately and
        the status of the task can the be queried via the [stat] call.
        """
        result = {}
        return result
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 id   | in        | id     | Unique identifier for a task  
## Method: `destroy`
[destroy task_id] should remove all traces of the task_id. This call  should fail if the task is currently in progress.

> Client

```json
{
  "method": "Task.destroy",
  "params": [ { "id": "id", "dbg": "dbg" } ],
  "id": 43
}
```

```ocaml
try
    let () = Client.destroy dbg id in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Task.destroy({ dbg: "string", id: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.destroy dbg id in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Task_myimplementation(Task_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def destroy(self, dbg, id):
        """
        [destroy task_id] should remove all traces of the task_id. This call
        should fail if the task is currently in progress.
        """
        result = {}
        return result
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 id   | in        | id     | Unique identifier for a task  
## Method: `ls`
[ls] should return a list of all of the tasks the plugin is aware of.

> Client

```json
{ "method": "Task.ls", "params": [ { "dbg": "dbg" } ], "id": 44 }
```

```ocaml
try
    let task_list = Client.ls dbg in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Task.ls({ dbg: "string" })
    print (repr(results))
```

> Server

```json
[ "task_list_1", "task_list_2" ]
```

```ocaml
try
    let task_list = Client.ls dbg in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Task_myimplementation(Task_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def ls(self, dbg):
        """
        [ls] should return a list of all of the tasks the plugin is aware of.
        """
        result = {}
        result = ["string"]
        return result
    # ...
```


 Name    | Direction | Type      | Description                   
---------|-----------|-----------|-------------------------------
 dbg     | in        | string    | Debug context from the caller 
 unnamed | out       | task_list |                               
## Errors
### exnt
```json
[ "Unimplemented", "exnt" ]
```
type `exnt` = `variant { Unimplemented }`

#### Constructors
 Name          | Type   | Description 
---------------|--------|-------------
 Unimplemented | string |             