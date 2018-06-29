Subsystem architecture
----------------------

Subsystems are not the same as components. Subsystem may go across
components to perform their function. A subsystem is a service provider
that performs one function or many functions, but does nothing until it
is requested. This section describes the data flow of each subsystem in
enough depth to fully illustrate how the system works.

Registration
~~~~~~~~~~~~

First, for a node to become a worker eligible to run computations, it
must first securely generate an Ethereum compliant ECC key-pair to be
used as a persistent identity. This key-pair is generated inside an SGX
enclave, and should never leave it. To persist it across sessions, we
will seal the key in the host’s system.

Once the key is generated, the enclave should generate a quote proving
that the key-pair was generated properly inside the enclave. The enclave
should then create an produce and sign a quote. The quote is then
verified with Intel and sent on-chain.

Now, everyone (whether it’s a user or some other stakeholder), can
independently verify that this node’s identity is linked to an enclave.
When Dapp users request a computation task, they can run through the
list on-chain and verify that all workers are legitimate, and cache a
whitelist of these. Workers will be able to perform the same
verification in future releases.

|image5|

The registration protocol is defined as follows:

1. The worker requests a quote from the enclave.

2. The enclave generates a proving key-pair from which it produces and
      signs a quote. Then, Surface extracts the public key and the
      quote.

3. Surface generates an address by hashing the public key. This is
      required in order to use the *ECRecover* method of Ethereum
      on-chain.

4. Surface calls Enigma’s Attestation service to verify the quote. This
      service internally stores an SPID certificate. It passes the SPID
      and the quote to Intel’s Remote Attestation Server to request a
      report which formally verifies the quote (see
      `Attestation <#attestation>`__ for the verification flow of this
      report).

5. Surface receives the report, then passes it along to the *register*
      function on the Enigma Contract with the address and the quote.
      The contract stores this data in a mapping where the key is the
      custodian address (i.e. the *msg.sender* of the transaction).

6. From here-on, nodes will know that the *prover address*, which never
      leaves the enclave, is used to prove computations; whereas the
      *custodian address* is in charge of receiving payouts.

Client Encryption and Storage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At a high-level, the protocol ensures that data parameters can flow
securely from a Dapp to a secure enclave through a Elliptic-curve
Diffie–Hellman (ECDH), an anonymous key agreement protocol. This scheme
allows both parties, each having an elliptic-curve public–private key
pair, to establish a shared secret in an untrusted area.

Optionally, encrypted data can be stored in the state of a Dapp smart
contract on Ethereum. The Enigma Protocol ensures that an enclave can
decrypt this data in context of a future computation task.

|image6|

In the encryption and storage protocol, functions of the `Enigma
Library <#enigma-js-client-library>`__ help executing these
instructions:

1. Request the application key pair from the Dapp config. In this
      development release, there is a single app key for all
      applications. In the future releases, each application will
      generate its own key pair.

2. Request an encryption public key from the Dapp config. Again, since
      this is a development release, the encryption key is simply a
      configuration parameter. All nodes in the network currently use
      the same encryption key pair. In future releases, encryption keys
      will be provided by the network. A key management protocol will
      control the lifecycle of the keys.

3. Derive a new key from the Dapp private key and the encryption key.
      Encrypting with this derived key will ensure that the selected
      enclave can decrypt the message, while proving that the message
      was encrypted by the Dapp (and nothing else).

4. Generate a random initialization vector (IV) [4]_, then use it to
      encrypt the message with the derived key. The IV will be
      concatenated at the beginning of the encrypted message. The IV and
      message can later be easily extracted because the IV always
      contain 12 bytes.

5. Depending on the use case, the encrypted message may or may not need
      to be stored in the world state. If the purpose of the message is
      to immediately serve as input to a transaction (and the
      transaction does not require any inputs stored on-chain), it can
      be sent directly to the Enigma Contract by requesting a
      computation task. As a general rule:

   a. If the encrypted message serves as input to an immediate
         transaction which does not have any other inputs stored
         encrypted in state, invoke to the Enigma Contract using web3.

   b. If the encrypted message must be stored for later or serve as an
         input to an immediate transaction along with other inputs
         stored encrypted in the state, use the Dapp contract to broker
         the transaction.

6. If the message must be used as input for future computation tasks, it
      can be stored as an attribute of the Dapp contract. In this case,
      the Dapp can use web3 to invoke a transaction function --
      *foo(encryptedData)* in the diagram. We assume that *foo* contains
      the instructions required to store the encrypted data in the Dapp
      contract.

This `Cryptography
Document <https://docs.google.com/document/d/1c9eReGipyBO7l82-n7U8AH8tSXyZeN9ZDzIJgTbVKSI/edit#heading=h.h4mmyxajdhy7>`__
describes the specific curves used for encryption and other cryptography
related considerations.

Worker Selection
~~~~~~~~~~~~~~~~

Sampling workers means that the entire Enigma network needs to reach an
agreement about the identity of a given worker (or one or more groups of
workers) at a given time. This sampling process happens once in every
period, known as *epoch*. The length of each epoch corresponds to a
configurable number of blocks.

|image7|

The protocol for selecting a worker does the following for each epoch:

1.  The principal node uses SGX’s *true random* (i.e. sgx_read_rand) to
       generate a fresh random value (256-bit), which will later be used
       by the nodes as a *seed*.

2.  It passes this value to the untrusted peer app running on the
       principal’s host. The untrusted peer then commits it to the
       Enigma Contract.

3.  The Enigma Contract stores a mapping of: 1) the current block
       number; 2) the seed; 3) a an ordered list representing a snapshot
       of all active workers.

4.  The Enigma Contract emits a *WorkersParameterized* event. Every node
       in the network can observe this value, as the are all watching
       the chain.

5.  Now, every node can independently run a pseudo-randomness algorithm
       which selects the winning worker’s address for each computation
       task.

6.  When the contract receives a compute request, it generates a taskId
       (see `Client Encryption and
       Storage <#client-encryption-and-storage>`__). Then, it emits a
       ComputeTask event (see `Computation <#computation>`__).

7.  Upon receiving a computation task, each worker run a
       pseudo-randomness algorithm to discover the selected worker. The
       input of the *selectWorker* function are: the seed; the taskId
       and the list of workers. Including the taskId ensures that a
       different worker is randomly selected for each computation task.

8.  Now, all nodes in the network know the address of the worker
       selected for the task. Only the selected worker executed the
       computation task.

9.  The selected worker commits the results on-chain including the block
       number which originated the task.

10. The Enigma Contract retrieves the worker selection parameters
       corresponding to the block number submitted.

11. The Enigma Contract re-runs the *selectWorker* pseudo-randomness
       algorithm to verify that the worker submitting the results is
       indeed the selected worker for the task. A greedy worker trying
       to compute more than its share of tasks would simply waste gas,
       as the unauthorized submissions get rejected by the this
       verification method.

Random sampling is one of the most important primitives in the network.
While in later versions, this would be achieved by a distributed MPC
algorithm, for Discovery it suffices to have a *principal* Enigma node
that generates this kind of randomness.

.. _section-1:

Computation 
~~~~~~~~~~~~

| When a worker executes a computation and signs its view (namely -
  H(input, code, output)) with his key, the user can be confident that
  these computations finished successfully – assuming the enclave is
  limited to only run computations inside the EVM and sign them. This is
  illustrated below.
| |image8|

This diagram assumes that *callableArgs* have been encrypted using the
`Client Encryption and Storage <#_rbm5765cidly>`__ subsystem described
above.

The computation protocol works as follows:

1. The Dapp users requests a computation tasks in one of the following
      ways. The choice usually depends on whether the Dapp stores
      encrypted values in the state of its contract.

   a. Directly from the Enigma Contract by using web3 to invoke the
         *compute* function.

   b. By invoking a function of the Dapp Contract which wraps the
         *compute* function of the Enigma Contract.

2. The Enigma Contract locks the fee (more details below)

3. The Enigma Contract emits a *ComputeTask* event. All nodes in the
      network will receive the event as they constantly monitor the
      chain.

4. Surface receives a task and runs the lottery to determine if it
      should execute the task (more details in `Worker
      Selection <#_7od30zs65dcs>`__).

5. If selected, Surface extracts the bytecode of the specified
      *dappContractAddress* and relays the call to Core.

6. Core executes the computation which involves the following steps:

   c. Deserialize and decrypt the encrypted arguments (some arguments
         may not be encrypted)

   d. Run the preprocessors if any. Inject the preprocessor outputs as
         additional arguments of the computation function.

   e. Gather the bytecode with all inputs and pass them to SputnikVM
         which will run the specified function of the secret contract.

   f. Sign a hash of the original callableArgs, outputs and bytecode
         using the enclave private key.

7. Surface receives the outputs and signature from Core. It relays them
      to the Enigma Contract along with the originating blockNumber,
      secretContract address and taskId using the *commitResults*
      function.

8. The Enigma contract verifies that the worker submitting the results
      1) is the worker selected for the task; 2) did not tamper with the
      inputs; 3) computed the task in a secure enclave. This
      verification protocol is composed of the following steps.

   g. With the workers parameters of the block originating the task, run
         the pseudo-random worker selection algorithm. This ensures that
         the worker committing the results is the worker selected by the
         network.

   h. Compute a hash function with the task parameters stored prior to
         broadcasting the task to the network -- which never left the
         contract so could not have been tampered with -- and the
         results submitted by the worker.

   i. Compute Ethereum’s *ECRecover*\  [5]_ function with the hash and
         the submitted signature. For a successful verification, this
         should return the signer address of the worker.

Payment of the Computation Fee
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Computation fees (tokens) flow from Dapp users to workers as follows:

1. The Dapp user calls the *approve* function of the ENG ERC20 contract
      to unlock a discretionary ENG payment for computing the task.

2. The Dapp user calls a payable function the Dapp contract which wraps
      the *compute()* function (or the Enigma Contract directly as
      illustrated in the diagram).

3. The Engima Contract locks the fee in a mapping for which the key is
      the *taskId*.

4. A worker is randomly selected to perform the task. In this release,
      it has no choice but to accept the computation fee proposed by the
      Dapp user. In future releases, it will be free to decline,
      creating a market effect which Dapp users will have to gauche in
      order to guess the optimal fee for their task.

5. Once the results are committed on-chain and passed the Enigma
      Contract verification steps, the fee is unlocked and transferred
      to the worker custodian wallet. This will also change in future
      releases, fees will be accumulated in each worker’s “bank”
      (mapping in the Enigma Contract). A withdrawal function will allow
      each worker to collect their accumulated rewards all at once.

Deserialization and Decryption
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The arguments of the *callable* function are RLP serialized in the
*callableArgss* parameter. Generally, at least one argument is encrypted
but necessarily all of them.

The protocol for deserializing and decrypting arguments works as
follows:

1. Deserialize *callableArgs* using
      `RLP <https://github.com/ethereum/wiki/wiki/RLP>`__

2. For each argument,

   a. Determine if the value is encrypted

   b. If encrypted, decrypt using the key derived from the encryption
         key and the Dapp user public key. See the `Cryptography
         Document <https://docs.google.com/document/d/1c9eReGipyBO7l82-n7U8AH8tSXyZeN9ZDzIJgTbVKSI/edit#heading=h.h4mmyxajdhy7>`__
         for details.

   c. Since encrypted arguments were RLP encoded after encryption, their
         type was not stored in the RLP bytes. To cast the value, find
         its type from the *callable* function signature using its
         position in the deserialized list. For example, if the callable
         signature is *foo(bytes,int8)*, and deserializing
         *callableArgs* result in *[1, 00sdfsd0000sdfjsd9990sdf9jhe]*;
         we know to cast the second argument as *int8* after decryption.

Preprocessing
^^^^^^^^^^^^^

A preprocessor is a static service which runs before before executing
the *callable* function in the EVM. The output of a preprocessor is
injected in the parameters of the *callable* function. An array of
preprocessors can be requested, each representing a function call:
*f()*; where *f* is the name of the preprocessor function.

The preprocessor execution protocol works as follows for each specified
value:

1. Parse the preprocessor function signature into function name and
      arguments

2. Retrieve the preprocessor business logic mapping to the function name
      in from the internal registry

3. If arguments are specified, find their value in the list of decrypted
      arguments referenced in the previous section

4. Run the preprocessor business logic

5. Inject the outputs after the parameters of the *callable* function.
      The existing parameters followed by the preprocessor outputs much
      match to the *callable* function signature.

This release supports only one preprocessor: *rand()*. It accepts no
argument.

Execution in EVM
^^^^^^^^^^^^^^^^

All arguments of the *callable* function are now available. In order to
execute the computation, the EVM requires bytes composed of the first
bytes of a hash of the *callable* signature followed by the encoded
arguments in order. The `Application Binary Interface
Specification <https://solidity.readthedocs.io/en/develop/abi-spec.html?highlight=encode>`__
describe the encoding specification.

The data required to invoke the callback function on-chain must be
encoded in the same manner. This is convenient because we know that the
*callable* outputs much match the *callback* inputs. This means that we
do not need to decode the EVM output, simply adding the first bytes of a
hash of the *callback* signature generates the required callback data.

On-Chain Verification
~~~~~~~~~~~~~~~~~~~~~

On-chain verification refers a set of instructions in the Enigma
Contract which verify the authenticity of some data committed on-chain.
This is done by signing a hash of this data in the enclave of a
registered node (worker or principal) with its private key. Then, in the
contract, a new hash is generated from the same data and verified using
the *ECRecover* method of Ethereum. If *ECRecover* outputs the address
of the correct node, we verified that this data originated from the
expected enclave (see `On SGX <#on-sgx>`__ for the guarantees offered by
this verification).

After Each New Epoch
^^^^^^^^^^^^^^^^^^^^

After each epoch, the principal node generates a random seed. Then, it
signs the seed in its enclave with it private key (see `Worker
Selection <#worker-selection>`__). Then, it commits it to the Enigma
Contract which which verifies the signature.

Post Computation
^^^^^^^^^^^^^^^^

After a computation task is executed, the worker signs a hash of all
parameters of the task in its enclave with its private key. Then, the it
commits this data to the Enigma Contract. The contract then recreates
this hash, notably using the input parameters stored in the task record
prior to broadcasting to the network. Once the signature of this hash is
verified, the rest of the transaction is relayed to the *callback*
method of the Dapp contract.

Attestation
~~~~~~~~~~~

Performing attestation involves a verifiable proof which guarantees that
a given worker runs an intact version of Core within a certified
enclave. Combined with `On-Chain
Verification <#on-chain-verification>`__, it offers strong guarantees
about the privacy and correctness of those tasks (see `On
SGX <#on-sgx>`__).

The attestation protocol of Enigma is adapted from the Remote
Attestation Protocol of Intel [6]_; a protocol developed by them for
establishing a secure stateful channel between two parties: an Enclave
and a Service Provider. The Remote Attestation protocol of SGX is
described in the SGX Attestation Process document [7]_. Technically
speaking, we stripped down the higher level API provided by Intel, in
methods *msg0* to *msg4* (from the diagram), and only used the things
that we need to offer the guarantees stated above.

Because this proof is the key premise which guarantees privacy and
correctness of a task, it is critical that Dapp users must be able to
verify it independently (i.e. without any intermediary) for themselves.
To ensure that Dapp users never need to send any data nor pay any fee
before obtaining such proof, they perform attestation before giving out
each task. This way, if a malicious worker made its way through
registration, it would never receive any task.

|image9|

The attestation protocol works as follows before each computation task:

1. The Dapp calls the Enigma Library with a *compute* request

2. If the Enigma Library has workers parameters cached, it checks if the
      current block number is lower than the associated block number +
      number for blocks before the next reparameterization event.

3. If the workers parameters are expired or not already in cache, it
      calls the Enigma Contract to get a new seed and ordered list of
      workers.

4. It generates a random number which will serve as a nonce to ensure
      that the taskId is always unique. Then, it uses it to generate a
      taskId and determine the selected worker using the
      pseudo-randomness algorithm described in the `Worker
      Selection <#worker-selection>`__ section.

5. If the worker has not yet been verified locally (i.e. not in cache),
      it requests a full report from the Enigma Contract. This report
      was already requested from Intel and stored in the contract during
      `Registration <#registration>`__.

6. It parses the report into its parts: body of the report, signature,
      the x509 certificate associated with the report and its root
      certificate.

7. Using standard crypto libraries, it verifies that the report is
      correctly signed by the attached x509 certificate. It also
      verifies that the attached root certificate matches Intel’s
      publically available root certificate issued by a Certificate
      Authority.