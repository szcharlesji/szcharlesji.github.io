---
layout: single
title: "No More Venmo Requests: Automatic Group Payment Splits on Base"
date: 2023-01-25
categories: [web3, product]
---

Planning a group trip. 8 people. Total cost: $3,200. Everyone says they'll pay their $400 share.

Two weeks later: 3 people paid, 5 haven't, and someone has to play collections officer sending awkward reminder texts.

I built a smart contract where money goes directly to the destination when the pot fills up. No manual collection. No "I'll pay you back later."

## The Core Idea

Create a funding pot with:

- Destination address (hotel, venue, whatever)
- Amount needed per person ($400)
- Number of people (8)
- Deadline (2 weeks)

Anyone can pay for any slot. Maybe Alice pays $800 to cover herself and her partner. Maybe Bob only pays $200 and Charlie covers the rest. The contract doesn't care who pays—it only cares that all 8 slots get filled.

When the pot is full, anyone can trigger payout and the destination receives the full amount. If the deadline hits and it's not full, everyone gets auto-refunded.

![Creating a treasury pot for group expenses](/assets/images/treasury-create.png)

## Why USDC Instead of ETH

My first version used ETH. Bad idea.

You tell people "send $400" and they send 0.2 ETH. Two days later ETH drops 15% and now their $400 is worth $340. The trip organizer has to chase people for the difference.

Switched to USDC. $400 USDC always equals $400. No price volatility. Everyone knows exactly what they're paying.

The code for a payment is simple:

```javascript
function contribute(uint256 potId, uint256 slots) external {
    TreasuryPot storage pot = pots[potId];
    require(block.timestamp < pot.deadline, "Deadline passed");
    require(!pot.executed, "Already paid out");

    uint256 amount = pot.amountPerSlot * slots;
    require(
        pot.currentSlots + slots <= pot.totalSlots,
        "Exceeds available slots"
    );

    usdc.transferFrom(msg.sender, address(this), amount);

    pot.contributors[msg.sender] += amount;
    pot.currentSlots += slots;

    if (pot.currentSlots == pot.totalSlots) {
        _executePayout(potId);
    }
}
```

No escrow delays. No "processing time." Money moves instantly when conditions are met.

## Manual Execution (Not Automatic)

I originally wanted full automation: deadline hits, contract auto-refunds everyone. Pot fills, contract auto-pays destination.

But gas costs for auto-execution were unpredictable. If 8 people need refunds and gas spikes, the transaction could cost $50+. Who pays that?

So I made it manual. When the pot fills, anyone can call `executePayout()`. When deadline passes without filling, anyone can call `executeRefund()`. Simple, cheap, predictable.

## The Flexible Contribution Model

This was the key insight: don't assign seats to people. Just track how many seats are filled.

Traditional models say "Alice owes $400." What if Alice can't pay but her friend wants to cover her? Now you need transfer logic and permission systems.

Instead: "8 slots available, $400 each." Anyone can fill any slot. Way simpler.

Group of friends going to a concert? Create a pot. Someone's friend last-minute wants to join but the original group is only 6 people? No problem, they fill the 7th slot themselves.

## What's Next

Adding milestone payments for bigger trips. Right now it's all-or-nothing: either the pot fills and pays out, or it expires and refunds.

For something like a $5,000 group vacation, you might want multiple payments: $1,500 for flights, $2,000 for hotel, $1,500 for activities. Each milestone has its own deadline and payout.

Also exploring recurring pots for monthly bills. Roommates paying utilities, subscription sharing, gym memberships—anything where the same group pays the same amount regularly.

**Source code:** [github.com/szcharlesji/group-expense-splitter](https://github.com/szcharlesji/group-expense-splitter)
