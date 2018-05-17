---
title: plugin

language_tabs:
 - json
 - ocaml
 - python

search: true
---
# plugin
The Datapath plugin takes a URI which points to virtual disk data  and chooses a Xen datapath implementation: driver domain, blkback  implementation and caching strategy.
## Type definitions
### query_result
```json
{
  "required_cluster_stack": [ "required_cluster_stack" ],
  "configuration": { "configuration": "configuration" },
  "features": [ "features" ],
  "required_api_version": "required_api_version",
  "version": "version",
  "copyright": "copyright",
  "vendor": "vendor",
  "description": "description",
  "name": "name",
  "plugin": "plugin"
}
```
type `query_result` = struct  { ... }
Properties of this implementation
#### Members
 Name                   | Type                   | Description                                                               
------------------------|------------------------|---------------------------------------------------------------------------
 plugin                 | string                 | plugin name, used in the XenAPI as SR.type                                
 name                   | string                 | short name                                                                
 description            | string                 | description                                                               
 vendor                 | string                 | entity (e.g. company, project, group) which produced this  implementation 
 copyright              | string                 | copyright                                                                 
 version                | string                 | version                                                                   
 required_api_version   | string                 | minimum required API version                                              
 features               | string list            | features supported by this plugin                                         
 configuration          | (string * string) list | key/description pairs describing required device_config parameters        
 required_cluster_stack | string list            | the plugin requires one of these cluster stacks to be active              
### srs
```json
[ "srs" ]
[]
```
type `srs` = string list

## Interface: `Plugin`
Discover properties of this implementation. Every implementation must support the query interface or it will not be recognised as a storage plugin by xapi.
## Method: `Query`
Query this implementation and return its properties. This is  called by xapi to determine whether it is compatible with xapi  and to discover the supported features.

> Client

```json
{ "method": "Plugin.Query", "params": [ { "dbg": "dbg" } ], "id": 1 }
```

```ocaml
try
    let query_result = Client.Query dbg in
    ...
with Exn (Unimplemented str) -> ...

```

```python

import xmlrpclib
import xapi
from storage import *

if __name__ == "__main__":
    c = xapi.connect()
    results = c.Plugin.Query({ dbg: "string" })
    print (repr(results))
```

> Server

```json
{
  "required_cluster_stack": [
    "required_cluster_stack_1", "required_cluster_stack_2"
  ],
  "configuration": { "field_1": "value_1", "field_2": "value_2" },
  "features": [ "features_1", "features_2" ],
  "required_api_version": "required_api_version",
  "version": "version",
  "copyright": "copyright",
  "vendor": "vendor",
  "description": "description",
  "name": "name",
  "plugin": "plugin"
}
```

```ocaml
try
    let query_result = Client.Query dbg in
    ...
with Exn (Unimplemented str) -> ...

```

```python

import xmlrpclib
import xapi
from storage import *

class Plugin_myimplementation(Plugin_skeleton):
    # by default each method will return a Not_implemented error
    # ...
    def Query(self, dbg):
        """Discover properties of this implementation. Every implementation must support the query interface or it will not be recognised as a storage plugin by xapi."""
        result = {}
        result = { "plugin": "string", "name": "string", "description": "string", "vendor": "string", "copyright": "string", "version": "string", "required_api_version": "string", "features": [ "string", "string" ], "configuration": { "string": "string" }, "required_cluster_stack": [ "string", "string" ] }
        return result
    # ...
```


 Name    | Direction | Type         | Description                       
---------|-----------|--------------|-----------------------------------
 dbg     | in        | string       | Debug context from the caller     
 unnamed | out       | query_result | Properties of this implementation 
## Method: `ls`
[ls dbg]: returns a list of attached SRs

> Client

```json
{ "method": "Plugin.ls", "params": [ { "dbg": "dbg" } ], "id": 2 }
```

```ocaml
try
    let srs = Client.ls dbg in
    ...
with Exn (Unimplemented str) -> ...

```

```python

import xmlrpclib
import xapi
from storage import *

if __name__ == "__main__":
    c = xapi.connect()
    results = c.Plugin.ls({ dbg: "string" })
    print (repr(results))
```

> Server

```json
[ "srs_1", "srs_2" ]
```

```ocaml
try
    let srs = Client.ls dbg in
    ...
with Exn (Unimplemented str) -> ...

```

```python

import xmlrpclib
import xapi
from storage import *

class Plugin_myimplementation(Plugin_skeleton):
    # by default each method will return a Not_implemented error
    # ...
    def ls(self, dbg):
        """Discover properties of this implementation. Every implementation must support the query interface or it will not be recognised as a storage plugin by xapi."""
        result = {}
        result["srs"] = [ "string", "string" ]
        return result
    # ...
```


 Name | Direction | Type   | Description                   
------|-----------|--------|-------------------------------
 dbg  | in        | string | Debug context from the caller 
 srs  | out       | srs    | The attached SRs              
## Method: `diagnostics`
Returns a printable set of backend diagnostic information.  Implementations are encouraged to include any data which will  be useful to diagnose problems. Note this data should not  include personally-identifiable data as it is intended to be   automatically included in bug reports.

> Client

```json
{ "method": "Plugin.diagnostics", "params": [ { "dbg": "dbg" } ], "id": 3 }
```

```ocaml
try
    let diagnostics = Client.diagnostics dbg in
    ...
with Exn (Unimplemented str) -> ...

```

```python

import xmlrpclib
import xapi
from storage import *

if __name__ == "__main__":
    c = xapi.connect()
    results = c.Plugin.diagnostics({ dbg: "string" })
    print (repr(results))
```

> Server

```json
"diagnostics"
```

```ocaml
try
    let diagnostics = Client.diagnostics dbg in
    ...
with Exn (Unimplemented str) -> ...

```

```python

import xmlrpclib
import xapi
from storage import *

class Plugin_myimplementation(Plugin_skeleton):
    # by default each method will return a Not_implemented error
    # ...
    def diagnostics(self, dbg):
        """Discover properties of this implementation. Every implementation must support the query interface or it will not be recognised as a storage plugin by xapi."""
        result = {}
        result["diagnostics"] = "string"
        return result
    # ...
```


 Name        | Direction | Type   | Description                                                         
-------------|-----------|--------|---------------------------------------------------------------------
 dbg         | in        | string | Debug context from the caller                                       
 diagnostics | out       | string | A string containing loggable human-readable diagnostics information 
## Errors
### exnt
```json
[ "Unimplemented", "exnt" ]
```
type `exnt` = variant { ... }

#### Constructors
 Name          | Type   | Description 
---------------|--------|-------------
 Unimplemented | string |             