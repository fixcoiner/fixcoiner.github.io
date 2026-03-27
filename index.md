---
layout: default
title: Fixcoiner
---

# Fixcoiner

A public record of what I have done with Bitcoin infrastructure, networking, and nodes over the last two years.

This site is intended to become a detailed public disclosure of the work, the flaw, the infrastructure behind it, the costs, the failed disclosure attempts, and the evidence I gathered over the last two years. I want the public record to be complete enough that the scale, seriousness, and implications of the issue can be evaluated directly.

## Purpose

Two recent posts on X made it clear to me that it is time to fully disclose this work in public. One was Luke Dashjr arguing that the Bitcoin network is catastrophically centralized. The other was Jameson Lopp pointing to what appears to be a Sybil attack against the Bitcoin network. From my perspective, both observations reflect real conditions, and parts of both connect to work I have done over the last two years.

More than two years ago, I found myself with extra time and decided to contribute to something I had believed in for many years: Bitcoin. I did not come into this as a software developer, and I still do not claim to understand every layer of Bitcoin. What I do understand is production infrastructure: network, systems, security, and availability. That is where I focused. I approached Bitcoin nodes as infrastructure behind a critical system, and I started looking at them the way I would look at any other high-value production environment.

After a few weeks of digging in, I found a flaw that, as far as I could tell, had not been properly explored. In the networking layers around Bitcoin Core, assumptions had been made about how Internet numbering and resources work. Those assumptions were not accurate, and because of that, there was a serious weakness in how nodes decide to communicate with each other.

This stopped feeling like a normal bug report pretty quickly. It turned into a security dilemma. Once a vulnerability like this exists, and there is no trusted way to shut it down, everyone has an incentive to prove it, use it, defend against it, or get there before the other side does. That is part of what makes it dangerous. Even people who think they are acting defensively can end up feeding the same cycle.

Since then, I have spent the last two years trying to prove out the problem, understand its consequences, and work toward ways to correct it. My purpose throughout has been to resolve it and make the network better. With where Bitcoin is now, and with the current civil war around its direction, I think the right move is to disclose the details publicly and let the facts stand on their own.

## Journey

The path from discovery to disclosure was not linear. I went through repeated cycles of raising the issue, not being believed, being told to prove it, expanding the work to prove it, discussing it further, seeking grants and other funding and being denied, and then continuing to build out the evidence anyway. That cycle repeated until it led to where things stand today, with both sides of the current Bitcoin civil war now observing infrastructure I stood up to demonstrate the flaw.

I started with a single /24 of address space, all running as full Bitcoin nodes. From there I expanded to four /24s, and then to twelve /24s spread across three different ASNs.

## Costs

The cost side is where this becomes more serious than most people assume. A common reaction is to imagine that operating infrastructure at this scale must require a massive ongoing budget. In my experience, that assumption is wrong. The setup I built, which runs roughly 3,000 IPv4-reachable full Bitcoin nodes, can be operated for about $2,000 per month, with roughly $5,000 in one-time hardware cost using refurbished Dell servers.

That matters because it means the barrier to building very large visible node fleets is far lower than many people think. The important point is not any one hosting vendor or pricing detail. The important point is that colocation, transit, address space, and commodity hardware can be combined in a way that makes this infrastructure surprisingly inexpensive.

I believe that low cost is itself a major risk to Bitcoin. If thousands of publicly reachable nodes can be stood up and maintained this cheaply, then the network is more exposed to infrastructure-driven distortion than many people want to admit.

## Steps

The mechanics were not exotic. They were mostly standard infrastructure, applied in a way that the Bitcoin network was not prepared for.

1. Establish typical network infrastructure: BGP sessions with transit provider(s), route advertisements, downstream topology to the nodes, and the rest of the ordinary network engineering needed to support the footprint.
2. Stand up Proxmox on the servers with ZFS storage, ZFS deduplication, and the host ready to run LXC containers.
3. Deploy a Bitcoin node in an LXC container and let the full blockchain sync complete.
4. Stand up a forward proxy to seed the full public IP footprint onto the Bitcoin network, and policy-route TCP source port `8333` toward that proxy.
5. Create `12` to `36` linked clones of the synchronized Bitcoin node, using ZFS linked clones with deduplication enabled. I also strongly recommend forcing roughly `128G` to `192G` of ZFS ARC in memory.
6. Configure those Bitcoin nodes to use the outbound forward proxy for proper IP seeding. In my case, `dante` and `haproxy` worked well for distributing outbound connections across the available IP space.
7. Stand up `4` to `8` inbound reverse proxies, again with `haproxy`, listening on the public IP addresses and forwarding inbound connections back toward the `12` to `36` linked clones.
8. Let it run. Once it is built correctly, the amount of Bitcoin traffic it attracts makes the point on its own.

This page will continue to expand with more implementation detail, evidence, and analysis over time.

## Details & Playbooks

This section will hold the technical detail, operational notes, and repeatable playbooks.

## Status

This is the first draft of a larger public disclosure. I will keep expanding it section by section until the full record is laid out in public.
