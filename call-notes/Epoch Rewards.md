# Epoch Rewards  V2

Meeting Date : 17 March 2023
Meeting Duration: 20 mins
[Video of the meeting
](https://www.youtube.com/watch?v=IbviAInuSHk&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5)

## Decisions

Spreading out reward distribution over multiple blocks as it takes a long time at epoch boundary.

## Speakers
1. [Jacob Creech](https://github.com/jacobcreech)
2. [Anatonly](https://github.com/aeyakovenko)
3. [Haoran](https://github.com/HaoranYi)


## Meeting Notes

### Introduction
Video |[ 00:00](https://www.youtube.com/watch?v=IbviAInuSHk&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5)
-|-

**Jacob Creech:** Okay, welcome everyone to today's core community call. We have on the agenda going through the epoch V2 rewards by Haoran. I'll let you take it away. Anatoly do you want to say anything first or do you want to just skip?

**Anatoly:** Let's go ahead.

**Haoran:** Good afternoon, everyone. My name is Haoran, and I work for Solana Labs. In this talk, I'm going to give a brief introduction about the new epoch reward proposal. We are discussing the following [link](https://github.com/solana-foundation/solana-improvement-documents/pull/15) as a SIMD pull request. 


### Background 
Video | [1:17](https://youtu.be/IbviAInuSHk?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=77)
-|-

Here is some background information, we are experiencing a very long block time at the epoch boundary, the following table shows the example of the timing for the block time and around  at the epoch 400 we can see that the the highest number around for 90 percent of the validators are mainnet for this particular block along the epoch boundary it will take around 38 seconds and we need to address this scalability issue.

**Jacob Creech:** Haron I believe you're not showing a presentation. You're just showing your code right now, must say you're showing your Visual Studio code.

**Haoran:**  I am seeing the PDF, let me re-share it again.
 How about it now?

**Jacob Creech:** Yeah, we can see it now.

**Haoran:** Sorry about that, let’s go back, so basically we are having a very long Epoch time at the Epoch boundary and this long time is because of paying out the rewards at the epoch boundary blocks, so the more stake accounts we have the longer it will take so the current approach won't be scalable.

### New Approach
 Video  |  [2:47](https://youtu.be/IbviAInuSHk?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=167)
-|-

That’s why we are proposing a change to pay out the Epoch rewards, so instead of paying out the rewards just for one block at the epoch boundary, the new proposal is to spread out the reward distribution over multiple blocks and the new approach were divided at the block distribution into two phases, the first one is the reward calculation phase, that basically is calculated the rewards are going to be paid out during the epoch after the reward calculation phase then there is going to be another phase called the rewarded distribution phase. In this phase, actual reward will be credited to the stake accounts. 


### Reward Calculation 
Video |   [3:30](https://youtu.be/IbviAInuSHk?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=210) 
-|-

So, as I mentioned earlier reward calculation is just computed rewards to be distributed, and based on the timing we have seen before so we think that 40 seconds may be a good time for the reward calculation period given that each block is 400 milliseconds, we estimate that it's going to take a thousand blocks for the reward calculation result to be available and the reward calculation is done by background services, services started at the epoch boundary and the last for a thousand blocks to compute the rewarded result. So for validators that are faster, they may finish the computation before a thousand block height if that's the case then the validator would need to cache the result and make it available until the thousand blocks and for some reason for some validators which are slow then they will have to wait at the thousand block height for the result to be available before they can enter into the next phase, a reward distribution phase.

### Rewards Distribution 
Video | [4:45](https://youtu.be/IbviAInuSHk?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=285) 
-|-

So, following the reward calculation is the reward distribution. 
Similarly, reward distribution happens over M blocks and to minimize the impact of all the block and other transaction processing during the rewarding period, we are targeting to reward basically only 64  accounts for every tick since each block has 64 ticks so that will give us about 4K total rewards that can be distributed in one block and the reward distribution of what happens in the block before any transaction processing.

### Epoch Reward History Sysvar Account  
Video | [5:31](https://youtu.be/IbviAInuSHk?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=331)
 -|-

With the new approach the rewards are not going to be distributed at one particular block instead is going to be spread out between multiple blocks. In order to track the progress of a rewarded distribution there will be two system sysvar of our account that are added to help to track the rewarded distribution the first one is the epoch rewarded history Sysvar. It is basically a 500 fixed-size array to be consistent with the stake history we choose that number to be 512 and it's a fixed-size array and each entry in the array contains three elements, three fields the first one is the total rewarding land port for the epoch. The second one is how much reward has already been distributed in import that's the progress and the other one the last one is the hash of all the rewards that are going to be paid out. This basically is introduced to verify the rewarded distribution. It's similar to the role of account hash but it's just more specific for rewards it's only harsh from all the rewards that are going to be paid out.

### Epoch Reward Reserve Sysvar Account 

Video | [6:55](https://youtu.be/IbviAInuSHk?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=416)
 -|-

The second is that the account to introduce is called Epoch reward reserve. As I mentioned earlier in reward distribution, each block of all the rewards are going to be distributed in M blocks so we introduced the second set of sysvars this is basically M sysvars that keeps tracks of all the rewards that are going to be distributed in one block and similarly it has a balance field in which it describes how much rewards are going to be distributed for this particular block and it also has a hash that's the hash of all the rewards are going to be distributed in this block and since we have a M sysvar account, so the address of the of the one particular system account for a particular block is basically determined by harsh, the base ID to the block height then we will get the unique address for the rewards going to be distributed.

### Reward Hash  

Video | [7:58](https://youtu.be/IbviAInuSHk?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=478)
 -|-

The relationship between the hash and the root hash in the Epoch history sysvar is the root hash as we seen earlier here, this is the root hash and it's computed by accumulating all the hashes from all the reserves together. So after we credit all the reserves, credit order rewards from the reserves we will compare the hash against the rule hash to make sure they match each other so that we can be sure that the reward distribution is correct. Okay,  that's the main changes of the proposal.

### Impact
Video | [8:40](https://youtu.be/IbviAInuSHk?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=520)
 -|-

There are a few differences with the new reward scheme than the older one. The first one is that we restrict stake account access during the rewarding period, which means any withdrawal, merge split stakes or any statement manipulation have to wait until the rewards finished. If any transactions that involve those operations was submitted during the reward paying out period. They will get a transaction error. We will introduce a new transaction error for it. Let's lock the reward icon during Epoc reward that's the first impact. 

The second impact is that since now the rewards are going to be paid out in multiple blocks there will be changes for snapshot and the cluster restart during those reward periods to accommodate this reward distribution over multiple blocks. So for any snapshots that are taken during the reward period, we will include a new field in a snapshot that stores the reward calculation result. That's how much rewards are going to be distributed and when the cluster restarts from a snapshot taken during the reward period it will have to load the result and resume the rewarded distribution process as if it was going on before it restarted.


### Q&A 
Video | [10:13](https://youtu.be/IbviAInuSHk?list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&t=613)
 -|-

That's the overview of all the new proposals for the new rewards and more details can be found on this link as the pull request [here](https://github.com/solana-foundation/solana-improvement-documents/pull/15). 

Yeah, let's go into Q and A sessions if you have any.

**Jacob Creech**: Well if anybody cannot talk feel free to raise your hand I will give access and get answers to your question.

**Audience 1**: Yeah, thank you for presenting this, I do remember a brief discussion about this form, in the Discord maybe three or four or five months ago, and I do remember asking this question, so I'm sorry if it's being reproduced here, but there are some dashboards that read rewards to be able to, you know, provide information to end users about rewards across the entirety of Solana. And up to this point, it's fairly easy to do. You just look at the first block of the first successful block of an epoch, and you get all the rewards there. How would a mechanism like that work in this new system? Would you have to be constantly watching for rewards blocks and sort of, you know, building up that information incrementally, or will there be some point where it'll be obvious that it's done and you can do one query to an RPC server to get all the information? 

**Haron**: Yeah. But if you just care about the total number of rewards, I think you can still do it at the first block of the epoch boundary. 

**Audience 2**: I can address a little bit of this. Since the rewards slot or blocks are gonna be paid out, and are gonna be deterministic, you're actually going to be able to generate a query based on which stake account or validator you want to know the rewards for. So we can actually make RPC queries that function the same. It will need some extension, but it'll be slightly different from what we have today.

**Audience 1**: Per validator? And have to make, like, 2,000 queries or something. Is that what you're saying? 

**Attendee 2**: No, no, I mean, you can just query all of the blocks in that case. Like, if you need to do a large range of rewards, you just get the entire payout or distribution block range for the slots. They're basically, gotta get blocks with a limit call with whatever the duration of the distribution.

**Attendee 1**:  So basically what you have been watching has been reached when all the distribution is completed, and then make that call. Is that what you're suggesting? 


**Attendee 3**: It’s deterministic, Yeah, you know, it's over by the thousandth block after the epoch, so you can just make the call right there instead, right?

**Attendee 2**:  So today, where you just know that it's, you know, the first block in the epoch, now it's going to be, you know, the thousandth block, or that, you know, however many blocks before. I don't think any of this is fully parameterized yet. 

**Attendee 1**: Okay, and so, well, so someone would have to have sort of code that understands that to be able to compute what that block would be, or there'll be an RPC call to ask that at some point. Like, is that how you anticipate it going? 


**Attendee 3**: Yeah, basically. It'll depend on how much complexity there ends up being in the final thing. I don't anticipate it being that difficult. This could probably be like an RPC helper versus an actual RPC endpoint, but I think it'll be fairly trivial to put together a way to equivalently get, I mean, admittedly, it's going to be a little heavier because you have to get multiple blocks, right? 


**Attendee 1** :  Okay. And I do have another question, but I don't want to monopolize this, so I'll wait to see if other people have a question first.

**Jacob Creech**: All right, next question.

I think it was Jeff, so go ahead, Jeffy, you should be able to unmute to ask your question. 

**Jeff** : Oh, sorry, my mistake. 

**Jacob Creech**: Okay, is there any other questions about this, or otherwise we'll go back to you, Zans. All right, Ashwin, go ahead.

**Aswin**: Hey just wanted to ask, if there's a cluster we started in the first thousand blocks, is there any change to change that procedure because the snapshot won't have the result in it? 

**Haoran** : Yeah, we have a talk about this, so in that case, when the snapshot is taken during the reward computation phase, the snapshot has to wait, it has to wait until the result is available and store the result there.

**Aswin**: Got it. 

**Jacob Creech**: Okay, Zans, you're welcome to ask your question. 

**Zantetsu**: Okay, so is it the case that validators will only validate a block if the leader producing that block has the appropriate amount of either computation work or rewards pair of work in that block? Or is it the case that a validator can just sort of decide not to do it on their block and be like, "I've modified the code to not do this because I don't want to burn the CPU on this and it doesn't give me any rewards, so I'm just not going to do it?" Or is that even a possibility? Or is it the case that the cluster won't even accept a block unless it includes these details because they're expected for... I mean, they're sort of built into the protocol that this has to happen in this order. 

**Haoran**: Yes, because the order is deterministic  for the current epoch, but there will be some randomness across different epochs. So if the leader, for example, doesn't include the expected set of rewards in the block, I think the hashes will mismatch and his block will be rejected. 

**Zantetsu**: Okay, so this part of the block does go into the hash so that validators have to agree on it to vote on it. So I guess that answers my question and we just kind of accept that the first leader kind of has to do his work without any compensation. That's just kind of built into the protocol. There's no need to add any compensation for doing this work. It's not expected to impact the transactions that could be fit into a block. Like, it doesn't take up extra block space or extra compute time that could prevent a validator from including as many transactions as it would otherwise.


**Attendee 4**:  It's the same problem as we have right now with only one, whatever that first leader or like whatever that first block is, kind of gets screwed on including transactions because everyone is computing this. So you'd have this, it's just spreading out the computation to make it more feasible. 

**Zantetsu**: Yeah, I agree. That sounds better. I was just wondering if there's an opportunity to sort of like make that even more addressed by sort of paying a validator for having to do this. Just putting it out there. 

**Attendee 5**: It samples a thousand slots, right? So it kind of spreads the workout out of everyone. You'd be kind of paying everyone equally, which means there's a no-up. 

**Attendee 6**: Except the dudes that are unfortunate and only get one block in the first, you know, about, well, whatever it tastes like. That's cool.

**Jacob Creech**:  All right, is there any other questions about this, the epoch reward changes? Okay, cool. So this is a SIMD still. I put the link in the chat for everyone to add to the discussion later, SIMD-15. You're welcome to discuss it for the next rest of the time on this call. I'd like to open the floor for anybody that has just general Q&A questions. Just feel free to raise your hand, and I'll give you access to speak, and you can ask your question.

If there's nobody with questions, we can end this early, so I'll give it a few more moments.

**Zantetsu**: Did you guys have a, you know, third-year anniversary party or anything like that? 

**Anatoly Yakovenko**: Yeah, man, we're so busy, we didn't. It's, uh, we should.

**Zantetsu**:  All right, we'll start planning for the fourth now, I suppose. 

**Jacob Creech:** All right, cool. Thank you, everyone, for joining today. There will be a space afterward that if anybody has any questions that they didn't ask here, you're welcome to join. So, be on Twitter, you can see it on Solana_devs, and we'll be happy to chat there as well. Thank you, guys, today.