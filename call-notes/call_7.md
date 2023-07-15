# Core Community Call 7 Notes

Meeting Date/Time: Friday, June 16th, 2023 19:00 UTC

Meeting Duration: 31 mins

[Video of the meeting](https://www.youtube.com/watch?v=9m_M8zEw1cE)

## Decisions

- There may be need to split the implementation of [SIMD 0052](https://github.com/solana-foundation/solana-improvement-documents/pull/52) into two or more different SIMDs to address different methods of implementation and concerns surrounding the SIMD 0052.

## Speakers

1. [Jacob Creech](https://github.com/jacobcreech)
2. [Anoushk | Tinydancer](https://github.com/anoushk1234)
3. [Richie Patel](https://github.com/ripatel-jump)
4. Alexander MeiBner
5. [Dubbel](https://twitter.com/dubbel06)
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

**Anoushk | Tinydancer:** Yeah, thanks for the intro Jacob, let me just share my screen.

Yeah, thanks everyone for joining in, giving your time. I am Anoushk from the Tinydancer team and we have been working on implementing the light clients for Solana. We drafted SIMD 0052 for adding transaction proof verification. So, today we are going to give a brief
overview of our research on it.

So, I had like to start with why we are doing this, the motivation behind this SIMD is primarily to implement light clients for Solana. The reason we need light clients is because users need to verify queries that they make to the RPC, and right now they cannot do that. Which is why they have to trust the RPC that they give them the correct data. These light clients need to be low Hardware pieces of software and they need to be able to run on a phone or browser. So, this is really important for security for blockchain network as we already know, more mature networks like Ethereum that have been there for longer already have light clients and this was a glaring problem in Solana.

So, the crux of this SIMD is adding something called transaction proof. So, on a high level, it is just a multi-proof, saying that your transaction that you sent was included in the block and it succeeded or failed. However, the change that we are making here requires adding some way of making sure that a particular signature and a status whether it's success or failure was included in that transaction in the block. The user needs to be able to verify the inclusion of transaction and the execution status.

We would also add a RPC method that would allow anyone to just call that method and get a proof for the particular transaction.

### Why Solana Need Light Clients

Video | [03:33](https://youtu.be/9m_M8zEw1cE?t=213)
-|-

***Anoushk | Tinydancer:** So, as I mentioned, the reason that we need this is because you know RPCs could give you incorrect information that the transaction actually succeeded or it was included in the block but it actually didn't and that's an attack vector. You can still verify this data if you have a snapshot, but obviously snapshots are you know 30, 40 Gigabytes and that is a lot of data for the end user. Also, relating to this SIMD is about staking attestations. You can actually use transaction proofs to verify that a particular validator has a certain stake off chain. As mentioned in the SPV proposal a validator could technically use transaction proofs to act like a checkpoint to verify a certain state. And as part of a different proposal that uses SPV, it can also be used for inter-chain verification.

Here is a list of important resources that you could take a look at this SIMD that we wrote SPV proposal. There is an open issue that talks about adding statuses to the [bank hash](https://docs.solana.com/proposals/simple-payment-and-state-verification#block-headers) and that is the inter-chain and SPV proposal. So, this is the part that there has been a lot of discussion on. And we think that there need to be more discussions on different parts of the code from community on like different ways to implement it.

### Implementation (Method 1)

Video | [05:30](https://youtu.be/9m_M8zEw1cE?t=330)
-|-

**Anoushk | Tinydancer:**  In our SIMD right now we are really implementing the first way which is modifying the block hash to include statuses but over the last week after discussing with some members of the [Jump team](https://jumpcrypto.com/firedancer/) and other members of Solana Labs. We also found different ways to implement it to tackle different issues that may arise while implementing the first method, that is modifying the block hash. So I am going to dive into each of these very briefly.

So, modifying the block hash is pretty straightforward and again part of the proposal so all it does is you add the transaction status along with the transaction signature into the merkle tree of the transactions which is part of each entry. And that gets hashed, gets mobilized into the block hash and becomes part of the bank hash. Currently, the block hash is a sequential hash of all the entries but making it would be better in terms of verification size and obviously having the status is important for verifying if transaction actually succeeded.

#### Pros and Cons (Method 1)

Video | [07:00](https://youtu.be/9m_M8zEw1cE?t=420)
-|-

**Anoushk | Tinydancer:** Here is a quick overview of the pros and cons.

So, the pros being that on the client side the verification is less computationally heavy so that is good for low Hardware devices and it is part of the core consensus protocol which means that all validators have to agree to it.

The downside of this being that it is quite a major change require like feature flag activation which means that the round trip from like implementing this to actually going live on Mainnet would be quite long. And also, there is a computational overhead of mortalizing entries that was initially taken into account.

### Implementation (Method 2)

Video | [07:50](https://youtu.be/9m_M8zEw1cE?t=470)
-|-

**Anoushk | Tinydancer:**  Moving on to the second method, which is adding separate transaction tree which would basically be a tree of all the receipts of each transaction. This would be part of the bank hash. So, basically just a markle tree of all these interaction signatures and statuses and would just be hashed into the bank hash.

#### Pros and Cons (Method 2)

Video | [08:20](https://youtu.be/9m_M8zEw1cE?t=500)
-|-

**Anoushk | Tinydancer:** The pros of this are mainly that it doesn't come the way of [bankless leaders](https://docs.solana.com/proposals/bankless-leader). So, a leader could basically just create the entries, create the block, propagate the block and then asynchronously update this data bank hash with the statuses. So they don't need to have it before propagating the block.

Again, this is also a major change so comes with the challenges of the first way to implement it.

Yeah, so this is part of the second method which we were told that implementing changing the bank hash would somehow come in the way of bankless leaders. So, if we implement this which is basically have the state of block `n` in block `n + 10`. You actually don't need to execute the transactions of block `n` to get the statuses and then you now have to compute it back to the bank hash. So, you could just do that in a sort of async way but this also would be a pretty major change to the protocol and that is also something that needs to be heavily considered if you are deciding to go over this.

### Implementation (Method 3)

Video | [10:05](https://youtu.be/9m_M8zEw1cE?t=605)
-|-

**Anoushk | Tinydancer:**  The last one is probably the most flexible and easy to work with because we are not really changing any core part of the validator like the consensus. So, we are just using gossip or could be a different network we are even to which is asynchronously push commercial commitment of all the transaction receipts and this could be done like let's say every 10 blocks or so.

So, you know it is not even doing at every block and you could then have validators that pull this change verified against their own state. And then, push an attestation saying that hey this checks out with the data that I have now or likely I could just pull that data, pull those attestations and just verify if `X` percent of the state has actually confirmed that this is the data attached and that their transaction was included in that.

#### Pros and Cons (Method 3)

Video | [11:08](https://youtu.be/9m_M8zEw1cE?t=668)
-|-

**Anoushk | Tinydancer:** The pros of this is that there is no overhead to block production because it is done asynchronously. It is not part of the block hash. There is low risk of liveness failures because it is not part of the core protocol and it is also easier to implement because it is part of gossip and not consensus.

The only downside would be that it is not baked into the core protocol. So, it would be optional to implement and the validators aren't really forced to make those attestations or commitments.

So, this is where we end this presentation and want to hear more about the thoughts on the code from the community and open to any critics or questions. 

Thank you!

### Further Discusions

Video | [12:05](https://youtu.be/9m_M8zEw1cE?t=725)
-|-

**Jacob Creech:**  Hello Richie!

**Richie Patel:**  Hey! So, want to say thanks a lot for working on this. I think this is a really nice initiative and it seems like a rather easy win complexity and wise for enabling this functionality.

I would just like to voice my preference for going the gossip route. Any related method of doing it this way where we don't modify the core data structures to support this feature. I think my main problem with modifying the Proof of History hash would be that it is a quite breaking change of the definition what the Proof of History hash currently is. Where it would basically go from committing to the block data contents of the current block and all blocks before that to going to committing to the state changes that this block induces. And this would matter for [Firedancer](https://solana.com/ecosystem/firedancer) for example because we might use the PoH hash to identify the chain that Firedancer and Salana Labs are currently on and let's say for example we have a temporary mismatch in the runtime where we derive a slightly different state on both clients by redefining the Proof of History hash to potentially differ on both clients. I think it would be much harder to tolerate such runtime mismatches or even detect them as that would basically stop validator from synchronizing entirely rather than continuing replay with a slightly different state. And similarly, I think going the force delay route doesn't seem too elegant to me because that would basically introduce minimum latency for the light clients where they would need to equip these and block speed 10 or so.

I think regardless which way we choose. Maybe, I thought of splitting up the proposal into a few separates SIMDs. And then it would be much easier to vote on each one specifically because it feels like if we try to incorporate this entire feature into one, there is going to be a bit of discussion on it for quite a while. So, the first one that I thought of would be just agreeing on how we actually compute the commitment for the transaction statuses only. So, you know given a vector of transaction statuses what goes into the hash. So, do we just commit the transaction result code or do we also hash logs in some way and then define what the actual root of the transaction statuses is.

And then, we can do a separate SIMD that says here is how you would propagate it over gossip, and then we might go back and say well in hindsight gossip was a bad idea. So, here is another SIMD of how we would propagate the transaction statuses over whatever other protocol. And one other way would be posting it on-chain itself which of course has the benefit that this kind of incentivizes validators to actually participate in this transaction status calculation. Whereas, if I wanted to be mean, I could say hey I don't want to implement this feature in Firedancer. So, I am not just gonna propagate it over gossip which would I think decrease the quality of service. I think we kind of want to incentivize validators to participate in this feature.

Again, thank you so much. I had love to hear your thoughts on these, and I am also gonna post the same feedback on the SIMD itself.

**Anoushk | Tinydancer:** Yeah, likewise I also respect your interest in this SIMD. I think that gossip is definitely a favorable route because it allows us to test a lot of the client UX and because it is actually simpler and less of a breaking change. It is easier to revert back if we messed up. If we think that's not the right way then doing that from you know making a change to the bank hash and then thinking that okay this is not the right way and going back. And, I think that regarding validators being incentivized to participate in network, that is something that again eventually be figured out and might even be another SIMD and yeah I think am leaning more towards what Richie said we are going with gossip.

**Richie Patel:**  Yeah, I totally agree regarding the incentivization. I had be also curious whether Solana Labs has any thoughts on this proposal. But would be really cool to see progress moving on this rather soon.

**Alexander MeiBner:**  I just want to say, first of all good effort. it's gonna have a lot I thing from the community. One of the issues I saw regarding the transaction status is we have generally been moving away from detailed transaction statuses more into very broad categories of failure and the reason for that is so for example originally we even had the runtime errors as part of the consensus luckily that is not the case anymore. So there is now only one error case for the entire runtime in the consensus. And the reason why we move away from that is that the success cases are already complex, but the failure modes are so much more complex and it really narrows down the implementation you can have to get the exact precise error response of every transaction right. And that makes it almost impossible to change anything or to re-implement it any other way.

So, yeah just be careful about the transaction results, don't include too much error state or logging into them otherwise we will all be stuck with exactly one implementation.

**Anoushk | Tinydancer:** Yeah, so we really just want to include like success or failure. There was a suggestion to include like transaction logs but we can actually avoid that by just re-executing the transaction on the client side by fetching the inputs. But so we just want successful or failure.

**Richie Patel:** I heard some feedback suggesting that we also add logs. I think logs are pretty scary because right now the truncation of logs is not well defined. But maybe, that's actually an opportunity to say that there is a recommendation to for example only do 256 byte long log lines. The concern I had there was how this would affect performance if we hash two or more char blocks for each program execution or so, would that limit the TPS in the future. I don't remember who it was, I think it was Mango, yeah I don't know.

**Anoushk | Tinydancer:** I think the only concern was like with the overhead added by logs. But, I think you mentioned that with Firedancer's implementation of char. I think that might not be a problem.

**Jacob Creech:** Dubbel, you ready? go ahead and start.

**Dubbel:** Yeah, no I just wanted to clarify actually for both Richie and Anoushk. When you say the gossip route, you mean that there are going to be no consensus changes right? Like even whatever the validators are voting on regarding the state or whatever state is necessary would be in like a separate smart contract? It wouldn't be part of the block?

**Richie Patel:** Yes correct. It wouldn't even be in a smart contract. It basically just so there is a structure called the [contact info](https://docs.rs/solana/latest/solana/contact_info/struct.ContactInfo.html). The contact info is a structure that every validator publishes regularly onto gossip. As far as I know there is already a snapshot hash and then the accounts hash. So, it seems like it would be fairly trivial to fit in another hash there so that is actually a bit more discussion around whether we should have an entirely separate hash for this, because usually if you are getting the transaction status commitments, you probably also want the account hashes and the problem with doing separate fields for this is that you basically have all validators signing separately. I think if we fit them all into the same contact info block that might not be a problem.

**Dubbel:** They also don't need to be at the same frequency right? How frequently is this contact info updated or how frequently do validators publish it?

**Richie Patel:** The question for Solana Labs I think.

**Dubbel:** That's an implementation detail anyway, but I just wanted to be clear that in the gossip route there are no consensus changes right? So, even the transaction status or anything. The block structure doesn't really change, so Tinydancer won't be blocked in any way on that right?

**Richie Patel & Anoushk | Tinydancer:** Yes!

**Dubbel:** Okay! yeah, just wanted to clarify that.

**Jacob Creech:** Are there any other questions for Anoushk and Harsh?

Kirill, I think you mentioned that the Mango was they suggested using bi-directional connection? Max, I think you are here. Do you want to speak a little bit more on that?

**Maximilian Schneider:** I am sorry, could you repeat on which one should I speak?

**Jacob Creech:** Kirill mentioned that you all suggested you use like bi-directional connection. I believe this was originally like a experimental feature that you all are implementing on the lab's client and it is like a client specific you all want to still relevant anymore?

**Maximilian Schneider:** Yeah, I can give some background. So, for the people who are interested. What we did is when we run benchmarks on a local network will enable a patch on the TPU side to give us back basically a status like it's just a very short status that summarizes what happened with the transaction in the scheduler. This is very similar to I would say like request tracing environment that you would see in a commercial microservice architecture deployed in a private company, send something in from a low balancer right you want to trace for a limited amount of the requests in the network why they are not getting scheduled right. So, you get a a per request measurement that is actually complete because right now the measurements we have are statistical and broad. And it is very hard to say in particular which transaction like we just see all five out of 100 transactions had this issue right. But, we don't know what is wrong with these five transactions, why didn't they get into the block right? What were the five transactions that hit the CU limit or what were the five transactions that hit the I don't know 2000 packets per 10 per 100 milliseconds on a quick connection right. There are different limits in a stack and it's very hard to identify why a certain transaction didn't pass.

That was the original intention there, but yeah I think there's a lot of pushback against this kind of I would say nice to have a feature. We really enjoyed having it for like local performance testing because we get more insights but yeah that's it.

**Jacob Creech:** Richie!

**Richie Patel:** I have looked at this a bit and it wouldn't seem that hard to support Firedancer and Solana Labs. and I think it had also be pretty nice to have clients to get explicit feedback about why the transaction might fail or not. And it seems like in financial kind of applications that would seem like a basic feature to have especially if they are user facing. Like if I go in my wallet and send a transaction and that gets dropped somewhere along along the path since this is a issue that would seem to directly affect the user experience of our clients. I think it would make sense to report that, but it requires a bit of Plumbing to get that data back to the networking layer. Because usually at the point of where you know where your transaction gets dropped you probably already anonymized the traffic flows where you don't have the IP addresses or quick connections anymore. So, I feel like this should be, I don't see why we wouldn't move to bi-directional connections just have the ability to do this kind of reverse for feedback in the future and then we could just maybe publish this in the SIMD specifying what the protocol should be for reverse flow feedback and then I think as time goes on we had see more Solana Labs clients adopt this and finance the clients.

What was the specific feedback why this wouldn't be a good idea?

**Maximilian Schneider:** I think there's a couple of like performance questions there right. So, I think the main issue is that it kind of like creates risk on the security DDOS protection side.

**Richie Patel:** Oh! I think I know where this is going so currently every transaction is a separate uni-direction streaming after you send it. It gets closed, so if there is reversal feedback maybe that can force the client to keep open these streams for a longer by just saying hey I dropped these packets please send them to me again. But that should be easy to fix by for example they are delivering this feedback with datagrams or with you know a persistent stream. I still don't really see why we shouldn't do it if the status is easily available.

I see Galactus from Mango.

**Jacob Creech:** hmmm!

**Galactus | Mango Market:** So, yeah it's actually like what we implemented is more like you have this with Solana client, we cannot really implement this bi-directional stuff because once you get that connection it is dropped immediately after reading. So, what we implemented is a separate service where like a validator can connect and then it can say like list of transactions that it wants feedback for. And so actually, whatever is executed like for X reason is dropped. Then we just send why it was dropped actually.

So, it is more like a separate service. It's not in the same even in the same like TPU or client TPU server.

**Richie Patel:** I see, I mean it would seem a bit cleaner to just deliver acknowledgments after.

**Galactus | Mango Market:** Yeah, I would also like to have like a bi-directional channel where you just send a transaction and get a feedback like why it was dropped. But, yeah like on the validators side, there are like a lot of limits and the connection can be dropped for a lot of reasons and it was quite impossible just to keep the connection up. So, we said okay forget, we will just have a new separate service. But yeah of course I agree like we can have this bi-directional connection of quick and then we will just get a feedback for our transaction.

**Richie Patel:** I think if you install a mechanism to signal optional features in the quick handshake itself. It should be pretty easy to trial out features like this on a real cluster without affecting reliability too much or without introducing breaking changes. But, it seems like it needs a bit more time on finding out what the actual right protocol is for delivering this feedback.

**Galactus | Mango Market:** Yeah! I agree. Yeah, because right now we don't have like a lot of bits remaining just to have and give accurate acknowledgment while closing or even in the acknowledgment packets. Yeah maybe, we have to find another mechanism to get this acknowledgment. I agree with you.

**Jacob Creech:** We are a few minutes over time. So, I think we will have to do as well to continue the discussion probably open on up the PR that is on this as well as the discussion on SIMD 0052 earlier.

Thank you all for coming this month and I will see you all next month for the next SIMD call.

Reminder if you have any agenda items, make sure you do a PR to the next agenda so that we get it earlier rather than later and people can read any additional information beforehand. So, we have a better discussion.

Thank you!
