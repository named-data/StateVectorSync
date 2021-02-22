# StateVectorSync
Spec and API description of the StateVectorSync (SVS) protocol

StateVectorSync (SVS) is an NDN transport layer protocol that allows efficient multi-producer/multi-consumer communication. The communication participants maintain a shared data set and whenever a participant publishes data to the data set, the other participants are informed about the data generation and can choose to retrieve the published data.

In SVS, the data set state is synchronized using a state vector. Every participant publishes Data under its own producer prefix, and to distinguish subsequent publications, the publications are enumerated using a sequence number. The highest sequence numbers of every participant are exchanged among all communicating participants in a state vector. Whenever receiving a state vector, new publications from other participants can be inferred and retrieved using Interest-Data exchange. The process of data set synchronization is elaborated in more detail in the Protocol Specification below.

Currently, there are three implementations of SVS available:

- C++: [named-data/ndn-svs](https://github.com/named-data/ndn-svs)
- TypeScript: [pulsejet/ndnts-svs](https://github.com/pulsejet/ndnts-svs)
- Python: [justincpresley/ndn-python-svs](https://github.com/justincpresley/ndn-python-svs) (in progress)

## Using State Vector Sync

Examples for using SVS can be found in the `./examples` folder of the individual implementations. If you'd enjoyed using StateVectorSync, or used it for your research, we'd appreciate a citations on the following publication:

Li, T., Kong, Z., Mastorakis, S., & Zhang, L. (2019). Distributed Dataset Synchronization in Disruptive Networks. 16th IEEE International Conference on Mobile Ad-Hoc and Smart Systems (IEEE MASS), November, 10. https://doi.org/10.1109/MASS.2019.00057

# Protocol Specification

For the detailed protocol specification, please see the [Specification.md](./Specification.md).

# API Description

The API is unified across the official libraries. Please find a detailed API description in [API.md](./API.md).

# Example Applications

- [ChronoChat](https://github.com/named-data/chronochat): A C++ chat client featuring the C++ version of SVS for group communication
- [SVChat](https://github.com/pulsejet/svchat): An Angular-based chat application featuring the TypeScript version of SVS for group communication.

# License
StateVectorSync is an open source project licensed under the LGPL version 2.1. See [LICENSE](./LICENSE) for more information.
