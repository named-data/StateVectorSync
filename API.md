# StateVectorSync API Description

This page describes the API of [StateVectorSync (SVS)](README.md). While there are implementations of SVS for multiple programming languages available, the basic API is the same across all official implementations.

The two components of an SVS library are:
* `SVSync` - provides an interface built on `SVSyncCore` to provide data delivery and synchronization.
* `SVSyncCore` - purely contains state vector sync logic, which can be used for synchronizing version vectors across nodes.

## SVSync API

`SVSync` provides an interface for data exchange and synchronization using SVS. The API is described below:

* `SVSync(Face, Name syncPrefix, Name nodePrefix, UpdateCallback, SecurityOptions)` - constructor.
* `SVSync::publishData(Buffer)` - publish the buffer as signed data on the next sequence number.
* `SVSync::fetchData(Name nodePrefix, SeqNo, Callback)` - fetch the data for a given node and sequence number.
* `SVSync::getCore()` - get the underlying `SVSyncCore` instance.

## SVSyncCore API

`SVSyncCore` provides a simple interface to run pure sync.

* `SVSyncCore(Face, Name syncPrefix, UpdateCallback, SecurityOptions)` - constructor to start SVS
* `SVSyncCore::getState()` - gets the current known version vector
* `SVSyncCore::updateSeqNo(SeqNo seq, NodeID nid)` - updates an entry in the current known version vector

On updating the version vector with `updateSeqNo`, the changes are propagated to other nodes running on the same sync prefix. When another node updates the version vector, `UpdateCallback` is called with state change information.

## SVSyncBase API (advanced usage)

A base class `SVSyncBase` is provided, that may be extended by applications to create derived `SVSync` classes, with arbitrary data semantics.

Derived classes must define:
* `SVSyncBase::getDataName(NodeID, SeqNo)` - return the name of the data packet corresponding to the given `NodeID` and sequence number `SeqNo`.

# License
StateVectorSync is an open source project licensed under the CC-BY-SA 4.0. See [LICENSE](./LICENSE) for more information.

![CC-BY-SA](https://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-sa.svg)

Different licenses for the implementations might apply.
