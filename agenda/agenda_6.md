# **[AGENDA 6 [Core Community Call - May 19, 2023 - Optimistic Cluster Restart Automation](https://www.youtube.com/watch?v=GA5AVg_svj8&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=5)]{.underline}**

-   Time: May 19, 2023

-   Link Video:
    > [[https://www.youtube.com/watch?v=GA5AVg_svj8&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=5]{.underline}](https://www.youtube.com/watch?v=GA5AVg_svj8&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=5)

```{=html}
<!-- -->
```
-   **Speakers profiles**

> **Will:** Mentioned at the beginning of the text, Will is going to
> talk about the upcoming release changes.
>
> **Wen:** Wen is part of Solana Labs since last September and is
> working on consensus.

-   **Key concepts mentioned:**

```{=html}
<!-- -->
```
-   **Optimistic Cluster Restart Automation -**
    > [[https://github.com/solana-foundation/]{.underline}](https://github.com/solana-foundation/)..
    > .

Welcome, everyone, to this month\'s core Community call. There are a
couple of things on the agenda. So, the first thing that we\'re going to
hear from Will, before you, Will, will be about the upcoming release
changes that they have or that they\'re pushing. And then, we\'ll hear
from Wen on 7046.

Before I get started, I wanted to first call out that if you ever want
to see the agenda for the meeting, you can always go to the core
Community call repo, which I just linked in the chat. And then, there
are two PRs that would be or are getting close to consensus on. So,
it\'d be great if people could find time to give their review on them.
The first one is PR #33, which is the timely vote credits. And then, the
previous one, which we actually already merged in, but would love to
have any extra eyes on it, would be PR #15, which is partition Epoch
Rewards.

Go ahead, well, you can go and see your spiel real quick. Thanks. So, as
you probably all know, the upgrade of mainnet to 11417 is underway. We
asked for 25 at the beginning of this week, and we\'re currently at 27.
Thank you; we appreciate the overachievement there. Everything\'s
looking good. Our plan is to ask for general adoption at the start of
next week. So you can anticipate that request in case anyone hasn\'t
seen it yet. I would encourage you to read the outage report from the
February outage. At the time, it was widely believed that the outage was
caused by the 114 upgrade. That was definitely not the case. I\'ll drop
a link to that report in just a moment in the chat here. It\'s also
linked in the MB announcements channel on Discord.

We also have some audit reports that we\'ll be publishing later today
related to 114. Feel free to peruse those. There\'s nothing scary in
them. If there were, we wouldn\'t be shipping the release. But you know,
I might give you some peace of mind and some interesting reading. So,
thanks to everyone who\'s already upgraded, and looking forward to
getting the rest of the questions on 114 soon.

All right, Wen, you\'re up. Oh, okay, let\'s share my slides first, and
I can present my slide show and share screens. Sorry, what I should
explain\... Oh my, so hello, and to those who I haven\'t met yet, I\'m
Wen, joined Solana Labs last September. And personally, I\'m working on
consensus, and I\'m interested in high-performance and high-reliability
system design.

So, how come I don\'t see my slides to share? Oops, no, sorry. Why
don\'t I have my slides to share? I have it previously, and let me do it
again. Sorry for wasting time. I don\'t see my slides to share. I can
share my whole screen. Maybe that\'ll work. Okay, do you see the slide
now? Yep.

Motivation Requirements. Okay, sounds good. So, motivation and
requirements. We are interested in making Solana more reliable, of
course. So, for high reliability, there are many things you can do.
First of all, you can write very high-quality code. Then you have fewer
outages. But no code is ever perfect, so you need testing to improve it.
But testing couldn\'t catch all problems, so you need monitoring to tell
you what problems you have and when.

Monitoring tells you there is a big outage; you kind of need outage
handling to get the cluster back to the same state. So, there are a lot
of efforts inside Solana to improve code quality, testing, monitoring,
and other stuff, and auditing and formal verification. And today, the
proposal is only about outage handling.

So, first of all, let\'s look at how we handle outages now. When I\'m
talking about outages, I\'m talking about the last outage where the
whole cluster just couldn\'t make progress anymore, not making progress.
And somehow, what we needed to do is, we need to restart validators,
keep them saying a start state to start with so the cluster can continue
functioning again. So, this is called cluster restart, which I have a
link here. It\'s very different from sporadic single validator restart,
which happens all the time and that doesn\'t impact reliability at all
normally because you still have a lot of validators functioning.

So, what we do now, first of all, we would try to find the highest
optimistically confirmed block. Optimistically confirmed means you have
a block which got the votes from the majority of the validators. We use
two-thirds here. So when the block is optimistically confirmed, ideally,
we shouldn\'t roll it back because it may contain user transactions. If
you roll it back, it will have economic impacts, very big economic
impacts.

So the whole design goal in this proposal is to try not to roll back
optimistically confirmed blocks if possible. So, in reality, today, we
also try to do the same. But since today we don\'t really have an
autonomous process to do this, today we use what\'s called social
consensus. So when there is an outage and people gather in the Discord
Channel, they would first confirm, \"Yes, there is an outage. All the
validators are not making progress.\" Then, they\'ll try to decide to go
for a restart because the cluster doesn\'t seem to be recovering. Then,
they\'ll try to see where to restart.

So, we need to have one block which everyone restarts from. And to make
sure we\'re not rolling back user transactions, we need to find this
highest optimistically confirmed block where we agree we will start. And
what we do now is we would normally do a consensus or like voting in the
Discord channel, say, \"My local confirmed block is X, and what do you
see?\" So if most people vote for X, then go with X. So that\'s how we
do it now.

And after that, after we decide which block to start from, the validator
monitors with operators would stop the validator, and sometimes if
there\'s a bug that could lead to an outage again, then we might install
a new binary. But that doesn\'t happen very often. And next, because we
decided we need to start from this block, so everyone needs to have the
same block. So you would create a snapshot with a hard fork at this
block we decided on. And if you don\'t have that block locally, you
would download it from a trusted source.

And after that, you would restart validators with the following
arguments. Two arguments: one is \"wait for a supermajority to add slot
X,\" and the next is \"I think the bank hash at this slot is blah.\" So
this makes sure you restart with the correct snapshot, you have the
correct block and correct hash. And then, you would wait for 80 percent
of the people to reach the same state as you. Then the whole cluster
begins to function. You start to make new blocks again, and you start to
vote, and all things go to normal.

So, there are a lot of problems with this current restart process. Maybe
the biggest problem is that it takes a long time. The whole cluster
restart takes about\... it could take about several hours, which would
make your reliability not very good. And so, in the future, we might
have other efforts to make it faster.

Today, we\'re focused on solving a small problem, which is improving the
process of finding the highest confirmed block. It\'s quite challenging
for Han to manually review the votes of two thousand to three thousand
validators to determine the block. Thus, we aim to develop a protocol
that enables machines to automatically find the highest confirmed block
without Han\'s intervention. Here are the design goals:

Avoid cross-part negatives: If a block was confirmed before the restart,
we should not discard it. Doing so would have severe consequences.

Allow false positives: It\'s acceptable if some blocks are not confirmed
before the restart but are mistakenly considered confirmed.

Let\'s consider a scenario with a 67% threshold. If a slot receives 66%
of the votes and no competing blocks exist, it\'s fine for 80% of the
validators in the restart to decide to start from there, even if it\'s
not yet confirmed. Confirming more is acceptable, but rolling back
confirmed blocks is not allowed. We\'ll prioritize avoiding false
negatives in all design choices, and false positives might be tolerated.

The proposed approach involves incorporating a silent repair phase where
validators negotiate amongst themselves to determine the block they
should restart from. During this phase, no new blocks will be created,
and the aim is to converge quickly. All validators should stick to their
votes from before the restart; no changes are allowed.

To achieve this, we\'ll use gossip to exchange most information during
the repair phase. To prevent interference between validators who have
restarted and those who haven\'t, we\'ll use a new shred version to form
separate gossip groups. Two new gossip messages will be sent: one
containing last-voted work slots and another to ensure everyone shares
the same metadata. This will ensure that all validators have the same
data and metadata by the end of the silent repair phase, allowing them
to make unanimous decisions and start from the same block.

So, last-voted Fork slots is where we share the last vote before the
restart because the validators might be on different slots. Some people
vote faster, some people vote slower, so just one last vote slot is
normally not enough. So we would send nine hours of slots on the same
Fork so that people can\'t get the whole Fork, and people can get a
better view of the metadata. Also, after you repaired everything and you
have all the metadata, then you send out your new vote. We don\'t call
this vote to distinguish between a normal vote and this vote, so here we
call it the Fork, which is actually just a vote. So you send out after
all the repairs are done, and you have automated data where you think we
should restart from. And also, you also send out how many heaviest Forks
you received from other people because we need to decide when we would
exit this silent repair phase after we see that 80 percent of the peers
received this heaviest fourth message from 80 to people. We check that
and everyone agrees on the same block, same slot, and hash. If yes, then
we exit this phase and proceed to a real restart. So we do what we
currently do; otherwise, if anything happens that makes you count,
proceed, just stop, print all debugging information can intervene or
maybe switch back to the old restart method. How much time do I have?
I\'m probably a little bit slow.

We have 11 minutes for both finishing questions, but we can see how far
we can get. Okay, that\'s fine.

So you are welcome to read the slide. So, I\'m just going to generally
introduce the silent repair phase, and then we can proceed to questions.
So first of all, when you restart a validator with this new arg,
immediately we send the last voted pork slots, which is what you voted
last, and all the slots on this fork. And after that, everyone
aggregates the last voted work slots from all the other restarted
validators, and you could start repairing a slot if you think this slot
could potentially have been up and confirmed before the restart. And we
could draw a line somewhere to say these are the candidates who could
have been confirmed before the slot, and the other blocks I don\'t care.
So you repair all the blocks you care about, and after we repair all of
them, you aggregate all the last votes and the last voted work slots and
choose your hub habits for it. So I think we are at 20 minutes now. I
don\'t know whether I\'ve introduced the new method enough so everyone
has a good graph, but we could see if anyone has any questions at this
point.

I can proceed if no one has questions. Does anybody have any questions
currently on the current approach, or should we just continue? Alright,
go ahead and continue then.

When? Okay, so\... Exit the Silent Repair Phase. To exit a silent repair
phase, I think the most important thing in audit handling is to make
sure everyone is on the same page. Otherwise, if you think everyone\'s
on the same page, you single-handedly enter the restart and start making
new blocks, while everyone still is people repairing blocks. That would
be disastrous, and another audit might be happening. So, here we are
very careful when we exit the silent repair phase. We would count
whether enough people are ready for action.

So the current check is whether 80% of the validators receive the
heaviest Fork response from 80% of people. So, we cut 80 here because
this is a current line we draw when we do a restart. We wait for 80% to
join the restart, and then we proceed. And also, even if we see that 80%
of people respond, we also linger for maybe two minutes because a gossip
message propagation takes time. So, linger for two minutes so that my
heaviest fork which contains how many responses I got\... Hence,
everyone can perform security checks.

Check one is: Did everyone agree on the same block, which means the same
slot and the same hash? Also, whether there\'s a local optimistically
confirmed block before the restart, right? So because my local confirmed
block means before the restart, I saw two-thirds of the votes on this
block already. So, you would imagine this block should be on the
selected Fork. It should be the ancestor or it should be the selected
block. If it\'s not the case, then the security check fails because
something\'s wrong. My local confirmed block is getting rolled back,
then you would also exit and hold and wait for an inspection. And if
every check succeeds, then perform the current restart logic. We
probably will click clear the gossip CRDs table so that we don\'t carry
the old restart messages into the new environment, and then we would
automatically start snapshot creation.

In contrast to what we\'re doing today, where we manually ask people to
manually do it using Ledger, here we might start earlier once we find
out everyone agrees on the same block. Then we exit and execute the same
logic we are doing now in this new discussion.

So I have a few links, one is a Google doc. I started this proposal just
starting, so it contains many details and a lot of designs we rejected
and why. Because I\'m currently modifying both the SM Med draft and this
Google doc, so this Google doc might be outdated, and some design
choices might be different if that\'s the case. SMED is the newest
proposal, and the actual SMID draft is here as well.

I also have some description here. Let me know if you have questions. Do
you have questions? Can you hear me? Yes, we can hear you. Yep, so
actually, I\'m from the Mango team, and we are currently implementing
SMD 47, which is like the last three-star slots. So, I\'m wondering if
this silent repair stage should be considered also a restart slot.
Sorry, I haven\'t checked that doc in detail, so I think your proposal,
correct me if I\'m wrong, your proposal is to expose the last restarted
slot somehow through Ledger, right? Exactly, so here I think it\'s
orthogonal, but once the silent repair phase is over, and we know where
we are restarting from, we could also connect to your code and expose
this information somewhere, right? Oh, exactly. So, when it\'s in the
silent repair phase, we don\'t know for sure it\'s a restart. They are
negotiating, but they don\'t know whether they can decide on the same
block. So in that case, I don\'t think we will expose anything, and once
that phase is over and we know we are really entering the restart, then
we can expose that restart slot. Did that answer the question? Okay, it
makes sense. There shouldn\'t actually be any changes to that proposal
because we\'re going, once we\'ve committed to restarting at the
coordinated restart slot, there\'s going to be a hard Fork anyway, and
we\'ll\... So that\'ll update this as far. Okay, that\'s cool. So when
it\'s in the silent repair phase, we don\'t know for sure it\'s a
restart. They are negotiating, but they don\'t know whether they can
decide on the same block. So in that case, I don\'t think we will expose
anything, and once that phase is over and we know we are really entering
the restart, then we can expose that restart slot. Did that answer the
question? Okay, it makes sense. There shouldn\'t actually be any changes
to that proposal because we\'re going once we\'ve committed to
restarting at the coordinated restart slot, there\'s going to be a hard
Fork anyway, and we\'ll\... So that\'ll update this as far. Okay,
that\'s cool. So that works nicely together.

And any other questions? We have a question. I\... I muted you. Say yes,
so I just\... Can you hear me? Okay, yep, okay, sure. So as a validator
operator, I want to say it\'s very encouraging to see how much really
good thought design seems to have gone into what you\'re proposing here.
I predict that what would happen during another restart event is that
there would still be hiccups that will get alerting, and there\'ll be a
lot of confusion and talk. So, and if we\'re kind of racing some
automated process, just keep in mind that you may want controls there,
as certainly messaging from the validator to sort of let us be very
aware of what\'s going on so that we can make decisions about oh, you
know, because if someone believes that there\'s a bug or a reason that
the restart shouldn\'t proceed, allowing something to run away with a
whole cluster restart that we want to pause. Just having controls and
informational messages to help us understand what\'s happening are very
important. I just want to emphasize that, that\'s all.

Yes, I think maybe later there will be\... I would try to get more
feedback from The Operators because this really impacts how you operate
during an outage, right? So it helps to get more feedback there. So I
think first of all, I totally agree, and the current approach is opt-in.
If you don\'t restart your battery data with that flag, nothing would
happen. We would keep the current approach if you feel more comfortable
with that, and also, of course, the outage handling is mostly to assist
people; it\'s not to replace people.

And we\'ll also, of course, give you ways to inspect what\'s happening
inside and ways to decide, \"No, this automatic restart is not working;
I should do something else." That\'s totally doable. It will be all
command-line controlled. Does that answer the question? Yes, it did. If
there\'s anybody else that wants to ask further questions on this in
SMID, I posted in the chat, we can take the discussion there. And thank
you all for joining another Core Community call. You all have a good
month. Thanks.
