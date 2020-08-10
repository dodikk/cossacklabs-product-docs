---
weight: 1
title: Inventory of Acra keys
bookCollapseSection: true
---

# Inventory of Acra keys

There are several types of keys used in Acra:

  - **Storage keypairs.**
<!-- TODO: describe keypairs in more details: what is keypair, private and public part, introduce "client-side" encryption and "server-side/proxy-side" encryption --> 
    AcraWriter uses the public key to encrypt data to be stored in the database.
    AcraServer and AcraTranslator use corresponding private storage key
    to decrypt data queried from the database.

  - **Keys for transport encryption.**
    Acra uses either [Themis Secure Session](/themis/crypto-theory/cryptosystems/secure-session/) or TLS
    as transport protection between AcraServer and AcraConnector, or AcraTranslator and AcraConnector.

    In Themis Secure Session mode, Acra uses transport keypairs:
    the two parties should exchange public keys, keeping the private keys to themselves.
    In TLS mode, Acra uses TLS certificates instead of transport keypairs.

  - Poison record key used to detect suspicious database access patterns.

  - Authentication storage key for encryption/decryption credentials of AcraWebConfig users.

  - [Acra Enterprise Edition](https://www.cossacklabs.com/acra/#pricing)
    uses more keys to power features like searchable encryption, tamper-proof audit log, etc.

    <!-- TODO: describe Acra EE keys in more detail? -->

Storage keys can be represented by either:

  - One keypair for each *Client ID* ([default zone mode](https://docs.cossacklabs.com/pages/documentation-acra/#zones)),
    which is used for encryption by AcraWriter and decryption by AcraServer and AcraTranslator.

  - A set of *zone keys* ([multiple zone mode](https://docs.cossacklabs.com/pages/documentation-acra/#zones)).
    Each zone represents a unique user or type of users and has corresponding encryption and decryption keys.
    Using zones complicates unauthorized decryption:
    the attacker not only needs to get the decryption key but to use a correct Zone ID, too.

## Acra components

Each Acra component needs to have its own key store (located in the `.acrakeys` directory by default)
which contains a set of keys necessary for correct operation of the component.

Here is an overview of core Acra keys and their locations:

| Purpose | Private key stays on | Public key exchanged with |
| ------- | -------------------- | ------------------------- |
| Transport: AcraConnector     | AcraConnector  | → AcraServer or AcraTranslator |
| Transport: AcraServer        | AcraServer     | → AcraConnector |
| Transport: AcraTranslator    | AcraTranslator | → AcraConnector |
| Data encryption (AcraStruct) | AcraServer     | → AcraWriter |
| Data encryption (AcraStruct) | AcraTranslator | → AcraServer (in Transparent proxy mode) |
| Authentication storage key (for AcraWebConfig users) | AcraServer | |

**AcraConnector** needs the following keys to establish a [Secure Session connection](/themis/crypto-theory/cryptosystems/secure-session/):

  - AcraConnector’s own transport private key.

  - Other party transport public key.

    You should put the transport public key of AcraServer or AcraTranslator
    into AcraConnector’s key store, depending on the connection type.

**AcraServer** works with the following keys:

  - AcraServer’s own transport private key as well as AcraConnector’s transport public key.

    They are necessary for accepting connections from the clients via Secure Session.

  - Storage private keys for decrypting AcraStructs generated by AcraWriter.

  - If AcraServer is running in [Transparent proxy mode](https://docs.cossacklabs.com/pages/documentation-acra/#transparent-proxy-mode),
    it should possess the storage public keys to be able to encrypt the values from SQL queries into AcraStructs.

  - If you’re using *AcraWebConfig* HTTP server to configure AcraServer remotely,
    the users’ credentials are stored encrypted with authentication storage key
    and are decrypted by AcraServer upon users’ login.

**AcraTranslator** needs to have the following keys:

  - AcraTranslator’s own transport private key along with AcraConnector’s transport public key.

    They are necessary for accepting connections from the clients via Secure Session.

  - Storage private keys for decrypting AcraStructs generated by AcraWriter.

**AcraWriter** should have the storage public keys.

  - Storage public keys are necessary for encrypting data into AcraStructs
    in such a way that would only be readable by AcraServer or AcraTranslator.

<!-- TODO: describe Acra EE keys? -->

Read about [key generation](operations/generation/)
to learn how to generate and distribute these keys between Acra components.