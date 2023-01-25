# State Vector Sync API Description

This page describes the API of [State Vector Sync (SVS)](/README.md). While there are implementations of SVS for multiple programming languages available, the basic API is the same across all official implementations.

The three components of an SVS library are:

* `SVSPubSub` - a high level pub/sub API built over `SVSync`.
* `SVSync` - provides an interface built on `SVSyncCore` to provide data delivery and synchronization.
* `SVSyncCore` - purely contains state vector sync logic, which can be used for synchronizing version vectors across nodes.

## SVSPubSub API

The `SVSPubSub` class provides a high level Pub/Sub interface built over SVS, originally described [here](https://doi.org/10.1145/3460417.3483376).

* `SVSPubSub(Name syncPrefix, Name nodePrefix, Face, UpdateCallback, SecurityOptions)` - registers prefixes and starts Sync.
* `SVSPubSub::publish(Name, Buffer)` - publishes a buffer of data with a given name to the Sync group.
* `SVSPubSub::subscribe(Name, Callback)` - subscribes to a given data name prefix. The callback will be called with the name, buffer, producer prefix and SVS sequence number for matching publications.
* `SVSPubSub::subscribeToProducer(Name, Callback)` - subscribes to all data from producers with names matching the given prefix.
* `SVSPubSub::unsubscribe(Handle)` - unsubscribes from a given handle (returned by the subscribe calls).

With the Pub/Sub API, the library handles Sync functions as well as segmentation, reassembly, signing and encapsulation of data packets. With the lower level `SVSync` API, the user is responsible for segmentation and reassembly.

## SVSync API

`SVSync` provides an interface for data exchange and synchronization using SVS. The API is described below:

* `SVSync(Name syncPrefix, Name nodePrefix, Face, UpdateCallback, SecurityOptions)`
* `SVSync::publishData(Buffer)` - publish the buffer as signed data on the next sequence number.
* `SVSync::fetchData(NodeID, SeqNo, Callback)` - fetch the data for a given node and sequence number.
* `SVSync::getCore()` - get the underlying `SVSyncCore` instance.

## SVSyncCore API

`SVSyncCore` provides a simple interface to run pure sync.

* `SVSyncCore(Face, Name syncPrefix, UpdateCallback, SecurityOptions)`
* `SVSyncCore::getState()` - gets the current known version vector.
* `SVSyncCore::updateSeqNo(SeqNo, NodeID)` - updates an entry in the current known version vector.

On updating the version vector with `updateSeqNo`, the changes are propagated to other nodes running on the same sync prefix. When another node updates the version vector, `UpdateCallback` is called with state change information. Note that the `NodeID` is equivalent to, and defaults to the `nodePrefix`.

## SVSyncBase API (advanced usage)

A base class `SVSyncBase` is provided, that may be extended by applications to create derived `SVSync` classes, with arbitrary data semantics.

Derived classes must define:

* `SVSyncBase::getDataName(NodeID, SeqNo)` - return the name of the data packet corresponding to the given `NodeID` and sequence number `SeqNo`.

## License

![CC-BY-SA](https://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-sa.svg)

State Vector Sync is an open source project licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/). See [LICENSE](/LICENSE) for more information.

Different licenses for the implementations might apply.
