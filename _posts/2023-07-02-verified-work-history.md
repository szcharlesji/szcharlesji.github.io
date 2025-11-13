---
layout: single
title: "Employment History That Can't Be Faked (Soulbound NFTs)"
date: 2023-07-02
categories: [web3, credentials]
---

A friend's company hired someone who claimed 5 years at Google. Three months in, they discovered the resume was completely fabricated. The person had never worked at Google.

Background checks exist, but they're slow, expensive, and still miss stuff. References can be friends pretending to be managers. LinkedIn profiles can be fictional.

I built a system where employers issue non-transferable NFTs proving employment. Your work history lives on-chain and can't be faked.

## How Soulbound Credentials Work

When you finish working at a company, your manager issues you a WorkCredential NFT with:

- Company name and address
- Your role and dates
- Optional: performance rating, skills verified

This NFT is soulbound—it cannot be transferred or sold. It's permanently tied to your wallet address.

```javascript
contract WorkCredentialNFT is ERC721 {
    struct Credential {
        address employer;
        string companyName;
        string role;
        uint256 startDate;
        uint256 endDate;
        string skillsVerified;
        bool revoked;
    }

    mapping(uint256 => Credential) public credentials;
    uint256 public tokenIdCounter;

    function issueCredential(
        address worker,
        string memory companyName,
        string memory role,
        uint256 startDate,
        uint256 endDate,
        string memory skills
    ) external onlyRegisteredEmployer {
        uint256 tokenId = tokenIdCounter++;

        _mint(worker, tokenId);

        credentials[tokenId] = Credential({
            employer: msg.sender,
            companyName: companyName,
            role: role,
            startDate: startDate,
            endDate: endDate,
            skillsVerified: skills,
            revoked: false
        });

        emit CredentialIssued(tokenId, worker, msg.sender);
    }

    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 tokenId
    ) internal override {
        require(
            from == address(0) || to == address(0),
            "Soulbound: Transfer not allowed"
        );
        super._beforeTokenTransfer(from, to, tokenId);
    }
}
```

The `_beforeTokenTransfer` hook prevents any transfers except minting (from zero address) or burning (to zero address).

![Employer interface for issuing credentials](/assets/images/verified-work-employer.png)

## Employer Reputation System

Anyone can claim to be "Google" and issue fake credentials. So we have an EmployerRegistry where companies stake funds and build reputation.

When you register as an employer, you stake 0.1 ETH and link your company website with a DNS verification challenge (similar to domain ownership verification).

Your employer reputation score starts at 100 and decreases if you:

- Issue credentials that get disputed
- Have high revocation rates (issuing credentials then revoking them)
- Get flagged for suspicious activity

Recruiters can filter candidates by employer reputation: only show credentials from employers with 80+ reputation scores.

This doesn't prevent all fraud, but it makes large-scale fake credential operations expensive and risky.

![Worker dashboard showing all credentials](/assets/images/verified-work-worker.png)

## Instant Verification

A recruiter looking at your resume can verify your work history in 10 seconds:

1. You share your wallet address or ENS name
2. They go to the verification page and paste it
3. The page queries the blockchain and shows all your credentials

No calling references. No waiting for background check companies. Just instant, cryptographic verification.

The verification page shows:

- All employers and dates
- Skills verified by each employer
- Employer reputation scores
- Whether any credentials have been revoked

![Verification page for recruiters](/assets/images/verified-work-verify.png)

## Revocation (But It's Permanent)

Employers can revoke credentials if they discover fraud or misconduct after someone leaves.

But revocations are permanent and visible. If you revoke a credential, everyone can see:

- The credential was revoked
- When it was revoked
- The reason provided

This prevents employers from weaponizing revocations. You can't just revoke someone's credential out of spite—it stays on-chain and they can dispute it publicly.

```javascript
function revokeCredential(
    uint256 tokenId,
    string memory reason
) external {
    Credential storage cred = credentials[tokenId];
    require(cred.employer == msg.sender, "Not the issuer");
    require(!cred.revoked, "Already revoked");

    cred.revoked = true;

    emit CredentialRevoked(tokenId, reason, block.timestamp);

    // Decrease employer reputation
    employerRegistry.decreaseReputation(msg.sender, 5);
}
```

Each revocation decreases the employer's reputation by 5 points. Companies that revoke frequently get flagged.

## Why Companies Would Use This

Faster hiring. No waiting weeks for background checks. Candidates with on-chain credentials can be verified immediately.

Fraud prevention. Much harder to fake a blockchain-verified credential than a PDF resume.

Reputation building. Companies with good employer reputations attract better talent. It's a signal of legitimacy.

Reduced HR overhead. No more manually responding to reference check calls. Just issue credentials when people leave and they're verifiable forever.

## What's Not Solved

Adoption is the hard part. This only works if enough employers issue credentials.

Right now it's 7 companies. That's not enough. You need hundreds or thousands before the network effect kicks in.

Also, older work history can't be verified this way unless former employers retroactively issue credentials. And most won't bother.

So this is more useful for recent grads building their work history than for people with 20 years of pre-blockchain employment.

Privacy is another concern. Right now all credentials are public. That's great for transparency but bad if you don't want everyone knowing your work history.

Exploring selective disclosure: you hold the credential privately and can prove specific facts (like "I worked as a SWE at Google from 2020-2023") without revealing other details.

## The Future (Maybe)

If this scales, every professional has a wallet address with their complete, cryptographically verified work history.

Recruiters don't ask for resumes. They just ask for your ENS name and verify everything on-chain.

Credentialism becomes harder to fake. The "LinkedIn influencer who worked at 6 FAANG companies in 2 years" gets exposed because none of them issued credentials.

But getting from 7 companies to 7,000 is the real challenge. The tech works. Adoption is unsolved.

**Source code:** [github.com/szcharlesji/verified-work-history](https://github.com/szcharlesji/verified-work-history)
