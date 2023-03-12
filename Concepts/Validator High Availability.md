---
title: High Availability
layout: home
---

# Validator High-Availability[](#validator-high-availability "Permalink to this headline")

Tendermint is a globally consistent, byzantine-fault tolerant consensus-based replicated state machine - the gold standard in distributed systems.

A single validator, however, is not highly available. It doesn’t need to be - the network tolerates single validator failures just fine.

However, as the validator operator, things look different and your tolerance for single validator failures is pretty low. Sooner or later, every validator operator is confronted with the question of making their validator fault-tolerant. Unfortunately, there’s no established procedure for this and each team is forced to build their own high availability solution.

This article portrays some possible HA topologies and their pros and cons.

Running a Tendermint validator is - like any distributed system - a delicate balance between ensuring availability and consistency/correctness (availability as in “up and signing blocks”, and consistency as in “no byzantine behavior”).

As is well known, the [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem) states that you can only pick two of partition tolerance, consistency and availability. Since network partitions are inevitable in any distributed system, you can’t pick CA, and the only choices are AP and CP.

With Cosmos validators, the penalty for inconsistency (double signing!) is much harsher than unavailability (you even get a few minutes to fix it before you are slashed). This means that the only reasonable choice is a CP system - if there’s even the slightest possibility that more than one node is active at the same time, it needs to sacrifice availability (i.e. stop signing).

## Single Node Validator[](#single-node-validator "Permalink to this headline")

Since inconsistency is so dangerous, running a single-node validator is a very reasonable choice. Modern datacenter hardware, in a modern datacenter, with redundant networking, designed by an experienced systems architect, has a very low failure rate sufficient for many low-stakes validators.

As any sysadmin will tell you, badly designed high availability setups tend to fail at a much higher frequency than reliable single-node ones.

By never running two nodes at the same time, the risk of double signing is very low - one would have to trick your replacement validator into signing an earlier block while it recovers or exploit a vulnerability in Tendermint.

However, if the hardware or datacenter does fail in a catastropic way, you will be down for an extended period of time until you have got replacement hardware in place and re-synced. While acceptable for low-stakes validators, most commercial validator operations won’t be able to accept this risk. Even redundant power and networking setups have a non-zero chance of failure, in fact, most systems aren’t fully isolated and failures often tend to be correlated.

(True story: your router has redundant PSUs? Oops, both fans just simultaneously ingested a packaging foil that someone left in the rack).

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

[[1]](#id1)

[https://github.com/tendermint/tendermint/issues/1758](https://github.com/tendermint/tendermint/issues/1758)

[[2]](#id2)

[https://github.com/tendermint/kms/issues/29](https://github.com/tendermint/kms/issues/29)

[[3]](#id3)

[Testing your tooling](testing.html)

### Network topology[](#network-topology "Permalink to this headline")

Raft is usually deployed within a single data center, however, the protocol does not _require_ low latencies and works just fine with higher latencies (at the expense of elections and consensus read/writes taking a multiple of the worst latency in the cluster), assuming proper tuning and timeouts. The acceptable latency depends on the block times in the Tendermint network. We’re running all nodes within central Europe with no node being farther away than 50ms to ensure that a consensus read completes within <1s.

We run at most one validator per data center, with no more than _n_ validators per autonomous system, where _n_ is the number of nodes that the cluster can lose without losing consensus. All nodes are BGP multi-homed with multiple transit providers.

We mapped out all routes between the data centers and ensured - through either private peering agreements or BGP traffic engineering - that even failure of critical transit networks or internet exchanges will not result in loss of consensus.

This allows us to survive the failure of multiple data centers, whole autonomous systems, as well as internet exchanges while at most losing one block.