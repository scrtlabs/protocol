.. _software-architecture:

Software architecture
---------------------
Overview
~~~~~~~~

The Enigma Network offloads private computation tasks from Ethereum.
These tasks are initiated by the end users of Ethereum Dapps. A JS
client library gives Dapp developers the tools to: 1) safely encrypt
sensitive data in-memory for immediate use or storage; 2) obtain an
authoritative proof that the target worker is securely running trusted
hardware (SGX) prior to sending data and paying fees.

Upon receiving a computation task, the Enigma contract deployed on-chain
broadcasts it in the Enigma network. Then, each registered worker runs a
random sampling lottery to determine whether they should execute the
task. This lottery is seeded with a random number generated using the
trusted hardware’s true randomness module. Each network participant may
independently run a pseudo-random algorithm to derive the selected
worker. This is how the Dapp knew which worker to verify.

.. image:: https://s3.amazonaws.com/enigmaco-docs/protocol/assigning-computation-task.png
    :align: center
    :alt: Giving Out Computation Task Overview

The selected worker then instructs its trusted hardware to unpack the
task, decrypt its arguments and delegate execution to its internal EVM.
After execution, a hash of the task attributes (i.e. inputs, code and
outputs) is signed with a private key which exists only in the trusted
hardware. This data is then committed on-chain where its provenance and
integrity are cryptographically verified. This on-chain verification
guarantees: 1) integrity of the task attributes; 2) safe execution
within trusted hardware; 3) execution by the selected worker. Finally,
the results are relayed to the Dapp contract and the worker collects its
fee.

.. image:: https://s3.amazonaws.com/enigmaco-docs/protocol/computation-task-overview.png
    :align: center
    :alt: Executing Computation Task Overview

Subsystem Decomposition
~~~~~~~~~~~~~~~~~~~~~~~

The system is comprised of the following subsystems. A subsystem is a
service provider that performs one function or many functions, but does
nothing until it is requested.

1. **Registration**: When a new node joins the network, it must register
      as a worker. The registration protocol ensures that the node runs
      on SGX hardware. It includes the node in a registry on-chain.

2. **Encryption / Storage**: Data flows privately between the Dapp and
      the Enigma Network using an Elliptic-curve Diffie–Hellman (ECDH)
      key agreement protocol. Encrypted data may be stored in the Dapp
      contract state for future computation.

3. **Worker Selection**: Multiple nodes participate in the network with
      the incentive of earning rewards by executing computation tasks as
      a worker. Worker selection happens for each computation tasks.
      Each node determine if they are the selected worker by running an
      algorithm which rely upon a shared random seed.

4. **Computation**: A computation task flows from the Dapp users to the
      secure enclave through multiple components. It executes a function
      of the Dapp smart contract with encrypted inputs.

5. **On-chain Verification:** The Enigma Contract verifies signed data
      submitted by a node in the Enigma Network.

6. **Attestation**: A Dapp may verify the authenticity of the selected
      worker through a remote attestation protocol prior to requesting
      or a computation task.

Logical Components
~~~~~~~~~~~~~~~~~~

The Enigma Network is not a standalone program. Its decentralized nature
and tight coupling with Ethereum increase complexity by requiring the
architecture to be molded according to highly opinionated existing
components. Here are the key components which compose the system.

1. **Enigma Library (EnigmaP.js):** A JavaScript library which performs
      the Enigma functions, to be included inside of Dapps.

2. **Dapp Contract:** A smart contract created by the Dapp author which
      stores encrypted data, the business logic of computation tasks and
      handles the callback.

3. **Enigma Contract:** A smart contract deployed on the Ethereum
      Network which orchestrates on-chain operations of the Enigma
      Network.

4. **Surface:** The untrusted component of an Enigma node which primary
      function is to coordinate computation task between the Enigma
      Contract and Core.

5. **Core:** The trusted component of an Enigma node which executes
      computation tasks. Core runs inside an SGX enclave.

6. **Principal Node:** A temporary centralized node which propagate
      random numbers to the rest of the network.

7. **Attestation Service:** A standalone service (not packaged in the
      local network) which verifies quotes with Intel.

The diagram below presents a composite view of the logical components of
the system.

.. image:: https://s3.amazonaws.com/enigmaco-docs/protocol/composite-structure.png
    :align: center
    :alt: Composite Structure of the System

Composite Structure of the System

Mapping Between Subsystems and Logical Components
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Given the complexity factor alluded to above, there is not a one-to-one
mapping between the subsystems and logical components, this table
illustrates in which component the core business logic of each subsystem
is implemented.

+----------------------+---------+---------------+--------------+---------+------+
|                      | Eng Lib | Dapp Contract | Eng Contract | Surface | Core |
+======================+=========+===============+==============+=========+======+
| Registration         |         |               | X            |         | X    |
+----------------------+---------+---------------+--------------+---------+------+
| Encryption / Storage | X       | X             |              |         | X    |
+----------------------+---------+---------------+--------------+---------+------+
| Computation          |         |               | X            | X       | X    |
+----------------------+---------+---------------+--------------+---------+------+
| Worker Selection     |         |               | X            | X       |      |
+----------------------+---------+---------------+--------------+---------+------+
| On-Chain Validation  |         |               | X            |         | X    |
+----------------------+---------+---------------+--------------+---------+------+
| Attestation          | X       |               | X            |         |      |
+----------------------+---------+---------------+--------------+---------+------+