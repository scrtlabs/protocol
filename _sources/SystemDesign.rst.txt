System design
==============
This section gives an overview of important components of our testnet
release for developers. It discusses each component at a high level:
more in-depth discussion can be found in System Architecture and
Subsystem Architecture sections.

The core of the Enigma network is the ability to perform private computations.
A private computation task is a function that is run over data that is not seen by any node. These tasks are initiated by the end-users of Ethereum dApps, then offloaded to the Enigma network via the Enigma Contract. The Enigma network selects workers running Enigma nodes to perform the computations, and then returns the result to the ethereum dApp contract. Encryption of the data happens on the client side, before data is submitted to the ethereum dApp.

To create a private computation task, developers use the Enigma-JS
client library.


Enigma-JS Client Library
~~~~~~~~~~~~~~~~~~~~~~~~

The Enigma client library, `Enigma-JS <#_ydldonl0i1f1>`__, is a
javascript library that interfaces with the Enigma protocol. The API for
this library describes tasks that a dApp will require to interact with
Enigma, such as encryption and verification. Enigma-JS contains tools
to: 1) safely encrypt sensitive data in-memory for immediate use or
storage; 2) obtain an authoritative proof that the target worker is
securely running trusted hardware prior to sending data and paying fees.
(Trusted hardware for this release means Intel SGX: for more information
on SGX, see sections On SGX and Registration)

.. _enigma-contract:

Enigma Contract
~~~~~~~~~~~~~~~

The Enigma contract primarily contains logic to secure the network. To do this, it has a list of registered worker nodes. It receives computation task requests from Dapps, and broadcasts them to the Enigma Network. It can also verify the integrity of the results submitted by each worker. The sections below describe the key characteristics of the Enigma contract.

Tracking of Computation Tasks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The computation tasks are stored in a mapping using the Task struct.
Each active task item contains the computation fee (in ENG) paid by the
Dapp user.

Events
^^^^^^

The following lists breaks down the key events emitted by the contract.

1. **ComputeTask**: 
Gives workers the parameters of a computation task

   a. **callable:** A variation of method signature (see Ethereum ABI [3]_) of a public function of the Dapp contract which contains the computation logic (callable). The only variation is an additional prefix which flags encrypted values. The type of each encrypted argument must have a *“_”* prefix in the signature. For example, if the first argument of *baz(uint32,bool)* is encrypted, the signature becomes *baz(_uint32,bool)*.
   b. **callableArgs:** The encrypted argument value(s) of the callable function. These inputs must be encrypted a priori (see Client section below). They cannot be provided in the clear. Their original types will be inferred from the callable function definition. In the Coin Mixing example, it is an array of encrypted destination addresses.
   c. **callback:** The method signature (see Ethereum ABI) of a public function of the Dapp contract which the worker calls when committing the results.
   d. **preprocessors**: A list of well-known preprocessor functions which will inject calculated arguments at runtime (when the callable function runs in the EVM). For example, a “random” pre-processor might inject an array of random integers for shuffling.
   e. **fee**: The computation fee (in ENG) to be in held in escrow until the computation is completed.

2. **WorkersParameterized**: 
Gives workers the parameters that are required for selection
   f. **seed**: The random sampling seed effective in the current block.
   g. **activeWorkers**: An ordered list of addresses of all registered workers

Principal Node
~~~~~~~~~~~~~~

The principal node is a temporary centralized node which performs two
key functions: 1) Generate random seeds for worker selection; 2)
Propagate the encryption key to other nodes joining the network. The
principal node exists in this developer release only. Using this
component achieves true randomness while simplifying the architecture.
In future releases, this component will be replaced by a completely
decentralized scheme which will also achieve true randomness.

Enigma Node
~~~~~~~~~~~

An Enigma Node is composed of two components: Surface and Core.

.. image:: https://s3.amazonaws.com/enigmaco-docs/protocol/surface-to-core.png
    :align: center
    :alt: Surface to Core Diagram

Surface
~~~~~~~

Surface is the untrusted component of an Enigma node which has the
primary function of coordinating computation tasks between the Enigma
Contract and Core. Surface is involved in worker selection and
computation tasks. For the tasks in each process, the Enigma network has
verification protocols that check correct execution. For information on
Surface’s role in the registration process, see
`Registration <#registration>`__. For more details on how Surface is
used in the computation process, see `Computation <#computation>`__.

Core
~~~~

Core is the trusted component of the Enigma network, and executes
computation tasks. Core runs inside of an `SGX enclave <#on-sgx>`__.
Core is involved in a number of processes, including registration,
encryption, computation and validation. Core is responsible for the
decryption of data, processing of computations, and returning results to
the Enigma function. The Enigma function is able to verify that each of
these tasks was executed correctly with Core. For more information about
these processes, see `Computation <#computation>`__,
`Registration <#registration>`__, and `Attestation <#attestation>`__
sections.