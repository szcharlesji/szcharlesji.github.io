---
layout: single
title: "Instant Referral Payouts Without Platform Fees"
date: 2023-12-16
categories: [web3, product]
---

I spent three hours debugging why referrers weren't getting paid. The smart contract worked perfectly in tests. Gas estimations were fine. But when I deployed to testnet and made a real purchase, the referrer balance stayed at zero.

The problem? I was sending the commission _after_ the NFT mint. If the mint failed, the whole transaction reverted—including the payment. Commissions were being paid and immediately undone, all in the same block.

## What I'm Building

A referral network where payouts happen instantly. Not "within 24 hours" instant. Not "pending review" instant. Actual instant—the referrer receives ETH in the same transaction where the buyer purchases.

No escrow. No manual releases. No support tickets asking "where's my commission?"

The flow is dead simple: Generate a referral code, share it, earn 5-15% commission the moment someone uses it. Everything is handled by smart contracts on Base.

## The Instant Payout Problem

Here's the tricky part. In traditional systems, you can separate concerns: process the purchase, then handle payouts asynchronously. In a smart contract, everything happens atomically. If any step fails, everything reverts.

My first attempt looked like this:

```javascript
function purchaseAccessPass(bytes32 referralCode) external payable {
    // 1. Mint NFT to buyer
    uint256 tokenId = accessPassNFT.mint(msg.sender);

    // 2. Calculate commission
    (uint256 commission, , ) = commissionEngine.calculateCommission(
        referrer,
        msg.value
    );

    // 3. Pay referrer
    (bool success, ) = referrer.call{value: commission}("");
    require(success, "Payment failed");

    // 4. Record referral
    registry.recordReferral(referralCode, msg.sender, commission);
}
```

This failed in production because if the `referrer.call` hit a contract with fallback logic that ran out of gas, the entire purchase reverted. Buyer lost their money temporarily, NFT didn't mint, and I got panicked messages.

The fix was reordering operations and adding proper error boundaries:

```javascript
// Pay referrer FIRST, before any complex state changes
(bool success, ) = referrer.call{value: commission}("");
require(success, "Referrer payment failed");

// Then update state
registry.recordReferral(referralCode, msg.sender, commission);

// Finally mint NFT (most complex operation)
uint256 tokenId = accessPassNFT.mint(msg.sender);
```

Now if the NFT mint fails, the referrer still gets paid. The purchase reverts, but we can retry without losing commission logic.

## Making Crypto Invisible

The instant payout works great. But then I watched someone try to use it.

"Connect your wallet to get started."

30 seconds of clicking around. "What's MetaMask?"

Another minute. "Do I need to buy crypto first?"

Then they gave up.

This is the real problem with consumer crypto. The technology works. The economics are better than Web2 platforms. But asking normal people to install MetaMask and understand gas fees is a non-starter.

I'm exploring embedded wallets next. Sign in with Google, auto-create a wallet behind the scenes, fund it with a credit card. The user never sees a seed phrase or knows they're using blockchain.

The goal is: click link → buy thing → referrer gets paid. No wallet setup, no "switch to Base Sepolia," no manual transaction approval.

## What Actually Shipped

The core system has four contracts:

**ReferralRegistry** - Generates unique codes, tracks referrer stats
**CommissionEngine** - Calculates tiered payouts (5% → 10% → 15%)
**PaymentProcessor** - Handles purchases and instant settlements
**NFT contracts** - Access passes for buyers, milestone badges for referrers

When you refer 10 people, you automatically get a Bronze badge NFT. At 50, Silver. At 100, Gold. These aren't just cosmetic—each tier unlocks higher commission rates.

The math is simple: refer more, earn more per referral. At 100+ referrals, you're taking home 15% of every purchase versus Fiverr's ~37.5% cut on the freelancer side alone.

![Referral dashboard showing stats and earnings](/assets/images/referral-dashboard.png)

![Leaderboard showing top referrers](/assets/images/referral-leaderboard.png)

## What's Next

The embedded wallet integration is the unlock. I'm testing Privy and Dynamic to see which has better conversion. Early tests show 70% of users complete signup when they can use Google instead of MetaMask.

Also adding email notifications when you earn commission. The instant payout is great, but if you don't check your wallet, you don't know you got paid. A simple "You earned 0.015 ETH from a referral" email makes it feel more real.

Still lots to figure out on dispute resolution and milestone tracking at scale. But the core loop works: generate code, share link, get paid instantly. No platform taking 20%. No week-long payout delays.

**Source code:** [github.com/szcharlesji/referral-network](https://github.com/szcharlesji/referral-network)
