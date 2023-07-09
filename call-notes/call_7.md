# Core Community Call 7 Notes

Meeting Date/Time: Friday, June 16th, 2023 19:00 UTC

Meeting Duration: 31 mins

[Video of the meeting](https://www.youtube.com/watch?v=9m_M8zEw1cE)

## Decisions

- There may be need to split the implementation of [SIMD 0052](https://github.com/solana-foundation/solana-improvement-documents/pull/52) into two different proposals.

## Speakers

1. [Jacob Creech](https://github.com/jacobcreech)
2. [Anoushk | Tinydancer](https://github.com/anoushk1234)
3. [Richie Patel](https://github.com/ripatel-jump)
4. [ Alexander MeiBner ]
5. [ Dubbel ]
6. [Maximilian Schneider](https://github.com/mschneider)
7. [Galactus | Mango Market](https://github.com/godmodegalactus)

## Meeting Notes

### Introduction

Video| [00:00](https://youtu.be/9m_M8zEw1cE?t=1)
-|-

**Jacob Creech:** Welcome everyone to this month's core community call. Today we have for discussion [Light Clients](https://docs.solana.com/proposals/simple-payment-and-state-verification#light-clients), and we have the [Tinydancer](https://solana.com/ecosystem/tinydancer) team, Anoushk and Harsh talking about different ways that they have been thinking about implementing it.

The agenda can be found as usual on the core community call repository and the specific SIMD is 52 [[SIMD 0052](https://github.com/solana-foundation/solana-improvement-documents/pull/52)] that I have just added to the chat.

Anoushk, you are welcome to take it away.

### Light Clients Overview

Video |  [00:35](https://youtu.be/9m_M8zEw1cE?t=35)
-|-

**Anoushk:** Yeah, thanks for the intro Jacob, let me just share my screen.

Yeah, thanks everyone for joining in, giving your time. I am Anoushk from the Tinydance team and we have been working on implementing the light clients for Solana. We drafted SIMD 0052 for adding transaction proof verification. So, today we are going to give a brief
overview of our research.

So, I had like to start with why we are doing this, the motivation behind this SIMD is primarily to implement light clients for Solana. The reason we need light clients is because users need to verify queries that they make to the RPC, and right now they cannot do that. Which is why they have to trust the RPC that they give them the correct data. These light clients need to be low Hardware pieces of software and they need to be able to run on a phone or browser. So, this is really important for security for blockchain network as we already know, more mature networks like Ethereum that have been there for longer already have light clients and this was a glaring problem in Solana.

So, the crux of this SIMD is adding something called transaction proof. So, on a high level, it is just a multi-proof, saying that your transaction that you sent was included in the block and it succeeded or failed. However, it should have it, the change that we are making here requires adding some way of making sure that a particular signature and a status whether it's success or failure was included in that transaction in the block. The user needs to be able to verify the inclusion of transaction and the execution status. 

We would also add a RPC method that would allow anyone to just call that method and get a proof for the particular transaction.

### Why Solana Need Light Clients

Video | [03:33](https://youtu.be/9m_M8zEw1cE?t=213)
-|-

***Anoushk:** So, as I mentioned, the reason that we need this is because you know RPCs could give you incorrect information that the transaction actually succeeded or it was included in the block but it actually didn't and that's an attack vector. You can still verify this data if you have a snapshot, but obviously snapshots are you know 30, 40 Gigabytes and that is a lot of data for the end user. Also, relating to this SIMD is about staking attestations. You can actually use transaction proofs to verify that a particular validator has a certain stake off chain as mentioned in the SPV proposal a validator could technically use transaction proofs to act like a checkpoint to verify a certain state. And as part of a different proposal that uses SPV, it can also be used for inter-chain verification.

Here is a list of important resources that you could take a look at this SIMD that we wrote SPV proposal. There is an open issue that talks about adding statuses to the block hash and that is the inter-chain and SPV proposal. So, this is the part that there has been a lot of discussion on. And we think that there need to be more discussions on different parts of the code from community on like different ways to implement it.

### Methods of Implementation

Video | [05:30](https://youtu.be/9m_M8zEw1cE?t=330)
-|-

**Anoushk:**  In our SIMD right now we are really implementing the first way which is modifying the block hash to include statuses but over the last week after discussing with some members of the [Jump team](https://jumpcrypto.com/firedancer/) and other members of Solana Labs. We also found different ways to implement it to tackle different issues that may arise while implementing the first method, that is modifying the block hash. So I am going to dive into each of these very briefly. 

So, modifying the block hash is pretty straightforward and again part of the history proposal so all it does is you add the transaction status along with the transaction signature into the merkle tree of the transactions which is part of each entry. And that gets hashed, gets mobilized into the block hash and becomes part of the block hash. Currently, the block hash is a sequential hash of all the entries but making it would be better in terms of verification size and obviously having the status is important for verifying if transaction actually succeeded.

### Pros and Cons

Video | [07:00](https://youtu.be/9m_M8zEw1cE?t=420)
-|-

**Anoushk:** Here is a quick overview of the pros and cons.

So, the pros being that on the client side the verification is less computationally heavy so that is good for low Hardware devices and it is part of the core consensus protocol which means that all validators have to write uh you know over either...

## In Progress

This is a work in progress and will continue to push updates within the next few days to complete this transcription.

Also, will appreciate any feedback and adjustment needed to incorporate to the final version.

Thanks!