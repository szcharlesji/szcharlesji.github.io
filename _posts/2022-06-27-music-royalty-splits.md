---
layout: single
title: "Instant Music Royalties Without Spotify's 70% Cut"
date: 2022-06-27
categories: [web3, music]
---

Spotify pays artists $0.003 per stream. After the label takes their cut, the producer gets their share, and everyone else involved takes a piece, the songwriter might see $0.0004.

On a song with 5 contributors, figuring out who gets paid what requires spreadsheets, lawyers, and quarterly settlement delays.

I built a system where every contributor's split is locked in a smart contract at upload. When someone streams the song for $0.01, everyone gets paid their percentage instantly and automatically.

## How Music Royalty Splits Work

Upload a song. Define contributors and their splits:

- Lead artist: 40%
- Producer: 30%
- Featured artist: 20%
- Songwriter: 10%

Lock these percentages in the smart contract. Now every time someone pays to stream the song, the payment automatically splits across all four wallets in real-time.

![How music royalty splits work](/assets/images/music-howitworks.png)

```javascript
struct Song {
    uint256 tokenId;
    string metadataUri;
    address[] contributors;
    uint256[] splitPercentages;
    uint256 streamPrice; // $0.01 in wei equivalent
    uint256 totalStreams;
    bool locked;
}

function streamSong(uint256 tokenId) external payable {
    Song storage song = songs[tokenId];
    require(msg.value >= song.streamPrice, "Insufficient payment");

    song.totalStreams++;

    // Instantly distribute to all contributors
    for (uint i = 0; i < song.contributors.length; i++) {
        uint256 amount = (msg.value * song.splitPercentages[i]) / 100;
        payable(song.contributors[i]).transfer(amount);
    }

    emit SongStreamed(tokenId, msg.sender, msg.value);
}
```

No quarterly payments. No label holding funds for 6 months. No mysterious "processing fees." The smart contract pays everyone the instant the stream happens.

## The NFT Certificate

Each song gets minted as an NFT (ERC-721) when uploaded. This isn't for collectibility—it's proof of ownership and contribution.

The NFT metadata contains:

- Song title and audio file hash (stored on Arweave)
- All contributors and their split percentages
- Upload timestamp
- Total streams and revenue

Anyone can verify who contributed to a song and what percentage they're receiving. It's all public.

This prevents the label from quietly renegotiating splits or claiming they're paying one percentage while actually paying less.

![Uploading a song with contributor splits](/assets/images/music-upload.png)

## Arweave for Permanent Storage

Audio files go to Arweave, not IPFS. Arweave has a one-time payment for permanent storage. Pay ~$5 once, and your song stays online forever.

IPFS requires constant pinning services or your files disappear when nodes go offline. That's not acceptable for something as important as music.

Metadata (song title, contributor info, album art) goes on IPFS via Pinata since it's small and doesn't need decades of permanence.

## One-Time Purchase Option

The streaming model is $0.01 per play. But if someone wants to "own" the song and play it unlimited times, they can buy it outright for $1.

```javascript
function purchaseSong(uint256 tokenId) external payable {
    Song storage song = songs[tokenId];
    uint256 purchasePrice = song.streamPrice * 100; // 100 streams worth

    require(msg.value >= purchasePrice, "Insufficient payment");

    // Distribute payment to contributors
    for (uint i = 0; i < song.contributors.length; i++) {
        uint256 amount = (msg.value * song.splitPercentages[i]) / 100;
        payable(song.contributors[i]).transfer(amount);
    }

    // Grant unlimited streaming rights
    hasAccess[tokenId][msg.sender] = true;

    emit SongPurchased(tokenId, msg.sender, msg.value);
}
```

This gives the same economic outcome as 100 streams but with a single transaction.

## Why Artists Would Use This

Transparency. On Spotify, you have no idea how much you're actually earning per stream until months later when the statement arrives.

With smart contracts, you watch your wallet fill up in real-time as people listen.

No middlemen taking mysterious cuts. Spotify takes ~30%, labels take 60-70%, and you're left with crumbs. Here, you define your splits upfront and that's what you get.

Instant settlement. No waiting 90 days to get paid. The money hits your wallet the second someone streams your song.

## What's Missing

Discovery. I built the payment rails but not the platform for people to find music. You need both.

Right now it's just "here's the smart contract address, pay to stream." That's not a music app, it's infrastructure.

Building the discovery layer (playlists, recommendations, social features) is its own massive project. The smart contracts are the easy part.

Also need streaming quality tiers. $0.01 per stream works for 128kbps MP3s, but audiophiles want lossless FLAC. Those files are 10x larger and should probably cost more per stream.

## Why This Probably Won't Replace Spotify

Network effects. Spotify has 600 million users and 100 million songs. You can't compete with that by having better payment infrastructure.

Crypto UX is still terrible for normies. "Install MetaMask to listen to music" is DOA.

But for independent artists tired of getting exploited by platforms, this works now. Upload your track, set your splits, share the link. People pay, you get paid instantly, no middleman.

**Source code:** [github.com/szcharlesji/music-royalty-splitter](https://github.com/szcharlesji/music-royalty-splitter)
