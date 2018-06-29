Introduction
============

The Enigma network provides a permissionless peer-to-peer network that
allows executing code (*secret contracts*) with strong correctness and
privacy guarantees. Another way to view the network is as a
smart-contract platform (e.g., similar to Ethereum) that enables the
development of decentralized applications (dApps), but with the key
difference that the data itself is concealed from the nodes that execute
computations. This enables dApp developers to include sensitive data in
their smart contracts, without moving off-chain to centralized (and less
secure) systems.

The initial testnet release is a self-contained Docker network. It
composed of multiple containers which fully encapsulate the dynamics of
a distributed version of the same topology (including a local Ethereum
blockchain). This release makes available multiple core components of
the Enigma protocol, and is highly simplified.

The goal of this simplified release is to familiarize developers with
our :doc:`software architecture<SoftwareArchitecture>`,
**secret contract development**, and is in
line of our goal of **open-source development**. This convenient
approach allows any developer to get started with limited friction.
Future releases will be deployed on Ethereum testnet and mainet.