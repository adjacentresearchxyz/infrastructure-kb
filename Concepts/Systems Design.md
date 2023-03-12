---
title: Systems Design
layout: home
---

# Systems Design[](#systems-design "Permalink to this headline")

The key to building and maintaining highly secure and reliable systems is **simplicity** - a good system will have nothing to take away, rather than nothing to add.

Each component you use you will have to secure, update and debug forever. Like open source projects which reject code not because it’s bad, but because they don’t have sufficient capacaties to maintain it, you need to be very careful about each piece of technology you introduce into your stack and carefully weight its benefits against the extra complexity it introduces.

Debugging and securing a system is an order of magnitude more complicated than building it. There’s a common saying that if you build the most clever system you can think of, you’re by definition unqualified to maintain it - and there’s a lot of truth to it.

## Why not Kubernetes?[](#why-not-kubernetes "Permalink to this headline")

We’ve heard about many validators who plan to use Kubernetes for their production setups.

We strongly recommend against the use of Kubernetes and similar technologies for your core validator operations - they solve problems validator operators don’t have. Validator core infrastructure are **pets, not cattle**. You can’t just deploy a cloud instance, you need to rent dedicated servers and plug HSMs into them. Even if you’re running an active-active setup like ours which tolerates full node outages, you’re unlikely to gain enough from Kubernetes to justify its costs. We recommend traditional configuration management tools like Ansible.

Even plain Docker usage should be carefully evaluated - while Docker itself is quite stable, even in 2018, the overlay filesystems and namespacing technology it uses haven’t stabilized yet (not for lack of trying, but any complicated piece of code in the kernel needs decades to mature; people are still finding bugs in ext4 to this day). Any large-scale production user of either Docker and Kubernetes will have tales of Docker daemon crashes in production, weird kernel issues that required node reboots and scheduler bugs that required them to read the Kubernetes source code at 3am in the morning. This is perfectly fine for the kind of stateless infrastructure Docker and Kubernetes are designed for - they are built to tolerate single node losses, or build systems. In fact, many production deployments can auto-update and reboot cluster nodes (like CoreOS/Container Linux does).

However, none of this applies to core validator infrastructure. cosmos-sdk compiles to a _single binary_, you don’t need Docker to deploy it (it’s quite useful for building it, though). You’re not going to need to scale your validator setup up and down across a fleet of thousands of machines. Being able to list all processes running on your nodes and knowing exactly what they’re doing is beautiful, and while it’s certainly possible with Kubernetes, it’s a lot easier with with a stripped-down, hardened node running nothing but your validator software.

Advanced security setups like eBPF seccomp policies, auditing, SELinux policies and just plain debugging get a lot easier when there are no kernel namespaces, Docker daemons, runC wrappers and overlay filesystems to reason about.

Both Docker and Kubernetes also add a lot of unnecessary attack surface (properly securing Kubernetes alone is pretty complicated - it’s a REST API which hands out omnipotent tokens which have root access to your nodes - that’s the opposite of _defense in depth_!).

Certus One has extensive experience using Docker, Kubernetes and Red Hat’s OpenShift k8s distribution in production and we’re confident that we could pull it off, just like we have no doubt that other teams will. However, we believe that a better result can be achieved without, with time better spent on security hardening and tooling.

That being said, we _do_ use Kubernetes for our auxiliary infrastructure like our monitoring stack, continous integration, testing setup and various other internal infrastructure needs. It’s perfect for that - our monitoring stack in particular consists of many small services talking to each other, and Kubernetes is perfect for that. We just don’t want it anywhere where its failure would affect the core of our business - running secure validators.

## Network topology[](#network-topology "Permalink to this headline")

We recommend a **flat network** topology where each node has its own unique IP address, with full end-to-end connectivity (only hindered by your firewall rules).

You should prefer a fully routed **L3 topology** over switched L2 topologies, especially those of the “magic SDN” variety which use a separate control plane (like OpenVSwitch, ZeroTier and most datacenter SDN technologies in general).

L2 topologies have no advantage over L3 routing for applications that have no special networking requirements. Their main users are service providers which need to span L2 network across their data centers to provide their customers with virtual networks. For everyone else, L3 networks are the better choice. L2 networks are brittle, hard to segment (they’re basically one large failure domain), and hard to debug.

L3 networks are built to scale, have well-defined interfaces and control plane protocols, and great troubleshooting tools (routing tables, traceroutes and BGP are more intuitive than an opaque SDN control plane which pretends to be a gigantic flat L2 network, but isn’t). The potential blast radius in a L3 network is much smaller than in a L2 network, where a stray spanning-tree packet can potentially take down an entire network.

For example, a GCE VPC is completely routed. Each virtual machine has its own transfer network with the VPC router, and the only broadcast packet you’ll ever see are the ones sent by either the router, or the instance. The VPC router maintains a /32 route for each VMs IP addresses. This is why you can’t set next hop routes in your virtual machines, but need to add them in the VPCs routing table - while it pretends to be a large /24 network, it’s in fact a routed network where each instance has its own little network.

Via VPC peerings, you can connect multiple networks to their respective IP ranges, which is “just” another entry in the VPC routing table pointing to the other VPC’s router.

Network address translation is best avoided. It adds an extra layer of complexity, makes it harder to attribute traffic once it left a NAT boundary and is another state machine on top of TCP and Tendermint’s consensus protocols.

It creates all sorts of operational headaches - you need to maintain a separate list of public/private IPs and port forwardings, you can’t easily whitelist traffic from single nodes once it crossed a NAT boundary (you would need to log NAT source ports), and so on.

**NAT is not a security tool**, firewalls are. You can achieve the same level of network isolation with and without NAT. It’s hard to avoid - most cloud provider’s VPC networks use RFC 1918 addresses internally with NAT for external connectivity, and you need _some_ sort of NAT at the network edges - but you should keep its use to a minimum and ensure that nodes can communicate without NAT inside of your network.

Kubernetes is another good example - each pod has its own IP address in a large flat network, which can be either routed or switched, depending on the network plugin you use - [Calico](https://www.projectcalico.org/) is what we use. Each pod can talk to each other across this flat network as long as the network policy allows it. However, pods need to talk to the outside world sometimes, so each node does outgoing NAT for the traffic originating from pods running on it. The outside world also needs to talk to some of the services running in the cluster, so there’s a load balancer which accepts traffic on its public IP address and load balances it to the internal IPs.

## Secret management[](#secret-management "Permalink to this headline")

**Avoid working with secrets**, especially bearer tokens, _especially_ if they can be used on public APIs without any additional layer of security (looking at you, public cloud platforms and k8s).

Use service-level certificate-based authentication to authenticate your services against a centralized secrets storage like [Hashicorp Vault](https://www.vaultproject.io/).

If possible, use Vault’s auto-provisioning feature which creates individual short-lived credentials for each service.

If you use stateless tokens like JWT, make sure to hardcode the signature algorithm and either use very short-lived tokens that expire after a few minutes, with stateful refresh tokens that you can invalidate, or another invalidation mechanism like revocation lists or a mechanism that invalidates all tokens issued before a certain timestamp (for emergencies).

Red Hat IDP/[Keycloak](https://www.keycloak.org/) is a good OAuth/JWT server which implements both short-lived tokens as well as not-before timestamps.

If you have a lot of services that need to authenticate at each other, you might reach a point where a service mesh like [Istio](https://istio.io/) is worth looking into.