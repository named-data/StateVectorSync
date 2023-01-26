# State Vector Sync

Specification and API description of the State Vector Sync (SVS) protocol.

[State Vector Sync (SVS)](https://named-data.net/wp-content/uploads/2021/07/ndn-0073-r2-SVS.pdf) is an NDN transport layer protocol that allows efficient multi-producer/multi-consumer communication. The communication participants maintain a shared data set and whenever a participant publishes data to the data set, the other participants are informed about the data generation and can choose to retrieve the published data.

In SVS, the data set state is synchronized using a state vector. Every participant publishes Data under its own producer prefix, and to distinguish subsequent publications, the publications are enumerated using a sequence number. The highest sequence numbers of every participant are exchanged among all communicating participants in a state vector. Whenever receiving a state vector, new publications from other participants can be inferred and retrieved using Interest-Data exchange. The process of data set synchronization is elaborated in more detail in the Protocol Specification below.

Currently, there are three implementations of SVS available:

- **C++**: [named-data/ndn-svs](https://github.com/named-data/ndn-svs)
- **TypeScript**: [@ndn/sync](https://github.com/yoursunny/NDNts/tree/main/packages/sync)
- **Python**: [justincpresley/ndn-python-svs](https://github.com/justincpresley/ndn-python-svs)

## Using State Vector Sync

Examples for using SVS can be found in the `examples` folder of the individual implementations. If you enjoyed using State Vector Sync, or used it for your research, we'd appreciate a citation on the following publication:

Philipp Moll, Varun Patil, Nishant Sabharwal, Lixia Zhang, *"A Brief Introduction to State Vector Sync"*, NDN Technical Report NDN-0073, Revision 2, July 2021. <https://named-data.net/wp-content/uploads/2021/07/ndn-0073-r2-SVS.pdf>

## Protocol Specification

For the detailed protocol specification, please see the [Specification](Specification.md). For the specification of the SVS-PS Pub/Sub protocol, see the [PubSub Specification](PubSubSpec.md).

## API Description

The API is unified across the official libraries. Please find a detailed API description in [API](API.md).

## Example Applications

- [SVChat](https://github.com/pulsejet/svchat): An Angular-based chat application featuring the TypeScript version of SVS for group communication.

## License

![CC-BY-SA](https://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-sa.svg)

State Vector Sync is an open source project licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/). See [LICENSE](/LICENSE) for more information.

Different licenses for the implementations might apply.
