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
Running a single node is the simplest and most common way to run a validator it is also a reasonable choice given modern hardware, redundant networking, and with well built out datacenters all leading to a pretty low failure rate for validators. 

Alas, there always comes a time when 
- there is a hardware failure 
- there are some hiccups in the networking
- you are [kicked off](https://www.infostor.com/news/hetzner-surprises-everyone-with-cloud-data-centre-ban-for-ethereum-nodes/) of the datacenter
- your datacenter literally [catches on fire](https://www.pcmag.com/news/ovhcloud-data-center-devastated-by-fire-entire-building-destroyed)

or for a variety of other reasons. 

When this happens (or ideally ahead of time) you need to consider your backup and restart procedure (how fast can your validator be back up operating?). Switching when you are not prepared means, getting a new server (potentially with a new hosting provider), setting up your machine with required software to operate (hopefully you already have a pipeline for this), and notably transferring your keys

> Quick note on keys, you should be using something such as a a key management system (KMS) or secret manager to store your keys. This is a system that allows you to store your keys in a secure and centralized location independent of the servers operating your nodes. This gives you access to the keys when you need them. Refer to the [Key Management](/concepts/key-management) and [Custody](/concepts/Custody.md) page for more information.

To top it all off you might be doing all of this in the early hours of the morning just to make sure you dona have extended periods of downtime... this is when you should start to consider alternative options like active-standby and active-active.

## Active-Standby Validator
Rather than just operating a single validator and then due to downtime needing to rush and set up a new setup an active-standby setup can be employed. The standby node has everything identical to the active node and is fully synced ready to go at a moments notice. The keys are the only thing that is missing. In the event of downtime on the active node, a systems administrator would be alerted and manually switch over the keys 

This would require an always on-call rotation in which administrators have very low response time for switch over. 

Active-Standby setups are quite common and you can find decent resources with [pacemaker](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/ch-introduction-haao) and [heartbeat](https://web.archive.org/web/20180829165659/http://www.linux-ha.org/wiki/Main_Page). Certus one also gives a great overview of this in [validator high availability](https://kb.certus.one/validator_ha.html#active-standby-validator).

## Active-Active Validator
The final option is to run an active-active validator. This is a setup where you have three or more *active* validators running at the same time. That is multiple validators, with the same keys running at the same time. 

Running an active-active setup has the following advantages 
- you can suffer downtime from multiple validators as long is one is up you do not suffer downtime and loose rewards
- validators can run in different datacenters, geographically distributed
- little to no manual intervention as the active-active setup will switch to validators that active with low response time if the current one sees downtime

Of course immediately you might be thinking, "but what if two validators are active at the same time and double sign?" This is a very valid concern but with some additional development at the validator client level, and some networking you can run a simple RAFT based consensus across all of your active validators allowing you to run with little to no downtime, securely, and effectively. 

There have been a few projects that have worked on this in the past such as [JANUS](https://github.com/certusone/janus) from Certus One. Notably [Kuutamo](https://kuutamo.cloud/) is working on building many active-active validators starting with [near](https://github.com/kuutamolabs/near-staking-knd) and [lightning](https://github.com/kuutamolabs/lightning-knd). In order to understand their implementation it might be helpful to read their [architecture](https://github.com/kuutamolabs/near-staking-knd/blob/main/docs/architecture.md)

We strongly believe that the future of validator operations is active-active. 