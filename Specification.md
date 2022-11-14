# State Vector Sync Protocol Specification

This page describes the protocol specification of [State Vector Sync](README.md).

**Last update to specification**: 2021-12-15

## 1. Basic Protocol Design

```
Node A       Node B      Node C
seq=10       seq=15      seq=24
   |            |           |          \
   |            |           |           \
                                        |
  <-------------------------->          |
     Periodic Sync Interest             | Steady
[/node-a/:10,/node-b/:15,/node-c/:24]   | State
                                        |
   |            |           |           |
   |            |           |           /
   |            |           |          /

Node A publishes Data                  \
seq=11                                  \
                                        |
  <-------------------------->          | Data Set State
           Sync Interest                | Reconciliation
[/node-a/:11,/node-b/:15,/node-c/:24]   |
                                        /
   |                        |          /
   |                        |          \
   |                        |           \
   |      Data Interest     |           |
   | <----------<---------- |           | Publication
   |       Publication      |           | Retrieval
   | ---------------------> |           |
   |                        |           /
   |                        |          /

```

## 2. Format and Naming

**Sync Interest Format**: `/<group-prefix>/<state-vector>/<signature>`

A state vector is appended to the name in TLV format:

Interest Lifetime: 1s

**Data Interest Format**: `/<node-prefix>/<group-prefix>/<seq-num>`

Note: _Choosing alternative Data Interest formats may be decided on application-level._

## 3. State Vector TLV Specification

```
StateVector = STATE-VECTOR-TYPE TLV-LENGTH
              *StateVectorEntry

StateVectorEntry = STATE-VECTOR-ENTRY-TYPE TLV-LENGTH
                   NodeID SeqNo

NodeID = Name
SeqNo = SEQ-NO-TYPE TLV-LENGTH NonNegativeInteger

STATE-VECTOR-TYPE = 201
STATE-VECTOR-ENTRY-TYPE = 202
SEQ-NO-TYPE = 204
```

- The encoded state vector in the Interest consists of State Vector Entries
- Each entry is a tuple of the NodeID of each node followed by its latest sequence number
- Node names in the encoded version vector are ordered in [NDN canonical order](https://named-data.net/doc/NDN-packet-spec/0.3/name.html#canonical-order) to allow for Interest aggregation.
- Definition: _A State Vector A is outdated to State Vector B, if A contains any entry with seq number strictly smaller than in B._

## 4. State Sync

### 4.1 Sync Interests are sent periodically

- Maintain a Sync Interest Timer (30 seconds, random ±10%)
- Decide whether to send a Sync Interest upon timeout

### 4.2 Send Sync Interests on new publication

- When the node generates a new publication, immediately emit a Sync Interest, reset the Sync Interest Timer.

### 4.3 Sync Ack Policy - Do not acknowledge Sync Interests

- Reason: Sending Sync Acks from multiple nodes result in unsolicited data. (the first one is delivered only, others are dropped)

### 4.4 Handling incoming Sync Interests

Nodes can either be in _Steady State_, or in _Suppression State_

- _Steady State_: The sync group is in sync. Incoming Sync Interests carry the latest known state.
- _Suppression State_: Incoming Sync Interests indicate a state inconsistency. The node tries reconciling the inconsistency by emitting a up-to-date Sync Interest. Use a suppression timer before sending to prevent flooding.

When a node is in _Steady State_:
- Incoming Sync Interest is up-to-date or newer.
  - No indication of inconsistencies. The scheduled Sync Interest can be delayed.
  - Eventually update the local state and reset Sync Interest Timer to 30 seconds (±10% uniform)
- Incoming Sync Interest is outdated: Node moves to _Suppression State_
  - Set Sync Interest Timer to 200ms (±50% uniform) - Time represents the suppression interval
  - Aggregate the state of consequent incoming Sync Interests in a separate state vector
  - On expiration of timer:
    - If aggregated received state vector is up-to-date:\
      No inconsistency - Reset Sync Interest Timer to 30 seconds (±10% uniform)
    - If aggregated received state vector is outdated:\
      Inconsistent State: Emit up-to-date Sync Interest. Reset Sync Interest Timer to 30 seconds (±10%)\
      Node moves to _Steady State_

When a node is in _Supression State_:
- Only aggregate state vector of incoming Sync Interests. No further action in _Suppression State_.

## 5 Examples

### 5.1 State Sync - Example without loss

Sync Group with 3 participants, node A, B, and C
- Data set state: [A=10, B=15, C=25], all nodes are in Sync
- Node A publishes new publication.
- A sends Sync Interests with [A=11, B=15, C=25]
- B and C receive Sync Interest and update their local states accordingly.
- _Consistent state is re-established_

### 5.2 State Sync - Example **with** packet loss

Sync Group with 3 participants, node A, B, and C

- Data set state: [A=10, B=15, C=25], all nodes are in Sync
- Node A publishes new publication.
- A sends Sync Interests with [A=11, B=15, C=25]
- B receives Sync Interest, but Interest does not reach C
- **Inconsistent state - C’s state is outdated.**
- C sends periodic Heartbeat Interest with [**A=10**, B=15, C=25]
  - A and B receive C’s outdated Sync Interest.
  - A and B set suppression timer.
  - A’s timer expires and sends up-to-date Sync Interest.
- B receives A’s Sync Interest during suppression interval. On suppression timeout, B suppresses the scheduled Sync Interest.
- C also receives A’s Sync Interest and updates the state accordingly.
- _Consistent state is re-established_

## 6 SVS State Machine

![SVS State Machine](./img/svs-state-machine.jpg)

## 7 Interest Authentication

- Sync Interests are signed using [signed interest V3](https://named-data.net/doc/NDN-packet-spec/0.3/signed-interest.html)
- All nodes must maintain the list of trusted publishers when using asymmetric signatures
  - This mechanism is beyond the scope of the Sync protocol
- Note: Interest aggregation cannot function when using asymmetric signatures

## License

State Vector Sync is an open source project licensed under the CC-BY-SA 4.0. See [LICENSE](LICENSE) for more information.

![CC-BY-SA](https://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-sa.svg)

Different licenses for the implementations might apply.
