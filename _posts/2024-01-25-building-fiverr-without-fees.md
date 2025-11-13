---
layout: single
title: "I Built Fiverr Without the 20% Fees (Here's What Actually Happened)"
date: 2024-01-25
categories: [web3, product]
---

Last month, I paid $12.50 in fees on a $50 freelance job. The freelancer lost another $12.50 to Fiverr's service fee. We both paid 25% to the platform for... what exactly? Holding money in escrow and showing my task in search results?

I figured I could build that with a smart contract. So I did.

## What I Built

**Bounty Board** is a task marketplace where clients and freelancers transact directly through Ethereum escrow. No middleman. Just a 2.5% platform fee instead of Fiverr's 20-25%. Built on Base so transactions cost about three-tenths of a cent. Payments are instant when approved, and the escrow is fully automatic.

Here's the flow: Client posts a task with crypto reward. Funds lock in the smart contract immediately. Freelancer claims the task, completes work, submits proof. Client approves and the freelancer gets paid instantly. But here's the innovation - if the client doesn't respond in 48 hours, payment releases automatically.

![Creating a bounty on the platform](/assets/images/bounty-create.png)

That last part is key. In Web2, forgotten escrow releases mean support tickets and frustrated freelancers. In Web3, I just made it automatic. No human intervention needed.

![How the bounty system works](/assets/images/bounty-howitworks.png)

## The Hard Parts Nobody Talks About

Wallet onboarding sucks. During testing, 30% of users couldn't figure out MetaMask. This is the real barrier to consumer crypto - not gas fees, not transaction speed. It's "what the hell is a seed phrase?" I spent hours watching test users struggle with something I thought was basic.

Explaining smart contracts is harder than building them. I spent more time writing tooltip copy than writing Solidity. Users don't care about "trustless escrow" - they care about "you'll definitely get paid." Every piece of blockchain jargon I removed increased conversion.

Gas spikes still happen, even on Base. Most transactions cost $0.003, but during network congestion I saw $0.15. That's 50x variance. For a $10 task, that matters. I can't tell users "don't worry about fees" when they're unpredictable.

## What Actually Works

The 48-hour auto-approve solved my biggest worry. I thought clients would abuse it by never reviewing work. Turns out, knowing there's a deadline makes people actually review on time. It's the same psychology as return windows - urgency creates action.

USDC payments made everything clearer. Early builds used ETH and users constantly asked "how much is that in real money?" Switching to USDC meant everyone understood exactly what they were earning. Small change, massive impact.

Base economics made micro-tasks viable. On Ethereum mainnet, a $5 task with a $3 gas fee is dead on arrival. On Base, a $5 task with a $0.003 gas fee actually works. This unlocked a whole category of small jobs that make sense for both sides.

## Why I'm Building This Anyway

Because the economics are undeniable. A freelancer earning $50 should get $47.50, not $37.50. That $10 difference pays for groceries. It's real money being extracted by platforms that, honestly, don't do that much.

The infrastructure exists to cut them out. Smart contracts can hold escrow. Blockchains can settle payments. The only missing piece is making it not feel like "crypto." That's solvable - Venmo felt weird in 2012 too.

I know the onboarding problems seem insurmountable right now. But I've been thinking about this a lot. The solution isn't better documentation or more tutorials. It's removing the need for users to know they're using crypto at all.

## What's Next

I'm exploring Privy for social login with embedded wallets. Let people sign in with Google, auto-create a wallet in the background, and never mention the word "blockchain." The tech should be invisible.

Also rebuilding the UI mobile-first. Most gig workers use phones, and my current desktop-focused design shows. If this is going to work, it needs to work where people actually are.

Still need to fix dispute resolution (too manual right now) and add milestone payments for bigger projects. But the core loop works. When someone completes a task and gets paid 2 seconds later, they get it. That moment of "holy shit, no payment processing?" is what I'm building for.

**Source code:** [github.com/szcharlesji/bounty-board](https://github.com/szcharlesji/bounty-board)
