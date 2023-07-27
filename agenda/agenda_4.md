# **[AGENDA 4 [Core Community Call - March 17, 2023 - Epoch Rewards V2](https://www.youtube.com/watch?v=IbviAInuSHk&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=4&t=2s)]{.underline}**

-   Time: March 17.2023

-   Link:
    > [[https://www.youtube.com/watch?v=IbviAInuSHk&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=4&t=2s]{.underline}](https://www.youtube.com/watch?v=IbviAInuSHk&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=4&t=2s)

```{=html}
<!-- -->
```
-   **Speakers profiles**

```{=html}
<!-- -->
```
-   Haoran Yi (Works for Solana Labs)

```{=html}
<!-- -->
```
-   **Key concepts mentioned:**

```{=html}
<!-- -->
```
-   SIMD 15 Epoch Rewards V2
    > [[https://github.com/solana-foundation/...]{.underline}](https://github.com/solana-foundation/%E2%80%A6)

Okay, welcome everyone to today\'s core Community call. We have on the
agenda going through the epoch VT rewards by Howardum. That is, I\'ll
let you take it away. It\'s incepted Anthony. Did you want to say
anything first or do you want to just skip ahead? Let\'s go, let\'s go.
Go ahead.

Okay. I'll show my screen. Okay, alright, we can see everything good.
Okay, so good afternoon, everyone. My name is Haran, I work for Solana
labs, and in this talk, I\'m going to give a brief introduction about
the new epoch rewarder proposal that we are discussing on the following
link as a CMD pull request.

So here is some background information. We are experiencing a very long
block time at the epoch boundary, and the following table shows an
example of the timing for the blocktime around the epoch 400. We can see
that the highest number around for 90 percent of the validators are
mainnet for this particular block along the epoch boundary. It will take
around 38 seconds.

Aaron, I believe you\'re not showing a presentation. You\'re just
showing your code right now. Okay, we see Visual Studio codeum instead
of seeing the PDF. Okay, let me reshare it again. How about that? Can
you see it now? Sorry about that.

So let\'s go back. So basically, we are having a very long Epoch time at
the epoch boundary, and at this long time is because of paying out the
rewards at the pocket boundary blocks. So the more stake accounts we
have, the longer it will take, so the current approach won\'t be
scalable. That\'s why we are proposing a change to pay out the epocket
rewards. So instead of paying out the rewards at just one block at the
poker boundary, the new proposal is to spread out the rewarded
distribution over multiple blocks, and the new approach was divided at
the block distribution into two phases. The first one is the reward
calculation phase that basically is calculated the rewards are going to
be paid out during the epoch. After the reward calculation phase, then
there is going to be another phase called the rewarded distribution
phase in this phase actual reward will be credited to the stake
accounts.

So as I mentioned earlier reward calculation is just computed the
rewards to be distributed and based on the timing we have seen before,
so we think that 40 seconds may be a good time for the reward
calculation period given that each block is 400 milliseconds so we
estimate that it\'s going to take a thousand blocks for the reward
calculation result to be available, and the reward calculation is done
by a background Services data services started Epoch boundary and the
last for a thousand blocks to compute the rewarded results for
validators which are faster they may finish the computation before a
thousand block height if that\'s the case then the validator would need
to Cache The result and make it available until the Thousand blocks and
for some reason for some validator which are slow then they will have to
wait at the thousand block height for the result to be available before
they can enter into the next phase a reward distribution phase.

So following the reward calculation is a reward distribution similarly
rewarded distribution were happens over M blocks and the two minimize
the impact of all the block and other transaction processing during the
rewarding period so we are targeting to reward basically earning 64.
accounts for every tick since each block has 64 ticks so that will give
us about 4K Total Rewards that can be distributed in one block and the
reward distribution what happens in the blog before any transaction
processing, and now since the reward with the new approach, the rewards
are not going to be distributed at one particular block, instead, it\'s
going to spread out between multiple blocks so in order to track the
progress of a rewarded distribution. There will be a two-system account
that are added to help to track the rewarded distribution. The first one
is the epoch rewarded history and then it is basically a 500 fixed-size
array to be consistent with the stake with the stake history we choose
that number to be 512, and it\'s a fixed-size array, and each entry in
the array contains three elements, three fields. The first one is the
total rewarding land port for the epoch, and the second one is how much
reward has already been distributed in import that\'s the progress, and
the other one, the last one, is the hash of all the rewards that are
going to be paid out. This is introduced to verify the rewarded
distribution. It\'s similar to the role of account hash, but it\'s more,
it\'s just like a more specific for rewards, it\'s only harsh from all
the rewards that are going to be paid out.

The second Israel account to introduce is called Epoch reward Reserve.
As I mentioned earlier in rewarded distribution, for each block all the
rewards are going to be distributed in M blocks, so we introduced the
second set of systems. This is basically MCS verse that keeps tracks of
all the rewards that are going to be distributed in one block similarly,
it has a balance field that describes how much rewards are going to be
distributed for this particular block, and it also has a hash that\'s
the hash of all the rewards are going to be distributed in this block,
and since we have M system accounts, the address of the one particular
system icon for a particular block is basically determined by harsh the
base ID to the block height then we will get the unique address for the
rewards going to be distributed for again.

As we\'ve seen earlier here, this is the rule hash, it\'s computed by
accumulating all the hashes from all the reserves together so after we
credit all the reserves credit order rewards from the reserves, we will
compare the hash against the rule hash, make sure they match each other
so that we can be sure that the reward distribution is correct.

That\'s the main changes of the proposal, and there are a few
differences with the new reward scheme than the older one, and the first
one is that we were restricted the stake account access during rewarding
period, that means any withdrawal merge split stick or any statement
manipulation have to wait until the rewards finished, if any
transactions that involve those operations were submitted during the
reward paying out period they will get a transaction error, we will
introduce a new transaction error for it, let\'s lock the reward icon
during epoch reward, that\'s the first impact, the second impact is that
since now the rewards are going to be paid out in multiple blocks, there
will be changes for snapshot and the cluster restart during those reward
periods, so to accommodate this reward distribution over multiple blocks
so for any snapshot that is taken during the reward period we will
include a new field in a snapshot that\'s stores to store the reward
calculation result that\'s how much rewards are going to be distributed,
and when the cluster restarts from a snapshot taken during the reward
period, it will have to load the result and resume the rewarded
distribution process as if it is was going on before it restarted, yeah,
that\'s the overview of all the new proposals for the new rewards and
the more details can be found on this link as the pull request here,

yeah, let\'s go into Q&A sessions if you have any. Well, if anybody
cannot talk, feel free to raise your hand, and I will give access to
your question, yeah, thank you for presenting this, um, I do remember a
brief discussion

About this form in the Discord maybe three or four or five months ago,
and I do remember asking this question, so I\'m sorry if it\'s being
reproduced here, but there are some dashboards that read rewards to be
able to, you know, provide information to end-users about rewards across
the entirety of Solana, and up to this point, it\'s fairly easy to do,
you just look at the first block of the first successful block of an
Epoch, and you get all the rewards there. How would a mechanism like
that work in this new system?

Would you have to be constantly watching for rewards blocks and sort of,
you know, building up that information incrementally, or will there be
some point where it\'ll be obvious that it\'s done and you can do one
query to an RPC server to get all the information? But if you just care
about the total number of rewards, I think you can still do it at the
first block of the Epoch boundary. I can address a little bit of this,
um, since the rewards slot or blocks are gonna be paid out, are you
gonna be deterministic, you\'re actually going to be able to generate a
query based on which um stake account or validator you want to know the
rewards for, so we can actually make RPC queries that function the same,
it will need some extension but it\'ll be slightly different from what
we have today per validator and have to make like 2,000 queries or
something, is that what you\'re saying?

I mean, you can just query all of the blocks in that case, like if you
need to do a large range of rewards you just get the entire payout or
distribution block range for the slots, they\'re basically, I gotta get
blocks with a limit call with whatever the um duration of the
distribution has been reached when all the distribution is completed,
and then make that call, is that what you\'re suggesting? Yeah, today,
right, yeah, yeah, you know, it\'s, it\'s over by the thousandth block
after the epoch, so you can just make the call right there instead,
right, so today, where you just know that it\'s, you know, the first
block in the epoch, now it\'s going to be, you know, the thousandth
block, or however many blocks before, I don\'t think any of this is
fully parametrized yet, okay, and so well, so someone would have to have
sort of code that understands that to be able to compute what that block
would be, or there\'ll be an RPC call to ask that at some point, it
like, is that how you anticipate it going?

Yeah, basically, it\'ll depend on how much complexity there ends up
being in the final thing. I don\'t anticipate it being that difficult,
this could probably be like an RPC helper versus an actual RPC endpoint,
but I think it\'ll be fairly trivial to put together a way to
equivalently get, I mean admittedly, it\'s going to be a little heavier
because you have to get multiple blocks, right, but okay, and I do have
another question, but I don\'t want to monopolize this, so I\'ll wait to
see if other people have a question first, all right, next question on
the answer is was Jeff, so go ahead, Jeffy should be able to unmute to
ask your question.

Just wanted to ask if there\'s a cluster we started in the first
thousand blocks, is there any change to change that procedure, because
the snapshot won\'t have the result in it, yeah, we have a talk about
this, so in that case when the snapshot is taken during the reward
computation phase, the snapshot has to wait, it has to wait until the
result is available and store the result there, okay, zans, you\'re
welcome to ask your question, okay, so, is it the case that validators
will only validate a block if the validator, the leader producing that
block has the appropriate amount of either computation work or rewards
pair at work in that block, or is it the case that a validator can just
sort of like decide not to do it on their block and be like, I\'ve
modified the code to not do this because I don\'t want to burn the CPU
on this and it doesn\'t give me any rewards, so I\'m just not going to
do it, or is that even a possibility, or is it the case the cluster
won\'t even accept a block unless it includes these details because
they\'re expected for I mean they\'re sort of built into the protocol
that this has to happen in this order. Because the order is
deterministic for the current Epoch, there will be some randomness
across different epochs, so if the leader. For example it doesn\'t
matter whether he\'s a leader or not, in fact, if the validator doesn\'t
include the expected set of rewards in the block, I think the harsh were
mismatched, and his block will be rejected, okay, so this part of the
block does go into the hash, so that validators have to agree on it to
vote on it, so I guess that answers my question, and we just kind of
accept that the first and leaders kind of have to do his work without
any compensation, that\'s just kind of like built into the protocol,
there\'s no, you know, need to add any compensation for doing this work,
it\'s not expected to impact the transactions that could be fit into a
block, like it doesn\'t take up extra block space or extra compute time
that could prevent a validator from including as many transactions as it
would, otherwise it\'s the same problem as we have right now with only
one, whatever that first leader or like whatever that first block is,
kind of gets screwed on including transactions because everyone is
Computing this, so you\'d have this is just spreading out the
computation to make it more feasible.

I was just wondering if there\'s an opportunity to sort of like make
that even more addressed by sort of paying a validator for having to do
this, just putting it out there, um, it samples a thousand slots, right,
so it kind of spreads the workout out of everyone, you\'d be kind of
paying everyone equally, which means there\'s no up except the dudes
that are unfortunate and only get one block in the first you know about,
well whatever it takes, like that\'s cool, all right, is there any other
questions about this, the epoch reward changes, okay, cool, so this is a
Cindy still, it\'s a I put the link in the chat for everyone to add to
the discussion later, um, 7015, um, you\'re welcome to discuss it for
the next rest of the time on this call, I\'d like to open the floor for
anybody that has just general Q&A questions, just feel free to raise
your hand and I\'ll give you access to speak and you can ask your
question, if there\'s nobody with questions, we can end this early, so
I\'ll give it a few more moments, did you guys have a, you know,
third-year anniversary party or anything like that?

Thank you, everyone, for joining today, um, there will be a space
afterward that if anybody has any questions or that they don\'t ask
here, welcome to join, so be on Twitter, you can see it on Solana and
underscore devs, and we\'ll happy to chat there as well, thank you, guys
today.
