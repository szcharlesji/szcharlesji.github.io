---
layout: single
title: "Creator Tips With Zero Platform Fees (Just ENS + USDC)"
date: 2023-05-17
categories: [web3, creator-economy]
---

Ko-fi takes 0% fees but charges for features. Patreon takes 5-12% depending on tier. Buy Me a Coffee takes 5%.

I thought: what if there was actually zero? No hidden charges, no "premium features," just send money directly to creators.

Built it with ENS names and USDC on Base. Total platform fee: $0. The only cost is network gas, which is ~$0.01 per tip.

## How It Works

Creators register their ENS name (like `yourname.eth`) in the smart contract. This maps their ENS name to their wallet address.

Fans visit `yoursite.com/tip/yourname.eth` and send USDC. The entire amount goes to the creator. The platform takes nothing.

![Creator registration interface](/assets/images/tipping-registerascreator.png)

```javascript
contract TippingPlatform {
    mapping(string => address) public creators;

    function registerCreator(string memory ensName) external {
        require(creators[ensName] == address(0), "Name taken");
        creators[ensName] = msg.sender;
        emit CreatorRegistered(ensName, msg.sender);
    }

    function tip(string memory ensName, uint256 amount) external {
        address creator = creators[ensName];
        require(creator != address(0), "Creator not found");

        usdc.transferFrom(msg.sender, creator, amount);
        emit TipSent(ensName, msg.sender, amount);
    }
}
```

That's it. No escrow, no holds, no processing fees. Money moves directly from fan to creator.

## Why ENS Names

Sharing a wallet address is ugly: `0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb`

Sharing an ENS name is memorable: `yourname.eth`

Plus ENS names are already owned by crypto-native people, so it reduces friction for creators who are already in the space.

For creators without ENS, we support raw addresses too. But the UX is way better with names.

## Why USDC Instead of ETH

Price stability. If someone tips you $20 in ETH and ETH drops 10% the next day, you actually got $18.

USDC is always $1. Creators know exactly what they're receiving.

Also, most people think in dollars, not crypto units. "$20 tip" is clearer than "0.012 ETH tip."

## The Zero Fee Model

There's no business model here. I'm not trying to monetize this.

The smart contract has no owner functions to extract fees. It's deployed and immutable. Even if I wanted to add fees later, I couldn't.

Why build something with no revenue? Because it's technically possible and the world doesn't need another platform taking cuts.

The only costs:

- Gas for the tip transaction (~$0.01 on Base)
- Time spent building the frontend
- Hosting costs for the website (~$10/month)

That's sustainable without charging creators or fans.

## Real-Time Tracking

The platform scans blockchain events to show creator stats:

- Total tips received
- Number of tippers
- Recent tip history

All of this is derived from on-chain `TipSent` events. No database, no backend scraping, just reading logs from the blockchain.

```typescript
const tips = await publicClient.getLogs({
  address: TIPPING_CONTRACT,
  event: parseAbiItem(
    "event TipSent(string indexed ensName, address tipper, uint256 amount)"
  ),
  fromBlock: DEPLOY_BLOCK,
  toBlock: "latest"
});
```

This makes the platform extremely light. No servers storing data. Just a frontend that reads blockchain state.

## Why Creators Should Use This

Zero fees. Seriously. Not "zero fees but we charge for analytics." Not "zero fees but premium features cost money." Actual zero.

Direct wallet-to-wallet transfers. No waiting for payouts. Tips arrive in your wallet instantly.

Global by default. Someone in Japan can tip someone in Brazil with zero friction. No PayPal region locks, no Stripe limitations.

Transparent history. You can verify every tip you've received on the blockchain. No platform can hide payments or manipulate numbers.

## Why This Won't Replace Patreon

No recurring payments. Patreon's model is subscriptions—creators get predictable monthly income. This is one-time tips.

No gating features. Patreon lets you gate content behind membership tiers. This is just tips.

No community features. Patreon has comments, DMs, polls. This is purely payment infrastructure.

So it's more of a "Buy Me a Coffee" replacement than a Patreon killer. For creators who just want a simple tip jar with zero fees, this works.

## What Could Be Better

ENS names cost $5/year to register. That's a barrier for new creators. Considering adding support for free subdomains like `yourname.tipeth.xyz` that we issue.

USDC onramp is still clunky. Most people don't have USDC. They have a credit card. Adding Stripe -> USDC conversion would make this usable for normal people.

Also the gas costs, while tiny, are still a mental barrier. "Why do I have to pay $0.01 to send a tip?" Exploring gasless transactions where the platform sponsors fees.

But even as-is, it works. Zero fees, instant settlement, full transparency. That's the core value.

**Source code:** [github.com/szcharlesji/tipping-platform](https://github.com/szcharlesji/tipping-platform)
