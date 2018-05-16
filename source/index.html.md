---
title: SMAPIv3 Xapi Storage Interface

language_tabs: # must be one of https://git.io/vQNgJ
  - json
  - ocaml
  - python

toc_footers:
  - <a href='https://github.com/xapi-project/xapi-storage'>Source Code of SMAPIv3 Interface</a>
  - <a href='https://github.com/xapi-project/xapi-storage/tree/slate'>Source Code of Website</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - plugin
  - datapath
  - volume
  - task

search: true
---

# Introduction

The SMAPIv3 storage interface provides an easy way to connect [xapi] to any
storage type.

## Who Is This For?

![Your bit here](your-bit-here.svg)

This is for anyone who has a storage system which is not supported by xapi
out-of-the-box.

This is also for anyone who wants to manage their storage in a customized way.
If you can make your volumes appear as Linux block devices or you can refer to
the volumes via URIs of the form `iscsi://`,  `nfs://`,  or `rbd://`, then this
documentation is for you.

**No Xapi or Xen specific knowledge is required.**

## Status of This Documentation

This documentation is a draft intended for discussion, which happens through
the [issues on GitHub](https://github.com/xapi-project/xapi-storage/issues).

# Learn

## Features

The xapi storage interface supports the following features:

* Storage Repositories: collections of Volumes on the same storage substrate
* Volumes may be created, cloned, snapshotted, queried, attached to and
  detached from VMs
* Storage Repositories can be incrementally probed; for example given only an
  iSCSI target portal, the IQNs and then the LUNs can be discovered.
* Everything is named by URIs, where the scheme describes the protocol used to
  access the data (e.g. file, iscsi)
* Implementations expose capabilities and these are reported via the XenAPI
* Dynamic properties (such as space utilisation, I/O bandwidth and latency) can
  be exposed as "datasources" which are compatible with the xapi toolstack RRD
  framework.
* The storage implementation can be used to store the xapi HA statefile and
  database redo logs
* If the storage implementation supports fast clone, then it will work with the
  xapi "clone on boot" feature

## Concepts

When a virtual machine looks at its disk, it sees something which looks like a
real physical disk. This is an illusion. In reality the bytes of data written
to the "virtual disk" probably reside in a file in a filesystem or in a logical
volume on some physical storage system that the VM cannot see.

We call the real physical storage system a **Storage Repository** (SR).

We call the virtual disks within the SR **volumes**.

### Manipulating volumes

When a VM is installed, a volume will be created. Typically this volume will be
deleted when the VM is uninstalled. The Xapi toolstack doesn't know how to
manipulate volumes on your storage system directly; instead it delegates to
"Volume plugins": implementation-specific plugins which know how to talk the
storage-specific APIs. These volume plugins can be anything from simple scripts
in domain 0 to sophisticated services running somewhere on the network.

Consider for example a system using Linux LVM, where individual LVs are mapped
to VMs as virtual disks. The volume plugin for LVM could implement the
`Volume.create` API by simply calling

`lvcreate -n name -L 64GiB -Z n`

Consider another example where volumes are simple sparse files stored on an NFS
share. The volume plugin could implement the `Volume.create` API by simply
calling:

`dd if=/dev/zero of=disk.name count=1 skip=64G`

### Connecting volumes to VMs

VMs running on the Xen hypervisor use special shared-memory protocols to access
their disks and network interfaces. There are several implementations of these
protocols including:

* the [Linux
  xen-blkback](http://lxr.free-electrons.com/source/drivers/block/xen-blkback/blkback.c)
  kernel module
* the [FreeBSD
  blkback](https://github.com/freebsd/freebsd/blob/master/sys/dev/xen/blkback/blkback.c)
  driver
* the [QEMU
  qdisk](https://github.com/qemu/qemu/blob/master/hw/xen/xen_backend.c)
  userspace implementation
* the [XenServer
  tapdisk3](http://xenserver.org/discuss-virtualization/virtualization-blog/entry/tapdisk3.html)
  userspace implementation
* the [MirageOS](https://github.com/mirage/mirage-block-xen) userspace and
  kernelspace implementation

With so many implementations to choose from, which one should we use for a
given volume? This decision - and how to configure the implementation for
maximum performance - is the job of the Datapath plugin.

### Disks as URIs

Every volume has one or more URIs, which describe how to access the data within
the volume. Examples include:

* `file:///local/block/device` - a local file or block device
* `nfs://server/export/path/file.qcow2` - a remote NFS server exporting a
  .qcow2 format file
* `smb://server/export/path/file.vhd` - a remote CIFS server exporting a .vhd
  format file
* `rbd://pool/volume` - a volume in a Ceph storage pool
* `iscsi://target/lun` - a LUN on an iSCSI array

The Xapi toolstack takes the list of URIs provided by the Volume plugin and
creates a connection between the VM and the disk. Xapi chooses a "Datapath
plugin" based on the URI scheme. The Datapath plugin returns Xen-specific
connection details, including choice of backend (kernel blkback or userspace
qemu) and caching options.

## Architecture

The system is divided into two parts, intended for people with different
expertise:

1. Volume Plugin: this is the storage control-plane. Xapi delegates Volume
   manipulation requests to these plugins, which know how to operate on the
   physical storage which could be anything from an NFS/CIFS server, an iSCSI
   target or a Ceph deployment. **No virtualisation knowledge is required to
   write a Volume Plugin.** The volume plugin do not perform I/O directly,
   instead they associate volumes with URIs which encode a particular method
   for accessing the disk data. For example an NFS plugin could expose URIs of
   the form `nfs://server/path/file`
2. Datapath Plugin: this is the storage data-plane. This is Xen-specific code
   which chooses the best way to connect VMs to individual disks. **Xen expertise
   is needed to write a datapath plugin.** Note that this code doesn't know about
   volumes; it only knows about specific volume access protocols such as NFS,
   iSCSI and RBD.

### Volume Plugins

Clients such as [OpenStack], [CloudStack], [XenCenter] and [Xen Orchestra]
create VMs and virtual disks by sending XenAPI requests to Xapi. Xapi doesn't
know how to manipulate storage directly, so it delegates storage operations to
a **Volume Plugin**. Each "Storage Repository" is associated with exactly one
Volume Plugin. These plugins know how to

* create volumes;
* destroy volumes;
* snapshot volumes;
* clone volumes; and
* list the ways in which a volume may be accessed.

The following diagram shows a XenAPI client sending a VDI.create request
causing Xapi to send a Volume.create request to a specific plugin:

![Diagram showing a XenAPI client sending a VDI.create request causing Xapi to
send a Volume.create request to a plugin.](arch-vol-create.svg)

### Datapath Plugins

The following diagram shows a XenAPI client sending a VM.start request. Xapi
calls `Volume.stat` to list the available access methods for the VM's disks. The
Ceph Volume Plugin returns a URL of the form `rbd://server/pool` so Xapi consults
its "rbd" Datapath plugin and asks "how should I connect this datapath to the
VM?". The datapath plugin tells Xapi to use QEMU qdisk's built-in support for
RBD via librados, so Xapi tells libxl to set this up.

![Diagram showing a XenAPI client sending a VM.start request.](arch-vm-start.svg)

## Frequently Asked Questions

How Do I...

### Test my code?

> The OCaml and python generated code includes a convenient command-line parser
> so if you write:

```ocaml
module Cmds = Xapi_storage.Control.Sr(Cmdlinergen.Gen ())

Cmdliner.Term.eval_choice default_cmd (List.map (fun t -> t rpc) (Cmds.implementation ()))
```

```python
class Implementation(xapi.volume.SR_skeleton):
    pass

if __name__ == "__main__":
    cmd = xapi.volume.SR_commandline(Implementation())
    cmd.attach()
```

> You'll be able to run the command like this:

```
$ ./SR.attach
usage: SR.attach [-h] [-j] dbg uri
SR.attach: error: too few arguments

$ ./SR.attach -h
usage: SR.attach [-h] [-j] dbg uri

[attach uri]: attaches the SR to the local host. Once an SR is attached then
volumes may be manipulated.

positional arguments:
  dbg         Debug context from the caller
  uri         The Storage Repository URI

optional arguments:
  -h, --help  show this help message and exit
  -j, --json  Read json from stdin, print json to stdout
```

Although it's not enforced by the interface, plugin implementations should
avoid interacting with the toolstack so that they can be easily tested in
isolation.

### Report dynamic properties like space consumption?

Dynamic properties like space consumption, bandwidth or latency should be
exposed as "datasources". The SR.stat function should return a list of URIs
pointing at these in "xenostats" format. The toolstack will hook up these
datasources to the [xcp-rrdd] daemon which will record history. XenAPI clients
can then use the [RRD API] to fetch the data, draw graphs etc.

### Expose backend-specific functions?

The SMAPIv3 is intended to be a _generic_ API. Before extending the SMAPIv3
itself, first ask the question: would this make sense for 3 completely
different storage stacks (e.g. consider Ceph, LVM over iSCSI and gfs2). If the
concept is actually general then propose an SMAPIv3 update via a [pull
request](https://github.com/xapi-project/xapi-storage/pulls). If the concept
is actually backend-specific then consider adding a new [XenAPI extension] for it
and name the API appropriately (e.g. "LVHD.foo").

### Call [xapi]?

Nothing in the interface prevents you from making RPC calls to xapi or other
toolstack components, however doing so will make it more difficult to test your
component in isolation.

In the past, a common reason to call xapi was to store data in the xapi
database, for example the "sm-config" fields. This was unreliable because

* The "sm-config" key,value pairs can disappear, for example if the SR is
  forgotten and then re-attached.
* The "sm-config" key,value pairs may be duplicated over VDI.clone (or not):
  the exact behaviour was never defined.
* If the storage is itself reverted to a previous point in time (imagine an
  SR-level snapshot), then the "sm-config" keys, value pairs will not be
  reverted.

It is strongly recommended to store all storage-related state on the storage
medium. This ensures that the metadata has a "shared fate" with the data: if
data is restored from backup, reverted to a snapshot, then so is the metadata.

### Tie my cluster to the [xapi] Pool?

Ideally a storage cluster would be managed separately from a xapi pool, with
it's own configuration and monitoring interfaces. The storage cluster could be
very large (consider Ceph-style scale-out) while the xapi pool is designed to
remain within a rack.

In the past, a common reason to tie a storage cluster to the xapi pool was to
piggyback on the xapi notions of a single pool master, HA and inter-host
authenticated RPC mechanisms to co-ordinate activities sych as vhd coalescing.
If it is still necessary to tie a storage cluster to a xapi pool then the
storage implementation should launch it's own "pool monitor" service which
could use the xapi pool APIs to track host membership and master status. Note:
this might require adding new capabilities to xapi's pool APIs, but they should
not be part of the storage API itself.

Note: in the case where a particular storage implementation requires a
particular HA cluster stack to be running, this can be declared in the
`Plugin.query` call.

# Develop

Below you can find the SMAPIv3 API Reference:

[xapi]: https://github.com/xapi-project/xen-api
[OpenStack]: http://www.openstack.org/
[CloudStack]: http://cloudstack.apache.org/
[XenCenter]: https://github.com/xenserver/xenadmin
[Xen Orchestra]: https://xen-orchestra.com/
[xcp-rrdd]: https://github.com/xapi-project/xcp-rrdd
[RRD API]: http://xapi-project.github.io/xen-api/metrics.html
[XenAPI extension]: http://xapi-project.github.io/devguide/howtos/add-api-extension.html
