---
layout: single
title: "Curated NFT Marketplace Where Bad Taste Costs You Money"
date: 2023-05-14
categories: [web3, nft]
---

I wanted curators to have skin in the game. So I made them stake 0.1 ETH to earn the right to curate art. Pick good art, your reputation grows and you earn more commission. Pick garbage, you get demoted.

Turns out people are way more careful about what they promote when their own money is at risk.

## The Problem With NFT Marketplaces

OpenSea has 80 million NFTs. Most are algorithmically generated profile pictures nobody wants. Finding actual quality art requires scrolling through pages of noise.

The curation problem is real. Anyone can mint anything. There's no quality filter. It's like if every Spotify artist could force their songs onto your homepage.

I wanted to flip the model: make curators stake capital, give them strong financial incentives to find good art, and let their reputation compound over time.

![How the art curation system works](/assets/images/art-howitworks.png)

## How Curator Staking Works

To become a curator, you stake 0.1 ETH (~$200). This gets locked in the contract and gives you the right to list art in the curated gallery.

When art you curated sells, you earn commission. The split is:

- Artist: 85%
- Curator: 10%
- Platform: 5%

But here's the key: your commission rate increases as you level up.

```javascript
enum ReputationTier {
    Bronze,   // 0-999 points, 10% commission
    Silver,   // 1000-2499, 10.5%
    Gold,     // 2500-4999, 11%
    Platinum, // 5000-7499, 11.5%
    Diamond   // 7500+, 12%
}

function calculateCuratorCommission(
    address curator,
    uint256 salePrice
) public view returns (uint256) {
    ReputationTier tier = getTier(curator);
    uint256 baseCommission = (salePrice * 10) / 100;

    if (tier == ReputationTier.Silver) return (baseCommission * 105) / 100;
    if (tier == ReputationTier.Gold) return (baseCommission * 110) / 100;
    if (tier == ReputationTier.Platinum) return (baseCommission * 115) / 100;
    if (tier == ReputationTier.Diamond) return (baseCommission * 120) / 100;

    return baseCommission;
}
```

Reputation points come from sales volume and early discovery. If you curate art that sells for 10 ETH, you earn way more reputation than curating a 0.1 ETH piece.

And if collectors buy art early after you list it, you get bonus points for discovering talent before it was obvious.

![Submitting art to the curated gallery](/assets/images/art-submit.png)

## What Collectors Get

There's a "tastemaker score" for collectors. Buy art early from an unknown artist who later blows up? Your score increases.

It's like having cred for buying Beeple NFTs in 2019 instead of 2021 after he sold for $69 million.

The tastemaker score doesn't do anything yet, but the idea is to eventually unlock early access to drops or reduced platform fees for good collectors.

## Artists Keep 85%

This was non-negotiable. Artists already get screwed on traditional platforms. Gallery commissions can be 50%. Spotify pays fractions of a cent per stream.

85% goes to the artist, 10% to the curator who found them, 5% to keep the lights on. Plus artists get 10% royalty on all secondary sales, enforced by the smart contract.

Secondary sales use the RoyaltySplitter contract to automatically split payments:

```javascript
function executeSecondarySale(
    uint256 tokenId,
    address buyer
) external payable {
    require(msg.value > 0, "Payment required");

    address artist = nft.getArtist(tokenId);
    address seller = nft.ownerOf(tokenId);

    uint256 royalty = (msg.value * 10) / 100; // 10%
    uint256 sellerAmount = msg.value - royalty;

    payable(artist).transfer(royalty);
    payable(seller).transfer(sellerAmount);

    nft.transferFrom(seller, buyer, tokenId);
}
```

No escaping royalties. No "optional creator earnings." The contract enforces it.

![Curator statistics and reputation dashboard](/assets/images/art-stats.png)

## What's Tricky

Getting artists to upload is hard. They're used to Twitter and Instagram being free. "Pay gas to mint your art" is a barrier.

I'm exploring gasless minting where the platform covers gas for the first piece. If it sells, the cost gets deducted from proceeds. If it doesn't sell within 90 days, the artist pays gas or the NFT gets delisted.

Also, Arweave storage adds ~$5 per upload for permanent storage. That's not expensive in absolute terms, but it's more friction than "upload to S3 for free."

**Source code:** [github.com/szcharlesji/digital-art-gallery](https://github.com/szcharlesji/digital-art-gallery)
