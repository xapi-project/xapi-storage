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
Unique identifier for a task.
### volume
```json
{
  "keys": { "keys": "keys" },
  "uri": [ "uri" ],
  "physical_utilisation": 0,
  "virtual_size": 0,
  "sharable": true,
  "read_write": true,
  "description": "description",
  "name": "name",
  "uuid": "uuid",
  "key": "key"
}
```
type `volume` = `struct { ... }`

#### Members
 Name                 | Type                   | Description                                                                                                                                                                                                                                                                                                                                                                         
----------------------|------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 key                  | string                 | A primary key for this volume. The key must be unique within the enclosing Storage Repository \(SR\). A typical value would be a filename or an LVM volume name.                                                                                                                                                                                                                    
 uuid                 | string option          | A uuid \(or guid\) for the volume, if one is available. If a storage system has a built-in notion of a guid, then it will be returned here.                                                                                                                                                                                                                                         
 name                 | string                 | Short, human-readable label for the volume. Names are commonly used by when displaying short lists of volumes.                                                                                                                                                                                                                                                                      
 description          | string                 | Longer, human-readable description of the volume. Descriptions are generally only displayed by clients when the user is examining volumes individually.                                                                                                                                                                                                                             
 read_write           | bool                   | True means the VDI may be written to, false means the volume is read-only. Some storage media is read-only so all volumes are read-only; for example .iso disk images on an NFS share. Some volume are created read-only; for example because they are snapshots of some other VDI.                                                                                                 
 sharable             | bool                   | Indicates whether the VDI can be attached by multiple hosts at once. This is used for example by the HA statefile and XAPI redo log.                                                                                                                                                                                                                                                
 virtual_size         | int64                  | Size of the volume from the perspective of a VM \(in bytes\)                                                                                                                                                                                                                                                                                                                        
 physical_utilisation | int64                  | Amount of space currently used on the backing storage \(in bytes\)                                                                                                                                                                                                                                                                                                                  
 uri                  | string list            | A list of URIs which can be opened and by a datapath plugin for I/O. A URI could reference a local block device, a remote NFS share, iSCSI LUN or RBD volume. In cases where the data may be accessed over several protocols, the list should be sorted into descending order of desirability. Xapi will open the most desirable URI for which it has an available datapath plugin. 
 keys                 | (string * string) list | A list of key=value pairs which have been stored in the Volume metadata. These should not be interpreted by the Volume plugin.                                                                                                                                                                                                                                                      
### async_result_t
```json
"UnitResult"
[
  "Volume",
  {
    "keys": { "keys": "keys" },
    "uri": [ "uri" ],
    "physical_utilisation": 0,
    "virtual_size": 0,
    "sharable": true,
    "read_write": true,
    "description": "description",
    "name": "name",
    "uuid": "uuid",
    "key": "key"
  }
]
```
type `async_result_t` = `variant { ... }`

#### Constructors
 Name       | Type   | Description 
------------|--------|-------------
 UnitResult | unit   |             
 Volume     | volume |             
### completion_t
```json
{ "result": "UnitResult", "duration": 0.0 }
```
type `completion_t` = `struct { ... }`

#### Members
 Name     | Type                  | Description 
----------|-----------------------|-------------
 duration | float                 |             
 result   | async_result_t option |             
### state
```json
[ "Pending", 0.0 ]
[ "Completed", { "result": "UnitResult", "duration": 0.0 } ]
[ "Failed", "state" ]
```
type `state` = `variant { ... }`

#### Constructors
 Name      | Type         | Description                                            
-----------|--------------|--------------------------------------------------------
 Pending   | float        | The task is in progress, with progress info from 0..1. 
 Completed | completion_t |                                                        
 Failed    | string       |                                                        
### task
```json
{
  "state": [ "Pending", 0.0 ],
  "ctime": 0.0,
  "debug_info": "debug_info",
  "id": "id"
}
```
type `task` = `struct { ... }`

#### Members
 Name       | Type   | Description 
------------|--------|-------------
 id         | string |             
 debug_info | string |             
 ctime      | float  |             
 state      | state  |             
### task_list
```json
[ "task_list" ]
[]
```
type `task_list` = `string list`

## Interface: `Task`
The task interface is for querying the status of asynchronous  tasks. All long-running operations are associated with tasks,  including copying and mirroring of data.
## Method: `stat`
\[stat task\_id\] returns the status of the task

> Client

```json
{
  "method": "Task.stat",
  "params": [ { "id": "id", "dbg": "dbg" } ],
  "id": 44
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
    print(repr(results))
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
        return {"id": "string", "debug_info": "string", "ctime": 1.1, "state": None}
    # ...
```


 Name   | Direction | Type   | Description                   
--------|-----------|--------|-------------------------------
 dbg    | in        | string | Debug context from the caller 
 id     | in        | id     | Unique identifier for a task. 
 result | out       | task   |                               
## Method: `cancel`
\[cancel task\_id\] performs a best-effort cancellation of an ongoing  task. The effect of this should leave the system in one of two  states: Either that the task has completed successfully, or that it  had never been made at all. The call should return immediately and  the status of the task can the be queried via the \[stat\] call.

> Client

```json
{
  "method": "Task.cancel",
  "params": [ { "id": "id", "dbg": "dbg" } ],
  "id": 45
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
    print(repr(results))
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
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 id   | in        | id     | Unique identifier for a task. 
## Method: `destroy`
\[destroy task\_id\] should remove all traces of the task\_id. This call  should fail if the task is currently in progress.

> Client

```json
{
  "method": "Task.destroy",
  "params": [ { "id": "id", "dbg": "dbg" } ],
  "id": 46
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
    print(repr(results))
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
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 id   | in        | id     | Unique identifier for a task. 
## Method: `ls`
\[ls\] should return a list of all of the tasks the plugin is aware of.

> Client

```json
{ "method": "Task.ls", "params": [ { "dbg": "dbg" } ], "id": 47 }
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
    print(repr(results))
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
        return ["string"]
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
type `exnt` = `variant { ... }`

#### Constructors
 Name          | Type   | Description 
---------------|--------|-------------
 Unimplemented | string |             