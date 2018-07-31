Appendix
==========

Additional reference materials for further reading.

.. _cryptography:

Cryptography
~~~~~~~~~~~~

The enclaves in the Enigma protocol use public-key cryptography, also known as asymmetrical cryptography, which use a pair of keys: a public key that may be disseminated widely, and a private key that is known only to the enclave. This accomplishes two functions: **authentication**, where the public key verifies that a holder of the paired private key sent the message, and **encryption**, where only the paired private key holder can decrypt the message encrypted with the public key. 

Within the realm of public-key cryptography, Enigma uses elliptic curve (EC) cryptography, which provide smaller key sizes and faster operations for approximately equivalent estimated security.

Elliptic Curve
^^^^^^^^^^^^^^

The Enigma Protocol uses the **secp256k1** curve for both signing and encryption keys, as it's standard in Bitcoin/Ethereum (but the keys themselves should not be the same). secp256k1 is not considered a safe curve, but it's unclear if that opens any vector of attack, and it's good enough for Bitcoin/Ethereum.
(`SafeCurves <https://safecurves.cr.yp.to>`_, all the attacks here on secp256k1 are theoretical only.)

secp256k1 was constructed in a special non-random way that allows for especially efficient computation. As a result, it is often more than 30% faster than other curves if the implementation is sufficiently optimized. Also, unlike the popular NIST curves, secp256k1's constants were selected in a predictable way, which significantly reduces the possibility that the curve's creator inserted any sort of backdoor into the curve.

Alternatives: 
Cardano/Ouroborous uses **secp256r1** curve for ECDSA, though we see no good reason to use **secp256r1**, as it has the almost the same security as **secp256k1**. The only difference is that **secp256k1** uses the "Koblitz curve" which means there is less chance for backdoors (because the constants aren't random). 
If at some point we doesn't need compatibility with bitcoin/ethereum then he should use something like **Curve25519** which is considered to be a "safe curve".

Signing
^^^^^^^

For signing, the Enigma Protocol uses **Elliptic Curve Digital Signature Algorithm** (ECDSA) with **keccak256** as the hashing function(for compatibility with ethereum’s solidity), a variant of the Digital Signature Algorithm that uses elliptic curve cryptography. ECDSA is used by most blockchains.


Encryption
^^^^^^^^^^

For encryption, the Enigma Protocol uses an adaptation of **Elliptic Curve Diffie-Hellman** (ECDH), an anonymous key agreement protocol that allows two parties to establish a shared secret over an insecure channel.

The enclave can make their public key known a priori. Any user that wants to encrypt a message for the enclave can generate a one-time asymmetric key-pair, and use that to derive a one-time symmetric key as in the usual ECDH protocol. The one-time symmetric key is generated with a key derivation function (KDF) that is yet to be specified. The symmetric-key algorithm used for encryption is most likely **Advanced Encryption Standard** (AES) with the **GCM** mode(**Galois/Counter Mode**) to add HMAC(Hash-based Message Authentication Code) to add message authentication to the encrypted message and to protect against a number of known vector attacks of AES. The encryption is used with a key length of 256 bits.

Now, the user can use that secret to encrypt a message, and send to the enclave both the encrypted message, and the public key it used. With that information, the enclave can derive the symmetric key and decrypt the message.

(Side note: if we want authentication as well, we may need to sign these messages as well, but that's to be determined. I don't think we care much about users authenticating towards the enclaves.)

*Elliptic Curve Integrated Encryption Scheme* (ECIES) was also considered as an option for encryption, while being similar to ECDH. The reason to prefer ECDH over ECIES is because ECIES isn’t standardized yet, it doesn’t add much security to the protocol, and because it isn’t standardized yet most of the known cryptography libraries doesn’t implement it. (which also means it wasn’t tested enough).

Implementation
~~~~~~~~~~~~~~

For the PoC, the Enigma Protocol implements all the above cryptographic algorithms in Python using the `pyca/cryptography <https://cryptography.io/en/latest/>`_ package. The Enigma implementation of the cryptographic primitives can be found in the core repository: `core/src/core/cryptography <https://github.com/enigmampc/core/tree/develop/src/core/cryptography>`_ (develop branch).

In the near future we may move to a C/C++ library that is compatible with the SGX implementation to be run natively inside an SGX enclave. The implementation under consideration is `Crypto++ <https://github.com/weidai11/cryptopp>`_, a free C++ class library of cryptographic schemes that includes all the algorithms outlined above.
