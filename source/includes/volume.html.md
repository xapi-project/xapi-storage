---
title: volume

language_tabs:
 - json
 - ocaml
 - python

search: true
---
# volume
The xapi toolstack delegates all storage control-plane functions to  "Volume plugins".These plugins allow the toolstack to  create/destroy/snapshot/clone volumes which are organised into groups  called Storage Repositories \(SR\). Volumes have a set of URIs which  can be used by the "Datapath plugins" to connect the disk data to  VMs.
## Type definitions
### configuration
```json
{ "configuration": "configuration" }
```
type `configuration` = `(string * string) list`
Plugin-specific configuration which describes where and how to locate the storage repository. This may include the physical block device name, a remote NFS server and path or an RBD storage pool.
### health
```json
[ "Healthy", "health" ]
[ "Recovering", "health" ]
```
type `health` = `variant { ... }`

#### Constructors
 Name       | Type   | Description                                         
------------|--------|-----------------------------------------------------
 Healthy    | string | Storage is fully available                          
 Recovering | string | Storage is busy recovering, e.g. rebuilding mirrors 
### sr_stat
```json
{
  "health": [ "Healthy", "health" ],
  "clustered": true,
  "datasources": [ "datasources" ],
  "total_space": 0,
  "free_space": 0,
  "description": "description",
  "uuid": "uuid",
  "name": "name",
  "sr": "sr"
}
```
type `sr_stat` = `struct { ... }`

#### Members
 Name        | Type          | Description                                                                                                                                                                         
-------------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 sr          | string        | The URI identifying this volume. A typical value would be a file:// URI pointing to a directory or block device                                                                     
 name        | string        | Short, human-readable label for the SR.                                                                                                                                             
 uuid        | string option | Uuid that uniquely identifies this SR, if one is available. For SRs that are created by SR.create, this should be the value passed into that call, if it is possible to persist it. 
 description | string        | Longer, human-readable description of the SR. Descriptions are generally only displayed by clients when the user is examining SRs in detail.                                        
 free_space  | int64         | Number of bytes free on the backing storage \(in bytes\)                                                                                                                            
 total_space | int64         | Total physical size of the backing storage \(in bytes\)                                                                                                                             
 datasources | string list   | URIs naming datasources: time-varying quantities representing anything from disk access latency to free space. The entities named by these URIs are self-describing.                
 clustered   | bool          | Indicates whether the SR uses clustered local storage.                                                                                                                              
 health      | health        | The health status of the SR.                                                                                                                                                        
### probe_result
```json
{
  "extra_info": { "extra_info": "extra_info" },
  "sr": {
    "health": [ "Healthy", "health" ],
    "clustered": true,
    "datasources": [ "datasources" ],
    "total_space": 0,
    "free_space": 0,
    "description": "description",
    "uuid": "uuid",
    "name": "name",
    "sr": "sr"
  },
  "complete": true,
  "configuration": { "configuration": "configuration" }
}
```
type `probe_result` = `struct { ... }`

#### Members
 Name          | Type                   | Description                                                                                                                                                                                                      
---------------|------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 configuration | (string * string) list | Plugin-specific configuration which describes where and how to locate the storage repository. This may include the physical block device name, a remote NFS server and path or an RBD storage pool.              
 complete      | bool                   | True if this configuration is complete and can be used to call SR.create or SR.attach. False if it requires further iterative calls to SR.probe, to potentially narrow down on a configuration that can be used. 
 sr            | sr_stat option         | Existing SR found for this configuration                                                                                                                                                                         
 extra_info    | (string * string) list | Additional plugin-specific information about this configuration, that might be of use for an API user. This can for example include the LUN or the WWPN.                                                         
### probe_results
```json
[
  {
    "extra_info": { "extra_info": "extra_info" },
    "sr": {
      "health": [ "Healthy", "health" ],
      "clustered": true,
      "datasources": [ "datasources" ],
      "total_space": 0,
      "free_space": 0,
      "description": "description",
      "uuid": "uuid",
      "name": "name",
      "sr": "sr"
    },
    "complete": true,
    "configuration": { "configuration": "configuration" }
  }
]
[]
```
type `probe_results` = `probe_result list`

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
----------------------|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 key                  | string                 | A primary key for this volume. The key must be unique within the enclosing Storage Repository \(SR\). A typical value would be a filename or an LVM volume name.                                                                                                                                                                                                   
 uuid                 | string option          | A uuid \(or guid\) for the volume, if one is available. If a storage system has a built-in notion of a guid, then it will be returned here.                                                                                                                                                                                                                        
 name                 | string                 | Short, human-readable label for the volume. Names are commonly used by when displaying short lists of volumes.                                                                                                                                                                                                                                                     
 description          | string                 | Longer, human-readable description of the volume. Descriptions are generally only displayed by clients when the user is examining volumes individually.                                                                                                                                                                                                            
 read_write           | bool                   | True means the VDI may be written to, false means the volume is read-only. Some storage media is read-only so all volumes are read-only; for example .iso disk images on an NFS share. Some volume are created read-only; for example because they are snapshots of some other VDI.                                                                                
 sharable             | bool                   | Indicates whether the VDI can be attached by multiple hosts at once. This is used for example by the HA statefile and XAPI redo log.                                                                                                                                                                                                                               
 virtual_size         | int64                  | Size of the volume from the perspective of a VM \(in bytes\)                                                                                                                                                                                                                                                                                                       
 physical_utilisation | int64                  | Amount of space currently used on the backing storage \(in bytes\)                                                                                                                                                                                                                                                                                                 
 uri                  | string list            | A list of URIs which can be opened and used for I/O. A URI could reference a local block device, a remote NFS share, iSCSI LUN or RBD volume. In cases where the data may be accessed over several protocols, he list should be sorted into descending order of desirability. Xapi will open the most desirable URI for which it has an available datapath plugin. 
 keys                 | (string * string) list | A list of key=value pairs which have been stored in the Volume metadata. These should not be interpreted by the Volume plugin.                                                                                                                                                                                                                                     
### volumes
```json
[
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
[]
```
type `volumes` = `volume list`
A list of volumes
### key
```json
"key"
```
type `key` = `string`
Primary key for a volume. This can be any string which is meaningful to the implementation. For example this could be an NFS filename, an LVM LV name or even a URI. This string is abstract.
### blocklist
```json
{ "ranges": [ [ 0, 0 ] ], "blocksize": 0 }
```
type `blocklist` = `struct { ... }`
List of blocks for copying
#### Members
 Name      | Type               | Description                                                                                        
-----------|--------------------|----------------------------------------------------------------------------------------------------
 blocksize | int                | size of the individual blocks                                                                      
 ranges    | int64 * int64 list | list of block ranges, where a range is a \(start,length\) pair, measured in units of \[blocksize\] 
### key_list
```json
[ "key_list" ]
[]
```
type `key_list` = `string list`

### changed_blocks
```json
{ "bitmap": "bitmap", "granularity": 0 }
```
type `changed_blocks` = `struct { ... }`

#### Members
 Name        | Type   | Description                                                                                                                                                                                                               
-------------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 granularity | int    | One bit in the changed block bitmap indicates the status of an area of this size, in bytes.                                                                                                                               
 bitmap      | string | The changed blocks between two volumes as a base64-encoded string. The bits in the bitmap indicate the status of consecutive blocks of size \[granularity\] bytes. Each bit is set if the corresponding area has changed. 
## Interface: `SR`
Operations which act on Storage Repositories
## Method: `probe`
\[probe configuration\]: can be used iteratively to narrow down configurations to use with SR.create, or to find existing SRs on the backing storage

> Client

```json
{
  "method": "SR.probe",
  "params": [
    {
      "configuration": { "field_1": "value_1", "field_2": "value_2" },
      "dbg": "dbg"
    }
  ],
  "id": 4
}
```

```ocaml
try
    let probe_result = Client.probe dbg configuration in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.SR.probe({ dbg: "string", configuration: {"string": "string"} })
    print (repr(results))
```

> Server

```json
[
  {
    "extra_info": { "field_1": "value_1", "field_2": "value_2" },
    "sr": {
      "health": [ "Healthy", "Healthy" ],
      "clustered": true,
      "datasources": [ "datasources_1", "datasources_2" ],
      "total_space": 0,
      "free_space": 0,
      "description": "description",
      "uuid": "optional_uuid",
      "name": "name",
      "sr": "sr"
    },
    "complete": true,
    "configuration": { "field_1": "value_1", "field_2": "value_2" }
  },
  {
    "extra_info": { "field_1": "value_1", "field_2": "value_2" },
    "sr": {
      "health": [ "Healthy", "Healthy" ],
      "clustered": true,
      "datasources": [ "datasources_1", "datasources_2" ],
      "total_space": 0,
      "free_space": 0,
      "description": "description",
      "uuid": "optional_uuid",
      "name": "name",
      "sr": "sr"
    },
    "complete": true,
    "configuration": { "field_1": "value_1", "field_2": "value_2" }
  }
]
```

```ocaml
try
    let probe_result = Client.probe dbg configuration in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class SR_myimplementation(SR_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def probe(self, dbg, configuration):
        """
        [probe configuration]: can be used iteratively to narrow down configurations
        to use with SR.create, or to find existing SRs on the backing storage
        """
        return [{"configuration": {"string": "string"}, "complete": True, "sr": None, "extra_info": {"string": "string"}}]
    # ...
```


 Name          | Direction | Type          | Description                                                                                                                                                                                         
---------------|-----------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 dbg           | in        | string        | Debug context from the caller                                                                                                                                                                       
 configuration | in        | configuration | Plugin-specific configuration which describes where and how to locate the storage repository. This may include the physical block device name, a remote NFS server and path or an RBD storage pool. 
 probe_result  | out       | probe_results | Contents of the storage device                                                                                                                                                                      
## Method: `create`
\[create uuid configuration name description\]: creates a fresh SR

> Client

```json
{
  "method": "SR.create",
  "params": [
    {
      "description": "description",
      "name": "name",
      "configuration": { "field_1": "value_1", "field_2": "value_2" },
      "uuid": "uuid",
      "dbg": "dbg"
    }
  ],
  "id": 5
}
```

```ocaml
try
    let configuration = Client.create dbg uuid configuration name description in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.SR.create({ dbg: "string", uuid: "string", configuration: {"string": "string"}, name: "string", description: "string" })
    print (repr(results))
```

> Server

```json
{ "field_1": "value_1", "field_2": "value_2" }
```

```ocaml
try
    let configuration = Client.create dbg uuid configuration name description in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class SR_myimplementation(SR_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def create(self, dbg, uuid, configuration, name, description):
        """
        [create uuid configuration name description]: creates a fresh SR
        """
        return {"string": "string"}
    # ...
```


 Name          | Direction | Type          | Description                                                                                                                                                                                         
---------------|-----------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 dbg           | in        | string        | Debug context from the caller                                                                                                                                                                       
 uuid          | in        | string        | A uuid to associate with the SR.                                                                                                                                                                    
 configuration | in        | configuration | Plugin-specific configuration which describes where and how to locate the storage repository. This may include the physical block device name, a remote NFS server and path or an RBD storage pool. 
 name          | in        | string        | Human-readable name for the SR                                                                                                                                                                      
 description   | in        | string        | Human-readable description for the SR                                                                                                                                                               
 configuration | out       | configuration | Plugin-specific configuration which describes where and how to locate the storage repository. This may include the physical block device name, a remote NFS server and path or an RBD storage pool. 
## Method: `attach`
\[attach configuration\]: attaches the SR to the local host. Once an SR is  attached then volumes may be manipulated.

> Client

```json
{
  "method": "SR.attach",
  "params": [
    {
      "configuration": { "field_1": "value_1", "field_2": "value_2" },
      "dbg": "dbg"
    }
  ],
  "id": 6
}
```

```ocaml
try
    let sr = Client.attach dbg configuration in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.SR.attach({ dbg: "string", configuration: {"string": "string"} })
    print (repr(results))
```

> Server

```json
"sr"
```

```ocaml
try
    let sr = Client.attach dbg configuration in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class SR_myimplementation(SR_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def attach(self, dbg, configuration):
        """
        [attach configuration]: attaches the SR to the local host. Once an SR is
        attached then volumes may be manipulated.
        """
        return "string"
    # ...
```


 Name          | Direction | Type          | Description                                                                                                                                                                                         
---------------|-----------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 dbg           | in        | string        | Debug context from the caller                                                                                                                                                                       
 configuration | in        | configuration | Plugin-specific configuration which describes where and how to locate the storage repository. This may include the physical block device name, a remote NFS server and path or an RBD storage pool. 
 sr            | out       | string        | The Storage Repository                                                                                                                                                                              
## Method: `detach`
\[detach sr\]: detaches the SR, clearing up any associated resources.  Once the SR is detached then volumes may not be manipulated.

> Client

```json
{
  "method": "SR.detach",
  "params": [ { "sr": "sr", "dbg": "dbg" } ],
  "id": 7
}
```

```ocaml
try
    let () = Client.detach dbg sr in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.SR.detach({ dbg: "string", sr: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.detach dbg sr in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class SR_myimplementation(SR_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def detach(self, dbg, sr):
        """
        [detach sr]: detaches the SR, clearing up any associated resources.
        Once the SR is detached then volumes may not be manipulated.
        """
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 sr   | in        | string | The Storage Repository        
## Method: `destroy`
\[destroy sr\]: destroys the \[sr\] and deletes any volumes associated  with it. Note that an SR must be attached to be destroyed; otherwise  Sr\_not\_attached is thrown.

> Client

```json
{
  "method": "SR.destroy",
  "params": [ { "sr": "sr", "dbg": "dbg" } ],
  "id": 8
}
```

```ocaml
try
    let () = Client.destroy dbg sr in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.SR.destroy({ dbg: "string", sr: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.destroy dbg sr in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class SR_myimplementation(SR_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def destroy(self, dbg, sr):
        """
        [destroy sr]: destroys the [sr] and deletes any volumes associated
        with it. Note that an SR must be attached to be destroyed; otherwise
        Sr_not_attached is thrown.
        """
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 sr   | in        | string | The Storage Repository        
## Method: `stat`
\[stat sr\] returns summary metadata associated with \[sr\]. Note this  call does not return details of sub-volumes, see SR.ls.

> Client

```json
{ "method": "SR.stat", "params": [ { "sr": "sr", "dbg": "dbg" } ], "id": 9 }
```

```ocaml
try
    let sr = Client.stat dbg sr in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.SR.stat({ dbg: "string", sr: "string" })
    print (repr(results))
```

> Server

```json
{
  "health": [ "Healthy", "Healthy" ],
  "clustered": true,
  "datasources": [ "datasources_1", "datasources_2" ],
  "total_space": 0,
  "free_space": 0,
  "description": "description",
  "uuid": "optional_uuid",
  "name": "name",
  "sr": "sr"
}
```

```ocaml
try
    let sr = Client.stat dbg sr in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class SR_myimplementation(SR_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def stat(self, dbg, sr):
        """
        [stat sr] returns summary metadata associated with [sr]. Note this
        call does not return details of sub-volumes, see SR.ls.
        """
        return {"sr": "string", "name": "string", "uuid": None, "description": "string", "free_space": 0L, "total_space": 0L, "datasources": ["string"], "clustered": True, "health": None}
    # ...
```


 Name | Direction | Type    | Description                   
------|-----------|---------|-------------------------------
 dbg  | in        | string  | Debug context from the caller 
 sr   | in        | string  | The Storage Repository        
 sr   | out       | sr_stat | SR metadata                   
## Method: `set_name`
\[set\_name sr new\_name\] changes the name of \[sr\]

> Client

```json
{
  "method": "SR.set_name",
  "params": [ { "new_name": "new_name", "sr": "sr", "dbg": "dbg" } ],
  "id": 10
}
```

```ocaml
try
    let () = Client.set_name dbg sr new_name in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.SR.set_name({ dbg: "string", sr: "string", new_name: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.set_name dbg sr new_name in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class SR_myimplementation(SR_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def set_name(self, dbg, sr, new_name):
        """
        [set_name sr new_name] changes the name of [sr]
        """
    # ...
```


 Name     | Direction | Type   | Description                   
----------|-----------|--------|-------------------------------
 dbg      | in        | string | Debug context from the caller 
 sr       | in        | string | The Storage Repository        
 new_name | in        | string | The new name of the SR        
## Method: `set_description`
\[set\_description sr new\_description\] changes the description of \[sr\]

> Client

```json
{
  "method": "SR.set_description",
  "params": [
    { "new_description": "new_description", "sr": "sr", "dbg": "dbg" }
  ],
  "id": 11
}
```

```ocaml
try
    let () = Client.set_description dbg sr new_description in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.SR.set_description({ dbg: "string", sr: "string", new_description: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.set_description dbg sr new_description in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class SR_myimplementation(SR_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def set_description(self, dbg, sr, new_description):
        """
        [set_description sr new_description] changes the description of [sr]
        """
    # ...
```


 Name            | Direction | Type   | Description                    
-----------------|-----------|--------|--------------------------------
 dbg             | in        | string | Debug context from the caller  
 sr              | in        | string | The Storage Repository         
 new_description | in        | string | The new description for the SR 
## Method: `ls`
\[ls sr\] returns a list of volumes contained within an attached SR.

> Client

```json
{ "method": "SR.ls", "params": [ { "sr": "sr", "dbg": "dbg" } ], "id": 12 }
```

```ocaml
try
    let volumes = Client.ls dbg sr in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.SR.ls({ dbg: "string", sr: "string" })
    print (repr(results))
```

> Server

```json
[
  {
    "keys": { "field_1": "value_1", "field_2": "value_2" },
    "uri": [ "uri_1", "uri_2" ],
    "physical_utilisation": 0,
    "virtual_size": 0,
    "sharable": true,
    "read_write": true,
    "description": "description",
    "name": "name",
    "uuid": "optional_uuid",
    "key": "key"
  },
  {
    "keys": { "field_1": "value_1", "field_2": "value_2" },
    "uri": [ "uri_1", "uri_2" ],
    "physical_utilisation": 0,
    "virtual_size": 0,
    "sharable": true,
    "read_write": true,
    "description": "description",
    "name": "name",
    "uuid": "optional_uuid",
    "key": "key"
  }
]
```

```ocaml
try
    let volumes = Client.ls dbg sr in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class SR_myimplementation(SR_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def ls(self, dbg, sr):
        """
        [ls sr] returns a list of volumes contained within an attached SR.
        """
        return [{"key": "string", "uuid": None, "name": "string", "description": "string", "read_write": True, "sharable": True, "virtual_size": 0L, "physical_utilisation": 0L, "uri": ["string"], "keys": {"string": "string"}}]
    # ...
```


 Name    | Direction | Type    | Description                   
---------|-----------|---------|-------------------------------
 dbg     | in        | string  | Debug context from the caller 
 sr      | in        | string  | The Storage Repository        
 volumes | out       | volumes | A list of volumes             
## Interface: `Volume`
Operations which operate on volumes \(also known as  Virtual Disk Images\)
## Method: `create`
\[create sr name description size\] creates a new volume in \[sr\] with  \[name\] and \[description\]. The volume will have size &gt;= \[size\] i.e. it  is always permissable for an implementation to round-up the volume to  the nearest convenient block size

> Client

```json
{
  "method": "Volume.create",
  "params": [
    {
      "sharable": true,
      "size": 0,
      "description": "description",
      "name": "name",
      "sr": "sr",
      "dbg": "dbg"
    }
  ],
  "id": 13
}
```

```ocaml
try
    let volume = Client.create dbg sr name description size sharable in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.create({ dbg: "string", sr: "string", name: "string", description: "string", size: 0L, sharable: True })
    print (repr(results))
```

> Server

```json
{
  "keys": { "field_1": "value_1", "field_2": "value_2" },
  "uri": [ "uri_1", "uri_2" ],
  "physical_utilisation": 0,
  "virtual_size": 0,
  "sharable": true,
  "read_write": true,
  "description": "description",
  "name": "name",
  "uuid": "optional_uuid",
  "key": "key"
}
```

```ocaml
try
    let volume = Client.create dbg sr name description size sharable in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def create(self, dbg, sr, name, description, size, sharable):
        """
        [create sr name description size] creates a new volume in [sr] with
        [name] and [description]. The volume will have size >= [size] i.e. it
        is always permissable for an implementation to round-up the volume to
        the nearest convenient block size
        """
        return {"key": "string", "uuid": None, "name": "string", "description": "string", "read_write": True, "sharable": True, "virtual_size": 0L, "physical_utilisation": 0L, "uri": ["string"], "keys": {"string": "string"}}
    # ...
```


 Name        | Direction | Type   | Description                                                                                                                                                                                                                           
-------------|-----------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 dbg         | in        | string | Debug context from the caller                                                                                                                                                                                                         
 sr          | in        | string | The Storage Repository                                                                                                                                                                                                                
 name        | in        | string | A human-readable name to associate with the new disk. This name is  intended to be short, to be a good summary of the disk.                                                                                                           
 description | in        | string | A human-readable description to associate with the new disk. This can  be arbitrarily long, up to the general string size limit.                                                                                                      
 size        | in        | int64  | A minimum size \(in bytes\) for the disk. Depending on the  characteristics of the implementation this may be rounded up to  \(for example\) the nearest convenient block size. The created disk  will not be smaller than this size. 
 sharable    | in        | bool   | Indicates whether the VDI can be attached by multiple hosts at once. This is used for example by the HA statefile and XAPI redo log.                                                                                                  
 volume      | out       | volume | Properties of the volume                                                                                                                                                                                                              
## Method: `snapshot`
\[snapshot sr volume\] creates a new volue which is a  snapshot of  \[volume\] in \[sr\]. Snapshots should never be written to; they are  intended for backup/restore only. Note the name and description are  copied but any extra metadata associated by \[set\] is not copied. This can raise Activated\_on\_another\_host\(host\_installation\_uuid\) if the VDI is already active on another host and snapshots can only be taken on the host that has the VDI active \(if any\). XAPI will take care of redirecting the request to the proper host

> Client

```json
{
  "method": "Volume.snapshot",
  "params": [ { "key": "key", "sr": "sr", "dbg": "dbg" } ],
  "id": 14
}
```

```ocaml
try
    let volume = Client.snapshot dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.snapshot({ dbg: "string", sr: "string", key: "string" })
    print (repr(results))
```

> Server

```json
{
  "keys": { "field_1": "value_1", "field_2": "value_2" },
  "uri": [ "uri_1", "uri_2" ],
  "physical_utilisation": 0,
  "virtual_size": 0,
  "sharable": true,
  "read_write": true,
  "description": "description",
  "name": "name",
  "uuid": "optional_uuid",
  "key": "key"
}
```

```ocaml
try
    let volume = Client.snapshot dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def snapshot(self, dbg, sr, key):
        """
        [snapshot sr volume] creates a new volue which is a  snapshot of
        [volume] in [sr]. Snapshots should never be written to; they are
        intended for backup/restore only. Note the name and description are
        copied but any extra metadata associated by [set] is not copied.
        This can raise Activated_on_another_host(host_installation_uuid)
        if the VDI is already active on another host and snapshots
        can only be taken on the host that has the VDI active (if any).
        XAPI will take care of redirecting the request to the proper host
        """
        return {"key": "string", "uuid": None, "name": "string", "description": "string", "read_write": True, "sharable": True, "virtual_size": 0L, "physical_utilisation": 0L, "uri": ["string"], "keys": {"string": "string"}}
    # ...
```


 Name   | Direction | Type   | Description                   
--------|-----------|--------|-------------------------------
 dbg    | in        | string | Debug context from the caller 
 sr     | in        | string | The Storage Repository        
 key    | in        | key    | The volume key                
 volume | out       | volume | Properties of the volume      
## Method: `clone`
\[clone sr volume\] creates a new volume which is a writable clone of  \[volume\] in \[sr\]. Note the name and description are copied but any  extra metadata associated by \[set\] is not copied.

> Client

```json
{
  "method": "Volume.clone",
  "params": [ { "key": "key", "sr": "sr", "dbg": "dbg" } ],
  "id": 15
}
```

```ocaml
try
    let volume = Client.clone dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.clone({ dbg: "string", sr: "string", key: "string" })
    print (repr(results))
```

> Server

```json
{
  "keys": { "field_1": "value_1", "field_2": "value_2" },
  "uri": [ "uri_1", "uri_2" ],
  "physical_utilisation": 0,
  "virtual_size": 0,
  "sharable": true,
  "read_write": true,
  "description": "description",
  "name": "name",
  "uuid": "optional_uuid",
  "key": "key"
}
```

```ocaml
try
    let volume = Client.clone dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def clone(self, dbg, sr, key):
        """
        [clone sr volume] creates a new volume which is a writable clone of
        [volume] in [sr]. Note the name and description are copied but any
        extra metadata associated by [set] is not copied.
        """
        return {"key": "string", "uuid": None, "name": "string", "description": "string", "read_write": True, "sharable": True, "virtual_size": 0L, "physical_utilisation": 0L, "uri": ["string"], "keys": {"string": "string"}}
    # ...
```


 Name   | Direction | Type   | Description                   
--------|-----------|--------|-------------------------------
 dbg    | in        | string | Debug context from the caller 
 sr     | in        | string | The Storage Repository        
 key    | in        | key    | The volume key                
 volume | out       | volume | Properties of the volume      
## Method: `destroy`
\[destroy sr volume\] removes \[volume\] from \[sr\]

> Client

```json
{
  "method": "Volume.destroy",
  "params": [ { "key": "key", "sr": "sr", "dbg": "dbg" } ],
  "id": 16
}
```

```ocaml
try
    let () = Client.destroy dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.destroy({ dbg: "string", sr: "string", key: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.destroy dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def destroy(self, dbg, sr, key):
        """
        [destroy sr volume] removes [volume] from [sr]
        """
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 sr   | in        | string | The Storage Repository        
 key  | in        | key    | The volume key                
## Method: `set_name`
\[set\_name sr volume new\_name\] changes the name of \[volume\]

> Client

```json
{
  "method": "Volume.set_name",
  "params": [
    { "new_name": "new_name", "key": "key", "sr": "sr", "dbg": "dbg" }
  ],
  "id": 17
}
```

```ocaml
try
    let () = Client.set_name dbg sr key new_name in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.set_name({ dbg: "string", sr: "string", key: "string", new_name: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.set_name dbg sr key new_name in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def set_name(self, dbg, sr, key, new_name):
        """
        [set_name sr volume new_name] changes the name of [volume]
        """
    # ...
```


 Name     | Direction | Type   | Description                   
----------|-----------|--------|-------------------------------
 dbg      | in        | string | Debug context from the caller 
 sr       | in        | string | The Storage Repository        
 key      | in        | key    | The volume key                
 new_name | in        | string | New name                      
## Method: `set_description`
\[set\_description sr volume new\_description\] changes the description  of \[volume\]

> Client

```json
{
  "method": "Volume.set_description",
  "params": [
    {
      "new_description": "new_description",
      "key": "key",
      "sr": "sr",
      "dbg": "dbg"
    }
  ],
  "id": 18
}
```

```ocaml
try
    let () = Client.set_description dbg sr key new_description in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.set_description({ dbg: "string", sr: "string", key: "string", new_description: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.set_description dbg sr key new_description in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def set_description(self, dbg, sr, key, new_description):
        """
        [set_description sr volume new_description] changes the description
        of [volume]
        """
    # ...
```


 Name            | Direction | Type   | Description                   
-----------------|-----------|--------|-------------------------------
 dbg             | in        | string | Debug context from the caller 
 sr              | in        | string | The Storage Repository        
 key             | in        | key    | The volume key                
 new_description | in        | string | New description               
## Method: `set`
\[set sr volume key value\] associates \[key\] with \[value\] in the  metadata of \[volume\] Note these keys and values are not interpreted  by the plugin; they are intended for the higher-level software only.

> Client

```json
{
  "method": "Volume.set",
  "params": [
    { "v": "v", "k": "k", "key": "key", "sr": "sr", "dbg": "dbg" }
  ],
  "id": 19
}
```

```ocaml
try
    let () = Client.set dbg sr key k v in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.set({ dbg: "string", sr: "string", key: "string", k: "string", v: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.set dbg sr key k v in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def set(self, dbg, sr, key, k, v):
        """
        [set sr volume key value] associates [key] with [value] in the
        metadata of [volume] Note these keys and values are not interpreted
        by the plugin; they are intended for the higher-level software only.
        """
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 sr   | in        | string | The Storage Repository        
 key  | in        | key    | The volume key                
 k    | in        | string | Key                           
 v    | in        | string | Value                         
## Method: `unset`
\[unset sr volume key\] removes \[key\] and any value associated with it  from the metadata of \[volume\] Note these keys and values are not  interpreted by the plugin; they are intended for the higher-level  software only.

> Client

```json
{
  "method": "Volume.unset",
  "params": [ { "k": "k", "key": "key", "sr": "sr", "dbg": "dbg" } ],
  "id": 20
}
```

```ocaml
try
    let () = Client.unset dbg sr key k in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.unset({ dbg: "string", sr: "string", key: "string", k: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.unset dbg sr key k in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def unset(self, dbg, sr, key, k):
        """
        [unset sr volume key] removes [key] and any value associated with it
        from the metadata of [volume] Note these keys and values are not
        interpreted by the plugin; they are intended for the higher-level
        software only.
        """
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 sr   | in        | string | The Storage Repository        
 key  | in        | key    | The volume key                
 k    | in        | string | Key                           
## Method: `resize`
\[resize sr volume new\_size\] enlarges \[volume\] to be at least  \[new\_size\].

> Client

```json
{
  "method": "Volume.resize",
  "params": [ { "new_size": 0, "key": "key", "sr": "sr", "dbg": "dbg" } ],
  "id": 21
}
```

```ocaml
try
    let () = Client.resize dbg sr key new_size in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.resize({ dbg: "string", sr: "string", key: "string", new_size: 0L })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.resize dbg sr key new_size in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def resize(self, dbg, sr, key, new_size):
        """
        [resize sr volume new_size] enlarges [volume] to be at least
        [new_size].
        """
    # ...
```


 Name     | Direction | Type   | Description                   
----------|-----------|--------|-------------------------------
 dbg      | in        | string | Debug context from the caller 
 sr       | in        | string | The Storage Repository        
 key      | in        | key    | The volume key                
 new_size | in        | int64  | New disk size                 
## Method: `stat`
\[stat sr volume\] returns metadata associated with \[volume\].

> Client

```json
{
  "method": "Volume.stat",
  "params": [ { "key": "key", "sr": "sr", "dbg": "dbg" } ],
  "id": 22
}
```

```ocaml
try
    let volume = Client.stat dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.stat({ dbg: "string", sr: "string", key: "string" })
    print (repr(results))
```

> Server

```json
{
  "keys": { "field_1": "value_1", "field_2": "value_2" },
  "uri": [ "uri_1", "uri_2" ],
  "physical_utilisation": 0,
  "virtual_size": 0,
  "sharable": true,
  "read_write": true,
  "description": "description",
  "name": "name",
  "uuid": "optional_uuid",
  "key": "key"
}
```

```ocaml
try
    let volume = Client.stat dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def stat(self, dbg, sr, key):
        """
        [stat sr volume] returns metadata associated with [volume].
        """
        return {"key": "string", "uuid": None, "name": "string", "description": "string", "read_write": True, "sharable": True, "virtual_size": 0L, "physical_utilisation": 0L, "uri": ["string"], "keys": {"string": "string"}}
    # ...
```


 Name   | Direction | Type   | Description                   
--------|-----------|--------|-------------------------------
 dbg    | in        | string | Debug context from the caller 
 sr     | in        | string | The Storage Repository        
 key    | in        | key    | The volume key                
 volume | out       | volume | Properties of the volume      
## Method: `compare`
\[compare sr volume1 volume2\] compares the two volumes and returns a  result of type blocklist that describes the differences between the  two volumes. If the two volumes are unrelated, or the second volume  does not exist, the result will be a list of the blocks that are  non-empty in volume1. If this information is not available to the  plugin, it should return a result indicating that all blocks are in  use.

> Client

```json
{
  "method": "Volume.compare",
  "params": [ { "key2": "key2", "key": "key", "sr": "sr", "dbg": "dbg" } ],
  "id": 23
}
```

```ocaml
try
    let blocklist = Client.compare dbg sr key key2 in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.compare({ dbg: "string", sr: "string", key: "string", key2: "string" })
    print (repr(results))
```

> Server

```json
{ "ranges": [ [ 0, 0 ], [ 0, 0 ] ], "blocksize": 0 }
```

```ocaml
try
    let blocklist = Client.compare dbg sr key key2 in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def compare(self, dbg, sr, key, key2):
        """
        [compare sr volume1 volume2] compares the two volumes and returns a
        result of type blocklist that describes the differences between the
        two volumes. If the two volumes are unrelated, or the second volume
        does not exist, the result will be a list of the blocks that are
        non-empty in volume1. If this information is not available to the
        plugin, it should return a result indicating that all blocks are in
        use.
        """
        return {"blocksize": 0L, "ranges": [[]]}
    # ...
```


 Name    | Direction | Type      | Description                   
---------|-----------|-----------|-------------------------------
 dbg     | in        | string    | Debug context from the caller 
 sr      | in        | string    | The Storage Repository        
 key     | in        | key       | The volume key                
 key2    | in        | key       | The volume key                
 unnamed | out       | blocklist | List of blocks for copying    
## Method: `similar_content`
\[similar\_content sr volume\] returns a list of VDIs which have similar content to \[vdi\]

> Client

```json
{
  "method": "Volume.similar_content",
  "params": [ { "key": "key", "sr": "sr", "dbg": "dbg" } ],
  "id": 24
}
```

```ocaml
try
    let key list = Client.similar_content dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.similar_content({ dbg: "string", sr: "string", key: "string" })
    print (repr(results))
```

> Server

```json
[ "key list_1", "key list_2" ]
```

```ocaml
try
    let key list = Client.similar_content dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def similar_content(self, dbg, sr, key):
        """
        [similar_content sr volume] returns a list of VDIs which have similar
        content to [vdi]
        """
        return ["string"]
    # ...
```


 Name     | Direction | Type     | Description                   
----------|-----------|----------|-------------------------------
 dbg      | in        | string   | Debug context from the caller 
 sr       | in        | string   | The Storage Repository        
 key      | in        | key      | The volume key                
 key list | out       | key_list | List of volume keys           
## Method: `enable_cbt`
\[enable\_cbt sr volume\] enables Changed Block Tracking for \[volume\]

> Client

```json
{
  "method": "Volume.enable_cbt",
  "params": [ { "key": "key", "sr": "sr", "dbg": "dbg" } ],
  "id": 25
}
```

```ocaml
try
    let () = Client.enable_cbt dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.enable_cbt({ dbg: "string", sr: "string", key: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.enable_cbt dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def enable_cbt(self, dbg, sr, key):
        """
        [enable_cbt sr volume] enables Changed Block Tracking for [volume]
        """
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 sr   | in        | string | The Storage Repository        
 key  | in        | key    | The volume key                
## Method: `disable_cbt`
\[disable\_cbt sr volume\] disables Changed Block Tracking for \[volume\]

> Client

```json
{
  "method": "Volume.disable_cbt",
  "params": [ { "key": "key", "sr": "sr", "dbg": "dbg" } ],
  "id": 26
}
```

```ocaml
try
    let () = Client.disable_cbt dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.disable_cbt({ dbg: "string", sr: "string", key: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.disable_cbt dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def disable_cbt(self, dbg, sr, key):
        """
        [disable_cbt sr volume] disables Changed Block Tracking for [volume]
        """
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 sr   | in        | string | The Storage Repository        
 key  | in        | key    | The volume key                
## Method: `data_destroy`
\[data\_destroy sr volume\] deletes the data of the snapshot \[volume\] without deleting its changed block tracking metadata

> Client

```json
{
  "method": "Volume.data_destroy",
  "params": [ { "key": "key", "sr": "sr", "dbg": "dbg" } ],
  "id": 27
}
```

```ocaml
try
    let () = Client.data_destroy dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.data_destroy({ dbg: "string", sr: "string", key: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.data_destroy dbg sr key in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def data_destroy(self, dbg, sr, key):
        """
        [data_destroy sr volume] deletes the data of the snapshot [volume]
        without deleting its changed block tracking metadata
        """
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 sr   | in        | string | The Storage Repository        
 key  | in        | key    | The volume key                
## Method: `list_changed_blocks`
\[list\_changed\_blocks sr volume1 volume2\] returns the blocks that have changed between \[volume1\] and \[volume2\] as a base64-encoded bitmap string

> Client

```json
{
  "method": "Volume.list_changed_blocks",
  "params": [
    {
      "length": 0,
      "offset": 0,
      "key2": "key2",
      "key": "key",
      "sr": "sr",
      "dbg": "dbg"
    }
  ],
  "id": 28
}
```

```ocaml
try
    let changed_blocks = Client.list_changed_blocks dbg sr key key2 offset length in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Volume.list_changed_blocks({ dbg: "string", sr: "string", key: "string", key2: "string", offset: 0L, length: 0L })
    print (repr(results))
```

> Server

```json
{ "bitmap": "bitmap", "granularity": 0 }
```

```ocaml
try
    let changed_blocks = Client.list_changed_blocks dbg sr key key2 offset length in
    ...
with Exn (Sr_not_attached str) -> ...
| Exn (SR_does_not_exist str) -> ...
| Exn (Volume_does_not_exist str) -> ...
| Exn (Unimplemented str) -> ...
| Exn (Cancelled str) -> ...
| Exn (Activated_on_another_host str) -> ...

```

```python

# import additional libraries if needed

class Volume_myimplementation(Volume_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def list_changed_blocks(self, dbg, sr, key, key2, offset, length):
        """
        [list_changed_blocks sr volume1 volume2] returns the blocks that
        have changed between [volume1] and [volume2] as a base64-encoded
        bitmap string
        """
        return {"granularity": 0L, "bitmap": "string"}
    # ...
```


 Name           | Direction | Type           | Description                                                          
----------------|-----------|----------------|----------------------------------------------------------------------
 dbg            | in        | string         | Debug context from the caller                                        
 sr             | in        | string         | The Storage Repository                                               
 key            | in        | key            | The volume key                                                       
 key2           | in        | key            | The volume key                                                       
 offset         | in        | int64          | The offset of the extent for which changed blocks should be computed 
 length         | in        | int            | The length of the extent for which changed blocks should be computed 
 changed_blocks | out       | changed_blocks | The changed blocks between two volumes in the specified extent       
## Errors
### exns
```json
[ "Sr_not_attached", "exns" ]
[ "SR_does_not_exist", "exns" ]
[ "Volume_does_not_exist", "exns" ]
[ "Unimplemented", "exns" ]
[ "Cancelled", "exns" ]
[ "Activated_on_another_host", "exns" ]
```
type `exns` = `variant { ... }`

#### Constructors
 Name                      | Type   | Description                                       
---------------------------|--------|---------------------------------------------------
 Sr_not_attached           | string | An SR must be attached in order to access volumes 
 SR_does_not_exist         | string | The specified SR could not be found               
 Volume_does_not_exist     | string | The specified volume could not be found in the SR 
 Unimplemented             | string | The operation has not been implemented            
 Cancelled                 | string | The operation has been cancelled                  
 Activated_on_another_host | string | The Volume is already active on another host      
### exns
```json
[ "Sr_not_attached", "exns" ]
[ "SR_does_not_exist", "exns" ]
[ "Volume_does_not_exist", "exns" ]
[ "Unimplemented", "exns" ]
[ "Cancelled", "exns" ]
[ "Activated_on_another_host", "exns" ]
```
type `exns` = `variant { ... }`

#### Constructors
 Name                      | Type   | Description                                       
---------------------------|--------|---------------------------------------------------
 Sr_not_attached           | string | An SR must be attached in order to access volumes 
 SR_does_not_exist         | string | The specified SR could not be found               
 Volume_does_not_exist     | string | The specified volume could not be found in the SR 
 Unimplemented             | string | The operation has not been implemented            
 Cancelled                 | string | The operation has been cancelled                  
 Activated_on_another_host | string | The Volume is already active on another host      