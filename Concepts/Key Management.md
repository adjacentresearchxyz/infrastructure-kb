---
title: Key Management
layout: home
parent: Concepts
---

# Key Management

When operating a validator the most important part is managing and custody of the keys. There are generally two types of keys in a staking setup.

**Validator Key**

A validator key is what is used to sign your attestation, commitment, or validation of a given block. This is what certifies the messages sent from your validator.

**Wallet Key**

Your wallet key(s) are more important than the validator keys as they are what give you access to unstake and withdraw your rewards and your collateral. Loss of your wallet key(s) is loss of your funds.

## Wallet

The wallet key(s) are what give and control complete access to all of your funds (staked collateral and rewards that are earned). Loosing this key means loss of all your funds, thus security and proper management is very important.

The ideal forms of custody are listed and detailed below

- A primary custodian
- Hardware wallet in a secure environment
- Software wallet in a secure environment
- Software wallet on the validator (**LAST RESORT**)

#### Primary Custodian

The ideal setup for custody of your collateral and earned rewards is by using a primary custodian like [fireblocks](https://www.fireblocks.com/), [coinbase custody](https://custody.coinbase.com/), etc.. Although many staked assets are still not yet available to be properly held within these platforms. Thus self-custodying the keys becomes required.

#### Hardware wallet in a secure environment

When managing the control of your own keys you ideally keep them in a segregated and air-gapped environment with redundancy and multi layers of access, stored within a hardware wallet.

This is the best form of self-custody is secure and gives you a lot of flexibility in wallet providers, etc..

> Ledger is an extremely popular hardware wallet, a decent amount of protocols are supported natively or through "experimental features". Often you can install applications on the Ledger but they are not supported on Ledger Live, meaning you will have to use a web wallet for the specific protocol with ledger. This is still secure since the keys are on a hardware device.

#### Software wallet in a secure environment

Newer projects and protocols might not have developed a ledger application (or something similar) and thus you will be unable to custody your keys on a hardware device. The next best form is using a software based wallet but store the device used to access it in a secure environment with a similar setup to the hardware wallet above.

#### Software wallet on the validator (**LAST RESORT**)

If a project does not have a hardware application built out, or support for a software based wallet, you will most likely need to custody the keys that hold your funds directly on the validator. This is an absolute **last resort** as hacks into your server open you up to potentially loosing all of your funds.

If maintaining custody on the server, ideally you can encrypt and password protect your private keys especially if they are holding staked collateral and rewards.

Unfortunately some protocols require you to have the private keys **unencrypted and non password protected** which is a very large security risk. But if needed it might be required in order to participate in the network, if possible you should also strongly encourage the developer team of the protocol to enable any (or all) of the following: encryption, password protection, hardware support, ledger support, primary custodian support.

## Validator Key

Custody of the validator key is much different than custody of a key that has funds against it.

The validator key does not have any actual value against it, however control over the validator key can lead to potential missed rewards and potential slashing of your collateral.

Additionally validator keys almost always need to be held on the validation node itself which poses a security risk (again see the #Linux and #Security sections for more information).

Ideally the protocol has built in KMS support ([here](https://docs.tendermint.com/master/tools/remote-signer-validation.html#running-against-kms) is a really guide guide to built in KMS support in tendermint protocols).

> If a protocol does not have built in KMS support, you should see if you can reach out to them and put it on their roadmap.

If the validation keys cannot be stored within a KMS, ideally they are password protected and encrypted.

If not same goes for above, reaching out to the core development team and seeing if they can put it on their roadmap becomes a priority.

## Software validator or wallet key that is not needed on the server

Sometimes you may have a validator or a wallet key that is not nessecarily needed to be on the node. For example it does not need to continually sign attestations and validate transactions. In this case you can store the `keystore` in an offline, coldstorage enviroment after generating it using the CLI. Generally the only case that you will need this is if the wallet is purely used just for withdrawing staked rewards. When you need to withdraw rewards simply import the keystore from the cold storage enviroment for use and then delete once down. 

## Further Resources
- [Ethereum staking Attestation Key Management](https://www.attestant.io/posts/protecting-validator-keys/)
- [Key Management Certus One](https://kb.certus.one/key_management.html)
- [HSM Certus One](https://kb.certus.one/hsm.html)