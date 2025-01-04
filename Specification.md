# State Vector Sync Protocol Specification

This page describes the protocol specification of [State Vector Sync (SVS)](/README.md) Version 2.

_Last update to specification: 2025-01-04_

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

**Sync Interest Name**: `/<group-prefix>/v=2/<parameters-digest>`

The State Vector is encoded in TLV format and included in the `ApplicationParameters` of the Sync Interest.
The State Vector SHOULD be the first TLV block in the `ApplicationParameters`.
The Interest Lifetime for Sync Interests is 1 second.

**Data Interest Name**: `/<node-prefix>/<group-prefix>/<seq-num>`

_Note:_ Choosing alternative Data Interest formats may be decided on application-level.

## 3. State Vector TLV Specification

```abnf
StateVector = STATE-VECTOR-TYPE TLV-LENGTH
              *StateVectorEntry

StateVectorEntry = STATE-VECTOR-ENTRY-TYPE TLV-LENGTH
                   NodeID
                   SeqNo

NodeID = Name
SeqNo = SEQ-NO-TYPE TLV-LENGTH NonNegativeInteger

STATE-VECTOR-TYPE = 201
STATE-VECTOR-ENTRY-TYPE = 202
SEQ-NO-TYPE = 204
```

- The encoded state vector in the Interest consists of State Vector Entries
- Each entry is a tuple of the NodeID of each node followed by its latest sequence number
- The sequence number is 1-indexed, i.e. the first valid sequence number in a state vector is 1
- If an entry is not present in the state vector, it is considered as 0 for any calculations.
- Node names in the encoded version vector are ordered in [NDN canonical order](https://docs.named-data.net/NDN-packet-spec/0.3/name.html#canonical-order) to allow for Interest aggregation.
- Definition: _A State Vector A is outdated to State Vector B, if A contains any entry with seq number strictly smaller than in B._

## 4. State Sync

### 4.1 Sync Interest Timer

SVS utilizes a single Sync Interest timer.
It can take on one of the two timeout values that are used by the protocol.
The values used by the timer are as follows.

- `PeriodicTimeout`: defaults to 30 seconds (±10% uniform)
- `SuppressionPeriod`: defaults to 200ms
- `SuppressionTimeout`: random value between 0 to `SuppressionPeriod`. \
  An exponential decay function SHOULD be used for the timeout value.
  ```
  c = SuppressionPeriod  // constant factor
  v = random(0, c)       // uniform random value
  f = 10.0               // decay factor
  SuppressionTimeout = c * (1 - e^((v - c) / (c / f)))
  ```

### 4.2 Send Sync Interests on new publication

- When the node generates a new publication, immediately emit a
  Sync Interest, and reset the Sync Interest Timer to `PeriodicTimeout`.

### 4.3 Sync Ack Policy - Do not acknowledge Sync Interests

- Reason: sending Sync Acks from multiple nodes result in unsolicited data.\
  (only the first one is delivered, others are dropped)

### 4.5 Sync Interest Processing and Timer Expiry

Nodes can either be in _Steady State_, or in _Suppression State_

- _Steady State_: The Sync group is synchronized.
  Incoming Sync Interests carry the latest known state.
- _Suppression State_: Incoming Sync Interests indicate a state inconsistency.
  The node tries to reconcile the inconsistency by emitting a up-to-date Sync Interest.
  Use a suppression timer before sending to prevent flooding.

**Steady State**

- When entering _Steady State_, reset the Sync Interest timer to `PeriodicTimeout`

- Incoming Sync Interest is up-to-date or newer.
  1. If the incoming state vector is newer, update the local state vector. \
     Store the current timestamp as the last update time for each updated node.
  1. Reset Sync Interest timer to `PeriodicTimeout`.

- Incoming Sync Interest is outdated.
  1. If every node with an outdated sequence number in the incoming state vector
    was updated in the last `SuppressionPeriod`, drop the Sync Interest.
  1. Otherwise, move to _Suppression State_

- On expiration of timer:
  1. Emit a Sync Interest with the current local state vector.
  1. Reset Sync Interest timer to `PeriodicTimeout`.

**Suppression State**

- When entering Suppression State_, reset the Sync Interest timer to `SuppressionTimeout`

- For every incoming Sync Interest:
  1. Update the local state vector with any newer sequence numbers.
  1. Aggregate the state vector into a `MergedStateVector`.

- On expiration of timer:
  1. If `MergedStateVector` is up-to-date; no inconsistency.
  1. If `MergedStateVector` is outdated; inconsistent state.\
     Emit up-to-date Sync Interest.
  1. Move to _Steady State_.

## 5. Examples

### 5.1 State Sync - Example without packet loss

Sync Group with 3 participants, node A, B, and C.

- Data set state: [A=10, B=15, C=25], all nodes are in Sync
- Node A publishes new publication.
- A sends Sync Interests with [A=11, B=15, C=25]
- B and C receive Sync Interest and update their local states accordingly.
- _Consistent state is re-established_

### 5.2 State Sync - Example with packet loss

Sync Group with 3 participants, node A, B, and C.

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

## 6. SVS State Machine

![SVS State Machine](img/svs-state-machine.jpg)

## 7. Interest Authentication

- Sync Interests are signed using the [Signed Interest v0.3](https://docs.named-data.net/NDN-packet-spec/0.3/signed-interest.html) format
- All nodes must maintain the list of trusted publishers when using asymmetric signatures
  - This mechanism is beyond the scope of the Sync protocol
- _Note:_ Interest aggregation cannot function when using asymmetric signatures

## License

![CC-BY-SA](https://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-sa.svg)

State Vector Sync is an open source project licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/). See [LICENSE](/LICENSE) for more information.

Different licenses for the implementations might apply.
