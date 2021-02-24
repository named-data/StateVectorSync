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

`Socket` provides an interface for data exchange over `Logic`.

A base class `SocketBase` is provided, that must be extended to create `Socket` classes. Two implementations are provided, `Socket` and `SharedSocket`. `Socket` uses the data prefix of a node as the `NodeID` in the version vector, while `SocketShared` uses a sub-namespace inside the sync prefix for data exchange.

* `SocketBase(Face, Name syncPrefix, Name dataPrefix, NodeID, UpdateCallback, SecurityOptions)` - constructor.
* `SocketBase::publishData(Buffer, SeqNo, NodeID)` - publish the buffer as signed data on the given `NodeID` and sequence number.
* `SocketBase::fetchData(NodeID, SeqNo, Callback)` - fetch the data for a given node and sequence number.
* `SocketBase::getDataName(NodeID, SeqNo)` - derived classes must override this method to return the name of the data packet corresponding to the given `NodeID` and sequence number. This allows usage of arbitrary data semantics.
* `SocketBase::getLogic()` - get the underlying `Logic`.

# License
StateVectorSync is an open source project licensed under the CC-BY-SA 4.0. See [LICENSE](./LICENSE) for more information.

![CC-BY-SA](https://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-sa.svg)

Different licenses for the implementations might apply.
