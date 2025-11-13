---
layout: single
title: "Anti-Scalping Event Tickets With Smart Contract Price Caps"
date: 2023-10-24
categories: [web3, nft]
---

Taylor Swift tickets hit $5,000 on StubHub for shows that originally sold for $200. Scalpers used bots to buy hundreds of tickets in seconds, then resold them at 25x markup.

I thought: what if the ticket itself enforced a maximum resale price?

So I built NFT tickets where the smart contract prevents selling above 1.5x face value. Try to list at 3x? Transaction reverts. The code says no.

## How Price Caps Work On-Chain

Each ticket is an ERC-721 NFT with metadata storing the original price and max resale multiplier:

```javascript
struct TicketMetadata {
    uint256 eventId;
    uint256 seatNumber;
    uint256 originalPrice;
    uint256 maxResaleMultiplier; // 150 = 1.5x
    uint256 timesTransferred;
    uint256 maxTransfers; // Usually 2
    bool redeemed;
}
```

When someone lists a ticket on the marketplace, the contract checks the price:

```javascript
function listTicket(
    uint256 tokenId,
    uint256 askingPrice
) external {
    require(ticket.ownerOf(tokenId) == msg.sender, "Not owner");
    require(!ticket.isRedeemed(tokenId), "Already redeemed");

    TicketMetadata memory metadata = ticket.getMetadata(tokenId);
    uint256 maxPrice = (metadata.originalPrice * metadata.maxResaleMultiplier) / 100;

    require(askingPrice <= maxPrice, "Price exceeds cap");
    require(
        metadata.timesTransferred < metadata.maxTransfers,
        "Transfer limit reached"
    );

    listings[tokenId] = Listing({
        seller: msg.sender,
        price: askingPrice,
        active: true
    });
}
```

Try to list above the cap and the transaction fails before it even hits the blockchain.

![Event creation interface](/assets/images/event-tickets-create.png)

## The Transfer Limit Problem

I originally had no transfer limit. That created a loophole: scalpers would buy tickets, list at 1.5x, buyer flips at 1.5x again, repeat. After 3 flips you're at 3.375x original price.

So I added a transfer counter. Most tickets allow 2 transfers max. After that, the ticket locks to the current owner until the event.

This isn't perfect—someone could still flip once for 50% profit—but it's way better than unlimited scalping.

## Identity Verification For High-Demand Events

Some events (like concerts with 50,000 people fighting for 5,000 tickets) need stronger protection. We added an optional identity whitelist.

Organizers can require buyers to pass KYC before purchasing. This stops bots from mass-buying tickets with thousands of wallets.

```javascript
mapping(address => bool) public verifiedBuyers;

function purchaseTicket(uint256 eventId) external payable {
    Event memory evt = events[eventId];

    if (evt.requiresVerification) {
        require(verifiedBuyers[msg.sender], "Not verified");
    }

    // ... rest of purchase logic
}
```

The admin panel lets organizers batch-verify addresses. It's manual right now, but it works.

## QR Code Check-In

Tickets get a QR code that venue staff scan at entry. The QR contains the tokenId, and scanning calls `redeemTicket()` on-chain:

```javascript
function redeemTicket(uint256 tokenId) external onlyVenue {
    require(!metadata[tokenId].redeemed, "Already used");
    metadata[tokenId].redeemed = true;
    emit TicketRedeemed(tokenId, block.timestamp);
}
```

Once redeemed, the ticket can't be transferred or resold. This prevents someone from entering the venue, immediately listing their ticket, and having someone else use the same ticket.

![Check-in interface for venue staff](/assets/images/event-tickets-checkin.png)

## Why This Probably Won't Replace Ticketmaster

Ticketmaster makes money from fees. They charge 20-30% in fees on both primary and secondary sales. Cutting scalping reduces their revenue.

They have no incentive to fix the problem. In fact, they own a huge stake in secondary markets through partnerships.

But for independent artists or smaller venues, this is viable now. No middleman, no hidden fees, just tickets that enforce fair pricing in code.

**Source code:** [github.com/szcharlesji/event-ticketing-platform](https://github.com/szcharlesji/event-ticketing-platform)
