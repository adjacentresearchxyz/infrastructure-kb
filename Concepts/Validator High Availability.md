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

- Additional Software for additional errors
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

## Active-Standby Validator[](#active-standby-validator "Permalink to this headline")

The next obvious step following a single-node validator is an active-standby setup with manually promoted slaves. If the active node dies, a sysadmin gets paged, ensures the active node is actually dead, and then manually promotes the standby node. You would be surprised how many large SaaS businesses rely on a single beefy MySQL or Postgres server with manual failover!

Both nodes would be identically configured, with the same validator key, mnode key and moniker.

This requires an on-call rotation with very low response times, which is expensive - you’re basically paying someone to sit at home all week, ready to react within minutes (most companies which do this do a follow-the-sun rotation with offices on three continents). It’s also susceptible to human error in determining node states.

Fortunately, active-standby setups are very common and there’s a lot of tooling for automated failover. The state-of-the art solution is [Pacemaker/Corosync](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/ch-introduction-haao), superseding [heartbeat](https://web.archive.org/web/20180829165659/http://www.linux-ha.org/wiki/Main_Page). Both were initially designed for VM clusters, which have very similar requirements to validators: no VM may ever run twice on the cluster, since it would result in immediate data corruption with both running from the same storage.

While Pacemaker/Corosync supports running with only two nodes, this is essentially a CA setup and _will not_ survive a network partition. This is common (and reasonable) for router/firewall clusters, with devices close to each other and redundant network interfaces and cabling, but we do not recommend a two-node cluster for validators due to the obvious risks e.g. of reaching a _split-brain_ scenario where both nodes go active at the same time.

Best practice for Pacemaker is a three node deployment - one active node, one standby node and a third quorum-only/witness node. Pacemaker will only _enable a resource_ (i.e. start the validator service) if it can establish a quorum, and will _self-fence_ (i.e. kill the validator service; the act of reliably excluding a broken node from the cluster is called _fencing_) if it loses quorum.

However, even with a quorum, self-fencing can fail. There are many edge cases around the interaction between pacemaker and the resources it protects. For instance, if pacemaker crashes, the node might be considered dead, but the validator service is still running, or your system’s service manager reporting the validator service as down when it’s actually still running.

To prevent these cases, a so-called _fencing device_ like a mechanical relay or out-of-band management interface (IPMI, iLO, …) is necessary which interrupts power for the other node to ensure that it’s really _really_ down (also called STONITH - “shoot the other node in the head”).

A Pacemaker/Corosync is a good option for validator high availability, but involves risks and drawbacks:

-   Pacemaker/Corosync is complicated to operate and has many edge cases. We recommend reading the Red Hat guide linked above which details common Pacemaker setups, and hiring someone who has experience with Pacemaker in critical production environments.
-   Fencing is failure-prone - it’s actually _really_ hard to ensure a node is down and won’t ever switch back on, either by resetting spontaneously or by operator mistake, and the fencing mechanisms needs to be tested extensively to reduce chance of failure.
-   Fencing and failover based on network load balancers or IP failover are unlikely to work as expected due to stateful connections and the P2P nature of the Tendermint network. You need to ensure that the actual processes (and ideally the physical hosts) are stopped.
-   You need to synchronize block height/round/step of the last signature to your standby node or risk double signing due to a race condition during failover or someone tricking your node. Do not use shared or replicated storage - it’s hard to reason about and introduces an additional point of failure (see our response in [this forum thread](https://forum.cosmos.network/t/backing-up-validator-server-physical-data-center/751/2?u=certus_zl)).
-   Pacemaker/Corosync aren’t designed to work across high latency networks, so this setup won’t scale beyond a single data center or even metro network (the corosync protocol expects latencies <2ms).

## Active-Active Validator[](#active-active-validator "Permalink to this headline")

Certus One has settled for an active-active validator setup. We run multiple validators, with the same key, _at the same time_, in multiple geographically distributed data centers. We built our own Raft-based consensus layer on top of the Tendermint core by implementing our own PrivValidator wrapper, which forces all signatures through a full Raft consensus.

While we’ve seen some proposals [[1]](#ha1) [[2]](#ha2) to use the Tendermint protocol itself to provide consensus, we decided it would introduce unnecessary complexity where a (comparably) simple protocol like Raft would suffice - we fortunately don’t need byzantine fault tolerance here.

This removes the brittleness of active-passive setups and the synchrony requirements of the corosync protocol. All of our validators are synchronized with the network at the current block height, ready to sign, while reliably preventing double signing through the Raft log.

This provides guaranteed consistency as well as very high availability (the CAP theorem still wins, though - there’s a small window of time where a node can crash just before submitting a signature to the network, where we cannot reliably retry the operation since we can’t know for sure whether it succeeded; this is deliberate and cannot be fixed without risking consistency).

While our active-active technology - called JANUS - currently isn’t an open source project, we open-sourced all of its dependencies and the testing framework [[3]](#testing) we use. We’re closely following upstream discussions and may decide to open source JANUS later.

We believe that active-active validator setups are the best way going forward, and look forward to contribute to the community discussions regarding active-active setups.

### Network topology[](#network-topology "Permalink to this headline")

Raft is usually deployed within a single data center, however, the protocol does not _require_ low latencies and works just fine with higher latencies (at the expense of elections and consensus read/writes taking a multiple of the worst latency in the cluster), assuming proper tuning and timeouts. The acceptable latency depends on the block times in the Tendermint network. We’re running all nodes within central Europe with no node being farther away than 50ms to ensure that a consensus read completes within <1s.

We run at most one validator per data center, with no more than _n_ validators per autonomous system, where _n_ is the number of nodes that the cluster can lose without losing consensus. All nodes are BGP multi-homed with multiple transit providers.

We mapped out all routes between the data centers and ensured - through either private peering agreements or BGP traffic engineering - that even failure of critical transit networks or internet exchanges will not result in loss of consensus.

This allows us to survive the failure of multiple data centers, whole autonomous systems, as well as internet exchanges while at most losing one block.