---
layout: single
title: "We Tried to Build 'DAO Democracy' (Here's Why It's Harder Than It Looks)"
date: 2022-11-14
categories: [web3, governance]
---

Three proposals. Zero votes. The dashboard showed 47 members with governance tokens, but when voting opened, nobody showed up.

I'd spent two weeks building a DAO voting platform with soulbound membership NFTs and gas-optimized voting contracts. The tech worked perfectly. The UX was clean. But apparently "decentralized governance" sounds more exciting than it actually is.

## The Vision

Build transparent community governance where every member has one vote. No plutocracy where whales control decisions. No delegate complexity. Just simple Yes/No/Abstain voting on proposals that matter.

We chose Base because voting transactions cost less than a cent. Ethereum mainnet would've been $5+ per vote—completely unusable for casual governance.

The core idea: mint soulbound membership NFTs that can't be transferred. One NFT = one vote. No buying influence, no Sybil attacks through multiple wallets. Your identity in the DAO is permanent.

![How the DAO governance system works](/assets/images/dao-howitworks.png)

## What Actually Shipped

The system has three smart contracts:

**MembershipNFT** - Non-transferable ERC-721 tokens for governance rights
**GovernanceCore** - Proposal creation, voting, and execution logic
**Treasury** - Holds funds and executes approved proposals

Creating a proposal is straightforward:

```javascript
function createProposal(
    string memory description,
    address target,
    bytes memory data
) external onlyMember returns (uint256) {
    uint256 proposalId = proposalCount++;

    proposals[proposalId] = Proposal({
        proposer: msg.sender,
        description: description,
        target: target,
        data: data,
        startTime: block.timestamp,
        endTime: block.timestamp + votingPeriod,
        executed: false,
        yesVotes: 0,
        noVotes: 0,
        abstainVotes: 0
    });

    emit ProposalCreated(proposalId, msg.sender, description);
    return proposalId;
}
```

Voting period is 7 days. Simple majority wins. If more yes votes than no votes by the deadline, the proposal passes and anyone can execute it.

![Creating a proposal in the DAO](/assets/images/dao-create-proposal.png)

## The Voter Apathy Problem

Week one: "This is amazing, on-chain governance!"

Week two: First real proposal goes up. "Should we allocate 1 ETH to marketing?"

Day 1: 3 votes
Day 3: 5 votes
Day 7: 8 votes out of 47 eligible members

The proposal failed because we needed 24 votes (simple majority of total members). Most people just didn't care enough to spend 30 seconds voting.

I tried everything. Discord reminders. Email notifications. Making the voting UI even simpler. Nothing worked. Turns out getting humans to participate in governance is a social problem, not a technical one.

## What Worked Better

We switched from "majority of all members" to "majority of votes cast" with a 10% quorum requirement. Now we just need 5 members to vote, and whichever side has more wins.

Engagement jumped to 15-20 votes per proposal. Still not amazing, but functional.

The soulbound NFT approach worked well. Nobody tried to game the system with multiple wallets because acquiring a membership NFT required real-world verification (we manually reviewed applications).

Gas costs were basically zero—$0.003 per vote on Base. If we'd launched on Ethereum mainnet, we'd have $0 votes because nobody would pay $8 to vote on a $50 treasury decision.

## Why This Still Matters

Traditional organizations have the same problem. Shareholder meetings have low turnout. HOA votes get ignored. Democracy is hard everywhere.

But at least with smart contracts, when someone does vote, it's recorded permanently and executed automatically. No trusting a board to "implement the decision later." The contract handles it.

For small communities that actually care about decisions, this works. For everyone else, it's a reminder that most people just want things to work without having to vote on every treasury allocation.

**Source code:** [github.com/szcharlesji/dao-treasury-dashboard](https://github.com/szcharlesji/dao-treasury-dashboard)
