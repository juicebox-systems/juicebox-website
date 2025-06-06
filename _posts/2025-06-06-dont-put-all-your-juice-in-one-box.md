---
layout: post
title:  "Don’t Put All Your Juice in One Box"
description: "At Juicebox, we believe key recovery should be secure, user friendly, and actually… work. That means it has to be more than cryptographic theater. It has to reflect the real world, where systems get hacked, people make mistakes, and trust boundaries aren’t just vibes."
date:   2025-06-06 09:00:00 -0400
author-name: Nora Trapp
author-github: imperiopolis
---
<div class="info-box" onclick="window.location.href='https://github.com/juicebox-systems/'">
	Juicebox is an open-source encryption key recovery protocol that provides high security coupled with a user-friendly design, to make encryption further accessible to larger numbers of people. Juicebox leverages programmable HSMs, distributed cryptography, and a user-friendly PIN-based recovery process to simplify key recovery without compromising security.
</div>

At Juicebox, we believe key recovery should be secure, user friendly, and actually… work. That means it has to be more than cryptographic theater. It has to reflect the real world, where systems get hacked, people make mistakes, and trust boundaries aren’t just _vibes_.

Good security, like good juice, comes from mixing the right ingredients, not overusing one. In the [Juicebox protocol](https://juicebox.xyz/assets/whitepapers/juiceboxprotocol_revision7_20230807.pdf), the key “ingredients” are independent trust boundaries. Juicebox is built on the idea that no single organization should hold all the keys (literally) to user secrets. 
<!--more-->

![Trust Boundaries](/assets/images/trust-boundaries.png)

At its heart, Juicebox uses a threshold cryptography approach to split a user's secret across multiple realms operated by different parties. Each realm holds only an encrypted fragment of the secret, and only a quorum of them together can recover the key. This design establishes _trust boundaries_ – essentially, lines that delineate who has access to each piece of the secret. 

A realm’s trust boundary is the line around who you'd need to compromise to get at that realm’s data. For this system to work, each realm must live within an independent trust boundary, typically by being run by a different organization or on fundamentally different infrastructure.

## The Illusion of “Boundaries”

It’s tempting to think that splitting trust across different services you own and operate is good enough. After all, they might live on different machines, use different credentials, or even run different code. But in reality, real attackers don’t care about your service boundaries. They care about organizational ones.

If a single subpoena, exploit, or admin mistake compromises all your realms at once, they’re not independent realms, they’re aliases. From the attacker’s perspective, it’s one big juicy target.

This isn’t paranoia. This is why [Juicebox was designed to rely on diverse, independently operated realms](https://juicebox.xyz/blog/key-to-simplicity-squeezing-the-hassle-out-of-encryption-key-recovery). When you deploy three realms and control all of them, you haven’t distributed trust, you’ve replicated infrastructure. And replication != distribution.

## Real Trust Looks Like Separation

The ideal Juicebox deployment distributes both control and infrastructure. You need separation of operators, separation of environments, and separation of responsibilities.

Some apps have started to deploy implementations relying on realms operated by independent organizations, which is a great start. When two different entities each hold part of the user’s secret, and neither can unilaterally reconstruct it, you’ve taken a step toward meaningful trust distribution.

But even a 2-of-2 setup, where both realms must agree to recover, isn’t ideal. It introduces brittleness: lose one, and you’ve lost everything. There’s no room for downtime, migration, or error.

Juicebox was designed with this in mind. Ideally, we’d recommend running a 2-of-3 or 3-of-5 configuration, mixing software realms run by separate orgs, hardware-backed realms (HSMs with remote attestation), and cloud-diverse deployments (AWS, GCP, on-prem).

This gives you fault tolerance, subpoena resistance, and true organizational separation, all while keeping recovery snappy and secure.

## PINs Are Just the Pulp

A common question we hear is: “Why use 4-digit PINs?” or “Is Argon2 strong enough?”

The important thing to remember is: PINs are not your primary line of defense. The realms are.

Every realm (hardware or software) enforces strict rate limits on recovery attempts. The number of guesses a client can make is controlled and your secret self destructs as soon as a predefined limit has been reached. This means, when operating as designed, no offline dictionary attack is possible. And a compromised realm can’t bypass this – it takes a quorum to verify a PIN is even correct. Even a weak PIN, when rate-limited and split across independently operated realms, offers strong protection.

And importantly, Juicebox doesn’t limit users to weak PINs – the protocol allows for using a passphrase of whatever length and complexity satisfies a given user’s security requirements. So while high-entropy passphrases are always an option, _Juicebox is designed to work even when users don’t choose them_. It’s important to provide that choice so a user can decide how heavily they wish to lean on distributed trust alone. For each of us, and each service we interact with, there’s likely a middle ground between security and convenience.

All of this is not to say hashing doesn’t matter. We use strong hashes like Argon2 for secondary resistance, but if an attacker can’t even make 10 guesses, the hash is mostly moot.

A PIN in Juicebox is more like a possession proof: something only the rightful owner knows, that when implemented as designed gets checked only after a gauntlet of verifications across organizational boundaries. If someone’s guessing it, they’re already past multiple lines of defense. And even then, they’ll get rate-limited into oblivion before they get close.

## The Recipe for Privacy Requires More Than Just Cryptography

It’s not enough to use threshold cryptography. The _social architecture_ matters just as much. One sip of the Juicebox doesn’t magically shield your secrets. It’s a protocol that works because it draws sharp boundaries between who holds what, and who can do what.

Realms are only as independent as the humans (and hardware) behind them.

If you’re deploying Juicebox, ask yourself:

* Who runs each realm?
* Who hosts each realm?
* Could one incident compromise them all?

If the answers to any of those questions intersect, it’s time to go back to the blender.

---

Want help designing or refining your deployment? We're [happy to chat](mailto:contact@trappdesign.net,d@juicebox.xyz) about how to use Juicebox in a way that actually delivers on its promises.

Juicebox was created by Alex Bochannek, Simon Fell, Moxie Marlinspike, Diego Ongaro, Daniela Perlein, and Nora Trapp. The project received valuable feedback from Trevor Perrin and was audited by NCC in June 2023.
