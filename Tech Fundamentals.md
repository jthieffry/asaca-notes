# Key Points

## Encryption

* Encryption-at-rest: usually only one entity involved, usually for storage. If storage is stolen, no compromission.
* Encryption in transit: usucally between multiple entities, usually for communication. Data encrypted during transit, usually via the setup of an encryption tunnel.
* Encryption objects: Algorithm (blowfish, aes, rc4, des, etc.) on key + plaintext -> cyphertext
* Symmetric encryption: Key used is the same for the encryption and decryption -> Cheap but how do you share the key between sender and receiver securely ?
* Asymmetric encryption: Private key used to decrypt, public key used to encrypt -> More expensive, but safe. Usually, communication starts with Asymmetric to share a symmetric key, then continues with symmetric encryption.
* Signing is to ensure somebody's identity. The party who wants to be identified will send a message encrypted with his private key. The receiver will decrypt with the sender's public key and then confirm that the sender is indeed who he claims to be.
* Steganography is hiding data into another piece of data, to hide the fact that an encrypted message has been sent.