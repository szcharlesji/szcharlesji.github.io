---
layout: single
title: "I Stake Crypto on My Morning Routine (Miss a Day, Lose Money)"
date: 2023-10-16
categories: [web3, mobile]
---

I've tried every habit app. Streaks, Habitica, Done, Loop. I'd use them for 2 weeks, miss a day, and abandon them forever.

The problem: no real consequences. Miss a workout? The app shows a sad face. Who cares.

So I built one where missing a day costs actual money. Stake $50 on doing something daily for 14 days. Complete it? Get your money back plus 10% bonus. Fail? Funds go to charity and you get nothing.

Day 6, I really didn't want to go to the gym. Then I remembered I had $50 on the line. I went.

## How Habit Staking Works

Pick a habit (exercise, meditation, reading, whatever). Choose your stake amount: $10, $25, or $50. Commit to 14 days.

Every day you check in by uploading proof—a photo, a note, whatever. Your check-in goes to 3 random validators who vote within 24 hours. Need 2 out of 3 approvals to count.

Complete all 14 days? Smart contract refunds your stake plus 10-20% bonus from the reward pool.

Fail? 80% of your stake goes to charity, 20% goes into the reward pool for successful completers.

## The Validator System Was Tricky

My first version had no validators. Just upload a photo and the check-in counted automatically.

Predictable result: people gamed it. One guy uploaded the same gym selfie 14 times. Another person submitted a blank image with "trust me I meditated."

So I added validators. Every check-in gets randomly assigned to 3 other users who vote yes/no within 24 hours.

```javascript
function submitCheckIn(
    uint256 habitId,
    string memory proofUrl
) external {
    Habit storage habit = habits[habitId];
    require(habit.participant == msg.sender, "Not your habit");
    require(habit.isActive, "Habit not active");

    uint256 checkInId = checkInCount++;
    checkIns[checkInId] = CheckIn({
        habitId: habitId,
        day: habit.currentDay,
        proofUrl: proofUrl,
        timestamp: block.timestamp,
        approvalsNeeded: 2,
        currentApprovals: 0,
        status: CheckInStatus.Pending
    });

    // Assign 3 random validators
    address[3] memory validators = _selectValidators(msg.sender);
    checkInValidators[checkInId] = validators;

    emit CheckInSubmitted(checkInId, habitId, proofUrl);
}
```

Validators have 24 hours to vote. Miss your voting duty too many times and you get banned from future campaigns.

This creates a weird incentive loop: you want your check-ins approved, so you approve others' check-ins generously. But if you approve obvious fakes, you lose validator reputation.

It's surprisingly self-regulating.

## Mobile-First With Embedded Wallets

Nobody's going to use a habit app on desktop. This had to be mobile from day one.

Built with React Native and Expo. Uses Privy for embedded wallets—users sign in with Google and a wallet gets auto-created in the background. They never see a seed phrase or sign a transaction manually.

Check-ins are gasless. The app sponsors transaction fees so users don't pay $0.50 every day to prove they went to the gym. It's invisible.

The entire crypto layer is hidden. It just works like a normal app.

## The Charity Mechanism

Failed stakes don't go to the platform. They go to charity.

The smart contract has a CharityRegistry where anyone can propose charities. The community votes monthly on which charities are approved.

When someone fails a habit, 80% of their stake goes to a random approved charity. This way you're not just losing money to some random platform—you're funding something useful even if you fail.

People actually feel less bad about failing when they know it went to charity. Still hurts to lose $50, but at least it's not wasted.

## Why This Is Different

Traditional accountability apps have no teeth. Social pressure only works if you care what internet strangers think.

Financial stakes are universal. Everyone cares about losing $50.

The mobile-first approach with embedded wallets makes it actually usable. No MetaMask, no confusing transaction confirmations, just "tap to check in."

And the charity component makes failing feel less terrible. You're still motivated to win, but losing doesn't feel like pure waste.

## What's Not Solved

Validator collusion is possible. If 3 friends all start habits, they could approve each other's fake check-ins.

Right now validators are semi-random from the active user pool. We exclude people you've validated before in the last 30 days, but a determined group could still game it.

Adding reputation scores helps—validators who consistently approve suspicious check-ins get flagged—but it's not perfect.

Also need better proof types. Photo proof is easy to fake. Thinking about integrating with Strava for exercise, Kindle for reading, meditation apps for practice time. Direct API verification instead of human validators.

**Source code:** [github.com/szcharlesji/habit-stakes](https://github.com/szcharlesji/habit-stakes)
