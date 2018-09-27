Technical Introduction
======================
This page contains an introductory overview of the Enigma protocol and this 
Docker testnet release aimed at experienced dApp developers. More 
thorough and indepth information can be found in the various architecture,
topology and cryptography related pages in the documentation index.

How Enigma Works
~~~~~~~~~~~~~~~~

The Enigma network provides a permissionless peer-to-peer network that
facilitates the execution of code (*secret contracts*) with strong 
correctness and privacy guarantees, similar to smart-contract platforms 
such as Ethereum - the key difference in Enigma is that the data itself 
is concealed from the nodes that execute computations. This allows
developers to include sensitive data in their smart contracts without 
moving off-chain to centralized (and less secure) systems, thus allowing
for truly private and scalable decentralized applications.

Secret contracts operate by being executed in a retrofitted EVM running 
inside a Trusted Execution Environment (TEE), based on Intel’s SGX technology. 
This supports out-of-the-box interoperability with the Ethereum network as 
well as Solidity.

The Enigma Network offloads private computation tasks initiated by end-users
of Ethereum dApps. These tasks are then handed by the Enigma-JS client
library, which encrypts sensitive data in memory for immediate use or 
storage. as well as obtain authoritative proof that their selected
worker will be securely running trusted hardware (Intel SGX) before sending 
any actual data or payment.
 
The Enigma Contract deployed on-chain broadcasts the task in the Enigma 
Network and performs a random sampling lottery to determine which node 
should execute it. The selected worker then instructs its trusted hardware 
to unpack the task, decrypt its arguments and delegate execution to its 
internal EVM (Ethereum Virtual Machine). After execution of the computation, 
a hash of the task is signed by the hardware and is then committed on-chain, 
where its integrity is cryptographically verified.

The Enigma Testnet - What's Inside?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This initial testnet release is a highly simplified containerized Docker release 
which includes every core component of a functional Enigma Network, including 
nodes and the blockchain itself. Using this, dApp developers will be able to 
get hands-on with the code and learn how to deploy secret contracts as well 
as gain a thorough understanding of how the various components of Enigmas 
functionality interconnect. 


For a visual of the Enigma Docker Network topology, see 
:doc:`this page <NetworkTopology>`.
