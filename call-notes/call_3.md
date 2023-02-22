# Core Community Call 3 Notes

Meeting Date/Time: Friday, February 17th , 2023 19:00 UTC

Meeting Duration: 30 mins

[Video of the meeting](https://www.youtube.com/watch?v=XkkxQAF-HhE)

## Discussions
- SIMD-16 - Application account write fees 
- SIMD-33 - Timely Vote Credits

### Introduction 
Video | [00:00](https://www.youtube.com/watch?v=XkkxQAF-HhE)
-|-

**Jacob Creech**: So Welcome all to the core Community call this month, today we'll have Galactus from Mango team talking about SIMD-16 which is application account right fees and then Zen Tetsu discussing SIMD-33 which is the timely vote credits, keeping in
mind time, I'd like to cap each discussion about 15 minutes to give each group their ability to present and discuss with the rest of the core team. But yeah, Galactus would you like to go ahead and start?

### SIMD-16 - Application account write fees 
Video | [00:37](https://youtu.be/XkkxQAF-HhE?t=37)
-|-

**Galactus**: Yeah! can you guys hear me well?

**Jacob Creech**: Yes.

**Galactus**: Okay, that's great. Thank you Jacob, so today I will present to you application fees. I'll just share my screen.

I'm Jim Galactus from Mango and today I will present to you application fees. We want to introduce a new kind of fees, it's like when you interact with a Dapp and this fees actually will go to the Dapp authority, like the account controlled by the Dapp instead of going to the validator itself and there will be a rebate mechanism where the Dapp can actually give back the fees to the user so that the overall transaction of the cluster won't increase.These fees actually will be paid even if the transaction eventually fails.

So, why do we want to introduce this application fees. So right now what we see in the cluster is like in Solana transaction scheduler we put transaction into batches and each batch is executed parallely. So what happens is, like a batch will take logs on all of its accounts, so like for example write logs, and so no other batch can actually use the same account in the same time, so these transactions are read, write or even forwarded to the next validator.So, this actually creates an incentive to spam the whole cluster, for example like high frequency market makers, they will just create new transactions spamming the network, so this actually create more and more transactions which will actually touch the same kind of accounts and eventually it will choke the the validator with these transcations.

So, we want to discourage this kind of behavior of spamming and even there are like custom programs which will only CPI into the Dapp if they can extract profit. To discourage this kind of behavior and to encourage actually creating proper and valid transactions we want to introduce this kind of fees. So, eventually they will reduce congestion in the Solana Network and Dapps will also have the ability to collect extra fees with this proposal.

These are the resources we have [SIMD-16](https://github.com/solana-foundation/solana-improvement-documents/pull/16), [PR](https://github.com/solana-labs/solana/pull/30137), we also created a POC, a Dapp based on the POC.

So we want to introduce a new native Solana program called application fees program with this program ID.

I will just talk a little bit about rebates, so the owner of the application fee like the owner of the the account can issue a rebate by CPI into like a special instruction like they can issue a partial or full rebate to the fees and the rebate will be transferred back to the payer at the end of the transaction. But actually if like the authority issue the rebate but eventually if the transaction fails then there will be no rebates. So we have multiple ways to implement this feature,the first thing was storing the application fees on The Ledger, this is the implementation done in the POC but recently actually there were some complications for calculating the fees because we have to load all the accounts and everything so we have decided to use setting application fees by instruction. So, I will just explain you more about this.

We have 3 instructions principally one is pay application fee in which payer agrees to pay application fee for a Dapp for a certain account then the Dapp actually can use the second instruction, check application fee, where it can just check for an account if this amount is paid or not and the third which is the rebate where the Dapp can rebate the user the fees he payed.

So the pay application fee instructions can take multiple accounts, so you can give a list of account in accounts and list of fees you want to pay for each account as an array of Uint

The transaction must include this instruction to interact with Dapps which have implemented application fee feature, even if the transaction fails the payer will endup paying this application fees. Where could even pay application fee if like he include this instruction but actually it was not needed or he may overpay the fees this instruction is used by the Solana runtime so you cannot CPI into this instruction from another smart contract and you can actually include multiple accounts belonging to different Dapps, this won't be an issue.

So the second instruction is check application fees instruction, where Dapp can CPI into, to check if they have paid application fees so here you take like one account and one argument like the account where we expect to have application fee and the expected application fee. So if the application fee is partially paid or not paid the instruction will return an error, application fee not paid error. If it's fully overpaid it will return okay. In case of partial payment the user will lose the amount he paid as application fees. The instruction can be called multiple time across multiple instructions it won't be an issue.

**Jacob Creech**: Jim Galactus just being mindful of time, we have five minutes okay you have to cut, I know that there's been a decent amount of discussion on this SIMD and just would like to open the floor for anybody with questions that have read this SIMD or have so far in the presentation.

### Q&A
Video | [09:13](https://youtu.be/XkkxQAF-HhE?t=553)
-|-

**Phillip**: Hey this is Phillip, from fire dancer. I still wanted to ask you about this, who exactly can call to rebate?

**Galactus**: I understood your question it was more like in case of PDAs who really called the rebate right? that was your question initially?

**Phillip**: No, if it's a PDA, I don't know I have two different interpretations of what you've been saying one is if it's a PDA then the program that can sign for the PDA can call rebate and other times you have talked about the owner of the account being able to call the rebate, so those are two different things and if it's an account which is larger than a PDA, anyways I just have two different understandings and I don't have full clarity on which one you mean.

**Galactus**: Ok, so if actually check like the metadata of an account, you always have an owner associated to each account. Usually it's a key pair like a normnal key pair, it's the same owner even for the PDA but with assign instruction actually you can transfer the ownership of the account to some other entity. In that case they have to sign this transaction, they have to sign this instruction to initiate the rebate.

**Phillip**: So like, if you have a PDA and the PDA is a token account then the owner of the token program, one you can sign for it is your program?

**Galactus**: No no, it's actually the token program, because the token program has to invoke sign instruction to sign the PDA, you cannot sign on behalf of token program.

**Phillip**: Don't think thats right, may be I'm mistaking where you derive the PDA from that program ID is the one that can invoke sign but I'll double check on this and get back to you.

**Galactus**: Yeah, okay, we can discuss more on that.

**Jerry**: Hey, this is Jerry from Ellipsis,Hey guys, So based on the signing topic I think Philip is correct where like when you have like invoked sign occur it's a PDA of the calling program not of like whatever program is like being invoked.

**Galactus**: I think it's like when you create a PDA, you have like a PDA and a bump so and then you yeah you give it to a program ID, this PDA and then you can just say invoke signed using. Let me get back on this.

**Jerry**: I can explain how it works I've explained this a number of times before, like the way that it works, just to make sure that we're all on the same semantically is you have a program and you create a new public key or you create a new address that is derived from that program ID but doesn't live on the ED25519 curve and from that program you're able to call invoke signed into a downstream program with the PDA as a signer.

**Galactus**: Okay.

**Jerry**: Just wanted to make sure there's no confusion around like how that process works here.

**Galactus**: Yeah but so we have the owner right, we have the owner which is the program which has created this PDA in the end. So we have a fixed owner of the PDA.

**Jerry**: Okay, when you say fixed owner do you mean like, system level owner or do you mean like owner of like this PDA account that has a certain structure tied to it, because like when we're talking about the owner of most PDAs like, if you have a PDA from any arbitrary program the system the owner is always going to start a system program there's like never going to be like an owner that's something else unless you explicitly allocate and assign.

**Galactus**: Yeah, I agree with you the system program is the owner.

**Jerry**: So, what are you saying when you say like fixed owner of PDA,like I'm not quite following like what you're talking about there.

**Galactus**: Okay, let me look into it more and I will write more about it in the SIMD. Okay guys?

**Jerry**: Sounds good. Would love to see the exact details for this.

**Jacob Creech**: Being mindful time we'll move on to the next person, thank you very much, further discussion on this SIMD, find the link I have posted in the chat I'll post it again in the chat. It's SIMD-16, we can both discuss there more as well as in Discord under core technology.

I'm moving on to the next one, let's go to Zentatsu with SIMD-33

### SIMD-33 - Timely Vote Credits
Video | [15:15](https://youtu.be/XkkxQAF-HhE?t=915)
-|-

**Zentatsu**: Hi, thanks, I'm gonna share my screen if that's okay and can you please tell me how much time should I expect to have here is it another 15 minutes or is it about more than that.

**Jacob Creech**: Untill the call ends, so roughly about 13 to 14 minutes.

**Zentatsu**: Okay, thank you, I will share my screen, oh shoot well I can't share my screen because this is the first time I've used this program on my new computer and I will have to go to some permission settings to be able to do that.So hope I can reference what I'm talking about and uh people will be able to follow.

So, I made a proposal quite a while ago for this thing I call timely vote credits. The purpose that I'm speaking about this today is I really just want to make sure that anybody at Solana Labs or anyone else who's a Solana developer that is unfamiliar with the topic can have some familiarity and can ask me any questions. In addition I was hoping that after I make my case for the idea that I maybe can get somebody at Solana labs to kind of a champion of the change to help me get it pushed through the various processes that are required to get it there. I've had a pull request open, and it's kind of achieved attention sometimes but I don't think very consistently and I think that, it's, I don't know what the existing mechanism is for Community contributions to the Solana code base, but I hope that there can be a process by which there's maybe a champion for changes that can act on their behalf from Solana Labs, because from the outside it's kind of hard to get traction without that.

Okay, so speaking about the change specifically for a long time we've noticed the validator set, kind of watches this since, you know a lot the profit we want to try to make comes from our vote credit achievements that there are some validators that are appear to be achieving higher vote credits essentially by delaying their votes and surveying the state of forks before casting votes, now of course, all that are validators do this to some degree because built into the Solana code base and also built into sort of same voting practices is the idea of waiting a little bit if you haven't seen consensus achieved on the fork that you're voting on so that you don't get too far ahead of consensus and then get locked very up for a very long time should it end up being the case you're on the wrong fork, so there's some natural amount of that that occurs, but there's also you know within the domain of making that choice there's also explicitly waiting for a long time to make a vote to try to make the most accurate vote and we don't want validators to wait on votes because that will slow down consensus which will then slow down user perception of transaction completion. 

So I propose this idea where vote credits would be calculated based upon the latency of the vote, which is how long it takes from the time that the slots that's being voted on, how long it takes for that vote to land on the chain and I've written that up in SIMD-33 and I've implemented a pull request for that and I've gotten lots of great feedback from Solana Labs over months of time and I've tried to incorporate all that that feedback into the change that I've proposed.

The current proposal is that it be done in a stage in several changes because the change requires updating the actual vote account state for all vote accounts, because extra data needs to be stored in vote accounts to track this latency and that requires new space in vote accounts and so that adds complexities because you know obviously it has to be done in a very careful manner so that nobody's prevented from voting or vote transactions don't fail for validators because of bugs and also because expanding the size of vote accounts requires adding lamports to achieve new rents with minimum levels and there's a consideration there, all of that is written up in the SIMD, I think and also you know sometimes these long-running changes have a lot of discussion that can be kind of hard to follow, so I hope that's not the case with the change I have open right now, but I'm more than happy to answer questions about the change and also about the proposal, if I had been able to share my screen I'd show the chart that I've been creating periodically where I have been looking at the historical vote state over every epoch or many epochs and then Computing the vote credits that were achieved by validators and the vote credits that would be achieved if this proposal were implemented and having a table showing sort of what the effects are and from that table I believe you can see kind of who the laggers are because you can see which validators have, say the top four or five achieved vote credits every epoch, would have significantly reduced credits each of them because a significant fraction of their votes are delayed.

And I think that is evidence that, this is what's happening and in fact economically speaking the ideal voting pattern right now is to vote only on finalized blocks because you'll get maximum possible credits and since none of your votes will ever be you know on a fork that is ever has a chance of dying, so you'll never get locked out and you'll always achieve maximum credits and in fact the validator code base and the protocol will accept all those votes because it accepts votes up to I think two or three hundred or four hundred slots old. So it's very likely that you'll be able to land every vote if you vote in that manner but of course that's a a strategy that would halt the cluster for everyone's in it. So we don't want people doing that and this proposal is a way to ensure that strategy and strategies that would artificially delay any amount are less profitable or potentially not profitable at all. 

So that's my presentation about the topic. I'm sorry I didn't share anything on the screen. Are there any questions and is there anybody who would be willing to partner up with me on this.

This topic has been somewhat beaten to death I know I've proposed it many times I've talked about it in Discord many times so maybe everyone's already familiar with it and if so great then I guess the only thing left is there anybody that would like to work.

I don't know how this works with Solana labs, how your organization works with the community but you know and I'm not demanding anything it's not like I want to be like taking anybody's time from Solana Labs outside of what the organization thinks is proper for that person. I'm just hoping that there can be some kind of collaboration here. 

**Jacob Creech**: Yeah, just to touch on that real quick. So in SIMD-1, we kind of outlined like, hey! the best way to get your proposal pushed into like the code base is kind of to be able to write it yourself. So that's step one, I know that you've made some PRs that's great, the second step is that there's also a contributor access policy now specifically for the Solana Labs repos, on the mono repo and you could request like triage access to start working with other people to get this through, other than that I would allow people from Solana Labs to talk about that since your target is the Solana Labs client for first implementation but overall in the long term it would be multiple client implementations.

### Views from Solana Labs and the community on timely vote credits
Video | [23:21](https://youtu.be/XkkxQAF-HhE?t=1401)
-|-

**Ashwin**: Hey, this is Ashwin from Solana Labs, I mean the first pull request looks like it's almost done. Yeah just keep pinging me, I don't know like GitHub notifications are pretty hard, so just keep pinging me. Every time you have an update and I know it takes a while because we move fast to resolve all the merge conflicts but I think you're doing everything right to get it on track if we're slow to respond you know just keep dming me on Discord.

**Zentatsu**: I didn't know that that was an option. I didn't want to pester anybody but I'm happy to pester.

**Ashwin**: I think he did it a couple times in the past and I've looked at it every time so yeah just keep pinging me if you're not getting a response and yeah I didn't see the SIMD, maybe we should add some reviewers. I'll add myself and Carl and hopefully we can get that merch too.

**Zentatsu**: If there aren't any other obvious questions, there was a question raised by someone at Jump last time I sort of brought this up in a validator meeting and that question I think was about, does this create any kind of perverse voting incentives for validators, does it disturb the possibility of chieving consensus, because validators may decide that it's more profitable to vote you know, to commit more to forks and then potentially get locked out a lot more, and what does that do does it make the blockchain bruttle and that's an interesting discussion to have and I'm happy to have that, I've thought about it a lot and I don't think it does.

Because right now it can only make things better right now like I said the best incentive is to simply not contribute a consensus and vote on finalized blocks only. Or if you're not going to do that if you don't have the stomach for something that you know anti-social you can choose your point of how far you want to wait before voting, when you otherwise could vote and that's what the current what I call ligers are doing.

Different ones doing it to different degrees, some of them are I think much better at it and as a result there be much less affected by this timely vote credits proposal but all of them will be to some degree.

**Richard**: Hey, Richard from financers jump team here, I'm not sure um where that concern was specifically raised but if anything that was probably more like out of caution since as far as you know Solano's consensus protocol has not been formally analyzed at least internally at jump, so we're having our security Auditors look at the the current implementation first of all and trying to achieve some formal proofs about how the network reaches consensus but certainly I think the final answer team gave you the commitment in Discord that we will Implement your proposal um I guess it's just a question for the way this indeed gets accepted about, how all this lands on mainnet I think that is mostly out of our control but for now we we haven't worked on implementing towerBFT for our current exist yet, so therefore it's also a bit tricky to get started on the work now but as soon as we do work on it we can commit to also implementing your proposal, and we really appreciate the work on on your side.

**Zentatsu**: Yeah no problem I mean I didn't mean to point a finger or something it's just it wasn't a DM after one of the meetings and they expressed very valid concerns I'm not even trying to say it was in some way inappropriate but there were good concerns and I just think that if someone wants to talk about those I'm happy to talk about them that's all.

Because I mean, like you said it hasn't been very formally analyzed so I think there's some math you can do to know, I'm not that great at math but some math you can do to say well given the chance of you know, if I make this vote the chance that it's on the wrong fork versus the potential credit I earn by making the vote, I think there's some you know self-referential function or something that decides what the benefit is of making a vote and of course we want that to be structured such that it's best for the cluster and I believe that what I'm proposing is is closer to that but yeah like you say until formal analysis is done I guess I can't prove. 

**Richard**: Yes there's also a the threshold check which stops you from voting too far consecutively so you're still within a balance of not locking yourself out and being detrimental to the cluster.

**Zentatsu**: Yeah and may I point out and you know I'm sure a lot of people know this already there are some of us validators who you know are voting and adding more credits and sort of committing harder to forks and earning more vote credits as a result of course with the potential also of getting longer lockouts should there be a fork and that's the risk we take and whether that's a better risk for the customer overall I mean I guess I can't say although thus far in the past year and a half or two years it hasn't seemed to be a problem you know tentatively but at any rate that strategy that I just described will be worse after this change like it will mean that you know that strategy will, well actually I guess I'll have to think about it I believe that it will reduce the the benefit of that. I'm sorry gohead.

**Firedancer Team**: We see this kind of change direction of being implemented in other protocols most notably to, they have an exponential drop-off in rewards on late attestations so yeah I don't see any last reason against implementing timely vote credits.

There's a bit more nuanced by the Solana Labs code Works which might in some cases be counter-intuitive to how BFT should resolve.I do hope that we get this proposal through maybe I don't know timelines.


### Outro
Video | [29:29](https://youtu.be/XkkxQAF-HhE?t=1769)
-|-

**Jacob Creech**: Okay so we've reached time, thank you all for coming today, if we have further discussion the two SIMDs I'll post them again in the chat are SIMD-16 and SIMD-33. So Zans was 33 which is the timely vote credits and then 16 is what Jim Galactus went over with the application write fees, we can continue discussion there also if you want to do something more athawk there's the core technology channel in Solana tech discord.

But thank you everyone for coming today






