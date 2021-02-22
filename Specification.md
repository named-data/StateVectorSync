# StateVectorSync Protocol Specification

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

**Sync Interest Format**: `/<group-prefix>/<version-vector>/<signature>`

A state vector is appended to the name in TLV format:

Interest Lifetime: 1s

**Data Interest Format**: `/<node-prefix>/<group-prefix>/<seq-num>`

Note: _Choosing alternative Data Interest formats may be decided on application-level._

## 3. State Vector TLV Specification

```
StateVector = VERSION-VECTOR-TYPE TLV-LENGTH
              *StateVectorComponent

StateVectorComponent = NodeID SeqNo
NodeID = VERSION-VECTOR-KEY-TYPE TLV-LENGTH *OCTET
SeqNo = VERSION-VECTOR-VALUE-TYPE TLV-LENGTH NonNegativeInteger
VERSION-VECTOR-TYPE = 201
VERSION-VECTOR-KEY-TYPE = 202
VERSION-VECTOR-VALUE-TYPE = 203
```

- The encoded version vector in the interest consists of the NodeID of each node followed by its latest sequence number
- Node names in the encoded version vector are ordered lexicographically to allow for interest aggregation.
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

- Sync Interests are signed using signed interest V3
  - The last name component of the Sync Interest is the signature
- The application may choose to use either HMAC or asymmetric signatures
  - Exchange of symmetric key (HMAC) is beyond the scope of the sync protocol
  - All nodes must maintain the list of trusted publishers when using asymmetric signatures
    - This mechanism is beyond the scope of the sync protocol
  - Note: Interest aggregation cannot function when using asymmetric signatures

# License
StateVectorSync is an open source project licensed under the LGPL version 2.1. See [LICENSE](./LICENSE) for more information.
