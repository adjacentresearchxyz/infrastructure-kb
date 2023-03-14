<!-- ---
title: Security Engineering
layout: home
parent: Concepts
--- -->

# Security Engineering[](#security-engineering "Permalink to this headline")

Someone once said that, paraphrasing:

> While most of systems engineering is about making computers do things, security is just about the opposite - keep your computers from doing _anything else_.

While much of the public focus on security engineering is about the low-level technical details - finding and exploiting bugs, cryptographic algorithms, code signing, chains of trust, and so on, these - while important - are only one piece of the puzzle.

Some of the hardest parts of security in most companies are actually not of the technical nature - from an academic point of view, most everyday security problems are downright boring - but making trade-offs, assessing risks, understanding your adversaries, and writing procedures, policies and checklists (also boring, both academically and practically).

Not every company needs a team of world-class security carrying out original research (though it certainly helps), but every company needs to understand their risks and plan accordingly.

That’s not to say that the technology isn’t important - quite the opposite. Validators, in particular, operate at a pretty low level, cryptographically speaking, and _do_ need to understand the cryptographic primitives that they’re using, especially if they build their own HSM integrations and similar pieces of security-critical code. Cutting-edge security technology makes risk mitigation much easier and can dramatically reduce costs. However, even the best security technology will fail if it’s not supported by a proper business framework.

Most security failures are not due to clever attackers discovering a complicated timing attack in your HMAC algorithm or exfiltrating data by [modulating your system bus](https://github.com/fulldecent/system-bus-radio), but dumb mistakes, human error in general and policy failures.

Remember the Equifax breach? They failed to mitigate a vulnerability that had been publicly known for _months_. Their fancy IDS which detected the breach had been turned off because the CSO thought the alerts were too noisy. Those were management failures.

Unfortunately, this is how most breaches happen, and - barring the occasional smart contract troubles - blockchain companies make no exception.

Most blue team security measures can be (roughly) categorized into these three disciplines:

1.  **Prevention** - properly hardening your systems and organization to prevent incidents from happening in the first place, just like sturdy walls and secure locks are the best defense against burglary, it’s hard to overstate how important proper systems design is for security. Most security issues are mundane and easily prevented.
    
2.  **Detection** - like with a good home alarm system, detecting an incident is the next best thing after preventing it. Studies say that the vast majority of attacks go on undetected for months until they’re discovered, often due to a mistake on the attacker’s part.
    
3.  **Response** - properly responding to a security incident in an art in itself and can dramatically alter the outcome. Compromises are inevitable, and you need to be able to competently deal with them as they happen.
    

The reality is that any sufficiently large network will have some compromised nodes at any given time, at the very least due to sheer scale and laws of probability (with a pinch of organizational incompetence inherent in large organizations sprinkled on top). It’s a common opinion that we have collectively failed at both incident prevention and detection and are now living in the age of incident response.

However, at the scale of a typical validator operation, minimizing risks by achieving excellence at all three disciplines is both possible and reasonable. There’s - quite literally - a lot at stake, and a single compromise can easily kill a validator company.

## Single Sign On and 2FA[](#single-sign-on-and-2fa "Permalink to this headline")

You need a secure single-sign on system. Unless you’re a large company who can afford to run and maintain their own SSO system, you should use Google’s GSuite.

Enforce two-factor authentication for all users. You should use two U2F tokens per user (they’re the only kind of 2FA authentication that can’t be phished).

Do not use SMS authentication - mobile phone providers are [un]surprisingly easy to social engineer into sending a replacement SIM card to someone else.

Shared passwords are evil.

Only ever use shared passwords as a last resort. Certus One has so far gotten away without a single shared password! Many third party suppliers support multiple user accounts - create an individual account for each team member.

Have a company wifi router? Either use WPA2 Enterprise, or treat it like a public hotspot.

Keep track of all shared passwords and be diligent about changing all of them when a team member leaves who had access to them, and deleting accounts at third parties.

If an application does not support SSO login, much of the time, you can use [oauth2_proxy](https://github.com/bitly/oauth2_proxy) or [keycloak-gatekeeper](https://github.com/keycloak/keycloak-gatekeeper) as authenticating reverse proxies.