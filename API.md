# StateVectorSync API Description

This page describes the API of [StateVectorSync (SVS)](README.md). While there are implementations of SVS for multiple programming languages available, the basic API is the same across all official implementations.

The two components of an SVS library are:
* `Logic` - purely contains state vector sync logic, which can be used for synchronizing version vectors across nodes.
* `Socket` - provides an interface built on `Logic` to provide data delivery and synchronization.

## Logic API

`Logic` provides a simple interface to run pure sync.

* `Logic(Face, Name syncPrefix, UpdateCallback, SecurityOptions)` - constructor to start SVS
* `Logic::getState()` - gets the current known version vector
* `Logic::updateSeqNo(SeqNo seq, NodeID nid)` - updates an entry in the current known version vector

On updating the version vector with `updateSeqNo`, the changes are propagated to other nodes running on the same sync prefix. When another node updates the version vector, `UpdateCallback` is called with state change information.

## Socket API

`Socket` provides an interface for data exchange over `Logic`. The `Socket` API is described below:

* `Socket(Face, Name syncPrefix, Name nodePrefix, UpdateCallback, SecurityOptions)` - constructor.
* `Socket::publishData(Buffer)` - publish the buffer as signed data on the next sequence number.
* `Socket::fetchData(Name nodePrefix, SeqNo, Callback)` - fetch the data for a given node and sequence number.
* `Socket::getLogic()` - get the underlying `Logic`.

## SocketBase API (advanced usage)

A base class `SocketBase` is provided, that may be extended by applications to create derived `Socket` classes, with arbitrary data semantics.

Derived classes must define:
* `SocketBase::getDataName(NodeID, SeqNo)` - return the name of the data packet corresponding to the given `NodeID` and sequence number `SeqNo`.

# License
StateVectorSync is an open source project licensed under the CC-BY-SA 4.0. See [LICENSE](./LICENSE) for more information.

![CC-BY-SA](https://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-sa.svg)

Different licenses for the implementations might apply.
