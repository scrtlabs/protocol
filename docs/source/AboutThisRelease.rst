About this release
==================
This initial testnet release prioritizes developer access, transparency,
and ease of use. We have included some essential features, and postponed
others or made temporary assumptions that are specific to this
development release. These choices are described at a high level in this
section. Help us improve by joining our developer forum and letting us
know what’s working, what’s not, and what you have questions about.

Selected features of this release:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Compatible with Solidity:** 
Executes any Solidity procedures which do not access the world state

2. **Randomly Selected Worker:** 
Each computation task is executed by a single randomly selected worker

3. **Multi-Node Network:** 
The network includes at least two nodes, all of which can be selected as a worker

4. **Token-based Payment:** 
Each worker receives a reward, equal to the ENG fee paid by the Dapp user, after successfully executing a computation

5. **Strong Privacy:** 
Sensitive data is never be exposed in clear to any intermediary or untrusted software -- local encryption by the dApp user tunneled through to secure hardware

6. **Independently Verifiable:**
dApp users are able to independently verify the enclave quotes with Intel

Assumptions made in this testnet release:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Static Keys / No Key Mgmt:** 
      Because of its complexity, the key management module will be deferred until the next release. This release will include a single encryption key pair and no key propagation. Doing this is only possible because this is a development release. Future release will propagate keys from a genesis node to other nodes.

2. **No Economic Incentives:** 
      This developer release focuses on the core components. Incentives will be added to the next release which will be deployed on the Ethereum testnet and mainnet.

3. **No Going Offline:** 
      In this release, once a node registers, the network assumes that it is always online and ready to receive tasks. Login and Logout operations will be included in the next release along with economic incentives based on staking. In subsequent releases, nodes will not be permitted to go offline without any economic penalty.

4. **No Bank:** 
      The payment mechanism is kept as simple as possible.
      Dapp users can pay any ENG fee when requesting a computation
      tasks. Workers will accept the proposed fee. The fee will be
      transferred to their wallet for each computation. The next release
      will contain a bank (deposit / withdrawal) along with a more
      sophisticated incentive layer.

5. **Principal Node:** 
      The network includes a Principal Node. This
      centralized component is used for the purpose of pushing random
      seeds to the network in order to randomly select workers. Using
      this component achieves true randomness while simplifying the
      architecture. In future releases, this component will be replaced
      by a completely decentralized scheme which will also achieve true
      randomness.

6. **Stateless EVM:** 
      During execution, SputnikVM cannot access the
      world state (i.e. account information, storage, or account’s
      code). These I/Os would require access to the Ethereum network. By
      design, SputnikVM can run without a connection to the Ethereum --
      a huge advantage over other EVM implementations. If the Solidity
      procedure requires access to Ethereum during execution, SputnikVM
      will raise an error and abort execution. This will change in
      future releases as smart contracts will be decoupled from Ethereum
      and the state will be stored in the Enigma Network.

7. **Centralized Software Attestation Service:** 
      Currently, the remote
      attestation protocol is bound by fairly restrictive rules which
      rely upon non-redistributable x509 certificates issued by Intel.
      For this release, requests to Intel’s attestation service are
      brokered by a gateway, hosted by Enigma, which injects the
      certificate issued by Intel to Enigma in the payload of each
      request. Future versions will allow each workers to request their
      own certificate and provide their own gateway to Intel’s
      attestation service.

8. **No Failed Tasks:** 
      We assume that all computation tasks will be
      successfully executed. If an error does occur, we return it to the
      console of the worker, but do not broadcast on the network. In
      future releases, computation errors will be tied with economic
      incentives.

9. **Single Preprocessors:** 
      We future releases will include a number of
      special functions (preprocessors) which extend the smart contract
      language to include features exclusive to the Enigma network, this
      version includes a single one: a random number generator.

On Rust
~~~~~~~

Using a new language like Rust is never an obvious choice especially
when C and C++ are available options. We feel that it is the best choice
for these reasons:

1. Compiles to machine instructions like C and C++. This gives us the granular level of control required.

2. Availability of the Rust SGX SDK. This SDK is maintained by Baidu. It includes many utility services built around best practices.

3. Availability of many high-quality Rust components for Ethereum, including SputnikVM, the Parity fork of WASM (which includes smart contracts) and many client libraries.

4. Designed as a more secure systems programming language (memory safety, no race conditions, etc). Security is paramount in blockchain applications.

5. Expressiveness of the language compared to C or C++ is expected to make up for the learning curve.

6. Interoperates with C when needed.

On SGX
~~~~~~

The Enigma Network uses Intel SGX enclaves because they provide strong
cryptographic guarantees. The following deductive statements explain why
the Enigma protocol can be trusted for privacy and correctness.

1. The attestation process provides verification for three things: the application’s identity, its intactness (that it has not been tampered with), and that it is running securely within an enclave on an Intel SGX enabled platform. [1]_

2. The signing key of an enclave never exists outside of it. It follows that data can only be signed with this key as part of the specified instruction set running in an enclave.

3. Computation tasks are signed in an enclave and verified on-chain. This guarantees integrity of all of their parameters: instructions, inputs and outputs. Since we know that all instructions and inputs are intact, outputs are necessarily correct.

4. The same guarantees apply to the principal node which generates a random seed. In addition, SGX only supports random number generators capable of true randomness.

These guarantees are critical. This is what allows the Enigma Protocol
to prove data privacy and correctness with minimal overhead (compared to
Ethereum for example). These guarantees offer enormous benefits both in
terms of scalability and privacy.

On Coupling with Ethereum
~~~~~~~~~~~~~~~~~~~~~~~~~

In this release the Enigma Network is tightly coupled with Ethereum in
multiple ways.

1. The Enigma Network shares many key characteristics with Oracles [2]_, including a similar pattern of asynchronous data exchange

2. The business logic of each computation task is included in dApp smart contracts deployed on Ethereum

3. The Enigma Network has no internal state, it must report each computation tasks to chain in order to update the state

4. Computation tasks are written in Solidity and executed in a standalone Ethereum Virtual Machine

5. Nodes of the Enigma Network cannot communicate with each other without going through the Ethereum chain

This strategic coupling allows us to deliver the Enigma Network in
planned phases without compromising on critical attributes like safety
of the funds. This release is the most tightly coupled with Ethereum.
Future releases will incrementally loosen this coupling by introducing
features (internal state, independent smart contracts, peer-to-peer data
exchange, etc).
