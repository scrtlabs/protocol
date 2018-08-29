Enigma-js
==========

The Enigma client library, `Enigma-JS <#_ydldonl0i1f1>`__, is a
javascript library that interfaces with the Enigma protocol. Enigma-JS
contains tools to: 1) safely encrypt sensitive data in-memory for
immediate use or storage; 2) obtain an authoritative proof that the
target worker is securely running trusted hardware prior to sending data
and paying fees. (Trusted hardware for this release means Intel SGX: for
more information on SGX, see sections `On SGX <#on-sgx>`__ and
`Registration <#registration>`__). 

**validateKeyHex**
~~~~~~~~~~~~~~~~~~
	validateKeyHex(key)

	- description:
			Checks whether the supplied key is valid.

	- arguments:
			key: the supplied key in hex

	- returns: 
			A valid key derived from the argument


**getDerivedKey**
~~~~~~~~~~~~~~~~~

	getDerivedKey (enclavePublicKey, clientPrivateKey)

	- description:
			Gets a shared key derived from the client and the enclave
	- arguments:
			enclavePublicKey: the public key of the enclave.

			clientPrivateKey: the private key of the client.
	- returns: 	
			A shared key derived from Elliptic Curve Diffie-Hellman


**encryptMessage**
~~~~~~~~~~~~~~~~~~
	
	encryptMessage (derivedKey, msg)

	- description: 
			Encrypts a user message.
	- arguments:
			derivedKey: the shared key resulting from the getDerivedKey function call.
			
			msg: the data to be encrypted.
	
	- returns: 
		
		An encrypted message using derivedKey and msg.


**getPublicKey**
~~~~~~~~~~~~~~~~
	
	getPublicKey (clientPrivateKey)
	
	- description: 
			gets the public key of the enclave
	- arguments:
			clientPrivateKey: the client's private key

	- returns:
			The public key of the enclave


**removeTrailing0x**
~~~~~~~~~~~~~~~~~~~~
	
	removeTrailing0x(str)

	- description: 
			Removes "0x" from the input
	- arguments:
			str: the string to be trimmed
	- returns:
			The shortened string if string contained "0x", the original string if not.

**recoverPubKey**
~~~~~~~~~~~~~~~~~

	recoverPubKey(msgHash, signature)

	- arguments:
	- returns:

**verifySignature**
~~~~~~~~~~~~~~~~~~~
	verifySignature(msg, signature, pubAddr)

	- arguments:
	- returns:

**selectWorker**
~~~~~~~~~~~~~~~~~

	selectWorker(taskId)

	- arguments:
	- returns: