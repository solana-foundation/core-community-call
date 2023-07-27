Solana Core Community Call - Feb 17th, 2023

Time: Feb 17th, 2023

Link Video:
https://www.youtube.com/watch?v=XkkxQAF-HhE&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=1

Speakers profiles

Galactus from the Mango team:https://github.com/godmodegalactus

Topic: Talking about application account right fees and the introduction
of a new kind of fee when interacting with DAPs.

Philip from Fire Dancer: https://twitter.com/jump_firedancer

Asked a question regarding who can call the rebate for application fees
and discussed the ownership of PDAs (Program-Derived Accounts).

Jerry from Ellipsis:
https://www.linkedin.com/in/jerry-francis-8033a8225?originalSubdomain=tz

Contributed to the discussion about who can call the rebate for
application fees and clarified the process for invoking signed
instructions for PDAs.

Zentatsu:

Presented on Simd 33, which is about \"timely vote credits\" to address
the issue of some validators achieving higher vote credits by delaying
votes and surveying forks before voting.

Richard from Finance JP Team:https://www.linkedin.com/in/rman/

Ashwin from Solana Labs:https://github.com/AshwinSekar

Key concepts mentioned:

SIMD 16 Application Write Fees - https://github.com/solana-foundation/..

SIMD 33 Timely Vote Credits - https://github.com/solana-foundation/...

Solana runtime -
https://docs.solana.com/developing/programming-model/runtime

Welcome all to the Core Communitycall this month. Today we\'ll have
Galactus from the Mango team talking about 70, 16 which is application
account right fees and then discussing Cindy 33 which is the timely vote
credits keeping in mind time.

I like to keep or I\'d like to cap each discussion about 15 minutes each
to give each group their ability to present and discuss with the rest of
the core team but Galactus would you like to go ahead and start.

Can you guys hear me well? Okay that\'s great! Thank you Jacob.

So today I will present to you the application fees. I\'ll just share my
screen. I\'m a gym character from mango and today I will present to you
application fees. We want to introduce a new kind of piece. It is like a
fee when you interact with the DAP and this is actually will go to the
DAPAuthority like the account controlled by the dab instead of going to
the validator itself. There will be a rebate mechanism where the DAP can
actually give back the fees to the user so that the overall transaction
of the cluster won\'t increase. These kind of fees actually will be paid
even if the transaction eventually fails.

So why do we want to introduce this application piece? So right now what
we see in the cluster is like in Solana transaction scheduler, we put
transactions into batches and each batch is executed parallelly. So what
happens is like a batch will take locks on its accounts. For example,
right locks and so no other batch can actually use the same account in
the same way at the same time. So these transactions are read right or
even forwarded to the next validator so if this actually creates an
incentive to span the whole spam the whole cluster. For example, like
high-frequency market-makers, they will just create new transactions
spamming the network. Actually, this will create more and more
transactions that will actually touch the same kind of accounts and
eventually it will choke the validator with these transactions. So we
want to discourage this kind of behavior of spamming and even there are
custom programs which will only CPI into the dab if they can extract
profit.

To discuss this kind of behavior and to encourage actually creating
proper and valid transactions we want to introduce this kind of fee.
Eventually this will reduce congestion in the Solana Network thank you
and actually Dapps will also have the ability to collect extra fees with
this proposal so these are the resources we have like the Simd 16. We
have also created a Pock and adapted based on the puck. We want to
introduce a new native Solana program called application fees program
with this program ID. I will just talk a little bit about three bits so
the owner of the application fee like the owner of the the account can
issue a rebate by CPI into like CPI into a special instruction like they
can issue a partial or full rebate to the fees and the rebate will be
transferred back to the pair in the same at the end of the transaction.
Actually if the authority issues the rebate but eventually if the
transaction fails then there will be no rebates. So we have multiple
ways to implement this feature: the first thing was storing the
application fees on The Ledger. This is the implementation done in the
park but recently there were some complications like calculating the
fees because we have to load all the accounts and everything.

So we have decided to go more like a setting application fee by
instruction so I will just explain more about this. About setting, we
have three instructions principally: one is pay application fee, in
which pair agrees to pay application fee for a dab for a certain account
then the dab actually can use; the second instruction check application
fee where it just check for an account if this amount is paid or not;
and the third, which is debate where that can rebate the user its
application history like the fees so the pay application fee
instructions can take multiple accounts.

So you can give a list of accounts and passes in accounts and the list
of fees you want to pay for each account as an array of events. So the
instruction must include this instruction of interacting with Dapps,
which have implemented an application fee feature.

Even if the transaction fails, the pair will end up paying this
application fees, where they could even pay the application fee if like
he include this instruction. But actually it was not needed or he may
overpay the fees, this instruction is used by the runtime - Solana
runtime. So you cannot CPI into this instruction from another smart
contract and you can actually include multiple accounts belonging to
different apps. This won\'t be an issue.

So the second instruction is check application fees instruction where
Dapps can issue CPI into to check if they have paid application fees.
Here you have one account and where you want to, where we expect to have
an application fee and the expected application fee. If the application
fee is partially paid or not paid the instruction will give an error
written or an error application fee not paid error. If it\'s fully or
overpaid it will return okay. In case of partial payment the user will
lose the amount he has paid as application fee. The instruction which
can be called multiple times across multiple instructions.

It won\'t be an issue with Jim Galactus just being mindful of time. We
have five minutes, you have to cut down.

I know that there\'s been a decent amount of discussion on this MD and
just like to open the floor for anybody with questions that have read
this MD or have so far read the presentation.

This is Phillip profile from fire dancer: I still wanted to ask you
about this: Who exactly can call to rebate?

Actually, I understood your question. It was more like in the case of
pdas who really called the rebate right? That was your question
initially? So suppose if it\'s a PDA, I guess I have two different
interpretations of what you\'ve been saying. One is if it\'s a PDA then
the program that can sign for the PDA can call rebate exactly and other
times you\'ve talked about the owner of the account being able to call
the rebate. Sure, those are two different things and if it\'s an account
which is larger than a PDA then anyways I just have two different
understandings and I still don\'t feel like I have full clarity on which
one you mean.

Okay so actually if you check the metadata of an account like account
metadata, you always have an owner associated with each account.
Usually, when it\'s a key pair like a normal key pair it\'s the same
owner even for the PDA. But with assign instructions you can transfer
the ownership of the account to some other keypad like some other
entity. So in that case they have to sign this transaction, they have to
sign this instruction to initiate the rebate.

If you have a PDA and the PDA is a token account then the owner of the
PDA is the token program exactly one you can sign for it is your
program?

No, it\'s actually the token program because a token program has to
invoke science like invoke sign instruction to sign the PDA, which you
cannot sign on behalf of the token program. The PDF you see doesn\'t
think that\'s right but maybe I\'m mistaken. I think it\'s where you
derive the PDA from that program ID is the one that can evoke a sign but
I\'ll double check on this and get back to you.

We can discuss it more and this is Jerry from Ellipsis. Hey guys, so
based on the signing topic, I think Philip is correct when you have an
invoked sign occur. it\'s a PDA of the calling program not of whatever
program is being invoked. Let me think, I actually think it\'s like when
you create a PDA you like, and then you give it to a program ID this
PDA, and then you can just say invoke signed using foreign. Let me get
back on this or maybe I can explain how it works. I\'ve explained this a
number of times before about the way that it works.

Just to make sure that we\'re all on the same semantically is you have a
program and you create a new public key or you create a new address that
is derived from that program ID but doesn\'t live on the curve. From
that program, you\'re able to call invoke signed into a downstream
program with the PDA as assigner so you just wanted to make sure
there\'s no confusion around like how that that process works here but
so we have we have the owner right we have the owner which is the
program which has created this PDA.

Right in the end so we have a fixed owner of the of the PDA. So the
fixed owner of the PDA so when you say fixed owner do you mean the
system level owner or do you mean owner of this PDA account that has a
certain structure tied to it? right because when we\'re talking about
the owner of most pdas if you create a PDA from any if you have a PDAv
from any arbitrary program the owner is always going to start a system
program. There is never going to be like an owner that\'s something else
unless you explicitly allocate and assign.

Okay, I Agree with you, the same program is the owner. So what are you
saying when you say fixed owner of PDA I\'m not quite following what
you\'re talking about there. I guess let me look into it more and I will
write more about that in this IMD.

Is it okay with you guys? Alright. sounds good. I would love to see the
exact details for this so I Think i\'m being mindful of time. We\'ll
move on to the next person. Thank you thank you very much for further
discussion on this Cindy you can find the link I have posted in the chat
I\'ll post it again in the chat. We can both discuss there more as well
as in Discordv under core technology.

I\'m moving on to the next one: let\'s go to Zentatsu with Simd 33. Hi
thanks, I\'m gonna share my screen if that\'s okay and can you please
tell me how much time I should expect to have here? Is it another 15
minutes or is it about more than that or until the until he calls end so
roughly about 13 14 minutes.

Okay, thank you, so I will share my screen. Oh shoot, well, I can\'t
share my screen because this is the first time I\'ve used this program
on my new computer, and I will have to go to some permission settings to
be able to do that. So, I hope I can just reference what I\'m talking
about, and people will be able to follow. So, I made a proposal quite a
while ago for this thing I call \'timely vote credits.\' The purpose
that I\'m speaking about this today is that I really just want to make
sure that anybody at Solana Labs or anyone else who\'s a Solana
developer, that is unfamiliar with the topic, can have some familiarity
and can ask me any questions. In addition, I was hoping that after I
make my case for the idea, that I maybe can get somebody at Sona Labs to
kind of be a champion of the change, to help me get it pushed through
the various processes that are required to get it there. I\'ve had a
pull request open, and it\'s kind of achieved attention sometimes, but I
don\'t think very consistently, and I think that it\'s\... I don\'t know
what the existing mechanism is for community contributions to the Solana
codebase, but I hope that there can be a process by which there\'s maybe
a champion for changes that can act on their behalf from Solana Labs
because from the outside, it\'s kind of hard to get traction without
that. Okay, so speaking about the change specifically, for a long time,
we\'ve noticed that the validator set kind of watches this since our\...
you know, a lot of what we do and the profit we want to try to make
comes from, you know, our vote credit, about credit achievements, that
there are some validators that are appearing to be achieving higher vote
credits essentially by delaying their votes and surveying the state of
forks before casting votes. Now, of course, all validators do this to
some degree because built into the Solana codebase and also built into
sort of sane voting practices is the idea of waiting a little bit if you
haven\'t seen consensus achieved on the fork that you\'re voting on, so
that you don\'t get too far ahead of consensus and then get locked very
up for a very long time should it end up being the case you\'re on the
wrong fork. So there\'s some natural amount of that that occurs. But
there\'s also, you know, within the domain of making that choice,
there\'s also explicitly waiting for a long time to make a vote to try
to make the most accurate vote. And we don\'t want validators to wait on
votes because that will slow down consensus, which will then slow down,
you know, user perception of transaction completion. So I propose this
idea where vote credits would be calculated based upon the latency of
the vote, which is how long it takes from the time that the vote is cast
to when it, or sorry, how the vote, the slot that\'s being voted on, how
long it takes for that vote to land on the chain. And I\'ve written that
up in CINDY 33, and I\'ve implemented a pull request for that, and I\'ve
gotten lots of great feedback from Sona Labs over months of time, and
I\'ve tried to incorporate all that feedback into the change that I\'ve
proposed. The current proposal is that it be done in stages, in several
changes because the change requires updating the actual vote account
state for all vote accounts because extra data needs to be stored in
vote accounts to track this latency, and that requires new space in vote
accounts, and so that adds complexities because, you know, obviously it
has to be done in a very careful manner so that nobody\'s prevented from
voting or vote transactions don\'t fail for validators because of bugs.
And also because expanding the size of vote accounts requires adding
lamps to achieve new rent-exemption levels, and there\'s a consideration
there. All of that is written up in the CINDY, I think.

And also, I\'ve, you know, sometimes these long-running changes have a
lot of discussions that can be kind of hard to follow, so I hope that\'s
not the case with the change I have open right now. But I\'m more than
happy to answer questions about the change and also about the proposal.
If I had been able to share my screen, I\'d show the chart that I\'ve
been creating periodically where I have been looking at the Historical
vote State over every Epoch or many epochs, and then Computing the vote
credits that were achieved by validators and the vote credits that would
be achieved if this proposal were implemented. And having a table
showing sort of what the effects are, and from that table, I believe you
can see kind of who the laggers are because you can see which validators
have, say, the top four or five achieved vote credits every Epoch would
have significantly reduced credits each of them because a significant
fraction of their votes are delayed, and I think that is evidence that
this is what\'s happening. And in fact, I think that economically
speaking, the ideal voting pattern right now is to vote only on
finalized blocks because you\'ll get the maximum possible credits. And
if you\'re a private validator, since none of your votes will ever be,
you know, on a fork that ever has a chance of dying, so you\'ll never
get locked out and you\'ll always achieve maximum credits. And in fact,
the validator code base and the protocol will accept all those votes
because it accepts votes up to, I think, two or three hundred or four
hundred slots old, so it\'s very likely that you\'ll be able to land
every vote if you vote in that manner. But of course, that\'s a strategy
that would halt the cluster for everyone in it, so we don\'t want people
doing that, and this proposal is a way to ensure that that strategy and
strategies that would artificially delay any amount are less profitable
or potentially not profitable at all. So that\'s my presentation about
the topic. I\'m sorry I didn\'t share anything on the screen. Are there
any questions, and is there anybody who would be willing to partner up
with me on this? This topic has been somewhat beaten to death. I know
I\'ve proposed it many times. I\'ve talked about it in Discord many
times, so maybe everyone\'s already familiar with it, and if so, great.

Then I guess the only thing left is, is there anybody that would be,
to\... I don\'t know how this works with small labs, how your
organization works with the community, but, you know, and I\'m not
demanding anything. It\'s not something I want to be in, you know, like
taking anybody\'s time from Salon Labs outside of what the organization
thinks is proper for that person. I\'m just hoping that there can be
some kind of collaboration here, just to touch on that real quick. So,
in some D1, we kind of outlined the best way to get your proposal pushed
into the codebase is to be able to write it yourself. So that\'s step
one. I know that you\'ve made some PRs, that\'s great. The second step
is that there\'s also a contributor access policy now for specifically
the Song Labs repos on the mono repo, and you could request triage
access to start working with other people to get this through. Other
than that, I would allow people from Solana Labs to talk about that
since you\'re going your target is the Solana Labs client for first
implementation, but overall in the long term, it would be multiple
client implementations. Hey, this is Ashwin from Solana Labs. I mean,
the first pull request looks almost done, just keep pinging me. I don\'t
know, GitHub notifications are pretty hard, so just keep pinging me in
Trent every time you have an update. I know it takes a while because we
move fast to resolve all the merge conflicts, but I think you\'re doing
everything right to get it on track. If we\'re slow to respond, you
know, just keep DMing me on Discord \[Music\]. I didn\'t know that that
was an option. I didn\'t want to pester anybody, but I\'m happy to
pester. So, I mean, I think he did it a couple of times in the past, and
I\'ve looked at it every time, so just keep me if you\'re not getting a
response. And I didn\'t see the 70. Maybe we should add some reviewers.
I\'ll add myself and Carl, and hopefully, we can get that merged too.
Great if there aren\'t any other obvious questions.\"

There was a question raised by someone at Jp last time. I sort of
brought this up in a validator meeting, and that question, I think, was
about: does this create any kind of perverse voting incentives for
validators? Does it disturb the possibility of achieving consensus
because validators may decide that it\'s more profitable to vote, you
know, to commit more to forks and then potentially get locked out a lot
more? And what does that do? Does it make the blockchain brittle? And
that\'s an interesting discussion to have, and I\'m happy to have that.
I\'ve thought about it a lot, and I don\'t think it does because right
now, it can only make things better. Right now, like I said, the best
incentive is to simply not contribute a consensus and vote on finalized
blocks only. Or if you\'re not going to do that, if you\'re not, if you
don\'t have the stomach for something that you know anti-social, you can
choose your point of how far you want to wait before voting when you
otherwise could vote, and that\'s what the current what I call ligers
are doing, different ones doing it to different degrees, some of them
are, I think, much better at it, and they and as a result, they be much
less affected by this timely vote credits proposal, but all of them will
be to some degree.\"

Hey Richard from finances jp team here, I\'m not sure where that concern
was specifically raised, but if anything, that was probably more like
out of caution since, as far as you know, Solano\'s consensus protocol
has not been formally analyzed, at least internally at jp, so we\'re
having our security auditors look at the current implementation first of
all and trying to achieve some formal proofs about how the network
reaches consensus. But certainly, I think the final answer team gave you
the commitment in Discord that we will implement your proposal. I guess
it\'s just a question for the way this indeed gets accepted about how
all this lands on mainnet. I think that is mostly out of our control,
but for now, we haven\'t worked on implementing tower BFT for the like
our current exist yet, so therefore it\'s also a bit tricky to get
started on the work now, but as soon as we do work on it, we can commit
to also implementing your proposal, and we really appreciate the work on
your site.\"

No problem, I mean, I don\'t, I didn\'t mean to point a finger or
something. It\'s just, it wasn\'t a DM after one of the meetings, and
they expressed very valid concerns. I\'m not even trying to say it was
in some way inappropriate, but there were good concerns, and I just
think that if someone wants to talk about those, I\'m happy to talk
about them, that\'s all because I mean, like you said, it hasn\'t been
formally very formally analyzed, so I think there\'s some math you can
do to, you know, I\'m not that great at math, but some math you can do
to say, well, given the chance of, you know, a fork if I make this vote,
the chance that it\'s on the wrong Court versus the potential credit I
earn by making the vote. I think there\'s some self-referential function
or something that decides what the benefit is of making a vote, and of
course, we want that to be structured such that it\'s best for the
cluster, and I believe that what I\'m proposing is closer to that. But,
like you say, until formal analysis is done, I guess I can\'t prove
that. Yes, there\'s also a sorry, there\'s also the threshold check
which stops you from voting too far consecutively so you\'re still
within a balance of not locking yourself out and being detrimental to
the cluster.\"

And may I point out, and you know, I\'m sure a lot of people know this
already, there are some of us validators who you know are voting and
adding more credits and sort of committing harder to forks and earning
more vote credits as a result, of course, with the potential also of
getting longer lockouts should there be a fork, and that\'s the risk we
take, and whether that\'s a better risk for the customer overall, I
mean, I guess I can\'t say, although thus far in the past year and a
half or two years, it hasn\'t seemed to be a problem, you know
tentatively but at any rate that strategy that I just described will be
worse after this change like it will mean that that strategy will well
actually I guess I\'ll have to think about it. I believe that it will
reduce the benefit of that because I\'m sorry go ahead we see this kind
of change direction of being implemented in other protocols most notably
is too they have an exponential drop-off in rewards on late attestations
so I don\'t see any last reason against implementing timely vote credits
either there\'s a bit more nuanced to the by the Solana Labs code Works
which might in some cases be counter-intuitive to how bft should
resolve. I do hope that we get this proposal through maybe I don\'t know
timelines. Okay, so we\'ve reached time, thank you all for coming today
if we have further discussion the two Some Ds I\'ll post them again in
the chat are simd 16 and some d33 so zans was 33 which is the timely
vote credits and then 16 is what GM Galactus went over with the
application right fees we can continue discussion there also if you want
to do something more ad hoc there\'s the core technology channel in
Salon attack Discord but thank you everyone for coming today.
