---
title: datapath

language_tabs:
 - json
 - ocaml
 - python

search: true
---
# datapath

The Datapath interfaces are provided to access the data stored
in the volumes. The `Datapath` interface is used to open and close the disks for read/write
operations from VMs and the `Data` interface is used for operations such as `copy` and `mirror`
    
## Type definitions
### persistent
```json
true
false
```
type `persistent` = `bool`
True means the disk data is persistent and should be preserved when the datapath is closed i.e. when a VM is shutdown or rebooted. False means the data should be thrown away when the VM is shutdown or rebooted.
### xendisk
```json
{
  "backend_type": "backend_type",
  "extra": { "extra": "extra" },
  "params": "params"
}
```
type `xendisk` = `struct { ... }`

#### Members
 Name         | Type                   | Description                                                                               
--------------|------------------------|-------------------------------------------------------------------------------------------
 params       | string                 | Put into the "params" key in xenstore                                                     
 extra        | (string * string) list | Key-value pairs to be put into the "sm-data" subdirectory underneath the xenstore backend 
 backend_type | string                 |                                                                                           
### block_device
```json
{ "path": "path" }
```
type `block_device` = `struct { ... }`

#### Members
 Name | Type   | Description                                                                      
------|--------|----------------------------------------------------------------------------------
 path | string | Path to the system local block device. This is equivalent to the SMAPIv1 params. 
### file
```json
{ "path": "path" }
```
type `file` = `struct { ... }`

#### Members
 Name | Type   | Description          
------|--------|----------------------
 path | string | Path to the raw file 
### nbd
```json
{ "uri": "uri" }
```
type `nbd` = `struct { ... }`

#### Members
 Name | Type   | Description                                                                                                                                                                                          
------|--------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 uri  | string | NBD URI of the form nbd:unix:&lt;domain-socket&gt;:exportname=&lt;NAME&gt; \(this format is used by qemu-system: https://manpages.debian.org/stretch/qemu-system-x86/qemu-system-x86\_64.1.en.html\) 
### implementation
```json
[
  "XenDisk",
  {
    "backend_type": "backend_type",
    "extra": { "extra": "extra" },
    "params": "params"
  }
]
[ "BlockDevice", { "path": "path" } ]
[ "File", { "path": "path" } ]
[ "Nbd", { "uri": "uri" } ]
```
type `implementation` = `variant { ... }`

#### Constructors
 Name        | Type         | Description                                             
-------------|--------------|---------------------------------------------------------
 XenDisk     | xendisk      | This value can be used for ring connection.             
 BlockDevice | block_device | This value can be used for Domain0 block device access. 
 File        | file         |                                                         
 Nbd         | nbd          |                                                         
### backend
```json
{
  "implementations": [
    [
      "XenDisk",
      {
        "backend_type": "backend_type",
        "extra": { "extra": "extra" },
        "params": "params"
      }
    ]
  ],
  "domain_uuid": "domain_uuid"
}
```
type `backend` = `struct { ... }`
A description of which Xen block backend to use. The toolstack needs this to setup the shared memory connection to blkfront in the VM.
#### Members
 Name            | Type                | Description                            
-----------------|---------------------|----------------------------------------
 domain_uuid     | string              | UUID of the domain hosting the backend 
 implementations | implementation list | choice of implementation technologies  
### uri
```json
"uri"
```
type `uri` = `string`
A URI representing the means for accessing the volume data. The interpretation  of the URI is specific to the implementation. Xapi will choose which  implementation to use based on the URI scheme.
### domain
```json
"domain"
```
type `domain` = `string`
A string representing a Xen domain on the local host. The string is guaranteed to be unique per-domain but it is not guaranteed to take any particular form. It may \(for example\) be a Xen domain id, a Xen VM uuid or a Xenstore path or anything else chosen by the toolstack. Implementations should not assume the string has any meaning.
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
### operation
```json
[ "Copy", [ "operation", "operation" ] ]
[ "Mirror", [ "operation", "operation" ] ]
```
type `operation` = `variant { ... }`
The primary key for referring to a long-running operation
#### Constructors
 Name   | Type            | Description                                                                                         
--------|-----------------|-----------------------------------------------------------------------------------------------------
 Copy   | string * string | Copy \(src,dst\) represents an on-going copy operation from the \[src\] URI to the \[dst\] URI      
 Mirror | string * string | Mirror \(src,dst\) represents an on-going mirror  operation from the \[src\] URI to the \[dst\] URI 
### status
```json
{ "progress": 0.0, "failed": true }
```
type `status` = `struct { ... }`
Status information for on-going tasks
#### Members
 Name     | Type         | Description                                                                     
----------|--------------|---------------------------------------------------------------------------------
 failed   | bool         | \[failed\] will be set to true if the operation has failed for some  reason     
 progress | float option | \[progress\] will be returned for a copy operation, and ranges  between 0 and 1 
### operations
```json
[ [ "Copy", [ "operations", "operations" ] ] ]
[]
```
type `operations` = `operation list`
A list of operations
## Interface: `Datapath`

Xapi will call the functions here on VM start / shutdown /
suspend / resume / migrate. Every function is idempotent. Every
function takes a domain parameter which allows the implementation
to track how many domains are currently using the volume. 

Volumes must be attached via the following sequence of calls:

1. \[open url persistent\] must be called first and is used to declare
   that the writes to the disks must either be persisted or not.
   \[open\] is not an exclusive operation - a disk may be opened on
   more than once host at once. The call returns `unit` or an
   error.

2. \[attach url domain\] is then called. The `domain` parameter is
   advisory. Note that this call is currently only ever called once.
   In the future the call may be made multiple times with different
   \[domain\] parameters if the disk is attached to multiple domains.
   The return value from this call is the information required to 
   attach the disk to a VM. This call is again, not exclusive. The
   volume may be attached to more than one host concurrently.

3. \[activate url domain\] is called to activate the datapath. This
   must be called before the volume is to be used by the VM, and 
   it is acceptible for this to be an exclusive operation, such that
   it is an error for a volume to be activated on more than one host
   simultaneously.
      
## Method: `open`
\[open uri persistent\] is called before a disk is attached to a VM. If persistent is true then care should be taken to persist all writes to the disk. If persistent is false then the implementation should configure a temporary location for writes so they can be thrown away on \[close\].

> Client

```json
{
  "method": "Datapath.open",
  "params": [ { "persistent": true, "uri": "uri", "dbg": "dbg" } ],
  "id": 29
}
```

```ocaml
try
    let () = Client.open dbg uri persistent in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Datapath.open({ dbg: "string", uri: "string", persistent: True })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.open dbg uri persistent in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Datapath_myimplementation(Datapath_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def open(self, dbg, uri, persistent):
        """
        [open uri persistent] is called before a disk is attached to a VM.
        If persistent is true then care should be taken to persist all writes
        to the disk. If persistent is false then the implementation should
        configure a temporary location for writes so they can be thrown away
        on [close].
        """
    # ...
```


 Name       | Direction | Type       | Description                                                                                                                                                                                                        
------------|-----------|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 dbg        | in        | string     | Debug context from the caller                                                                                                                                                                                      
 uri        | in        | uri        | A URI which represents how to access the volume disk data.                                                                                                                                                         
 persistent | in        | persistent | True means the disk data is persistent and should be preserved when the datapath is closed i.e. when a VM is shutdown or rebooted. False means the data should be thrown away when the VM is shutdown or rebooted. 
## Method: `attach`
\[attach uri domain\] prepares a connection between the storage named by \[uri\] and the Xen domain with id \[domain\]. The return value is the information needed by the Xen toolstack to setup the shared-memory blkfront protocol. Note that the same volume may be simultaneously attached to multiple hosts for example over a migrate. If an implementation needs to perform an explicit handover, then it should implement \[activate\] and \[deactivate\]. This function is idempotent.

> Client

```json
{
  "method": "Datapath.attach",
  "params": [ { "domain": "domain", "uri": "uri", "dbg": "dbg" } ],
  "id": 30
}
```

```ocaml
try
    let backend = Client.attach dbg uri domain in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Datapath.attach({ dbg: "string", uri: "string", domain: "string" })
    print (repr(results))
```

> Server

```json
{
  "implementations": [
    [
      "XenDisk",
      {
        "backend_type": "backend_type",
        "extra": { "field_1": "value_1", "field_2": "value_2" },
        "params": "params"
      }
    ],
    [
      "XenDisk",
      {
        "backend_type": "backend_type",
        "extra": { "field_1": "value_1", "field_2": "value_2" },
        "params": "params"
      }
    ]
  ],
  "domain_uuid": "domain_uuid"
}
```

```ocaml
try
    let backend = Client.attach dbg uri domain in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Datapath_myimplementation(Datapath_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def attach(self, dbg, uri, domain):
        """
        [attach uri domain] prepares a connection between the storage named by
        [uri] and the Xen domain with id [domain]. The return value is the
        information needed by the Xen toolstack to setup the shared-memory
        blkfront protocol. Note that the same volume may be simultaneously
        attached to multiple hosts for example over a migrate. If an
        implementation needs to perform an explicit handover, then it should
        implement [activate] and [deactivate]. This function is idempotent.
        """
        return {"domain_uuid": "string", "implementations": [None]}
    # ...
```


 Name    | Direction | Type    | Description                                                                                                                            
---------|-----------|---------|----------------------------------------------------------------------------------------------------------------------------------------
 dbg     | in        | string  | Debug context from the caller                                                                                                          
 uri     | in        | uri     | A URI which represents how to access the volume disk data.                                                                             
 domain  | in        | domain  | An opaque string which represents the Xen domain.                                                                                      
 backend | out       | backend | A description of which Xen block backend to use. The toolstack needs this to setup the shared memory connection to blkfront in the VM. 
## Method: `activate`
\[activate uri domain\] is called just before a VM needs to read or write its disk. This is an opportunity for an implementation which needs to perform an explicit volume handover to do it. This function is called in the migration downtime window so delays here will be noticeable to users and should be minimised. This function is idempotent.

> Client

```json
{
  "method": "Datapath.activate",
  "params": [ { "domain": "domain", "uri": "uri", "dbg": "dbg" } ],
  "id": 31
}
```

```ocaml
try
    let () = Client.activate dbg uri domain in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Datapath.activate({ dbg: "string", uri: "string", domain: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.activate dbg uri domain in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Datapath_myimplementation(Datapath_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def activate(self, dbg, uri, domain):
        """
        [activate uri domain] is called just before a VM needs to read or write
        its disk. This is an opportunity for an implementation which needs to
        perform an explicit volume handover to do it. This function is called
        in the migration downtime window so delays here will be noticeable to
        users and should be minimised. This function is idempotent.
        """
    # ...
```


 Name   | Direction | Type   | Description                                                
--------|-----------|--------|------------------------------------------------------------
 dbg    | in        | string | Debug context from the caller                              
 uri    | in        | uri    | A URI which represents how to access the volume disk data. 
 domain | in        | domain | An opaque string which represents the Xen domain.          
## Method: `deactivate`
\[deactivate uri domain\] is called as soon as a VM has finished reading or writing its disk. This is an opportunity for an implementation which needs to perform an explicit volume handover to do it. This function is called in the migration downtime window so delays here will be noticeable to users and should be minimised. This function is idempotent.

> Client

```json
{
  "method": "Datapath.deactivate",
  "params": [ { "domain": "domain", "uri": "uri", "dbg": "dbg" } ],
  "id": 32
}
```

```ocaml
try
    let () = Client.deactivate dbg uri domain in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Datapath.deactivate({ dbg: "string", uri: "string", domain: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.deactivate dbg uri domain in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Datapath_myimplementation(Datapath_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def deactivate(self, dbg, uri, domain):
        """
        [deactivate uri domain] is called as soon as a VM has finished reading
        or writing its disk. This is an opportunity for an implementation which
        needs to perform an explicit volume handover to do it. This function is
        called in the migration downtime window so delays here will be
        noticeable to users and should be minimised. This function is idempotent.
        """
    # ...
```


 Name   | Direction | Type   | Description                                                
--------|-----------|--------|------------------------------------------------------------
 dbg    | in        | string | Debug context from the caller                              
 uri    | in        | uri    | A URI which represents how to access the volume disk data. 
 domain | in        | domain | An opaque string which represents the Xen domain.          
## Method: `detach`
\[detach uri domain\] is called sometime after a VM has finished reading or writing its disk. This is an opportunity to clean up any resources associated with the disk. This function is called outside the migration downtime window so can be slow without affecting users. This function is idempotent. This function should never fail. If an implementation is unable to perform some cleanup right away then it should queue the action internally. Any error result represents a bug in the implementation.

> Client

```json
{
  "method": "Datapath.detach",
  "params": [ { "domain": "domain", "uri": "uri", "dbg": "dbg" } ],
  "id": 33
}
```

```ocaml
try
    let () = Client.detach dbg uri domain in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Datapath.detach({ dbg: "string", uri: "string", domain: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.detach dbg uri domain in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Datapath_myimplementation(Datapath_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def detach(self, dbg, uri, domain):
        """
        [detach uri domain] is called sometime after a VM has finished reading
        or writing its disk. This is an opportunity to clean up any resources
        associated with the disk. This function is called outside the migration
        downtime window so can be slow without affecting users. This function is
        idempotent. This function should never fail. If an implementation is
        unable to perform some cleanup right away then it should queue the
        action internally. Any error result represents a bug in the
        implementation.
        """
    # ...
```


 Name   | Direction | Type   | Description                                                
--------|-----------|--------|------------------------------------------------------------
 dbg    | in        | string | Debug context from the caller                              
 uri    | in        | uri    | A URI which represents how to access the volume disk data. 
 domain | in        | domain | An opaque string which represents the Xen domain.          
## Method: `close`
\[close uri\] is called after a disk is detached and a VM shutdown. This is an opportunity to throw away writes if the disk is not persistent.

> Client

```json
{
  "method": "Datapath.close",
  "params": [ { "uri": "uri", "dbg": "dbg" } ],
  "id": 34
}
```

```ocaml
try
    let () = Client.close dbg uri in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Datapath.close({ dbg: "string", uri: "string" })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.close dbg uri in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Datapath_myimplementation(Datapath_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def close(self, dbg, uri):
        """
        [close uri] is called after a disk is detached and a VM shutdown. This
        is an opportunity to throw away writes if the disk is not persistent.
        """
    # ...
```


 Name | Direction | Type   | Description                                                
------|-----------|--------|------------------------------------------------------------
 dbg  | in        | string | Debug context from the caller                              
 uri  | in        | uri    | A URI which represents how to access the volume disk data. 
## Interface: `Data`

This interface is used for long-running data operations such as
copying the contents of volumes or mirroring volumes to remote
destinations.

These operations are asynchronous and rely on the Tasks API to
report results and errors.

To mirror a VDI a sequence of these API calls is required:

1. Create a destination VDI using the Volume API on the destination
   SR. This must be the same size as the source. To minimize copying
   the destination VDI may be cloned from one that has been previously
   copied, as long as a disk from which the copy was made is still
   present on the source \(even as a metadata-only disk\)

2. Arrange for the destination disk to be accessible on the source
   host by suitable URL. This may be `nbd`, `iscsi`, `nfs` or
   other URL.

3. Start mirroring all new writes to the destination disk via the
   `Data.mirror` API call.

4. Find the list of blocks to copy via the CBT API call. Note that if
   the destination volume has not been 'prezeroed' then all of the
   blocks must be copied to the destination.

5. Start the background copy of the disk via a call to `DATA.copy`,
   passing in the list of blocks to copy. The plugin must ensure that
   the copy does not conflict with the mirror operation - ie., all
   writes from the mirror operation must not be overwritten by writes
   of old data from the copy operation.

6. The progress of the copy operation may be queried via the `Data.stat`
   call.

7. Once the copy operation has succesfully completed the destination
   disk will be a perfect mirror of the source.

     
## Method: `copy`
\[copy uri domain remotes blocks\] copies \[blocks\] from the local disk  to a remote URI. This may be called as part of a Volume Mirroring  operation, and hence may need to cooperate with whatever process is  currently mirroring writes to ensure data integrity is maintained. The \[remote\] parameter is a remotely accessible URI, for example, `nbd://root:pass@foo.com/path/to/disk` that must contain all necessary authentication tokens

> Client

```json
{
  "method": "Data.copy",
  "params": [
    {
      "blocklist": { "ranges": [ [ 0, 0 ], [ 0, 0 ] ], "blocksize": 0 },
      "remote": "remote",
      "domain": "domain",
      "uri": "uri",
      "dbg": "dbg"
    }
  ],
  "id": 35
}
```

```ocaml
try
    let operation = Client.copy dbg uri domain remote blocklist in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Data.copy({ dbg: "string", uri: "string", domain: "string", remote: "string", blocklist: {"blocksize": 0L, "ranges": [[]]} })
    print (repr(results))
```

> Server

```json
[ "Copy", [ "Copy_1", "Copy_2" ] ]
```

```ocaml
try
    let operation = Client.copy dbg uri domain remote blocklist in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Data_myimplementation(Data_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def copy(self, dbg, uri, domain, remote, blocklist):
        """
        [copy uri domain remotes blocks] copies [blocks] from the local disk
        to a remote URI. This may be called as part of a Volume Mirroring
        operation, and hence may need to cooperate with whatever process is
        currently mirroring writes to ensure data integrity is maintained.
        The [remote] parameter is a remotely accessible URI, for example,
        `nbd://root:pass@foo.com/path/to/disk` that must contain all necessary
        authentication tokens
        """
        return None
    # ...
```


 Name      | Direction | Type      | Description                                                     
-----------|-----------|-----------|-----------------------------------------------------------------
 dbg       | in        | string    | Debug context from the caller                                   
 uri       | in        | uri       | A URI which represents how to access the volume disk data.      
 domain    | in        | domain    | An opaque string which represents the Xen domain.               
 remote    | in        | uri       | A URI which represents how to access a remote volume disk data. 
 blocklist | in        | blocklist | List of blocks for copying                                      
 operation | out       | operation | The primary key for referring to a long-running operation       
## Method: `mirror`
\[mirror uri domain remote\] starts mirroring new writes to the volume  to a remote URI \(usually NBD\). This is called as part of a volume  mirroring process

> Client

```json
{
  "method": "Data.mirror",
  "params": [
    { "remote": "remote", "domain": "domain", "uri": "uri", "dbg": "dbg" }
  ],
  "id": 36
}
```

```ocaml
try
    let operation = Client.mirror dbg uri domain remote in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Data.mirror({ dbg: "string", uri: "string", domain: "string", remote: "string" })
    print (repr(results))
```

> Server

```json
[ "Copy", [ "Copy_1", "Copy_2" ] ]
```

```ocaml
try
    let operation = Client.mirror dbg uri domain remote in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Data_myimplementation(Data_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def mirror(self, dbg, uri, domain, remote):
        """
        [mirror uri domain remote] starts mirroring new writes to the volume
        to a remote URI (usually NBD). This is called as part of a volume
        mirroring process
        """
        return None
    # ...
```


 Name      | Direction | Type      | Description                                                     
-----------|-----------|-----------|-----------------------------------------------------------------
 dbg       | in        | string    | Debug context from the caller                                   
 uri       | in        | uri       | A URI which represents how to access the volume disk data.      
 domain    | in        | domain    | An opaque string which represents the Xen domain.               
 remote    | in        | uri       | A URI which represents how to access a remote volume disk data. 
 operation | out       | operation | The primary key for referring to a long-running operation       
## Method: `stat`
\[stat operation\] returns the current status of \[operation\]. For a  copy operation, this will contain progress information.

> Client

```json
{
  "method": "Data.stat",
  "params": [
    { "operation": [ "Copy", [ "Copy_1", "Copy_2" ] ], "dbg": "dbg" }
  ],
  "id": 37
}
```

```ocaml
try
    let status = Client.stat dbg operation in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Data.stat({ dbg: "string", operation: None })
    print (repr(results))
```

> Server

```json
{ "progress": 0.0, "failed": true }
```

```ocaml
try
    let status = Client.stat dbg operation in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Data_myimplementation(Data_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def stat(self, dbg, operation):
        """
        [stat operation] returns the current status of [operation]. For a
        copy operation, this will contain progress information.
        """
        return {"failed": True, "progress": None}
    # ...
```


 Name      | Direction | Type      | Description                                               
-----------|-----------|-----------|-----------------------------------------------------------
 dbg       | in        | string    | Debug context from the caller                             
 operation | in        | operation | The primary key for referring to a long-running operation 
 unnamed   | out       | status    | Status information for on-going tasks                     
## Method: `cancel`
\[cancel operation\] cancels a long-running operation. Note that the  call may return before the operation has finished.

> Client

```json
{
  "method": "Data.cancel",
  "params": [
    { "operation": [ "Copy", [ "Copy_1", "Copy_2" ] ], "dbg": "dbg" }
  ],
  "id": 38
}
```

```ocaml
try
    let () = Client.cancel dbg operation in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Data.cancel({ dbg: "string", operation: None })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.cancel dbg operation in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Data_myimplementation(Data_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def cancel(self, dbg, operation):
        """
        [cancel operation] cancels a long-running operation. Note that the
        call may return before the operation has finished.
        """
    # ...
```


 Name      | Direction | Type      | Description                                               
-----------|-----------|-----------|-----------------------------------------------------------
 dbg       | in        | string    | Debug context from the caller                             
 operation | in        | operation | The primary key for referring to a long-running operation 
## Method: `destroy`
\[destroy operation\] destroys the information about a long-running  operation. This should fail when run against an operation that is  still in progress.

> Client

```json
{
  "method": "Data.destroy",
  "params": [
    { "operation": [ "Copy", [ "Copy_1", "Copy_2" ] ], "dbg": "dbg" }
  ],
  "id": 39
}
```

```ocaml
try
    let () = Client.destroy dbg operation in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Data.destroy({ dbg: "string", operation: None })
    print (repr(results))
```

> Server

```json
null
```

```ocaml
try
    let () = Client.destroy dbg operation in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Data_myimplementation(Data_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def destroy(self, dbg, operation):
        """
        [destroy operation] destroys the information about a long-running
        operation. This should fail when run against an operation that is
        still in progress.
        """
    # ...
```


 Name      | Direction | Type      | Description                                               
-----------|-----------|-----------|-----------------------------------------------------------
 dbg       | in        | string    | Debug context from the caller                             
 operation | in        | operation | The primary key for referring to a long-running operation 
## Method: `ls`
\[ls\] returns a list of all current operations

> Client

```json
{ "method": "Data.ls", "params": [ { "dbg": "dbg" } ], "id": 40 }
```

```ocaml
try
    let operations = Client.ls dbg in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import necessary libraries if needed
# we assume that your library providing the client is called myclient and it provides a connect method
import myclient

if __name__ == "__main__":
    c = myclient.connect()
    results = c.Data.ls({ dbg: "string" })
    print (repr(results))
```

> Server

```json
[ [ "Copy", [ "Copy_1", "Copy_2" ] ], [ "Copy", [ "Copy_1", "Copy_2" ] ] ]
```

```ocaml
try
    let operations = Client.ls dbg in
    ...
with Exn (Unimplemented str) -> ...

```

```python

# import additional libraries if needed

class Data_myimplementation(Data_skeleton):
    # by default each method will return a Not_implemented error
    # ...

    def ls(self, dbg):
        """
        [ls] returns a list of all current operations
        """
        return [None]
    # ...
```


 Name    | Direction | Type       | Description                   
---------|-----------|------------|-------------------------------
 dbg     | in        | string     | Debug context from the caller 
 unnamed | out       | operations | A list of operations          
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
### exnt
```json
[ "Unimplemented", "exnt" ]
```
type `exnt` = `variant { ... }`

#### Constructors
 Name          | Type   | Description 
---------------|--------|-------------
 Unimplemented | string |             