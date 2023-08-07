# Core Community Call 5 Notes

Meeting Date/Time: Friday, April 21st, 2023 19:00 UTC

Meeting Duration: 27 mins

[Video of the meeting](https://www.youtube.com/watch?v=AEnkivbha0k)

## Decisions

- Retroactive proposal for QUIC connection handling in TPU
- [SIMD-0047](https://github.com/solana-foundation/solana-improvement-documents/pull/47) Syscall to get the last restart slot

## Speakers

1. [Jacob Creech](https://github.com/jacobcreech)
2. [Maximilian Schneider](https://github.com/mschneider)
3. [Stephen Akridge](https://twitter.com/stephenakridge)
4. [Kevin Bowers](https://www.linkedin.com/in/kevinjbowers)
5. [Tom Pointon](https://www.linkedin.com/in/thomaspointon/)
6. Zantetsu
7. [Alexander MeiBner](https://github.com/Lichtso)

## Meeting Notes

### Introduction

Video| [00:00](https://youtu.be/AEnkivbha0k?t=1)
-|-

**Jacob Creech:** Welcome to this month's core community call. So I have posted the agenda in the chat.

So, Max from Mango will be presenting on the [SIMD-0047](https://github.com/solana-foundation/solana-improvement-documents/blob/main/proposals/0047-syscall-and-sysvar-for-last-restart-slot.md), on the new sysvar for the last voted on the slot. And then, he will also want to talk about how the different tuning parameters were chosen for QUIC.

Go ahead Max.

**Maximilian Schneider:** Sorry for being late.

**Jacob Creech:** So the floor is yours. Max, you can go ahead and get started with whichever one you want to get started with. Either the QUIC tuning parameters or your SIMD that you have been working on.

### QUIC

Video| [01:02](https://youtu.be/AEnkivbha0k?t=62)
-|-

**Maximilian Schneider:** Yeah, so for the QUIC tuning parameters, the thing with that was like my request which was that we have someone who worked on it previously lead the discussion. I don't know if someone is here that worked on it. And I can only give the outside perspective of reverse engineering some of those things. But you know, reverse engineering requirements is very difficult sometimes and that was the main request that we get someone who knows all the requirements and maybe to present them. But if there is no one here, then maybe we put it for next time.

**Jacob Creech:** Stephen Akridge, could you talk a little bit how you all choose tuning for QUIC.

**Stephen Akridge:** Yeah, I can talk a little bit. Yeah, we had a somewhat specific performance idea in mind right. We want it to be in that like 50k to 100K TPS range in the frontend or at most try to split the streams that way. And allow write enough streams for each client to be able to fit within that. So, you know it was mainly from like benchmarking in the test environment and seeing kind of what bandwidth we could obtain unlike a somewhat optimal connection right which would be like the worst case in terms of TPS in like the real validator. And trying to calibrate that to what we thought like the backend behind QUIC could sustain or handle. And also like keeping the memory within something that was feasible for like a normal validator. So obviously, you have got more streams, you need to have more memory to hold all the packets that are kind of in flight. And potentially reconstruct a transaction that might cross multiple packets. So the more outstanding streams you allow, the more memory you need to frame all those incoming streams.

**Maximilian Schneider:** Yeah, one limit we found and I think we were curious about was I think it's like 2,000 transactions every 50 milliseconds like a leaky bucket model.


**Stephen Akridge:** That's just to verify constants right. Yeah, I think like we were talking about in Discord right that's not like a hard limit. The 50 milliseconds is kind of a target for that stage to not like that's the time it is spending to verify and things are coming in behind it right into the channel to get backed up. So we want to make sure that we are pinging the channel like pretty frequently like within 100 milliseconds to make sure that the channel can back up faster than the incoming packet flow. So, I mean it doesn't have a hard requirement right like you have a machine that can basically verify 10,000 transactions in 50 milliseconds. It will clear all of those and it will bring up the new batch and it will verify it. But it is just a way to keep the queue from doing an exponential fill-up. If you have a large batch and you let it fill in behind you and that is even larger than the next batch then you are just like an unbounded condition and that it's not good for the memory in the validator.

So there is a couple different solutions to that but you know we didn't like the bounded channel solution because the visibility into the channel is not very good. You can't drop intelligently inside the channel. So we felt like it was better to just pull everything out of the channel and we have that kind of the dumb random thing for like if we are getting an extreme amount of packets that we really don't have a time to look at really anything in the packet list at all. But if we are in a less extreme scenario where the machine can handle the flow you know we do like a round between senders kind of dropping. But that algorithm can't handle some cases of we have to handle like the case of a really wide pipe coming in and a very small amount of compute and then a very narrow pipe coming in a large amount of computing so that all those two variables can be different between across machines. So trying to handle any kind of configuration I guess machines.

**Maximilian Schneider:** Yeah that makes sense. I think this limit is a bit surprising. At least for us it was a bit surprising. There is another one I think it is the receive window size that I think have caused some discussion a few times and I think the idea that was like you have basically based on the stake different we receive window sizes which limits the number of parallel streams there can be per connection. So, it's like eight connections per identity and then I think there is some depending on the stake there, each connection can have a certain number of stream.

**Stephen Akridge:** Right, yeah the eight connections is just to kind of if you had clients behind like a router or you had a condition where you got disconnected and you needed to reconnect you might have some connections overlapping. So just not to like kick you out immediately if you had a stale connection in the connection pool. But yeah, we are using the streams to throttle based on stake and then of course a budget for unstaked as well. And those are subject to tweaking, I think we wanted to roll out of the first version and then see kind of monitor the metrics and then update them potentially as we see the use in the validator. I don't think they are, I mean they seem to have been fairly reasonable defaults but open to tweaking I think.

**Jacob Creech:** Just a question for any of people from the Firedancer team. I know that you all have been working on QUIC. Did you all choose similar like received window parameters in tuning or have you all not gotten to that part yet?

**Kelvin Bowers:** I don't think Nick is on this call, he would be the best person to answer it. Let me go see if I can find Nick.

**Jacob Creech:** Okay, thank you.

**Tom Pointon:** I think he's on vacation today.

**Kelvin Bowers:** Oh then, I probably won't be finding Nick. I don't have the answer to that off top my head.

**Jacob Creech:** Okay, maybe probably we probably should sync as we go further into the implementation of like what are the different parameters that we have seen because in the future I think as Stephen as you said it was a good default to start with but what is a better performance in the long run would be nice.

**Stephen Akridge:** Saw a question in the chat real fast about the state-based variables. So more stake generally gets more bandwidth and more resources and you know say of transactions into the validator. So that's the idea.

**Maximilian Schneider:** Which is really cool and like novel. It is like a Solana Innovation you know you've seen that before.

**Jacob Creech:** Zantetsu did you have a question on this?

**Zantetsu:** Yeah, I just wanted to expand a little bit on that is it because the larger stake is assumed to have larger pipes or is it because larger stake actually sort of does more in some way that requires different variables. And the only reason I ask is because if it is because larger stick is assumed as a proxy for larger pipes then maybe it would be better to have larger pipes actually be something that one could specify on the command line or something you could say how much bandwidth you expect to be able to utilize and then smaller validators can do more if they are able to and larger validators could do less if they are you know it doesn't if that makes sense. Did I come through there?

**Stephen Akridge:** Yeah, it's not necessarily a hardware thing. Although, yeah more higher state validators should have more hardware, more resources I guess and more hardware to handle a higher load.

**Zantetsu:** If I have 10 gigabits but maybe Jump has 100 gigabits or something. So I don't know if they may have more or less stake than I have. So I was just wondering if you know if that's stake is assumed as a proxy for bandwidth or if it's something else that that causes these variables they need to be different.

**Stephen Akridge:** Standard proxy it's just that stake is really the only you know civil resistant identifier that we have in the network right to determine like how much resources and and things that you should be in control of essentially. Like how much block space right should you have. I mean I would say you know those overall bandwidth, those could be like a scaling like if the validator had customized hooks for scaling the overall numbers. You could do that, so you would have like let's say the validator has you know 100,000 connections now or outstanding streams maybe if you had a 10 gig pipe or 100 gig pipe you would want to make it like you know 10,000 or something but you would still like distribute those streams across the stake.

**Zantetsu:** Yeah that's kind of what I'm saying like if there's any opportunity for a validators to custom tune their numbers in ways that better match their actual configuration regardless of their stake that is kind of what I was getting at.

**Stephen Akridge:** Yeah, there isn't today in terms of like a command line flag or anything but that potentially could be added if it seemed to be a limiter or helpful.

### SIMD 0047

Video| [13:20](https://youtu.be/AEnkivbha0k?t=800)
-|-

**Jacob Creech:** In the interest of time, we probably should move on to SIMD that you wanted to talk to. Max, so if you wanted to go ahead and chat about that, we can get it started there.

**Maximilian Schneider:** Yes let me bring it up. can I screen share?

**Jacob Creech:** You should be allowed to let me know if you are not able to.

**Maximilian Schneider:** Okay let's open this file. This is like the newest one we wanted to propose basically. The motivation here is on a high level what do DeFi protocols do when the cluster restarts. I think some protocols have been built in certain slot limits for orders or things like that but especially in I think in lending protocols there is a lot of first come first serve and basically when the network starts up we need to assume that not necessarily all RPC nodes are ready yet like that's something we have seen before is that sometimes a lot of things will not work on a coordinated restart. So the idea here is well, I think right now it is just a little bit random and uncontrolled how things behave. We can expose to the application developer that there was a controlled restart so this is currently the last hard fork on the bank in the reference client. So this is basically just a very simple syscall that you know allows you to access the data and then implement custom logic. Maybe you want to lock down liquidations for another 100 slots until the Oracles have time to update etcetera. Could be that the chain was at a very different state or the price of certain assets were in a very different state before the restart occurred and so this is, it's kind of like just more programmability. The proposal goes in detail what exactly we want to expose and I think there was some feedback already that this would be also a good mechanism to maybe actually expose other things. So just curious before we touch it and create an implementation if there are other data needed than just the slot. It will be good to add it now to the proposal. So we are kind of do one pass over it and it is everything we need.

**Jacob Creech:** I think Zane has a question in the chat for you Max. Question is if the protocol allowed validators to snap the slot timestamp the current time on the restart instead of having to catch up gradually would that be another solution?

**Maximilian Schneider:** Oh so I did some analysis on like how the timestamp works right now in the last restart. I am trying to find it, but I'll post the data later but basically you have the restart slot then usually it's a few more slots vote only and only once those votes come in actually the cluster time gets updated so I think around like three slots after restart you'll have actually a correct cluster time or a somewhat updated plus the time so suddenly jumps from like before the restart after the restart but these are the things that are kind of like can cause actually issues on protocols. Slots and times plus the time is supposed to move fairly connected to each other but in those moments it doesn't. There is a real disconnect there for a few slots and then transactions usually start piling in around that time as well as the cluster time starts moving again for user transactions not vote transactions.

I think this is just one of the effects that we're seeing right so that's the one you are describing this is like one of the symptoms then we can circumvent with this measure but I think in general this information is a little bit more rich. I think there were some ideas about lending protocols blocking liquidations and allowing people to deposit more collateral kind of like a margin call scenario where you have at least you know 2 minutes to pop up your balance because there is a freaking cluster restart you know it's not happening every day and it's maybe not the user's fault that they couldn't manage their position. I think that was that was like one of the main concerns why we wanted to be as configurable as possible and as exposed as possible to the application developer rather than just fixing just one particular issue about cluster time.

**Jacob Creech:** Okay cool. So if anybody has any questions, you can voice them now until we end this. If not there is also I posted the link to the PR for the SIMD in the chat so that if you can't get to it here we can do the discussion on the SIMD as well. I have some time for anybody that has any questions for this.

**Alexander MeiBner:**  Yeah excellent idea from proper runtime developments. So two things, one is terminology so I think it's also written in the comments that restarts slot and hard fork are two different things. So hard fork would be a scenario in which a part of the cluster on purpose tries to diverge from the rest and make up their own cluster essentially which is properly not meant to. The other thing is that we are trying to move away from syscall specifically and we will be using or completely replacing them by bit and programs in the future for all the things which actually do compute anything right but in this case this is just a lookup of some global value without any computation behind it so it should probably go into a syscall so system variable. And I can't say what the Jump team is going to do but in our implementation we feed all the syscall with system variables anyway. So that would only be like a through code identity function this is called kind of useless in that sense. Yeah, just saying probably easier to start off with sysvar and think about what else you want to put in there.

**Maximilian Schneider:** Anyone has proposals for what else to put in?

**Jacob Creech:** I think if no one has any other proposals, we can end here. We can continue the discussion on the SIMD. Zantetsu, you have a question? Go ahead.

**Zantetsu:** Yeah my question is only is there a value in knowing more than the last restart like say there were two restarts within I don't know two hours because there was some problem that recurred would that be useful to know or is it always only the last restart is useful to know? In other words does it really need to be a single value or do you want it to be some like set of `n` values that end most recent restarts.

**Maximilian Schneider:** I think I would rather see what adoption looks like on the single value and then go from there. Is a new feature historic value I find it hard to reason about historic values.

**Zantetsu:** Well but I mean in particular you are identifying a use case that you believe covers something that's important to you. I am asking you if you had two restarts that happened within 4 hours, would that change what you would want to know or not?

**Maximilian Schneider:** No probably not. But I think you want to get back to normal operation as quickly as possible and I think that the delays probably within 100 slots for most applications. So that the time frame concerned are just very short.

I wouldn't know what 4 hours ago another restart would change on the current situation right so of course unfortunately.

**Zantetsu:** Okay thank you that does answer my question. Thank you.

**Maximilian Schneider:** I had a follow-up for Alex, you mentioned that like the current hard fork slot is currently synonymous for a restart but there's other ways to trigger a restart but there's no way there's no consensus stage shared about those resources right or is there another way to get those?

**Alexander MeiBner:**  Yeah it's a kind of the other problem the security issue how are you gonna reach consensus about the restart slots right. But I was really talking just about the terminology that hard fork means that you are diverging on purpose because part of the cluster wants to do something else yeah
and otherwise it's just a fork like any other fork that will get pruned at some point.

**Maximilian Schneider:** Yeah it is a slow block right out basically so yeah I think the the differentiation between like a slow block and the restart is probably not so like on an application it doesn't really make a difference right if you have a one hour slow block or you have a hard fork it's probably the same right if it took like an hour to organize that or 24 hours.

**Alexander MeiBner:** What I mean there's no like terminology called this or anything but.

**Maximilian Schneider:** Yeah

**Alexander MeiBner:** If people for example say like Bitcoin and Bitcoin Go these are hard folks, well like what they did on Ethereum back in the day. Because the community actually switched to an entirely different protocol version in the restart slot and this is not I mean this is obviously depends on the exact reset scenario but if you are just reloading the same version again always only with minor bug fixes and all that we on the same thing then this is not really hard fork because it's not you're not forking off from anything else. But like what people understand on that term hard fork means that then there are two blockchains and two sets of validators and two networks essentially which do different things. And I mean usually restart tries to avoid that scenario of hard forking and instead come back to one consensus of global networking.

**Jacob Creech:** Alright just being cognisant of time because we are one minute overtime. Let's bring this discussion further into the SIMD. I will post it again in the chat and then we can further the discussion on this specifics SIMD 47 with Max and Alexander and more.

Thank you all for coming on today on this month's core community call.

**Maximilian Schneider:** Thanks for hosting us Jacob.

**Kevin Galler:** Thank you.