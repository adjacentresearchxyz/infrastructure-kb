---
title: Custody
layout: home
parent: Concepts
---

# Custody 
When operating a validator the most important part is managing and custody of the keys. You can read more about the types of and the management of keys in the [Key Management](Key%20Management.md) section as often the type of key relates to how you might want to securly store it. There are a few different types of custody solutions possible. This is broadly split up into Hot and Cold wallet catergories, although we would also like to introduce the category of Warm wallets. 

## Hot 
A hot wallet is a wallet that is connected to the internet and is not air-gapped and can be thought of as "always on". Generally there are little to no access control policies, levels of approval, monitoring/alerts, and usually only a single password that protects the signing capacity of the key. This means that if someone was able to access your computer and brute force the password they would have full control over your keys. 

While this is the least secure form of custody (and not absolutely not fit for enterprise scale), it is also the most convenient and flexible. This is often the type of wallet that bots, validators, and other types of infrastructure use along with very active traders. 

Examples of a Hot wallet would include applications like
- Metamask
- MyEtherWallet
- A private key generated via `geth` and stored locally

## Cold 
A cold wallet is a wallet that is not connected to the internet and is air-gapped and can be thought of as "fully offline". A cold wallet is fantastic for storing funds in an extremely secure way for a long time. It is not good at being constantly accessed or signing a lot of transactions given the process to do so is often quite manual and can be time consuming. 

There are a few different types of cold storage solutions. The most common is a hardware wallet. A hardware wallet is a device that is designed to be air-gapped and is often used to store private keys. The hardware wallet is connected to the computer via USB and the private key is then used to sign transactions (you also can offline sign on most hardware wallets and then broadcast the transaction on the computer). Hardware wallets are great for self custody but often does not support the flexability and speed that enterprise needs. For that most institutions have turned to one of two options

### HSM 
A Hardware Security Module (HSM) is a device that generates, stores and protects cryptographic keys and provides hardware acceleration for cryptographic operations and is common across many industrys as a very common solution for storing sensitive information like credit cards, other PII, or private keys. 

HSMs provide a few key benifits like 
- High levels of trust and authentication
- Tamperproof while each implementation of an HSM has their own generally an HSM is able to respond to tampering or show evidence that data was tampered with
- Access control and audit logs, all access to the HSM is logged and can be audited and can be constantlly monitored for suspicious activity
- A single, central point of control for all keys, this makes it easy to manage and audit all keys and makes it easy to rotate keys

### MPC
Multi Party Computation (MPC) is another method of custody that is becoming more popular. MPC is a method of computing a function over multiple parties without revealing the input data to any of the parties. This is done by splitting the data into shares and then having each party compute a function on their share and then combining the results. This is a very secure way to store keys as it requires multiple parties to collude in order to access the keys.

Similar to an HSM, MPC provides a few key benifits like
- High levels of trust and authentication
- Access control and audit logs, all access to the MPC is logged and can be audited and can be constantlly monitored for suspicious activity
- A single, central point of control for all keys, this makes it easy to manage and audit all keys and makes it easy to rotate keys
- Requires multiple parties to agree on a transaction, this makes it harder for a single party gain access and steal funds

## Warm
Jump Crypto has introduced a concept of [Warm Custody](https://jumpcrypto.com/custody-bft-policy-checking-threshold-signatures/) a solution that provides the flexability and speed of a hot wallet along with the control, access, security, and auditability of a cold storage solution like MPC. 

While cold storage providers generally implement their solutions in either a hardware wallet, via an HSM, or using MPC technology warm solutions are still actively being developed and researched. Jump implements their solution with threshold signatures running on a private blockchain they call it `Silo`. Other solutions are slowly coming on the market such as [Entropy](https://entropy.xyz/) (created by ex-[nucypher](https://www.nucypher.com/)) developers or [mpch labs](https://www.mpch.io/). 

The advantages for a infrastructure provider (validator, bot operator, high frequency trader, etc.) are everything good about hot and cold custody 
- Flexibility and speed in the types, number of, and frequency of transactions
- High levels of trust and authentication
- Access control and audit logs, all access to the MPC is logged and can be audited and can be constantlly monitored for suspicious activity
- A single, central point of control for all keys, this makes it easy to manage and audit all keys and makes it easy to rotate keys

While these solutions continue to built out there is software you can implement to get the benefits of a warm wallet today. A common example of this would be using [Hashicorp Vault](https://www.hashicorp.com/products/vault), with a plugin to handle signing (such as [vault-ethereum](https://github.com/immutability-io/vault-ethereum)). This would allow you to have fine grained access control policies, various levels of telemetry and monitoring, redundency and backup, while still maintaining the flexability and speed of a hot wallet (you can even use an HSM on the Vault backend to control it's seal status). 

## Resources 
- [Key Management](Key%20Management.md)
- [The omnibus model for custody](https://medium.com/@FidelityDigitalAssets/the-omnibus-model-for-custody-96b69710f92d)
- [What is an HSM](https://realsec.com/en/blog/what-is-a-hardware-security-module-hsm/)
- [HSM Guide](https://github.com/snowch/hsm-guide/blob/master/book.md)
- [What is MPC](https://www.fireblocks.com/what-is-mpc/)
- [HSM vs MPC](https://custody.bitpanda.com/insights-events/hsm-vs-mpc)