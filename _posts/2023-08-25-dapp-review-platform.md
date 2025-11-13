---
layout: single
title: "I Built Yelp for Crypto Apps (On Sui, With Move)"
date: 2023-08-25
categories: [web3, sui]
---

Learning a new smart contract language to build a review platform seemed unnecessary. Then I tried doing it on Ethereum and the gas costs killed the project before it launched.

Writing a review on Ethereum mainnet: $15. Upvoting someone else's review: $8. For a product review. The economics made zero sense.

So I learned Move and deployed on Sui instead. Now reviews cost $0.02 and the whole thing actually works.

## What I Built

A review platform for Sui dApps with five-dimensional ratings:

- Security (30%)
- Usability (25%)
- Performance (20%)
- Documentation (15%)
- Innovation (10%)

Each dApp gets a composite score from 0-100 calculated as a weighted average. Users can upvote or downvote reviews, changing their vote anytime. Reviews are immutable once posted, but the community ranking them isn't.

The anti-spam rule: one review per user per dApp. You can't flood the platform with fake reviews because each Sui address can only review a specific dApp once.

![Submitting a review with multi-dimensional ratings](/assets/images/review-submit.png)

## Learning Move Was Weird

Coming from Solidity, Move felt backwards. No global state. Everything is object-based. You can't just reference a mapping—you have to pass objects around.

This is a Sui review structure:

```move
public struct Review has key, store {
    id: UID,
    dapp_id: ID,
    reviewer: address,
    security_score: u64,
    usability_score: u64,
    performance_score: u64,
    documentation_score: u64,
    innovation_score: u64,
    comment: String,
    upvotes: u64,
    downvotes: u64,
    timestamp: u64,
}
```

To create a review, users call:

```move
public entry fun create_review(
    registry: &mut DAppRegistry,
    dapp_id: ID,
    security: u64,
    usability: u64,
    performance: u64,
    documentation: u64,
    innovation: u64,
    comment: String,
    ctx: &mut TxContext
) {
    // Validate scores are 0-100
    assert!(security <= 100, EInvalidScore);
    assert!(usability <= 100, EInvalidScore);
    // ... other validations

    let review = Review {
        id: object::new(ctx),
        dapp_id,
        reviewer: tx_context::sender(ctx),
        security_score: security,
        usability_score: usability,
        performance_score: performance,
        documentation_score: documentation,
        innovation_score: innovation,
        comment,
        upvotes: 0,
        downvotes: 0,
        timestamp: tx_context::epoch(ctx),
    };

    transfer::public_share_object(review);
}
```

Everything is an object. The review is an object. The registry is an object. You share objects to make them globally accessible.

It took me three days to stop thinking in Solidity patterns and start thinking in Move's object model.

## What Actually Works

The composite scoring system is surprisingly good. Most review platforms just average all ratings equally, but security should matter more than "innovation" for financial dApps.

Weighting security at 30% means a dApp with amazing UX but sketchy security still gets a mediocre score. That feels right.

Allowing vote changes was controversial but necessary. Someone upvotes a review, then the dApp fixes the issue mentioned—they should be able to change their vote to reflect that.

Gas costs on Sui are absurdly low. Creating a review: ~$0.02. Upvoting: ~$0.005. I could never build this on Ethereum without making it unusable.

![Analytics dashboard showing review statistics](/assets/images/review-analytics.png)

## Why Sui For This

Move's object model makes sense for reviews. Each review is an independent object that can be queried, voted on, and referenced without touching global state.

The parallel execution means multiple people can upvote different reviews simultaneously without conflicts. On Ethereum, everything would serialize and cost way more gas.

And the economics just work. $0.02 per review is impulse-buy territory. $15 per review is "I'll write this review never."

I'm planning to add zkLogin so people can sign in with Google instead of managing Sui wallets. The tech exists, I just haven't implemented it yet.

**Source code:** [github.com/szcharlesji/decentralized-rating](https://github.com/szcharlesji/decentralized-rating)
