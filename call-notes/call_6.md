# Core Community Call 6 Notes

Meeting Date/Time: Friday, May 19th, 2023 19:00 UTC

Meeting Duration: 27 mins

[Video of the meeting](https://www.youtube.com/watch?v=GA5AVg_svj8)

## Decisions

- The automation introduced by [SIMD-0046](https://github.com/solana-foundation/solana-improvement-documents/pull/46) Optimistic cluster restart automation should not replace humans (validator operators) in critical decision-making during restart processes but instead, enhance them while making it easier to automate some hard-to-do tasks by humans during this process.
- Also, there should be options for humans to fully take charge in case things seems to be going wrong with the Optimistic cluster restart automation process.

## Speakers

1. [Jacob Creech](https://github.com/jacobcreech)
2. [Will Hickey](https://github.com/willhickey)
3. [Wen Xu](https://github.com/wen-coding)

## Meeting Notes

### Introduction

Video| [00:00](https://youtu.be/GA5AVg_svj8?t=1)
-|-

**Jacob Creech:** So, welcome everyone to this month's core community call. There is a couple of things on the agenda. So, the first thing that we are going to hear from Will before Xu Wen.

Will about the upcoming release changes that they have or that they are pushing.

And then, we will hear from Wen on [SIMD-0046](https://github.com/solana-foundation/solana-improvement-documents/pull/46).

Before I get started I wanted to first call out that if you ever want to see the agenda for the meeting, you can always go to the core community call repo which I just linked in the chat. And then, there's two SIMDs that would be or that are getting close to consensus on. So, it would be great if people could find time to give your review on them. The first one is [SIMD 33](https://github.com/solana-foundation/solana-improvement-documents/blob/main/proposals/0033-timely-vote-credits.md) which is the timely vote credits. And then, the previous one which we actually already merged in but would love to have any extra eyes on it would be the [SIMD 15](https://github.com/solana-foundation/solana-improvement-documents/blob/main/proposals/0015-partitioned-epoch-reward-distribution.md) which is partition Epoch Rewards.

And go ahead `Will`, you can go and see you speak real quick.

### Mainnet Upgrade

Video |  [01:06](https://youtu.be/GA5AVg_svj8?t=66)
-|-

**Will Hickey:** Yeah thanks. So, as you probably all know the upgrade of Mainnet to [v1.14.17](https://forum.solana.com/t/version-1-14-17-release-summary/175) is underway. We asked for 25% at the beginning of this week, we are currently at 27%. Thank you, we appreciate the over-achievement there. Everything is looking good. Our plan is to ask for general adoption at the start of next week. So, you can anticipate that request.

In case anyone hasn't seen it yet I would encourage you to read the [outage report from the February outage](https://solana.com/news/02-25-23-solana-mainnet-beta-outage-report). At that time, it was widely believed that outage was caused by the v1.14.16 upgrade. That was definitely not the case. I will drop a link to that report in just a moment in the chat here it is also linked in the MB announcements channel on Discord. We also have some audit reports that we will be publishing later today related to [v1.14](https://github.com/solana-labs/security-audits). Feel free to peruse those there is nothing scary in them. If there were, we wouldn't be shipping the release but you know it might give you some peace of mind and some interesting reading.

So yeah, thanks everyone who's already upgraded. And looking forward to getting the rest of the cluster on [v1.14](https://solana.com/news/solana-validators-v1-14-upgrade) soon.

**Jacob Creech:** All right, Wen you're up?

### SIMD 0046

#### Motivation & Requirements

Video |  [02:35](https://youtu.be/GA5AVg_svj8?t=150)
-|-

**Wen Xu:** Oh! okay, let's share my slides first and I can present it. Slide show and share screen. Sorry, what I should explain. So, hello and to those who I haven't met yet, I am Wen, joined  Solana Labs last September and personally I am working on consensus. Personally, I am interested in like high performance and high-reliability system design.

Sorry for wasting time, I don't see my slides to share. I can share my whole screen maybe.

**Jacob Creech:** That will work.

**Wen Xu:** Okay, do you see the slide now?

**Jacob Creech:** Yep!

**Wen Xu:** Okay sounds good. So, motivation and requirements. We are interested in making Solana more reliable of course, and so for high reliability there are many things you can do. First of all, you can write very high quality code then you have less outages but no code is ever perfect. So, you need testing to improve it, but testing couldn't catch all problems. So, you need monitoring to tell you what problems you have. And when monitoring tells you there is a big outage, you kind of need outage handling to get the cluster back to a same state. So, there are a lot of efforts inside Solana to improve code quality or testing or monitoring and other stuff and auditing and formal verification. And today the proposal is only about outage handling.

 So, first of all let's look at how we handle outages now. So, when I am talking about outages, I am talking about like the last outage where the whole cluster just couldn't make routes anymore, is not making progress. And so, somehow what we needed to do is, we need to restart validators, give them a same state to start with. So, the cluster can continue function again. This is called [cluster restart](https://docs.solana.com/running-validator/restart-cluster), which I have a link here. It is very different from like sporadic single validator restart which happens all the time and that doesn't impact reliability at all normally, because you still have a lot of validators functioning. So, what we do now first of all we would try to find the highest optimistically confirmed block. So, Optimistically Confirmed Block means you have a block which got the votes from majority of the validators. Majority means we use two-thirds here. So, when the block is optimistically confirmed, ideally we shouldn't roll it back because it may contain user transactions. If you roll it back, it will have economic impacts, very big economic impact. So, the whole design goal in this proposal is to try not to roll back optimistically confirmed block if possible. So, in reality today we also try to do the same but since today we don't have really autonomous process to do this.

 Today, we use what we call social consensus. So, when there is outage and people would gather in the Discord Channel and they would first confirm yes there is outage. All the validators are not making progress then they will try to, if they decide to go for restart because the cluster doesn't seem to be recovering. Then, they will try to see where to restart from. So, we need to have one block which everyone restarts from. And to make sure we are not rolling back user transactions, we need to find this highest Optimistically confirmed block, where we agree we will start with. And what we do now is we would normally do consensus or like a voting in Discord channel. Say my local confirmed block is `X` and what do you see? So if most people vote for `X` then go with `X`. So that is how we do it now. And after we decide which block to start from, the validator monitors with operators would stop the validator. And sometimes if there is a bug which could lead to like outage again then we might install new binary. But that doesn't happen very often. And next because we decided we need to start from this block, everyone needs to have the same block. So, you would create a snapshot with a hard fork at this block we decided on. And if you don't have that block locally, you would download it from a trusted source. And after that you would restart validators with the following two arguments. One is `--wait-for-supermajority SLOT_X` and next is `--expected-bank-hash NEW_BANK_HASH`. So this makes sure you restart with the correct snapshot, you have the correct block and correct hash. And then you would wait for 80% of the people to reach the same state as you. Then, the whole cluster begin to function. You start to make new blocks again, and you start to vote and all things go normal.

So, there are a lot of problems with this current restart process. Maybe the biggest problem is it takes a long time. So the whole cluster restart takes about, it could take about like several hours. Which would make your reliability not very good. And so, in the future, we might have other efforts to make it faster, but today we are just trying to solve a small problem. And so we are trying to improve the process of finding the highest optimistically confirmed block because it is really hard for human to pull like two thousand to three thousand validators and see what each one voted for, and then decide what the block is. This is really a job better for machine. So, we are trying to see whether we can develop a protocol, so that the machines can automatically find the highest confirmed block without human intervention.

So, the main design goal here is not to have false negatives. That means if a block was confirmed before the restart that is why not roll it back because that will have very bad impact. And, but false positives might be okay. It means maybe some block is not confirmed before the restart but somehow we think it is confirmed which is fine. Let's say there's our limit is at 67. Let's say there is a block which got 66% of the votes and there are no competing blocks. So this is the only one we know which got 66% of the votes. So it is okay if 80% of the validators in restart decide that we should start from here even though it is not confirmed. It is okay to confirm more, but it's not okay to roll back confirmed block. So, in all the choices we have after this, we will see that we try to not have false negatives at all. And false positives might be fine in the design choices. So, the new approach we think because we now want to have machines automatically negotiate what is the block we want to restart that from. So we would still have humans in the room, they would still start restart validators with the argument. Let's say optimists restart, but they don't need to tell you which slot and which hash they need to start from. That's for machines to decide and we would add additional phase, the silent repair phase here is where the validators negotiate with themselves what is a block they should restart from. So, because they are in this repair phase, no one would make new blocks anymore. So there would be no [turbine](https://docs.solana.com/cluster/turbine-block-propagation) traffic at all. The goal here is to just converge as soon as possible.

So, nothing changes here. You should still stick with the vote you had before the restart. Don't change your votes. So the current proposal, we would use gossip to exchange most information in this phase. And because people are worried because we do restart. People do restart like some people restart earlier, some people restart later. So, in the cluster it is possible to have some validators to restart and some which are not. So we are worried that these two gossip message will pollute each others' internal data structure. So, here we would probably use a new shred version to form a new gossip group. So the validators in restart talk among themselves while the others form a different gossip ring. And we would send two new gossip messages, one is `LastVotedForkSlots` because the goal here is to have everyone share the same metadata so they know, so they repair the same data blocks. So at the end of Silent repair phase, everyone should have the same data and some same metadata. So that they can make the same decision on the same block and start from here.

So LastVotedForkSlots is where we share the last vote before the restart. And because like the validators might be on different slots like some people vote faster, some people vote slower. So just one last vote slot is normally not enough. So we would send nine hours of slots on the same fork so that people can get the whole fork and people can get a better view of the metadata. And also after you repaired everything and you have all the metadata, then you send out your new vote. We don't call this vote to distinguish between normal vote and this vote. So here we call it `HeaviestFork` which is actually just a vote. So you send out after all the repairing is done, you have all the metadata, what you think, where we should restart from. And also, you also send out how many HeaviestFork you received from other people. Because we need to decide when we would exit this silent repair phase. After we see that 80 percent of the peers received this HeaviestFork message from 80% of the people. We check that and everyone agrees on the same block, same slot and hash. If yes, and then we exit this phase and proceed to real restart. So we do what we currently do. Otherwise, if anything happens that makes you can't proceed, just stop, print all debugging information and halt. And then so human can intervene or maybe switch back to the old restart method.

**Wen Xu:** How much time do I have? I am probably a little bit slow.

**Jacob Creech:** Yeah, we have 11 minutes for both finishing questions but we can see how far we can get.

**Wen Xu:** Okay, that's fine. I put all the information, all the details in the SIMD and I put more details in this slide. So, you are welcome to read the slide.

#### The Silent Repair Phase

Video |  [16:40](https://youtu.be/GA5AVg_svj8?t=1000)
-|-

**Wen Xu:** So, I am just going to generally introduce the silent repair phase and then we can go proceed to questions. So first of all when you restart a validator with this new arguments. Immediately we send the LastVotedForkSlots which is what you voted last and all the slots on this fork. And after that, everyone aggregates the LastVotedForkSlots from all the other restarted validators. And you could start repairing your slot if you think this slot could potentially have been up, it's confirmed before the restart. And we could draw a line somewhere to say these are the candidates which could have been confirmed before the slot and the other blocks I don't care. So, you repair all the blocks you care and after we repaired all of them, you aggregate all the last votes in the LastVotedForkSlots and choose your HeaviestFork.

So, I think we are at 20 minutes now. I don't know whether I have introduced the new method enough so everyone has good grasp. But, we could we could see if anyone has any questions at this point. Hello, or I can proceed if no one has questions.

**Jacob Creech:** Does anybody have any questions currently on the current approach or should we just continue?

Alright go ahead and continue then Wen.

#### Exit Silent Repair Phase

Video |  [18:26](https://youtu.be/GA5AVg_svj8?t=1106)
-|-

**Wen Xu:** Okay! So, to exit a silent repair phase. I think one of the most important thing in outage handling is to make sure everyone is on the same page. Otherwise, you think everyone is on the same page, you single-handedly enter the restart and start making new blocks while everyone still repairing blocks, that would be disastrous. And another outage might be happening. So here we are very careful when we exit the silent repair phase. We would count whether enough people are ready for action. So the current check is whether 80% of the validators received the heaviest fork response from 80% of people. So we count 80% here because this is the current line we draw when we do a restart. We wait for 80% to join the restart, then we proceed. And also, even if we see that 80% of people respond, got response from 80%, we also linger for maybe we haven't totally decide the numbers here, we would say linger for two minutes because a gossip message propagation takes time. So linger for two minutes so that my HeaviestFork which contains how many responses I got can reach everyone.

And then, you can perform security checks. Check one is, did everyone agree on the same block, which means same slot and same hash. And whether I also have my local optimistically confirmed block before the restart right. So because my local confirmed block means before the restart, I saw two-thirds of the votes on this block already. So you would imagine this block should be on the selected Fork. So it should be the ancestor or it should be the selected block. If it's not the case then the security check fails because something is wrong, my local confirmed block is getting roll back then you would also exit and halt and wait for human inspection. And if every check succeeds, then perform the current restart logic. We probably will clear the gossip CRDS table so that we don't carry the old restart messages into the new environment. And then, we would automatically start snapshot creation. In contrast to what we are doing today is we manually ask people to manually do it using Ledger. Here we might start earlier once we find out everyone agrees on the same block. Then we execute the same logic, what we are doing now in this new arguments.

So, I have a few links. One is a [Google doc](https://docs.google.com/document/d/1RkNAyz-5aKvv5FF44b8SoKifChKB705y5SdcEoqMPIc/edit?usp=sharing) I started when the proposal was just starting. So it contains many details and a lot of designs we rejected and why. Because I am currently modifying both the SIMD draft and this Google doc. So this Google doc might be outdated and so some design choices might be different if that's the case SIMD is the newest proposal. And the actual SIMD draft here as well. And this is why I'm asking whether you have any questions again. 

So, I haven't described the details of the algorithm. I have backup slide at the end to talk about. There are basically two algorithms, one is how we calculate which blocks I should repair when I have all the LastVotedForkSlots. And the other is how do I select a block in the HeaviestFork. So these two algorithms are in the SIMD and I also have some description here.

#### Questions

Video |  [22:59](https://youtu.be/GA5AVg_svj8?t=1379)
-|-

**Wen Xu:** Let me know if you have questions.

**Mango Team:** Can you hear me?

**Jacob Creech:** Yes, we can hear you.

**Wen Xu:** Yep, yep!

**Mango Team:** Yeah, so actually I am from Mango team and like we are currently implementing [SIMD 47](https://github.com/solana-foundation/solana-improvement-documents/blob/main/proposals/0047-syscall-and-sysvar-for-last-restart-slot.md) which is like the last restart slot. So,  I am wondering if this silent repair stage should be considered also a restart slot?

**Wen Xu:** Sorry I haven't checked that document in detail. So I think, correct me if I am wrong, your proposal is to expose the last restarted slot somehow through Ledger right?

**Mango Team:** Yeah, exactly.

**Wen Xu:** Yeah so, here I think it is orthogonal but once the silent repair phase is over and we know where we are restarting from. We could also like connect to your code and expose this information somewhere right?

**Mango Team:** Oh yeah exactly, so like silent repair should be considered as a restart right? Because it may take time to restart the slot silently right? That's my question.

**Wen Xu:** So when it is in the silent repair phase, we don't know for sure it's a restart. They are negotiating but they don't know whether they can decide on the same block. So in that case I don't think we will expose anything. And once that phase is over and we know we are really entering the restart, then we can expose that restart slot. Did that answer the question?

**Mango Team:** Okay! yeah, it makes sense yeah.

**Anonymous:** There shouldn't actually be any changes to that proposal because once we've committed to restarting at the coordinated restart slot there is going to be a hard fork anyway and we will. So that will update the Sysvar.

**Mango Team:** Okay, okay that's cool.

**Wen Xu:** Yeah. So, that works nicely together. And any other questions?

**Jacob Creech:** Any have a question?

**Validator Operator** Okay sure so as a validator operator, I want to say it is very encouraging to see how much really good thought design seems to have gone into what you are proposing here. I predict that what would happen during another restart event is that there would still be humans that will get alerting and there will be a lot of confusion and talk. So, if we are kind of racing some automated process, just keep in mind that you may want controls there and certainly messaging from the validator to sort of let us be very aware of what's going on so that we can make decisions about. Because if someone believes that there's a bug or a reason that the restart shouldn't proceed allowing something to run away with a whole cluster restart that we want to pause. Just having controls and informational messages to help us understand what's happening are very important. I just want to emphasize that. That's all.

**Wen Xu:** Yes yes, I think maybe later there will be I would try to get more feedback from the Operators because this really impacts how you operate during outage right? So, it helps to get more feedback there. So I think first of all I totally agree and the current approach is opt-in if you don't restart your validator with that flag nothing would happen. We would keep the current approach if you feel more comfortable with that and also of course the outage handling is mostly to assist people, it's not to replace people. And we will also of course give you ways to inspect what's happening inside and ways to like decide no this automatic restart is not working. I should do something else, that's totally doable. It will be all be command line controlled. Does that answer a question?

**Validator Operator** Yes, it did. Thank you.

**Jacob Creech:** Thank you everyone. Thank you Wen for presenting today.

If there is anybody else that wants to ask further questions on this SIMD, I posted in the chat. We can take the discussion there. And thank you all for joining another core Community call.

You all have a good month.

**Wen Xu:** Thanks!
