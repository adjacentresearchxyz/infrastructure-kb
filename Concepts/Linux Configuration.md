---
title: Linux Configuration
layout: home
parent: Concepts
---

# Linux Best Practices[](#linux-best-practices "Permalink to this headline")

This article details some basic best practices for Linux server configuration.

## Configuration management[](#configuration-management "Permalink to this headline")

You should strictly do all systems configuration through config management - it’s a investment that pays off almost immediately. Some companies go as far as disabling SSH access, though we consider that a bit extreme (you still need SSH login to debug).

We recommend [Ansible](https://www.ansible.com/) for being both simple and powerful - it has a good learning curve, is easy to reason about and it’s the [fastest-growing](https://trends.google.com/trends/explore?date=all&q=ansible%20download,saltstack%20download,puppet%20download) tool with the largest community.

There are some systems like Puppet which are more powerful, but also a lot more complicated. We’ve seen a pattern in devops team where there’s only a few “Puppet wizards” who know how to use it properly, with everyone else avoiding it. Ansible is less sophisticated - its execution model is sequential and it has no concept of resources, but it’s also _much_ easier to use.

This turns out to be a significant advantage, since it greatly lowers the barrier to entry and ensure the team is actually using the configuration management tool rather than trying to bypass it or getting the team’s resident wizards to implement features for them.

Red Hat has bought Ansible and is moving many of its products away from Puppet.

## User authentication[](#user-authentication "Permalink to this headline")

Unless you have dozens of users, you should provision a **local user** - including a password - for each team member using your favorite configuration management tooling and add the users to your sudo group (i.e. `sudo` or `wheel`, depending on distro).

Make sure to use a password that’s not re-used for anything else, and only ever put the password hash into your config files (seems self-evident to a Linux admin, but did you know it’s impossible to set a user’s password hash on Windows?).

Make sure to use **consistent UIDs in the >70000** range for each user - this prevents UID conflicts and consistency across all machines makes (make sure your UID range is outside of what is configured in `/etc/login.defs`).

You need to treat each individual user like a root user - if the account were to be compromised, it would be trivial to escalate these privileges by adding a LD_PRELOAD backdoor (or similar) and waiting until the next time the user uses sudo.

Then, disable the root user’s password by setting an invalid password hash (usually `NP`).

This has a number of advantages:

-   It’s a lot easier to audit each individual user’s actions on the server (see auditing section in the security article). auditd can associate SSH keys to user sessions, but that takes some work and isn’t 100% accurate. The user’s login/audit UID (`auid`) will be carried along through sudo and su invocations.
-   You can have your configuration management tool log into the server using its own user account, allowing you to easily filter out audit events caused by deployment.
-   Users can log into a server’s local console using their own password.
-   If a team member leaves, you don’t need to change a shared root password.

If you have many different users, the overhead of adding local users on each machine will become notable and you might want to look into using something like [sssd](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/sssd), though that will introduce additional complexity - how to ensure that you can still log into servers without network?

### SSH authentication[](#ssh-authentication "Permalink to this headline")

Make sure to use public key authentication, and only public key authentication.

Have a redundant set of bastion/VPN hosts and restrict SSH access to these hosts (along with any other potentially dangerous services). Don’t use SSH agent forwarding for your bastion host, instead, forward SSH sessions _through_ the bastion host using plain OpenVPN, sshuttle or a `ProxyCommand` ssh_config directive.

If your team is large, you probably want to use something like [Teleport](https://github.com/gravitational/teleport) to tie SSH authentication and auditing into your existing, two-factor-secured single-sign-on solution.

However, if your team is small, just hand everyone two [YubiKey 4](https://www.yubico.com/product/yubikey-4-series/) tokens and use their OpenPGP feature for SSH authentication. Never use plain local SSH keys - they’re too easy to steal, and an attacker who has sufficient access to steal your keys can likely keylog the passphrase. They’re so cheap that there’s little excuse for not using them :-)

Don’t use fail2ban (it’s pointless with key auth and adds additional attack surface; Chinese bots shouldn’t be able to talk to your SSH service to begin with).

Todo

How to use a YubiKey 4 for SSH authentication

## Choice of distribution[](#choice-of-distribution "Permalink to this headline")

We recommend **Red Hat Enterprise Linux** (RHEL), **CentOS** or **Debian**, in that order.

Red Hat’s quality assurance is the best in the industry - they are very diligent about release notes and backwards compatibility within a major release.

They have an in-house kernel engineering team and are good at making sure their kernel works well on enterprise bare-metal server hardware. When the Spectre/Meltdown vulnerabilities hit, not only did they have a functional patch on day one, but they managed to develop it in-house despite not being able to discuss it with the Linux kernel community (due to a severely mismanaged vulnerability disclosure process).

Red Hat is generally the fastest to respond to security vulnerabilities and has writeups detailing impact, workarounds and available patches.

Red Hat’s support is top notch (once you get past the first level), and has direct access to Red Hat’s engineers. Needless to say, this is very valuable.

Licensing is a hassle, but we recommend buying RHEL support if you can afford it - at least for your critical hosts (no, we’re not getting any sales commissions ;-)).

[CentOS](https://www.centos.org/) is the community edition of RHEL. It used to be a separate project, but has since been assimilated by Red Hat. Due to the cooperation between Red Hat and CentOS, it barely lags behind the upstream releases. You don’t get any support, lesser assurances as far as supply-chain security goes and the packages have no security metadata (so you can’t set it to auto-install security updates, but you should have a proper patch management procedure, anyway), but it’s otherwise identical to RHEL.

RHEL and CentOS have [very long support periods](https://access.redhat.com/support/policy/updates/errata) of more than ten years - RHEL 6, released in 2010, is getting updates until 2024.

[Debian](https://en.wikipedia.org/wiki/Debian) is _the_ community server linux distribution and the other pole of the RPM/DEB ecosystem split. While it has a number of corporate sponsors nowadays - companies which use Debian and support it - it’s a volunteer-driven community project at its core, developed entirely in the open. The project is community-run, has a mature governance model and community manifesto, is independent of any single company, and - naturally - much less subject to corporate agendas than other distributions. It’s a conservative, slow-moving distribution just like RHEL.

Security updates are usually fast, but depend a lot on individual maintainers and whether or not a particular issue is being handled by their security team. We’ve seen situations where less widely used packages with known security issues weren’t updated since its maintainer was unreachable, so be prepared to package and deploy your own security fixes (this applies to any distro - you should be able to build and sign your own packages).

[Debian releases](https://wiki.debian.org/DebianReleases) are more frequent than RHEL and are only supported for 2-3 years, with the [Debian LTS](https://wiki.debian.org/LTS/) project continuing support for two more years.

Debian has good QA and is a very solid choice, especially if you have team members who have a lot of experience with Debian-based distributions like Ubuntu, and none with RPM. Don’t expect it to be as polished as RHEL. Many things which work “out of the box” in RHEL are harder on Debian, especially enterprise features like auditing.

### Why not Ubuntu?[](#why-not-ubuntu "Permalink to this headline")

We do not recommend Ubuntu for production, especially if you’re going to run on it bare metal hardware. Ubuntu is a very common choice for devops teams, especially startups, so we’ll elaborate a little to explain why.

We have had bad experiences with their quality assurance and have seen things breaking horribly in LTS releases. They’re still doing a great job, considering that they’re a much smaller company than Red Hat, but it’s not at the same level. Often, relatively simple bugs take years to fix.

Canonical also has a habit of building their own solutions without ensuring backing of the wider Linux community. Those solutions then fail to be adopted by the community.

This is a hit and miss strategy, and they often miss, leaving their customers scrambling to migrate to whatever the community settled on.

This happened multiple times in the recent past:

-   [Eucalyptus](https://en.wikipedia.org/wiki/Eucalyptus_(software)) was shelved in favor of [OpenStack](https://en.wikipedia.org/wiki/OpenStack).
-   [Bazaar](https://en.wikipedia.org/wiki/GNU_Bazaar) - though some people still swear by it - was superseded by Git.
-   [Upstart](https://en.wikipedia.org/wiki/Upstart) ended up being abandoned in favor of [systemd](https://en.wikipedia.org/wiki/Systemd) (Ubuntu users had to first migrate to Upstart, then migrate _again_ to systemd - wasn’t fun!).
-   [Unity](https://en.wikipedia.org/wiki/Unity_(user_interface)) was abandoned for [Gnome 3](https://en.wikipedia.org/wiki/GNOME_Shell).
-   Ubuntu 14.04 LTS wasn’t released with the stable Linux kernel release at the time, but they chose to maintain their own stable kernel.
-   The [Mir](https://en.wikipedia.org/wiki/Mir_(software)) display server did not gain adoption and was replaced by [Wayland](https://en.wikipedia.org/wiki/Wayland_(display_server_protocol)).
-   [Ubuntu One](https://en.wikipedia.org/wiki/Ubuntu_One) and [Ubuntu Phone](https://en.wikipedia.org/wiki/Ubuntu_Touch) were discontinued.
-   [Juju](https://en.wikipedia.org/wiki/Juju_(software)) is basically unheard of outside the Ubuntu ecosystem, and will probably become irrelevant with the adoption of Kubernetes and Ansible. It’s also pretty crude (basically, a stateful collection of bash scripts) compared to something like Helm charts.

It’s a shame some of these didn’t work out and hopefully they [they learnt from it](https://blog.ubuntu.com/2017/04/05/growing-ubuntu-for-cloud-and-iot-rather-than-phone-and-convergence) - the ecosystem would greatly benefit from better collaboration between Canonical and Red Hat.

The latest case is [Snappy](https://snapcraft.io/) vs [Flatpak](https://flatpak.org/) - right now, Snappy has a larger user base and commercial backing, but Flatpak is technically superior with OSTree, Portals and Wayland integration, focusing on deep integration with desktop applications, whereas Snappy is trying to be a general-purpose packaging mechanism, with an unclear relationship with Docker (which, for better or worse, is on track to become the actual general-purpose packaging mechanism).

Part of the issue is that Canonical often fails to engage the community or doesn’t wait for community consensus. Their [CLA](https://en.wikipedia.org/wiki/Contributor_License_Agreement) adds a lot of friction and their Launchpad platform has bad usability compared to modern platforms, while many Red Hat projects are developed on GitHub, which people are more familiar with (they also [have no CLA](https://blog.openshift.com/keep-calm-and-merge-on-lowering-barriers-to-open-source-contributions-with-apache-v2/) for most projects).

That being said, the positive influence of Ubuntu Desktop on the desktop Linux ecosystem is hard to overstate. They were the first distribution to popularize desktop Linux and did a lot of hardware compatibility and user experience work. For the longest time, Ubuntu was the only major distribution which had acceptable font rendering out-of-the box. It’s the only popular desktop Linux distribution that offers long-term support.

### Applications vs OS divide[](#applications-vs-os-divide "Permalink to this headline")

One common pain point with conservative distributions like Debian and RHEL is their slow pace - LTS distributions make promises of backwards compatibility, and any major upgrades within a major release are hard-to-impossible without breaking compatibility.

For example, RHEL has - to many people’s surprise - been rebased on a newer OpenSSL release in 7 .4 in order to to enable HTTP/2. They had to rebuild half of all packages that depended on OpenSSL, which was a massive effort, and test them all to ensure they all were still working, and the only reason this was possible to begin with was that OpenSSL itself didn’t break backwards compatibility or change their API.

Having a slow-moving, rock-solid operating system is critical for production, however, it’s painful for development. For this reason, we believe that attempts to develop and deploy applications against the distribution’s runtime and library are misguided (with the obvious exception of systems software which is part of the distribution). It’s impossible to reconcile the stability requirements of the operations side with developer’s need for the latest and greatest programming languages and libraries, both of which are reasonable.

The right solution is to **completely separate application runtime and operating system**. A best example of this approach is Golang - Go binaries have no userland dependencies except for minimal usage of libc, and even that can be disabled. This also applies to the Go compiler - you can download the latest Go compiler and run it on any Linux kernel >2.6.23.

The same thing is possible with any other language runtime.

Docker makes this even easier - containers ship their own userspace with minimal interaction with the host OS (mostly through the kernel). You can run a rock-solid host OS, while running Fedora or Ubuntu containers with the latest and greatest

## Custom software[](#custom-software "Permalink to this headline")

Try to **stick with your Linux disto repositories**. Your Linux distro takes care of reviewing and maintaining the software, a tasks that would otherwise fall on you.

If you need custom software outside of your distro’s repositories (and you will, sooner or later), do not install it manually/using _make install_. Ideally, you wouldn’t even have compilers on your production systems! Rather, **build custom packages** or Docker containers (whatever you’re more comfortable with - building DEB and RPM packages can be intimidating)

If you’re familiar with it, Certus One recommends a custom DEB/RPM package repository and signing infrastructure since it allows you to rebuild and deploy core packages, and doesn’t require any extra dependencies on your nodes. However, there’s nothing wrong with standardizing on Docker builds, especially if your team is familiar with Docker and you’re using it anyway, so you can avoid maintaining two separate build pipelines and delivery methods.

You also need to keep track of all custom software you use in a **software library** - this can be as simple as a spreadsheet or internal Wiki page, but it’s crucial to have.

The software library should list, at the very least:

-   The software’s author and source.
-   The team member who is the internal “package maintainer”.
-   Instructions for building the package.
-   A mailing list or comparable channel for **security bulletins** - since your distro isn’t going to look out for security vulnerabilities, you will have to do so yourself.

Having processes for maintaining and patching custom packages is very important - they are easily forgotten about.

## Docker images[](#docker-images "Permalink to this headline")

**Build all Docker images from scratch**. We strongly recommend against using pre-built Docker images in production, for multiple reasons:

-   Experience shows that you’ll sooner or later need to modify the Docker containers you use, and you should be prepared by having a proper build .

Certus One uses the OpenShift/[okd](https://www.okd.io/) build system and registry for container builds.

If you build your own images, try to use the same operating system base image as your host OS, or at least the same operating system family (i.e. Fedora on CentOS, more recent Ubuntu releases on Ubuntu LTS). This makes it a lot easier for your team to debug containers since they’ll be familiar with the operating system, and it avoids introducing additional security dependencies.

## Network time[](#network-time "Permalink to this headline")

We recommend using chronyd, a modern (and safer) alternative to ntpd. Many Linux distributions come with chronyd as the default. Do not use systemd-timedated - it’s great for

Time-keeping is extremely important for any modern protocol. Some databases like [CockroachDB](https://www.cockroachlabs.com/) even rely on nanosecond-precision timestamps for global transaction ordering (and therefore consistency).

If you can, configure each data center’s local time servers as preferred peers. Many high-end data centers will even have a local stratum This protects you against outside NTP

In case you run a database like CockroachDB, having a high-accuracy local time source is crucial.

## DNS resolvers[](#dns-resolvers "Permalink to this headline")

Keep in mind that Linux has no DNS caching - it consults the resolver configured in `/etc/resolv.conf` on each lookup. Depending on the amount of lookups your application does, this can be really expensive. You need to ensure that you either:

-   ..have a resolver very close in your local network (<1ms)
-   ..or run a caching stub resolver on each host.

If you have a local resolver, use that. Any cloud or datacenter provider will have local resolvers, and it’s one less thing you need to care about and deal with when it breaks.

However, if you do a lot of DNS queries or need DNSSEC validation, or your local resolver is untrustworthy/unreliable, by all means, run a local forwarding resolver on each node and familiarize yourself with it.

Here’s some common choices:

**nscd**

nscd is a generic caching daemon not only for DNS, but any [NSS](https://en.wikipedia.org/wiki/Name_Service_Switch) names, including local users and groups. It’s present on most systems, though seldom enabled by default. We do not recommend to use it, and to disable its DNS caching if you do use it for other reasons.

It has very simplistic positive and negative caches, disregards the DNS TTL and many other parts of the DNS RFCs and likes to treat lookup failures as negative responses and cache them, resulting in hard-to-debug issues.

We don’t recommend its use. If you need NSS caching, use the modern [sssd](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/sssd) instead.

**systemd-resolved**

systemd-resolved does more than just manage /etc/resolv.conf - it also has its own NSS resolver implementation, nss-resolve. You can think of it as an advanced nscd that understands DNS TTLs and DNSSEC, but has little customization or security features on top of that.

In particular, it’s designed as a _stub resolver_ that forwards to a real resolver in the same network, and is not particularly resistant against network-level attacks. In particular, they do not employ source port randomization (which makes it easier to race against the reply from the upstream resolver) or bother validating certificates with DNS-over-TLS.

We recommend systemd-resolved if all you need it caching and DNSSEC validation, and want to avoid pulling in an extra dependency like dnsmasq, and your risk model excludes or accepts attackers in the local network (as a validator, it probably doesn’t, and you might not want to use resolved).

**dnsmasq**

[dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) is the most commonly used forwarding resolver (and DHCP server - make sure to disable that!). It’s well-audited, standards-compliant, DNSSEC-validating and has none of the drawbacks that systemd-resolved has, though it’s “just” a local resolver without fancy NSS integration.

We recommend dnsmasq.

**unbound**

[Unbound](https://www.nlnetlabs.nl/projects/unbound/about/) is a full recursive DNSSEC-validating resolver which is able resolve domains by itself (by querying the root zone), without forwarding to a higher-tier resolver. While you can use it as a forwarding resolver, there’s little point in doing so. However, if you _do_ want to run a full resolver, Unbound is what we recommend.

### DNS-over-TLS[¶](#dns-over-tls "Permalink to this headline")