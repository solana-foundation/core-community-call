# **[AGENDA 5]{.underline}** [**[Core Community Call - April 21, 2023 -]{.underline}** **[QUIC, Syscall for Restart Slots]{.underline}**](https://www.youtube.com/watch?v=AEnkivbha0k&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=6)

-   Time: April 21, 2023

-   Link Video:
    > [[https://www.youtube.com/watch?v=AEnkivbha0k&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=6]{.underline}](https://www.youtube.com/watch?v=AEnkivbha0k&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=6)

```{=html}
<!-- -->
```
-   **Speakers profiles**

```{=html}
<!-- -->
```
-   Max from Mango:
    > [[https://twitter.com/m_schneider]{.underline}](https://twitter.com/m_schneider)

```{=html}
<!-- -->
```
-   **Key concepts mentioned:**

```{=html}
<!-- -->
```
-   **Syscall to get the last restart slot
    > [[https://github.com/solana-foundation/]{.underline}](https://github.com/solana-foundation/)..
    > .**

-   **QUIC Tuning Parameters**

Welcome to this month\'s core Community Call. I have posted the agenda
in the chat. So, Max from Mango will be presenting on the Simds 46th or
on the new Cisfar for the last vote. Last voted on the slot, and then
he\'ll also want to talk about how quickly the different tuning
parameters were chosen for Quick. And Max just joined. I will promote
you as a panelist. Go ahead, Max. Sorry for being late.

So, the floor is yours, Max. You can go ahead and get started with
whichever one you want to start with---either the quick tuning
parameters or your CMD that you\'ve been working on. For the quick
tuning parameters, the thing that was, like, my request, which I was
hoping for, was that we have someone who worked on it. You know, lead
the discussion. I don\'t know if someone is here that worked on it, and
I can at least give the outside perspective of reverse engineering some
of those things. But, you know, reverse engineering requirements is very
difficult sometimes. And that was the main request, that we get someone
who knows all the requirements, maybe to present them. If there\'s no
one here, then maybe we put it for next time.

Could you talk a little bit about how y\'all chose the tuning for Quick?
I can talk a little bit. We had a somewhat specific performance idea in
mind. We wanted it to be in that 50 to 100K TPS (transactions per
second) range in the front end or, like, at most at 100K TPS. So, we
tried to split the streams that way and allow enough streams for each
client to be able to fit within that. So our approach was mainly from
benchmarking in the test environment and seeing what bandwidth we could
obtain under a somewhat optimal connection---like the worst-case TPS in
the real validator---and trying to calibrate that to what we thought the
back end behind Quick could sustain or handle. Also, we had to consider
keeping the memory within something that was feasible for a normal
validator. Obviously, if you have more streams, you need to have more
memory to hold all the packets that are kind of in flight and
potentially reconstruct a transaction that might cross multiple packets
and things.

So, the more outstanding streams you allow, the more memory you need to
frame all those incoming streams. You have one limit we found, and we
were curious about it. I think it\'s like 2,000 transactions every 50
milliseconds, like a leaky bucket model. That would mean that\'s just to
verify, concentrate. I mean, I think we were talking about it in
Discord, right? That\'s not like a hard limit. The 50 milliseconds is
kind of a target for that stage. It\'s spending time in
\'instigverify,\' and things are coming in behind it, right into the
channel, to get backed up. So, we want to make sure that we\'re pinging
the channel pretty frequently, like within 100 milliseconds, to ensure
that the channel can\'t back up faster than the incoming packet flow.

So, I mean, it doesn\'t have a hard requirement. Like, if you have a
machine that can basically verify 10,000 transactions in 50
milliseconds, it will clear all of those, and it will bring up the new
batch, and it will verify it. But it\'s just a way to keep the queue
from doing an exponential fill up. Right, if you have a large batch and
you let it fill in behind you, and that\'s even larger than the next
batch, then you\'re just, you know, like an unbounded condition, and
that\'s not good for the memory in the Valter. So, there are a couple of
different solutions to that, but, you know, we didn\'t like the
boundedChannel solution because you can\'t really have good visibility
into the channel. You can\'t really drop intelligently inside the
channel. So, we felt like it was better to just pull everything out of
the channel, and we have that kind of, you know, the db random thing for
like if we\'re getting an extreme amount of packets that we really
don\'t have time to look at really anything in the packet list at all.
But if we\'re in a less extreme scenario where the machine can handle
the flow, you know, we do like a round-robin between senders, kind of
dropping, but that algorithm can\'t handle some cases where we have to
handle the case of a really wide pipe coming in and a very small amount
of compute and then a very narrow pipe coming in a large amount of
compute, right? So, all those variables can be different between across
machines. So, trying to handle any kind of configuration, I guess, makes
sense. I think this limit is a bit surprising, right? Because, at least
for us, it was a bit surprising.

There\'s another one I think it\'s the receive window size that we
discussed a few times, and I think the idea that was, like, you have
basically based on the stake, different receive window sizes, which
limits the number of parallel streams there can be per connection. It\'s
like eight connections per identity, and then I think there\'s some,
depending on the stake, each connection can have a certain number of
streams. The eight connections are just to kind of, if you had clients
behind a router or you had a race condition where you got disconnected
and you needed to reconnect, you might have, you know, some connections
overlapping, so just not to kick you out immediately if you had a stale
connection in the connection pool.

But we\'re using the streams to throttle based on stake, and then, of
course, a budget for unstaked as well, and those are subject to
tweaking, I think. We wanted to roll out the first version and then see,
kind of, you know, monitor the metrics and then update them potentially
as we see, you know, the use in the validator. I don\'t think they\'re,
I mean, they seem to have been fairly reasonable defaults but open to
tweaking, I think. For the just a question for any of the people from
the fire dancer team, I know that you all have been working on it quick.
Did y\'all choose similar like received window parameters in tuning, or
have you all not gotten to that part yet? I don\'t think Nick is on this
call; he\'d be the best person to answer it. Let me go see if I can find
Nick. Okay, thank you. I think he\'s on vacation today. Oh, then I
probably won\'t be finding Nick. I don\'t have the answer to that off
the top of my head. Okay, maybe probably we probably should sync as we
go further into the implementation of like what are the different
parameters that we have seen because in the future I think as Steven, as
you said, it was a good default to start with, but what is better
performance in the long run would be nice. So a question that charges
real fast about the state-based variables, right? So, Morse, you have
more stake generally gets more bandwidth and more resources, and you
know, say of transactions into the value, so that\'s the idea, which is
really cool and like a novel, it\'s like a Solana Innovation, you know,
you\'ve seen that before. As Antipsy, did you have a question on this? I
just wanted to expand a little bit on that. Is it because the larger
stake is assumed to have larger pipes, or is it because larger stake
actually sort of does more in some way that requires different
variables? And the only reason I ask is that if it\'s because larger
stake is assumed as a proxy for larger pipes, then maybe it would be
better to have larger pipes actually be something that one could specify
on the command line or something. You could say how much bandwidth you
expect to be able to utilize, and then smaller validators can do more if
they\'re able to, and larger validators could do less if they\'re, you
know, it doesn\'t if that makes sense. Did I come through there? It\'s
not necessarily a hardware thing, although more higher state validators
should have more, you know, hardware, more resources, I guess, and more
hardware to handle a higher load---10 gigabits, but maybe JP has 100
gigabits or something, so I don\'t know if they may have more or less
stake than I have. So I was just wondering if you know if that\'s stake
is assumed as a proxy for bandwidth or if it\'s something else that
causes these variables to need to be different. Standard proxy, it\'s
just that stake is really the only, you know, civil-resistant identifier
that we have in the network right to determine how much us resources and
things that you should be in control of essentially, like how much block
space right should you have. I mean, I would say, you know, those
overall bandwidths could be like a scaling like if the validator had
customized hooks for scaling the overall numbers right, you could do
that right. So you would have like let\'s say the value has you know
100,000 connections now or outstanding streams maybe if you had a 10 GB
or 100 GB you would want to make it like you know 10,000 or something
but you would still like to distribute those streams across the stake.

That\'s kind of what I\'m saying. If there\'s any opportunity for a
protocol to custom-tune their numbers in ways that better match their
actual configuration, regardless of their stake, that\'s kind of what I
was getting at. There isn\'t today in terms of a command line flag or
anything, but that potentially could be added if it seemed to be a
limiter or helpful.

In the interest of time, we probably should move on to this MD that you
wanted to talk to, Max. So, if you want to go ahead and chat about that,
we can get it started there.

\"Okay, let\'s open this file. This is the newest one we wanted to
propose. Basically, the motivation here is, on a high level, what do D5
protocols do when the cluster restarts? I think some protocols have been
built in with certain slot limits for orders or things like that. But
especially in lending protocols, there\'s a lot of \'first-come,
first-serve.\' Basically, when the network starts up, we need to assess
that not necessarily all RPC nodes are already at the right state. Like,
that\'s something we\'ve seen before, is that sometimes a lot of things
will not work on a coordinated restart. So the idea here is, well, right
now, it is just a little bit random and uncontrolled how things behave.
We can expose to the application developer that there was a controlled
restart. So this is currently the last hard time for Hard Fork on the
bank in the reference client. So this is basically just a very simple
system variable that allows you to access that data and then implement
custom logic. Maybe you want to lock down liquidations for another 100
slots until the oracles have time to update, etc. Could be that the
chain was in a very different state or the prices of certain assets were
in a very different state before the restart occurred. So this is kind
of like just more programmability. The proposal goes into detail what
exactly we want to expose, and I think there was some feedback already
that this would also be a good mechanism to maybe actually expose other
things. So just curious, you know, before we touch it and create an
implementation, if there are other data needed than just the slot, it
would be good to add it now to the proposal. So we\'re kind of on the
one pass over it, and it\'s everything we need.\"

\"I think that\'s a question in the chat for you, Max. The question is
if the protocol allowed validators to stop the slot timestamp at the
current time on the restart instead of having to catch up gradually,
would that be another solution?\"

\"Oh, so I did some analysis on how the timestamp works right now in the
last restart. I\'m trying to find it, but I\'ll post the data later. But
basically, you have the restart slot, and then usually it\'s a few more
slots to go, and only once those votes come in, actually, the cluster
time gets updated. So I think around three slots after restart, you\'ll
actually have a correct cluster time or a somewhat updated plus the
time. So suddenly, you\'ll have jumps from before the restart to after
the restart. But these are the things that are kind of like can cause
actual issues in protocols, right? A lot of times, the cluster time is
supposed to move fairly connected to each other, but in those moments,
it doesn\'t. There\'s a real disconnect there for a few slots, and then
transactions usually start piling in around that time as well, as the
cluster time starts moving again due to user transactions, not both
transactions.\"

\"You know, I think this is just one of the effects that we\'re seeing,
right? So that\'s the one you\'re describing. This is like one of the
symptoms that we can circumvent with this measure. But I think, in
general, this information is a little bit more rich. Right now, I think
there were some ideas about lending protocols blocking liquidations and
allowing people to deposit more collateral, kind of like a margin call
scenario where you have at least two minutes to top up your balance
because there\'s a freaking cluster restart. You know, it\'s not
happening every day, and it\'s maybe not the user\'s fault that they
couldn\'t manage their position. I think that was like one of the main
concerns why we wanted to be as configurable as possible and as exposed
as possible to the application developer, rather than just fixing just
one particular issue about cluster time.\"

\"Okay, cool. So if anybody has any questions, they can voice them now.
If not, there\'s also a link to the PR for the Sunday in the chat, so
that if you can\'t get to it here, we can do the discussion on the Cindy
as well. I\'ll ask some time for anybody that has any questions for this
excellent idea from proper runtime developments.\"

\"Two things: one is terminology. I think it\'s also written in the
comments that restarts, slots, and halves are two different things. So a
hard fork would be a scenario in which a part of the cluster on purpose
tries to diverge from the rest and make up their own cluster
essentially, which is probably not meant to be. The other thing is that
we are trying to move away from Cisco specifically, and we will be using
or completely replacing them by fit and programs in the future for all
the things which actually do compute anything. But in this case, this is
just a lookup of some global value without any computation behind it. So
it should probably go into a system variable, and I can\'t say what the
JP scene is going to do, but in our implementation, we feed all the
switches with system variables anyway. So that would only be like a
trivial identity function, which is kind of useless in that sense. Just
saying, probably easier to start off with this voice and think about
what else you want to put in there. Anyone has proposals for what else
to put in?\"

\"Well, here, I think if no one has any other proposals, we can end
here. We can continue the discussion on the Cindy, though. You have a
question? Go ahead.\"

\"Is there ever a value in knowing more than the last restart? Like, say
there were two restarts within, I don\'t know, two hours because there
was some problem that recurred. Would that be useful to know, or is it
always only the last restart that is useful to know? In other words,
does it need to be a single value, or do you want it to be some like a
set of n values that end most recent restarts?\"

\"I think I would rather see what adoption looks like on the single
value and then go from there. Having a new feature, a historic value, I
find it hard to reason about historic values. Well, but I mean, in
particular, you\'re identifying a use case that you believe covers
something that\'s important to you.

I\'m asking you if you had two restarts that happened within four hours,
would that change what you would want to know or not? No, probably not,
but I think you want to get back to normal operation as quickly as
possible. And I think that the delays, probably within 100 slots for
most applications, so as far as the time frame is concerned, are just
very, very short. I wouldn\'t know what four hours ago, another restart
would change on the current situation, right? So, of course,
unfortunately, okay, thank you. That doesn\'t answer my question, thank
you.

I had a follow-up for Alex. You mentioned that the current hard Fork
slot is currently synonymous for a restart, but there are other ways to
trigger a restart, but there\'s no consensus stage shared about those
resources, right? Or is there another way to get those? It\'s a kind of
the other problem, the security issue. How are you gonna reach consensus
about the researchers, right? But I was really talking just about the
terminology that a hard Fork means that you are diverging on purpose
because part of the cluster wants to do something else, and otherwise,
it\'s just a fork like any other fork that will get removed at some
point. It\'s a slow block, right?

Basically, so I think the differentiation between a slow block and the
restart is probably not so\... on an application, it doesn\'t really
make a difference, right? If you have a one-hour small block or you have
a hard fork, it\'s probably the same. If it took like an hour to
organize that or 24 hours, so what I mean, there\'s no like terminology
called this or anything, but if people, for example, say like Bitcoin
and Bitcoin Gold, these are hard forks, well, like what they did one
there back in the day because the community actually switches to an
entirely different protocol version in the restart slide. And this is
not\... I mean, this is obviously depends on the exact reset scenario,
but if you are just relearning the same version again, always only minor
black faces and all that, we are on the same thing, then this is not
really hard for because it\'s not\... you\'re not off from anything
else.

But like what people understand on that term, a hard fork means that
then there are two blockages and two sets of validators and two
networks, essentially, which do a different phase. And I mean, usually,
there is that tries to avoid that scenario of hard forking and instead
come back to one consensus of global networking.

All right, interesting. Coming in sometime because we are one minute
overtime. Let\'s bring this discussion further into the CMD. I will post
it again in the chat, and then we can further the discussion on this
specific MD for 47 with Max and Alexander and more. But thank you all
for coming on today on this month\'s core Community call. Thanks for
watching us, Jacob.
