---
title: High Availability
layout: home
parent: Concepts
---

# High Availability
Operating distributed systems at scale requires a high degree of availability. This is especially true for validators, which are a critical component of the network. This page describes the various approaches to validator high availability.

Operating blockchain infrastructure (minus RPC/Archive nodes) often requires you to run with a private key. Of the many things to think about when operating a distributed system management and access control around the private key is of upmost importance. 

## TLDR

When standing up a validator for staking considering high availability is very important. The common ways for operating validators are a single validator, an active-passive, or an active-active setup. Each has its own advantages and disadvantages and you might deploy different protocols with different HA setups.

### Single

Running a single validator node is the simplest to setup and ensures the lowest risk for double-signing but trades off potential downtime

#### Advantages
- Lowest risk of double signing and slashing

#### Disadvantages
- Due to networking, datacenter, or hardware failures long periods of downtime can be expected
- In the event of downtime you will suffer a loss of rewards, which is concerning for large staking balances
- If you cannot access the machine during downtime you cannot safely activate a new validator at risk of the first coming back online and double signing

### Active Passive
Operating an active validator with a passive validator waiting (completely synced, keys ready, just needs to be activated) with a manual switch over.

#### Advantages
- Due to a manual switch over between the active and passive nodes you can ensure that double signing is minimized
- An extra node in a different location, different hardware (in case of a hardware failure) ready to go when needed

#### Disadvantages
- Requires manual switch over which is a lot of risk on an engineer, best to design with separate levels of access control
- If you cannot access the machine during downtime you cannot safely activate a new validator at risk of the first coming back online and double signing

### Active Active
Running a consensus layer on top of two or more fully active validators, where you have a switch to choose which is actively voting, while the other's votes are not submitted to the chain. An example is [Raft Consensus](https://raft.github.io/)

#### Advantages
- Next to zero downtime
- Never missed rewards

#### Disadvantages
- Additional software for additional errors
- If there is an error high chance you are slashed

## Single Node Validator
Running a single node is the simplest and most common way to run a validator it is also a reasonable choice given modern hardware, redundent networking, and with well built out datacenters all leading to a pretty low failure rate for validators. 

Alas, there always comes a time when 
- there is a hardware failure 
- there are some hiccups in the networking
- you are [kicked off](https://www.infostor.com/news/hetzner-surprises-everyone-with-cloud-data-centre-ban-for-ethereum-nodes/) of the datacenter
- your datacenter literally [catches on fire](https://www.pcmag.com/news/ovhcloud-data-center-devastated-by-fire-entire-building-destroyed)

or for a variety of other reasons. 

When this happens (or ideally ahead of time) you need to consider your backup and restart proceedure (how fast can your validator be back up operating?). Switching when you are not prepared means, getting a new server (potentially with a new hosting provider), setting up your machine with required software to operate (hopefully you already have a pipeline for this), and notably transferring your keys

> Quick note on keys, you should be using something such as a a key management system (KMS) or secret manager to store your keys. This is a system that allows you to store your keys in a secure and centralized location indpendent of the servers operating your nodes. This gives you access to the keys when you need them. Refer to the [Key Management](/concepts/key-management) and [Custody](/concepts/Custody.md) page for more information.

To top it all off you might be doing all of this in the early hours of the morning just to make sure you dont have extended periods of downtime... this is when you should start to consider alternative options like active-standby and active-active.

## Active-Standby Validator
Rather than just operating a single validator and then due to downtime needing to rush and set up a new setup an active-standby setup can be employed. The standby node has everything identical to the active node and is fully synced ready to go at a moments notice. The keys are the only thing that is missing. In the event of downtime on the active node, a systems administrator would be alerted and manually switch over the keys 

This would require an always on-call roatation in which administrators have very low response time for switch over. 

Active-Standby setups are quite common and you can find decent resources with [pacemaker](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/ch-introduction-haao) and [heartbeat](https://web.archive.org/web/20180829165659/http://www.linux-ha.org/wiki/Main_Page). Certus one also gives a great overview of this in [validator high availability](https://kb.certus.one/validator_ha.html#active-standby-validator).

## Active-Active Validator[](#active-active-validator "Permalink to this headline")

Certus One has settled for an active-active validator setup. We run multiple validators, with the same key, _at the same time_, in multiple geographically distributed data centers. We built our own Raft-based consensus layer on top of the Tendermint core by implementing our own PrivValidator wrapper, which forces all signatures through a full Raft consensus.

While we’ve seen some proposals [[1]](#ha1) [[2]](#ha2) to use the Tendermint protocol itself to provide consensus, we decided it would introduce unnecessary complexity where a (comparably) simple protocol like Raft would suffice - we fortunately don’t need byzantine fault tolerance here.

This removes the brittleness of active-passive setups and the synchrony requirements of the corosync protocol. All of our validators are synchronized with the network at the current block height, ready to sign, while reliably preventing double signing through the Raft log.

This provides guaranteed consistency as well as very high availability (the CAP theorem still wins, though - there’s a small window of time where a node can crash just before submitting a signature to the network, where we cannot reliably retry the operation since we can’t know for sure whether it succeeded; this is deliberate and cannot be fixed without risking consistency).

While our active-active technology - called JANUS - currently isn’t an open source project, we open-sourced all of its dependencies and the testing framework [[3]](#testing) we use. We’re closely following upstream discussions and may decide to open source JANUS later.

We believe that active-active validator setups are the best way going forward, and look forward to contribute to the community discussions regarding active-active setups.