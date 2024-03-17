# State Vector Sync Pub/Sub Specification

This page describes the specification of the SVS-PS protocol. SVS-PS runs on top of [State Vector Sync](Specification.md) and performs the additional functions described here.

_Last update to specification: 2023-05-19_

## Overview

For each call, SVS-PS performs the functions in the order below:

1. `publish`
    1. Segment, sign and encapsulate the data
    1. Provide the name mapping
    1. Increment the sequence number
1. `subscribe` - events after receiving a new sequence number
    1. Fetch the name mapping
    1. Fetch the data segments
    1. Decapsulate and reassemble the data
    1. Validate all signatures

## Publishers

This section describes the functions performed at data publishers.

### Segmentation and Encapsulation

Data blobs passed to the `publish` call MUST be segmented if it is larger than the network MTU. It SHOULD NOT be segmented if it is small enough to fit directly in the network MTU.

If segmentation is needed, each segment is named per the NDN segmentation guidelines, with version number of `0`. Each segment MUST contain the `FinalBlockId` field set to the name of the last segment. Segmentation MAY be done asynchronously.

Each segment MUST be encapsulated inside an "outer" Data packet, named following SVS Data naming convention. The SVS sequence number is the next available sequence number for the publisher. The `ContentType` field of the outer Data packet MUST be set to `6` (i.e. TLV-TYPE number of `Data`). The `FreshnessPeriod` field of the outer Data packet MUST be set to a non-zero value. The `FinalBlockId` field of the outer Data packet MUST be set to the name of the last segment.

```text
Segment (encapsulated): /<app-name>/v=0/seg=<seg>
Segment (outer): /<node-prefix>/<sync-prefix>/<seq-num>/v=0/seg=<seg>

Examples:
Application Data Name: /ndn/data/example
First Segment (encapsulated): /ndn/data/example/v=0/seg=0
First Segment (outer): /node/a/some/group/%19/v=0/seg=0
```

If no segmentation is done, the Data packet MUST NOT contain a `FinalBlockId` field. The Data packet MUST, likewise, be encapsulated inside an "outer" Data packet. The name of outer Data packet MUST NOT contain version and segment components. The outer Data packet MUST NOT contain a `FinalBlockId` field.

```text
Unsegmented (encapsulated): /<app-name>
Unsegmented (outer): /<node-prefix>/<sync-prefix>/<seq-num>

Examples:
Application Data Name: /ndn/data/example
Unsegmented (encapsulated): /ndn/data/example
Unsegmented (outer): /node/a/some/group/%19
```

On completion of segmentation (or immediately if no segmentation or asynchronous segmentation), the Name Mapping MUST be updated and the SVS sequence number MUST be incremented to the next available sequence number to indicate the data is published.

### Name Mapping

The publisher MUST maintain a name mapping table for each NodeID it publishes to. The table MUST be updated on each publish call, to contain the tuple of the application Name of the data and the SVS sequence number.

Each SVS-PS instance MUST respond to Interests for mapping data for each NodeID it publishes at. The name of the `query` Interest is defined as follows:

```
/<node-prefix>/<sync-prefix>/MAPPING/<low-seq>/<high-seq>
```

On receiving a matching Interest, the SVS-PS instance MUST respond with a Data packet containing the appropriate subset of the SeqNo-Name tuples for the node matching the `<node-prefix>`, from `low-seq` to `high-seq` both inclusive. The content of the data packet MUST be encoded as a TLV block of type `MappingData` as defined below.

```abnf
MappingData = MAPPING-DATA-TYPE TLV-LENGTH
              NodeID
              *MappingEntry

MappingEntry = MAPPING-ENTRY-TYPE TLV-LENGTH
               SeqNo
               ApplicationName

NodeID = Name
ApplicationName = Name
SeqNo = SEQ-NO-TYPE TLV-LENGTH NonNegativeInteger

MAPPING-DATA-TYPE = 205
MAPPING-ENTRY-TYPE = 206
SEQ-NO-TYPE = 204
```

A `MappingEntry` MAY contain additional information such as the time of publication of the data. Any additional information MUST be encoded as one or more TLV blocks following the `ApplicationName` block. SVS-PS implementations SHOULD provide a mechanism to include additional mapping information, and SHOULD allow applications to filter incoming publications based on the received mapping entry. Implementations MAY also provide mechanisms for automatic handling of well-known additional blocks, such as fetching only recent data based on the `Timestamp` block included in the mapping entry.

The following well-known additional blocks are RECOMMENDED.

```abnf
Timestamp = TimestampNameComponent
```

### Name Mapping Delivery Optimization

To optimize the delivery of mapping data, SVS-PS implementations MAY piggyback mapping data in the Sync Interest's `ApplicationParameters`. If present, the mapping data SHOULD be appended as a single block after the StateVector in the `ApplicationParameters`. The mapping data MUST be encoded as a TLV block of type `MappingData` as defined above.

When this optimization is implemented, Mapping Data SHOULD be inserted in every outgoing Sync Interest sent as a result of new data production. On receiving a Sync Interest, this mapping data can be utilized only after the Interest signature has been validated.

## Subscribers

This section describes the functions performed at data subscribers.

### Subscribing

On receiving a `subscribe` call, the topic prefix and callback MUST be added to a table of `Subscriptions`. The `subscribe` calls MUST return immediately. For `subscribeToProducer` calls, the producer prefix and callback MUST be added to a separate table of `ProducerSubscriptions`.

### Data Fetching

On receiving a new sequence number, the application name mapping for the sequence number MUST be fetched if ALL of the following are satisfied:

1. There is at least one entry in the `Subscriptions` table
1. The producer does not match any entry in the `ProducerSubscriptions` table

The name of the query Interest is defined in the publishers section.

The data segments MUST be fetched if ANY of following are satisfied:

1. There is at least one entry in the `Subscriptions` table that matches the application data name in the mapping (after the mapping is received).
1. The producer matches an entry in the `ProducerSubscriptions` table

The data segment is fetched using the name of the outer data packet described in the producers section. All data packets upto the `FinalBlockId` MUST be fetched.

If any requests time out, the implementation MUST provide an option to retry infinitely (with backoff and delay) or a finite number of times. If the number of retries is finite, the implementation SHOULD provide a callback to notify the application of the failure.

### Segment Decapsulation and Reassembly

Each segment MUST be decapsulated as soon as it is received. The received encapsulated segments MUST be reassembled in-order of the segment number. The reassembled data blob MUST be passed to the callback function for each subscription from the `Subscriptions` table that matches the application data name, and each subscription from `ProducerSubscriptions` table that matches the producer. Decapsulation and reassembly MAY be done asynchronously.

## Security

Implmentations of SVS-PS SHOULD provide security primitives to applications. If the implementation includes security features, it MUST implement the following features.

1. Publishers MUST accept separate data signers for encapsulated and outer data packets.
1. Publishers MUST accept a separate data signer for the name mapping data packet.
1. Each data packet MUST be signed with the appropriate signer if specified.
1. Subscribers MUST accept corresponding data validators for each type of data.
1. Each data packet MUST be validated before being returned to the application. The outer data packet MUST be validated before decapsulation. Only validated segments must be reassembled and returned to the application.
1. If data validation fails, the application SHOULD be notified of the failure. Implementations SHOULD provide an option to retry fetching segments that could not be validated.
1. If desegmentation is done asynchronously, the application MUST be notified of the failure to validate a segment if some data has already been returned to the application.

## License

![CC-BY-SA](https://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-sa.svg)

State Vector Sync is an open source project licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/). See [LICENSE](/LICENSE) for more information.

Different licenses for the implementations might apply.
