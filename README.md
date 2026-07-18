# State Vector Sync

Specification and API description of the State Vector Sync (SVS) protocol.

[State Vector Sync (SVS)](https://named-data.net/wp-content/uploads/2021/07/ndn-0073-r2-SVS.pdf) is an NDN transport layer protocol that allows efficient multi-producer/multi-consumer communication. The communication participants maintain a shared data set and whenever a participant publishes data to the data set, the other participants are informed about the data generation and can choose to retrieve the published data.

In SVS, the data set state is synchronized using a state vector. Every participant publishes Data under its own producer prefix, and to distinguish subsequent publications, the publications are enumerated using a sequence number. The highest sequence numbers of every participant are exchanged among all communicating participants in a state vector. Whenever receiving a state vector, new publications from other participants can be inferred and retrieved using Interest-Data exchange. The process of data set synchronization is elaborated in more detail in the Protocol Specification below.

Several implementations of SVS are currently available:

- **C++**: [named-data/ndn-svs](https://github.com/named-data/ndn-svs)
- **TypeScript**: [@ndn/svs](https://ndnts-docs.ndn.today/typedoc/modules/_ndn_svs.html)
- **Golang**: [ndnd/std/sync](https://github.com/named-data/ndnd/blob/main/std/sync)
- **Python**: [justincpresley/ndn-python-svs](https://github.com/justincpresley/ndn-python-svs) (outdated)

## Protocol Specifications

For the detailed SVS protocol, see the [SVS Specification](Specification.md).
For the SVS-PS Publish/Subscribe protocol, see the [Pub/Sub Specification](PubSubSpec.md).

## Using State Vector Sync

Examples for using SVS can be found in the `examples` folder of the individual implementations.

### API Description

The API is unified across the official libraries. See [API](API.md) for a high-level description.
Please also refer to the individual library documentation for more details.

### Example Applications

- [Ownly](https://github.com/UCLA-IRL/ownly): A secure decentralized workspace built over NDN.
- [ndn-dv](https://github.com/named-data/ndnd): A distance vector routing protocol implementation.

## References

If you enjoyed using State Vector Sync, or used it for your research, we would appreciate a citation on the following publications:

- Philipp Moll, Varun Patil, Nishant Sabharwal, Lixia Zhang, *"A Brief Introduction to State Vector Sync"*, NDN Technical Report NDN-0073, Revision 2, July 2021. <https://named-data.net/publications/techreports/ndn-0073-r2-SVS/>
- Philipp Moll, Varun Patil, Lixia Zhang, Davide Pesavento, *"Resilient Brokerless Publish-Subscribe over NDN"*, 2021 IEEE Military Communications Conference (MILCOM). <https://doi.org/10.1109/MILCOM52596.2021.9652885>

## License

![CC-BY-SA](https://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by-sa.svg)

State Vector Sync is an open source project licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/). See [LICENSE](/LICENSE) for more information.

Different licenses for the implementations might apply.
