# Core Community Call 4 Notes

Meeting Date/Time: Friday, March 17th, 2022 19:00 UTC

Meeting Duration: 21 mins

[Video of the meeting](https://www.youtube.com/watch?v=IbviAInuSHk)

## Decisions
- [SIMD: Partitioned Epoch Reward Distribution](https://github.com/solana-foundation/solana-improvement-documents/pull/15)

### Introduction 
Video | [00:00](https://www.youtube.com/watch?v=IbviAInuSHk)
-|-

**Jacob Creech**: Welcome everyone to today's core Community call, we have on the agenda going through the epoch V2 rewards by Haoran,that is all. I'll let you take it away except Anthony did you want to say anything first or do you want to just skip ahead?

**Anthony**: let's go, let's do it.

**Jacob Creech**: Ok, let's go ahead.

### SIMD - Partitioned Epoch Reward Distribution 
Video | [00:26](https://youtu.be/IbviAInuSHk?t=26)
-|-

**HaoranYi**: Okay, how do I show my screen? okay, 

**Jacob Creech**: all right we can see everything.

**HaoranYi**: Okay, so, good afternoon everyone my name is Haran I work for sonana labs and in this talk I'm going to give a brief introduction about the new epoc rewarder proposal that we are discussing on the following link as a SIMD pull request. 

So, here is some background information, so we are experiencing a very long block time at the epoch boundary and the following table shows an example of the timing for the block time, around at the epoch 400 we can see that the the highest number around for 90 percent of the validators are mainnet for this particular block along the epoch boundary it will take around 38 seconds.

**Jacob Creech**: Aaron I believe you're not showing your presentation, You're just showing your code right now.

**HaoranYi**: Really ok,

**Jacob Creech**: We see Visual Studio code instead of seeing the PDF.

**HaoranYi**: okay let me reshare it again. How about that you can see it now?

**Jacob Creech**: We can see it now.

**HaoranYi**: Yeah so, let's go back, so basically we are having a very long Epoch time at the epoch boundary and at this long time is because of paying out the rewards at the pocket boundary blocks. So the more stake account we have the longer it will take. So the current approach won't be scalable that's why we are proposing a change to pay out the epoch rewards.

So instead of paying out the rewards at just one block at the epoch boundary, the new proposal is to spread out the rewarded distribution over multiple blocks and the new approach would divided at the Block distribution into two phases, the first one is the reward calculation phase that basically is calculated the rewards are going to be paid out during the epoch after the reward calculation phase. Then there is going to be another phase called the rewarded distribution phase, in this phase actual reward will be credited to the stake accounts.

As I mentioned earlier reward calculation is just computed the rewards to be distributed and based on the timing we have seen before so we think that 40 seconds may be a good time for the reward calculation period given that each block is 400 milliseconds, so we estimate that it's going to take a thousand blocks for the reward calculation result to be available and the reward calculation is done by a background Services. That services started Epoch boundary and the last for a thousand blocks to compute the rewarded result.

So for validators which are faster they may finish the computation before a thousand block height, if that's the case then the validator would need to Cache the result and make it available until the thousand blocks and for some reason for some validators which are slow they will have to wait at a thousand block height for the result to be available before they can enter into the next phase, a reward distribution.

So following the reward calculation is a reward distribution. Similarly reward distribution were happens over M blocks and the two minimize the impact of all the block and other transaction processing during the rewarding period so we are targeting to reward basically earning 64 accounts for every tick since each block has 64 ticks so that will give us about 4K Total Rewards that can be distributed in one block and the reward distribution, what happens in the blog before any transaction processing and now since the reward with the new approach the rewards are not going to be distributed at one particular block instead is going to spread out between multiple blocks, so in order to track the progress of a rewarded distribution there will be a two system  system var of our account that are added to help to track the reward distribution, the first one is the epoch rewarded history sysva and then it is basically a 500 a fixed size array to be consistent with the stake history we choose that number to be 512 and it's a fixed size array and each entry in the array contains three elements three Fields the first one is the total rewarding land port for the epoch and the second one is the how much reward has already been distributed in import that's the progress and the other one the last one is the hash of all the rewards that are going to be paid out this is basically is introduced to verify the rewarded distribution it's similar to the role of account hash but it's more it's just like a more specific for rewards it's only harsh from all the rewards that are going to be paid out.

The second sysvar account to introduce is called Epoch reward Reserve as I mentioned earlier in rewarded distribution each block, all the rewards are going to be distributed in M blocks. So we introduced the second set of sysvar, this is basically Msysvar that keeps tracks of all the rewards that are going to be distributed in one blocks and similarly it has a balance for it has a balanced field which describe how much rewards are going to be distributed for this particular block and it also have a hash, that's the hash of all the rewards are going to be distributed in this block and since we have Msysvar account so the address of the of the one particular sysvar account for a particular block is basically determined by hash the base ID to the block height then we will get the unique address for the rewards going to be distributed. And that the relationship between the hash the root hash in the epoch var history epoch history system var is the root hash as we seen earlier here this is the root hash it's computed by accumulating all the hashes from all the reserves together so after we credit all the reserves credit rewards from the reserves we will compare the hash against the root hash, make sure they match each other so that we can be sure that the reward distribution is correct.

Okay that's the main changes of the proposal and there are a few different with the new reward scheme than the older one and the first one is that we were restrict the stake account access during rewarding period that means any withdrawal merge, split stakes or any statement manipulation have to wait until the rewards finished.

If any transactions that involve those operations was submitted during the reward paying out period they will get a transaction error we will introduce a new transaction error for it. That's log the the reward account during epoc reward that's the first impact, the second impact is that since now the rewards are going to be paid out in multiple blocks there will be changes for snapshot and the cluster restart during those reward period. So to to accommodate this reward distribution over multiple blocks, so for any snapshot that are taken during the reward period we will include a new field in a snapshot that's stores to store the reward calculation result that's how much rewards are going to be distributed, and when the cluster restart from a snapshot taken during the reward period it will have to load the result and resume the rewarded distribution process. As if it is was going on before it's restarted. yeah that's the overview of all the new proposals for the new rewards and the more details can be found on this link as the pull request here.

Yeah let's go into Q and A sessions, if you have any.

### Q&A
Video | [10:37](https://youtu.be/IbviAInuSHk?t=637)
-|-

**Jacob Creech**: Well, if anybody that cannot talk feel free to raise your hand I will give access. Zentatsuto have a question.

**Zentatsu**: Yeah, thank you for presenting this. I do remember a brief discussion about this from in the Discord maybe, three or four or five months ago and I do remember asking this question, so I'm sorry if it's being reproduced here but there are some dashboards that read rewards to be able to, you know provide information and users about rewards across the entirety of Solana and up to this point it's fairly easy to do you just look at the first block of the, first successful block of a Epoch and you get all the rewards there. How would a mechanism like that work in this new system, would you have to be constantly watching for rewards blocks and sort of you know building up that information incrementally or will there be some point where it'll be obvious that it's done and you can do one query to an RPC server to get all the information but

**HaoranYi**: If you just care about the total number of rewards I think we can, you can still do it at the first block of the epoch boundary.

**Solana Team member**: I can address a little bit of this, since the rewards slot or blocks are gonna be paid out are gonna be deterministic, you're actually going to be able to generate a query based on which stake account or validator you want to know the rewards for so we can actually know the rewards for, so we can actually make RPC queries that function the same, it will need some extension but it'll be slightly different from what we have today.

**Zentatsu**: So per validator we have to make like 2000 queries or something, is that what you're saying.

**Solana Team member**: No no I mean, you can just query all of the blocks in that case like if you if you need to do a large range of rewards you just get the entire payout or distribution block range for the slots. They're basically I gotta get blocks with limit call with whatever the duration of the distribution.

**Zentatsu**: When that point has been reached when all the distribution is complete and make that call, is that what you're suggesting?

**Anthony**: yeah you know it's over by the thousandth block after the epoch so you can just make the call right there instead.

**Solana Team member**: Right so today where you just know that it's you know the first block in the epoch now it's going to be you know the the thousandth block or that you know however many blocks, before I don't think any this is fully parametrized yet.

**Zentatsu**: Okay so, well uh so someone would have to have sort of code that understands that to be able to compute what that block would be or there'll be an RPC call to ask that at some point it like is that how you anticipate it going.

**Solana Team member**: yeah basically it'll depend on how much complexity there ends up being in the final thing I don't anticipate it being that difficult this could probably be like an RPC helper versus an actual RPC endpoint but I think it'll be fairly trivial to put together a way to equivalently get, I mean admittedly it's going to be a little heavier because you have to get multiple blocks right but.

**Zentatsu**: Okay and I do have another question but I don't want to monopolize this so I'll wait to see if other people have a question first.

**Jacob Creech**: Okay, so next person to raise hands was jeff, so go ahead. Jeff should be able to unmute to ask your question.

**Jeff**: Sorry, my mistake.

**Jacob Creech**: okay, is there any other questions about this or otherwise we'll go back to you Zans.
Alright Ashwin, go ahead.

**Ashwin**: Just wanted to ask, if there's any cluster we started in the first thousand blocks is there any change to uh change that procedure because the snapshot won't have the result in it.

**HaoranYi**: Yeah we have a talk about this so in that case when the snapshot is taken during the reward computation phase the snapshot has to wait it has to wait until the result is available and store the result there.

**Ashwin**: Got it.

**Jacob Creech**: Zans, you're welcome to ask your question.

**Zentatsu**: Ok so, is it the case that validators will only validate a block if the validator the leader producing that block has the appropriate amount of either computation work or rewards pair at work in that block, or is it the case that divider can just sort of like decide not to do it on their block and be like, I've modified the code to not do this because I don't want to burn the CPU on this and it doesn't give me any rewards so I'm just not going to do it or is it, is that even a possibility or is it the case the cluster won't even Val, won't even accept a block unless it includes these details because they're expected for I mean they're sort of built into the protocol that this has to happen in this order.

**HaoranYi**: Yes, because, the order is deterministic for the current Epoch but there will be some Randomness across different epochs so if the leader for example, he doesn't, it doesn't matter whether he's a leader or not in fact if the validator doesn't include the expected set of rewards in the block, I think the hash were mismatched and his block will be rejected. 

**Zentatsu**: Okay so these this part of the block does go into the hash so that validators have to agree on it to vote on it so I guess that answers my question and we just kind of accept that the first and leaders kind of have to do his work without any compensation that's just kind of like built into the protocol there's no you know need to add any compensation for doing this work it's not expected to impact then transactions that could be fit into a block like it doesn't take up extra block space or extra compute time that could prevent a validator from including as many transactions as it would otherwise.

**Anthony**: It's the same problem as we have right now with only one whatever that first leader or like whatever that first block is kind of gets screwed on including transactions because everyone is Computing this so you'd have this is just spreading out the computation to make it more feasible.

**Zentatsu**: yeah I agree that sounds better I was just wondering if there's an opportunity to sort of like make that even more addressed by sort of paying a validator for having to do this just putting it out there.

**Anthony**: It samples a thousand slots right so it kind of spreads the workout out of everyone you'd be kind of paying everyone equally which means there's a no-up.

**Zentatsu**:  except the dudes that are unfortunate and only get one block in the first you know about, well whatever it's fine, that's cool.

**Jacob Creech**: All right, is there any other questions about uh this the epoch reward changes.
Okay cool, so this is a Cindy still. I put the link in the chat for
everyone to add to the discussion later, SIMD-15 you're welcome to discuss it, for the next rest of the time on this call I'd like to open the floor for anybody that has just general questions just feel free to raise your hand and I'll give you access to speak and you can ask your question.

If there's nobody with questions we can end this early so I'll give it a few more moments

**Zentatsu**: Did you guys have a you know, third year anniversary party or anything like that.

**Anthony**: Yeah man we're so busy we didn't, we should.

**Zentatsu**: all right we'll start planning for the fourth now I suppose.

**Anthony**: Yep.

### Outro
Video | [19:41](https://youtu.be/IbviAInuSHk?t=1181)
-|-

**Jacob Creech**: All right cool, thank you everyone for joining today, there will be a space afterwards that if anybody has any questions or that they don't ask here welcome to join, it'll be  on Twitter you can see it on Solana and underscore devs and we'll happy to chat there as well thank you guys today.