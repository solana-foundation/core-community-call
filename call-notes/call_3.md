# [Core Community Call 3 Notes](https://github.com/solana-foundation/core-community-call/blob/main/call-notes/call_7.md?fbclid=IwAR0ilgueYg0ZF6L9OVID2V08Ytw02hpyIWzWbZMeSDEXSoBvp-ciT9Mzb2w#core-community-call-7-notes)

Meeting Date/Time: Friday, Feb 17th, 2023

Meeting Duration: 30 mins

[Video of the meeting](https://www.youtube.com/watch?v=XkkxQAF-HhE)

## Decisions

- Use more a setting application fee in [SIMD-16](https://github.com/solana-foundation/)
- Timely Vote Credits in [SIMD-33](https://github.com/solana-foundation/%E2%80%A6)
- [Solana runtime](https://docs.solana.com/developing/programming-model/runtime)

## Speakers

- [Jacob Creech](https://github.com/jacobcreech)
- [Galactus | Mango Market](https://github.com/godmodegalactus)
- [Philip from Fire Dancer](https://twitter.com/jump_firedancer)
- [Jerry from Ellipsis](https://www.linkedin.com/in/jerry-francis-8033a8225?originalSubdomain=tz) 
- Zantetsu
- [Richard from Finance JP Team](https://www.linkedin.com/in/rman/) 
- [Ashwin Sekar from Solana Labs](https://github.com/AshwinSekar)

## Meeting notes

### Introduction

Video: [00:00](https://youtu.be/XkkxQAF-HhE?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5)

**Jacob Creech:** So welcome all to the core Community call this month. Today we’ll have Galactus from mango team talking about SIMD-16, which is application account right fees. And then Zantetsu discussing SIMD- 33 which is the timely vote credits keeping in mind time. I like to keep or i’d like to cap each discussion about 15 minutes each to give each group their ability to present and discuss with the rest of the core team. Galactus, would you like to go ahead and start?

**Galactus | Mango Market:** Yeah, can you hear me well?

**Jacob Creech:** Yes 

**Galactus | Mango Market:** Okay that’s great 

### Application fees

Video: [00:41](https://youtu.be/XkkxQAF-HhE?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=41)

**Galactus | Mango Market:** Thank you, Jacob, so today I will present to you the application fees. I'll share my screen. So I’m a gym character from Mango, and I will present to you the application fees. So we want to introduce a new kind of piece. It’s like a fee when you interact with the Dapp. And this will actually go to the Dapp authority, like the account controlled by Dapp, instead of going to the validator itself. And there will be a rebate mechanism where the Dapp can actually give back the fees to the user so that the overall transaction of the cluster won’t increase. These fees will actually be paid even if the transaction eventually fails.

### The reasons to have an application fees

Video: [1:55](https://youtu.be/XkkxQAF-HhE?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=115)

**Galactus | Mango Market:** So why do we want to introduce this application piece. Right now what we see in th cluster like in Solana transaction scheduler. We put transaction into batches and each batch is executed parallely. So what  happens like a batch will take like locks on all of its account for example right locks. And  no other batch can actually use the account at the same time. So these transaction are read right or even forwarded to the next validator. If this actually creates an incentive to span the whole cluster for example high frequency, market markers they will just create new transaction spamming the network. So actually this will create more and more transactions which will touch the same kind of accounts and eventually it will choke the validator with these transactions.  So we want to have the like discourage, this is kind of behavior of spamming. And even there are like custom  programs  which will only CPI into the Dapp if they can extra profit. To discuss this kind of behavior and to encourage actually creating proper and valid transactions. We want to introduce this kind of fees so they will reduce congestion in the Solana Network. 

Thank you. And actually Dapps will also have ability to collect fees and to extra fees with this proposal. These are the resources that we have like the SIMD- 16. We have also created a POC and adapted basing on the POC

### A new native Solana program

Video: [4:07](https://youtu.be/XkkxQAF-HhE?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=247)

**Galactus | Mango Market:** So we want to introduce a new native. Solana program called application fees program with this program ID. I will talk a little bit about three bits. So the owner of the application fee like the owner of the account can issue a rebate by CPI into a special instruction. They can issue a partial or full rebate to the fees. And the rebate will be transferred back to the pair in the same at the end of the transaction. If the authority issue the rebate and the transaction fails, there will be no rebates.

So we have actually multiple ways to implement this feature. The first thing was storing the application fees on The Ledger. This is the implementation done in the POC but recently there were some complications for  calculating the fees. Because we have to load all the accounts and everything, we have decided to use more like a setting application fee by instruction. So I will explain you more about this. To setting, we have three instructions principally. One pay application fee in which pair agrees to pay application fee for a Dapps, for a certain account. Then the Dapps actually can use the second instruction check application fee. It can check for an account if this amount is paid or not.  And the third which is debate that can rebate the user its application history like the fees paid foreign. So the pay application fee instructions can take multiple accounts. So you can give a list of accounts as in accounts. And the list of  fees you want to pay for each account like an array of event. So the instruction must include this transaction to interact with Dapps which have implemented application fee feature. Even if the transaction fails the pair will end up paying this application fees where could pay application fee if he include this instruction. But actually it was not needed or he may overpay the fees. This instruction is used by the Solana runtime so you cannot CPI into this instruction from another smart contract. And you can actually include multiple accounts belonging to different apps. This won't be an issue.

The second instruction is check application fees instruction where Dapps can issue like CPI into to check if they have paid application fees. So you have to like one account and one argument like the account which you want to. We expect to have application fee and the expected application fee so the application fee is partially paid or not paid. The instruction will give an error and write an error application fee not paid error. if it's fully or overpaid, it will return okay. In case of partial payment, the user will lose the amount. He has paid as application fee the instruction which can be called multiple time across multiple instructions. It won't be an issue.

**Jacob Creech:** Jim Galactus just being mindful. We have five minutes

**Galactus | Mango Market:** Ah, okay. 

**Jacob Creech:** You have to cut. I know that there's been a decent amount of discussion on SIMD. And I would like to open the floor for anybody with questions that have read this SIMD or have so far in the presentation.

**Phillip:** This is Phillip. Profile from fire dancer, I still wanted to ask you about this. who can call to rebate 

**Galactus | Mango Market:** So actually I understood your question. it was more in case of PDA who really called the rebate right that was your question initially 

**Phillip:** No so suppose if it's a PDA

**Galactus | Mango Market:** Yeah

**Phillip:** I guess you've. I don't know I have two different interpretations of what you've been saying one. If it's a PDA then the program that can sign for the PDA and can call rebate.

**Galactus | Mango Market:** Yeah exactly 

**Phillip:** And other times you've talked about the owner of the account being able to call the rebate

**Galactus | Mango Market:** Yeah sure

**Phillip:** Those are two different things and if it's an account which is larger than a PDA.  Then anyways I just have two different understandings and I haven't still liked. I feel like. I don't have full clarity on which one you mean.

**Galactus | Mango Market:** Okay so actually if you check the metadata of an account metadata you always have an owner Associated to each account. So when it's a key pair like a normal key pair, it's the same owner even for the . But with assign instruction, you can transfer the ownership of the account to some other keypad, like some other entity. In that case they have to sign this transaction to initiate the rebate.

**Phillip:**  If you have a PDA and the PDA is a token account. Then the owner of the PDA is the token program.

**Galactus | Mango Market:** Exactly

**Phillip:** You can sign for  your program

**Galactus | Mango Market:** No no  it's actually the token program because uh token program has to invoke sign instruction to sign the PDA. You cannot sign on behalf of token program. You see.

**Phillip:** Don't think that's right. But maybe I'm mistaken. I think it's where you drive the PDA from that program SIMD is the one that can evoke sign. But I'll double check on this and get back to you 

**Galactus | Mango Market:** okay. We can discuss it more 

**Jerry:** This is Jerry from Ellipsis. Hey guys so based on the signing topic. I think Philip is correct. When you have invoked sign occur, it's a PDA of the calling program not of whatever program is like being invoked

**Galactus | Mango Market:** okay. Let’s me think. I actually think it's like when you create a PDA and a bump. And then you give it to a program ID this PDA.  You can just say invoke signed using foreign. Let me get back on this 

**Jerry:** I can explain how it works. I've explained this a number of times before the way that it works to make sure that we're all on the same semantically is you have a program. And you create a new public key or you create a new address that is derived from that program ID. But It doesn't live on the ED25519 curve and from program. You're able to call invoke signed into a downstream program with the PDA as a signer 

**Galactus | Mango Market:** Okay

**Jerry:** So you just wanted to make sure. There's no confusion or confusion around like how process works here. 

**Galactus | Mango Market:** So we have the owner right. We have the owner which is the program, has created this PDA right in the end so we have a fixed owner of the PDA.

**Jerry:** Okay so the fixed owner of the PDA. This is okay so when you say fixed owner do you mean like system level owner. Or do you mean like owner of like this PDA account that has a certain structure, tied to it right. When we're talking about the owner of most PDA like if you create a PDA from any. If you have a PDA from any arbitrary program the system, the owner is always going to start a system program. There's never going to be like an owner that's something else unless you explicitly allocate and assign.

**Galactus | Mango Market:** Yeah okay,  I agree with you. The same program is the owner.

**Jerry:** So what are you saying when you say fixed owner of PDA. Like I'm not quite following what you're talking about there.

**Galactus | Mango Market:**  Okay. I guess let me look into it more and  I think  I will write more about that in this SIMD. Is it okay with you, guys?

**Jerry:** All right sounds good I would love to see the exact details for this.

**Jacob Creech:** Okay, so I think in being mindful time we'll move on the next person. Thank you very much further discussion for the  SIMD. You can find the link. I have posted in the chat, I'll post it again in the chat it's SIMD- 16. We can both discuss more as well as in discord under core technology. I'm moving on to the next one. Let's go to Zantetsu with SIMD-33.

### The propose about credit vote in SIMD- 33

Video: [15:15](https://youtu.be/XkkxQAF-HhE?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=915)

**Zantetsu:** Hi thanks. I'm gonna share my screen if that's okay. Can you please tell me how much time should I expect to have here.

**Jacob Creech:** Another 15 minutes or is it about more than or until the calls end roughly about 13,14 minutes.

**Zantetsu:** Okay, thank you, so I will share my screen. Well I can't share my screen because this is the first time I've used this program on my new computer. And I will have to go to some permission settings to be able to do that. So I hope I can reference what I'm talking about. And people will be able to follow. I made a proposal quite a while ago for this thing. I call timely vote credits the purpose. I'm speaking about this today.  I really just want to make sure that anybody at Solana Labs or anyone else who's a Solana developer that is unfamiliar with the topic can have some familiarity and can ask me any questions. In addition, I was hoping that after I  make my case for the idea that I maybe can get somebody at Sona labs to kind of a Champion of the change  to help me get. It pushed through the various processes that are required to get it there. I've had a pull request open and it's kind of achieved attention sometimes. But I don't think very consistently and I think that  I don't know what the existing mechanism is for community contributions to the Islamic code base. But I hope that there can be a process by which there's maybe a champion for changes that can act on their behalf from Solana Labs. Because from the outside  it's kind of hard to get traction without that. So speaking about the change specifically for a long time we've noticed us the validator set kind of watches this since our you know a lot of what we do and the profit. We want to try to make comes from. You know, our vote credit about credit achievements that. There are some validators that are appear to be achieving higher vote credits essentially by delaying their votes and surveying the state of forks before casting votes. Of course all that are validators do this to some degree because built into the Solana code base and also built into sort of sane. Voting practices is the idea of waiting a little bit if you haven't seen consensus achieved on the fork that you're voting on. You don't get too far ahead of consensus and then get locked very up for a very long time. It end up being the case you're on the wrong Fort so there's some natural amount of occurs. But there's also you know  within them domain of making that choice. There's also explicitly waiting a long time to make a vote to try to make the most accurate vote. And we don't want valuers to wait on votes because that will slow down consensus, will then slow down. You know, user perception of of transaction completion so I propose this idea where vote credits would be calculated based upon the latency of the vote. Which is how long it takes from the time that the vote is cast to when it or sorry how the vote the slot. That's being voted on how long it takes for that vote to land on the Chain. And I've written up in SIMD- 33, I've implemented a pull request for that.  I've gotten lots of great feedback from Sona Labs over months of time. And I've tried to incorporate all feedback into the change that I've proposed. The current proposal is that it be done in a stage in several changes because the change requires updating. The actual vote account state for all vote accounts because extra data needs to be stored in vote accounts to track this latency. That requires new space and boat accounts and so that adds complexities. Because you know obviously it has to be done in a very careful manner. Nobody's prevented from voting or vote transactions don't fail for validators because of bugs. And also because expanding the size of vote accounts requires adding landports to achieve new rent exam minimum levels. There's a consideration there all of that is written up in the SIMD. I think and also I've sometimes these long-running changes, have a lot of discussion that can be kind of hard to follow. So I hope that's not the case with the change I have open right now. But I'm more than happy to answer questions about the change and also about the proposal. If I had been able to share my screen. I'd show the chart that I've been creating periodically where I have been looking at the Historical vote State over every epoch or many epochs. And then computing the vote credits that were achieved by validators and the vote credits that would be achieved. If this proposal were implemented and having a table showing sort of what the effects are. And from that table I believe you can see kind of who the laggers are. Because you can see which validators have say the top four or five achieved vote credits. Every Apoc would have significantly reduced credits each of them because a significant fraction of their votes are delayed. I think that is evidence. This is what's happening and in fact I think that economically speaking the ideal voting pattern. Right now is to vote only on finalized blocks because you'll get maximum possible credits. If you're a private validator since none of your votes will ever be you know on a fork that is ever has a chance of dying. So you'll never get locked out and you'll always achieve maximum credits and in fact the validator code base. The protocol will accept all those votes because it accepts votes up to. I think two or three hundred or four hundred slots old. It's very likely that you'll be able to land every vote if you vote in that manner but of course. That's a strategy, would halt the cluster for everyone's in it. So we don't want people doing that and this proposal is a way to ensure. The strategy and strategies that would artificially delay. Any amount are less profitable or potentially not profitable at all. So that's my presentation about the topic.  I'm sorry, I didn't share anything on the screen. Are there any questions? Is there anybody who would be willing to partner up with me on this topic and has been somewhat beaten to death. I know,  I've proposed it many times. I've talked about it in discord many times. So maybe everyone's already familiar with it and so great then I guess the only thing left is is anybody. That would like to. I don't know how this works with small labs how your organization works with the community. You know I'm not demanding anything it's not like. I want to be in you know like taking anybody's time from Salon Labs outside of what the organization thinks is proper for that person. I'm just hoping, there can be some kind of collaboration here.

### Question

Video: [22:27](https://youtu.be/XkkxQAF-HhE?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=1347)

**Jacob Creech:** Yeah just to touch on that real quick. So in SIMD-1, we kind of outlined like the best way to get your proposal pushed into like the code base is kind of to be able to write it yourself. So that's step one I knoư that you've made some PR. That's great. The second step is that there's a contributor access policy now for the specifically the song Labs repos on the mono repo. And you could request like triage access to start working with other people to get this through other. That I would allow people from Solana labs to talk about that since you're going your target is the salon Labs client for first implementation. But overall in the long term it would be multiple client implementations.

**Ashwin Sekar:** Hey this is uh Ashwin from Savannah Labs  I mean the first pull request looks like it's almost done,  just keep paying me. I don't know like GitHub notifications are pretty hard so just keep pinging me in Trent. Every time you have an update. I know it takes a while because we move fast to reserve resolve all the merge conflicts. But I think you're doing everything right to get it on track. If we're slow to respond, just keep dming me on discord.

**Zantetsu:** I didn't know that was an option I didn't want to pester anybody but I'm happy to pester
Ashwin Sekar: Yeah I mean. I think he did it a couple times in the past and I've looked at it every time so just keeping me. If you're not getting a response and yeah I didn't see SIMD maybe. we should add some reviewers. I'll add myself and Carl. And hopefully we can get that merch too.

**Zantetsu:** Great. If there aren't any other obvious questions.There was a question raised by someone at Jump. Last time I sort of brought this up in a validator meeting. And that question I think was about does this create any kind of perverse voting incentives for validators It disturb the possibility of achieving consensus. Because validators may decide that it's more profitable to vote. you know to commit more to forks and then potentially get locked out a lot more. And what does that do does it make the blockchain brittle and that's an interesting discussion to have. And I'm happy to have that I've thought about it a lot and I don't think it does. Because right now it can only make things better right now like. I said the best incentive is to simply not contribute a consensus and vote on finalized blocks only or if you're not going to do that if you're not if you don't have the stomach for something. That you know antisocial you can choose. Your point of  how far you want to wait before voting when you otherwise could vote. That's  the current what I call ligers are doing different ones doing it to different degrees some of them are. I think much better at it and as a result there be much less affected by this timely vote credits proposal. But all of them will be to some degree. 

**Richard:** Hey Richard from finances jump team. I'm not sure that concern was specifically raised but if anything that was probably more like out of caution since as far as.  You know Solano's consensus protocol has not been um formally, analyzed at least  internally at jump. So we're having our security auditors look at the current implementation. First of all and trying to achieve some formal proofs about how the network reaches consensus. But certainly I think the final answer team gave you the commitment in discord. That we will implement your proposal. I guess it's just a question for the  way this SIMD gets accepted about how all this lands on mainnet. I think that is mostly out of our control but  for now we haven't worked on implementing tower for the like our current exit. Therefore it's also a bit tricky to get started on the work now but as soon as we do work on it. We can commit to also implementing your proposal and we really appreciate the work on on your site.

**Zantetsu:** Yeah, no problem I mean, I didn't mean to point a finger or something. It wasn't a DM after one of the meetings and they expressed very valid concerns. I'm not even trying to say it was in some way uh inappropriate. But there were good concerns and I just think that if someone wants to talk about those. I'm happy to talk about them. That's all. Because I mean I don't like you said it hasn't been very formally analyzed so I think there's some math. You can do to you know. I'm not that great at math but some math you can do to say well given the chance. You know a fork if I make this vote the chance that it's on the wrong Court versus the potential credit I earn by making the vote. I think there's some you know self-referential function or something that decides. what the benefit is of making a vote and of course. We want that to be structured such that it's best for the cluster.  I believe that what I'm proposing is is closer to that but you say until formal analysis is done I guess I can't prove that.

**Ashwin Sekar:** There's also a sorry there's also uh the threshold check which stops you from voting to far consecutively so you're still within a balance of not locking yourself out. And being detrimental to the cluster.

**Zantetsu:** I point out and you know. I'm sure a lot of people know this already there are some of us validator. who you know are uh voting  and adding more credits and sort of committing harder to forks. And earning more vote credits as a result of course with the potential also of getting longer lockouts should. There be a fork and that's the risk we take and that's a better risk for the customer. Overall I guess I can't say. Although thus far in the past year and a half or two years it hasn't seemed to be a problem. You know tentatively but at any rate that strategy that I just described will be worse after this change like it. It will mean that strategy will. Actually I guess I'll have to think about it. I believe that it will reduce the benefit of that. Because I'm sorry go ahead.

**Richard:** We see this kind of change direction of being implemented in other protocols most notably is too. They have an exponential drop-off in rewards on late attestations. So I don't see any last reason against implementing timely World credits either. There's a bit more nuanced to the  by the Solana Labs code Works. Which might in some cases be counter-intuitive to how BFT should resolve. I do hope that we get this proposal through maybe. I don't know timelines.

**Jacob Creech:** Okay so we've reached time, thank you. All for coming today if we have further discussion the two Some Ds. I'll post them again in the chat are SIMD-16 and SIMD- 33. Zans was 33 which is the timely vote credits and then 16 is what GM Galactus went over with the application right fees. We can continue discussion. If you want to do something more. There's the core technology channel in Salon attack discord but thank you everyone for coming today.
