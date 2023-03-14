<!-- ---
title: Validator Operations
layout: home
parent: Concepts
--- -->

# Validator Operations Guide[](#validator-operations-guide "Permalink to this headline")

Running a proof-of-stake validator puts a much greater emphasis on technical correctness, sound systems architecture, security, and overall operational excellence than most distributed systems. In a proof-of-stake cryptocurrency, operational skills take the place of raw computational power. This means that the design process, documentation and knowledge sharing are particularly important for validator operations.

This guide is a living document which details a set of best practices for running a validator service as implemented by Certus One, as well as technical background to help you design your own validator architecture.

The aim of this document is to provide a baseline for validator operations, both to make it easier for new validators to get started, and to provide input to other teams. We believe that collaboration and openness strongly benefits the overall ecosystem - the more well-run validators there are, the more resilient will the network be.

While this document’s focus is running blockchain validators, much of its content is applicable to operating any highly available, distributed service.

While it’s hard to provide an implementation that fits all use cases, we try to provide reference implementations which implement our guidelines.

This guide assumes practical Linux systems administration experience and at least basic knowledge of the blockchain you’re validating on (we recommend reading at least their whitepaper). For Cosmos, this includes both Tendermint and the Cosmos SDK.

Numerous books have been written about each of the topics in this knowledge base - keep in mind that a knowledge base like ours is only ever a starting point and a guide, not an exhaustive treatment of any of the topics we’re discussing. We took a look at our bookshelves (and e-readers, and browser bookmarks) and many articles have a literature list of books and articles which we can personally recommend.

The document’s source code is available [on GitHub](https://github.com/certusone/kb). Contributions and bug reports/feedback are greatly appreciated (feel free to use the GitHub issues for discussion as well as bug reports).

## Contents[](#contents "Permalink to this headline")

-   [Monitoring, Alerting and Instrumentation](monitoring.html)
    -   [Symptoms-Based Alerting](monitoring.html#symptoms-based-alerting)
    -   [What To Monitor](monitoring.html#what-to-monitor)
    -   [What To Alert](monitoring.html#what-to-alert)
    -   [How to Monitor](monitoring.html#how-to-monitor)
    -   [How to Alert](monitoring.html#how-to-alert)
    -   [Events](monitoring.html#events)
    -   [Further Reading](monitoring.html#further-reading)
-   [Tendermint P2P Layer](peers.html)
    -   [Intro](peers.html#intro)
    -   [The peer reactor](peers.html#the-peer-reactor)
    -   [Operation Notes](peers.html#operation-notes)
    -   [The Sentry architecture](peers.html#the-sentry-architecture)
    -   [Sentry-Auto-Scaling](peers.html#sentry-auto-scaling)
-   [Systems Design](systems.html)
    -   [Why not Kubernetes?](systems.html#why-not-kubernetes)
    -   [Network topology](systems.html#network-topology)
    -   [Secret management](systems.html#secret-management)
-   [Security Engineering](security.html)
    -   [Single Sign On and 2FA](security.html#single-sign-on-and-2fa)
-   [Linux Best Practices](linux_config.html)
    -   [Configuration management](linux_config.html#configuration-management)
    -   [User authentication](linux_config.html#user-authentication)
    -   [Choice of distribution](linux_config.html#choice-of-distribution)
    -   [Custom software](linux_config.html#custom-software)
    -   [Docker images](linux_config.html#docker-images)
    -   [Network time](linux_config.html#network-time)
    -   [DNS resolvers](linux_config.html#dns-resolvers)
-   [Validator High-Availability](validator_ha.html)
    -   [Single Node Validator](validator_ha.html#single-node-validator)
    -   [Active-Standby Validator](validator_ha.html#active-standby-validator)
    -   [Active-Active Validator](validator_ha.html#active-active-validator)
-   [HSM for Signing](hsm.html)
    -   [Why use a HSM](hsm.html#why-use-a-hsm)
    -   [HSM implementations](hsm.html#hsm-implementations)
    -   [How to setup a Cosmos validator with Aiakos YubiHSM2 support](hsm.html#how-to-setup-a-cosmos-validator-with-aiakos-yubihsm2-support)
    -   [HSM hardware](hsm.html#hsm-hardware)
-   [Key Management](key_management.html)
    -   [Handling of the Account Key](key_management.html#handling-of-the-account-key)
-   [Testing your tooling](testing.html)
    -   [Case Study at Certus One](testing.html#case-study-at-certus-one)
    -   [How to deploy](testing.html#how-to-deploy)
-   [Building your tools and Cosmos](building.html)
    -   [Performing reproducible builds](building.html#performing-reproducible-builds)
    -   [Maintaining a mirror/fork](building.html#maintaining-a-mirror-fork)
-   [Business Continuity](business_continuity.html)
    -   [Shareholder agreements](business_continuity.html#shareholder-agreements)