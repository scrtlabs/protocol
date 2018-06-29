Enigma-js
==========

The Enigma client library, `Enigma-JS <#_ydldonl0i1f1>`__, is a
javascript library that interfaces with the Enigma protocol. Enigma-JS
contains tools to: 1) safely encrypt sensitive data in-memory for
immediate use or storage; 2) obtain an authoritative proof that the
target worker is securely running trusted hardware prior to sending data
and paying fees. (Trusted hardware for this release means Intel SGX: for
more information on SGX, see sections `On SGX <#on-sgx>`__ and
`Registration <#registration>`__)

**validateKeyHex**

	validateKeyHex(key)

	- Checks whether the supplied key is valid.

**getDerivedKey**

	getDerivedKey (enclavePublicKey, clientPrivateKey)


**encryptMessage**
	
	encryptMessage (derivedKey, msg)


**getPublicKey**
	
	getPublicKey (clientPrivateKey)
	
**removeTrailing0x**
	
	removeTrailing0x(str)

**recoverPubKey**

	recoverPubKey(msgHash, signature)

**verifySignature**

	verifySignature(msg, signature, pubAddr)


**selectWorker**

	selectWorker(taskId)