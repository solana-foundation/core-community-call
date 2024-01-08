# Core Community Call 10 Notes

Meeting Date/Time: Friday, October 20th, 2023 19:00 UTC

Meeting Duration: 22 minutes 29 seconds

[Video of the meeting](https://www.youtube.com/watch?v=DOaQ_VBFyIw) 

## Speakers

1. [Jacob Creech](https://github.com/jacobcreech)
2. [Joe C](https://github.com/buffalojoec)
3. [Liam Heeger](https://github.com/Iheeger-jump)
4. [Tyera Eulberg](https://github.com/CriesofCarrots)
5. [Galactus | Mango Market](https://github.com/godmodegalactus)
6. Josh Siegel

## Meeting Notes

### Introduction

Video| [00:00](https://youtu.be/DOaQ_VBFyIw?t=1)
-|-

**Jacob Creech:** Alright, welcome everyone to this month's core Community call, today we will be having Joe present on, I believe it's the feature gate threshold activation.

Go ahead and take it away Joe. 

### Feature Gate Threshold Automation

Video |  [00:18](https://youtu.be/DOaQ_VBFyIw?t=18)
-|-

**Joe C:** Cool sounds good thanks Jacob. Hey guys I'm Joe for those of you guys who haven't met me. Yeah, I just wanted to talk a little bit about the SIMD, if any of you guys haven't seen it, it's number 72 [[SIMD 0072](https://github.com/solana-foundation/solana-improvement-documents/pull/72)]. Should I just share my screen, What do you think Jacob or ?

**Jacob Creech:** Go for it.  

#### Overview

Video |  [00:35](https://youtu.be/DOaQ_VBFyIw?t=35)
-|-

**Joe C:** Yeah, I am not going to go through every crazy nit bit detail but we can talk a little bit about the key point and stuff, whatever you guys kind of want to jam on but basically, this is the proposal [here](https://github.com/buffalojoec/solana-improvement-documents/simd-feature-gate-threshold-automation/proposals/0072-feature-gate-threshold-automation.md). If someone coud please provide the link, that'd be great.

And the whole idea is to basically, create a process that is going to be like automated, for kind of doing what we already have laid out with like the manual process for activating a feature. So, like right now you would run like `solana feature activate` and you're supposed to have like, 95% of stake before you do that right,  before like you try to activate a certian feature on a certain version but it's kind of up to you as like a person to go make sure like the CLI will check for you, but you're supposed to make sure that that stake support is there. 

So this process is going to kind of like automate that through the runtime and through a program, and basically the way it's going to kind of work is, if you go through this design part, you can see it like for example there's two like nodes running different versions they have different feature sets and we want to make sure that only the feature sets that have majority like 95% would be activated and these other ones would have to wait. 

So again 95% is the number that we've been going with for now, and we can sort of discuss that later but that seems to be like what we want to stay with, and in a nutshell these three bullets here kind of summarise like on an epoch boundary, the runtime is going to generate a list of Feature Gates that are queued, the validators are going to signal which of the queued feature gets the support and on the next epoch boundary the runtime will activate the ones that have the necessary support and this going to be an automated process, at least being proposed here.  

#### Representing the Current Feature Set

Video |  [02:22](https://youtu.be/DOaQ_VBFyIw?t=142)
-|-

**Joe C:** So for starters like we've been working a little bit to replace this feature 111 account which actually doesn't exist right now, with a program, because this is the owner of like all the feature accounts  that we have deployed when we go to activate. 

So we want to swap that non-existent account for BPF program and that process may or may not take a SIMD like we haven't really I guess landed on that part of it is program itself there is some questions to answer about  the program itself that we can touch on later, but either way just imagine this program sits there and then basically what it's going to do is it's going to use two PDA through this proposal atleast, one of them is going to be sort of take the new requests for activation as the come in, and then at the end of the epoch it will take that and it will finalize it, and that finalised list we're going to call the queue, is what the node will vote on in the following epoch. 

So you are going to have much like two epochs of like process here, so one to activate or queue to activate, and then the second one to actually vote on for support, and then that epochs at the end there of epoch 2 is when you know the ones that have support will be activated. 

##### Signaling Support for Features 

Video |  [03:35](https://youtu.be/DOaQ_VBFyIw?t=215)
-|-

**Joe C:** So what this kind of looks like, is going to be done through what's called a bit mask. So we will have a bit field for all of the features that are sort of like have been proposed. So you'll have this big bit field  and then well however big right, like we've talked a little bit about how what the size should be and then every node will sort of signal a mask of that with a one(1) for features they have and zeros(0) for features they don't, and then everybody will signal that and at the end of the epoch, you'll take a look at all these signalled support and just figure out who's got the proper stake support and who doesn't, and just use the bit field to go ahead and activate only the proper ones and then kind of reset everything.

So that's what this kind of talks about here, this is what that would look like, there's going to be an instruction that I should probably outline a little more that you would basically send this bit mask into the program, it will store this information in  PDAs. So there is one PDA for the central authority bit mask, the bit field, the features that have been requested and then there's, each node will have a PDA of the ones that they support right and this will all kind of refresh every epoch. 


##### Activating Supported Features 

Video |  [04:45](https://youtu.be/DOaQ_VBFyIw?t=265)
-|-

**Joe C:** And so the activation the activation will still kind of work the same in a way, there will just be that check for the stake support. so this is where things are a little bit dicey, because it's, you know computationally intensive, but or can be basically it will walk these accounts to tally up the stake support and then only activate the ones that have the support. But once it knows who, like once we've tallied up who has the support, the rest is kind of like the same, like it'll run the activations that have been written into the code. 

#### Alternatives considered

Video |  [05:17](https://youtu.be/DOaQ_VBFyIw?t=265)
-|-

**Joe C:** So that's like the proposal there are some alternatives I've included here too, like one alternative is we could sort of like not use two PDAs, we could use just one, but that kind of makes things dicey with like the timing of when these signals go out, and another alternative as well, is we could not use transactions, we can instead use the block header. 

But there doesn't seem to be be like a ton of support for that idea, but I would love to sort of open up the conversation to just any thoughts on this especially considering like the multi-client world we're going to be in, want to make sure that everybody has like a voice on this, and we can kind of figure out what looks best for this. 

So anybody has a question to start yet? Yeah Lian go ahead.

### Questions

Video |  [06:11](https://youtu.be/DOaQ_VBFyIw?t=371)
-|-

**Liam Heeger:** Yeah so, I am from the Firedancer team, I have been looking at this, we have been thinking about this issue a lot of new features that have been added as we've been developing here, and so we are intimately kind of involved in how features are being added and having to track all of that. So when this proposal came up and and it was not the first discussion that we've had about this, in fact we've been talking about this since probably day one, atleast internally. 

So I guess like one of the big questions and I thinik like overall the proposal, like the bones of the proposal, are or at least like the philosophy of this proposal is really good that, like okay, maybe like we should be enshrining how features get like activated base on like a stake weighted, you know base on you know which validators are actually running the feature based on their stake weight and then you know allowing them to be activated after that. I think that that's a really good and it's one that we had internally just didn't think about how to do it. 

I think the big question or I guess like there's a few big questions and is and one of them was one that you just mentioned which was like iterating over all of the stake Accounts at the end of an Epoch to determine whether a feature is ready for activation. It's kind of an expensive operation given the number of stake accounts that there are or even to iterate over the number of vote of accounts that number could also end up exploding at some point. 

So it's just something to think about whether there should be just one account maybe where theses things are stored, where we do all the accumulation, I think there's a comment about this too that we've been going back and forth about.  

**Joe C:**  Yeah I think you're right, I think I saw that so you think maybe use one account that everybody sort of appends to?
 
**Liam Heeger:** Maybe not appends to but like there's a counter in there for each individual feature, so like, I know this was kind of mentioned in the comment that I made about like an alternative process and it's certainly I think a worthwhile, instead of having this bit mask having each individual feature have its own account, and every time and have validators have to vote into each feature that is not active yet that  they support and say like, hey I'm good on this feature and they just increment one number in there and then once you reach like you know some percentage of the number of validators right or the number percentage of the amount of stake that is neccessary, you know that feature is ready to be activated or is activatable. 

**Joe C:** So would you suggest that they would write their stake in there then? 

**Liam Heeger:** They have access to that right? I suppose, and I think they can also. 

**Joe C:** I think the problem is the program, doesn't right? like we have no way to like prove that that's true right? like from a program's context or is that wrong? 

**Liam Heeger:** I can't remember if vote accounts actually contain the current actor state. 

**Tyera Eulberg:** I think the, sorry Tiara here, the way that Richard wrote it out on Discord was using the leader schedule and so that is ...
and then having bits that can, you know reflect each slot in the leader schedule and a validator could set the bits for their own leaders slots. 

The concern that I have with that approach is that we now have a lot of contention on those single accounts, if we're concerned about scalability with the number of validators going up, now suddenly you know we have thousands of transactions all trying to hit those same accounts at the same time.What are your thoughts about that. 

**Liam Heeger:** Yeah, so one of the things there, so like the idea there was that it's not a transaction, like the leader implicitly is updating ...
all the time so this would just be another one of those that happens either before after transaction processing and so it really 

**Tyera Eulberg:** So then we have, we'd come back to that same problem with timing in the epoch right? if your leader slots are all at the very beginning, you can't see support later in the epoch.

**Liam Heeger:** But I guess the Issue is that, that person also has a very small stake the if they only get one, they only get one set of slots in the leader schedule, they are probably like or even a handful, they are probably not a large, like amount of the stake that's currently active and so they probably don't constitute like the whole network as a whole is what matters. In my mind and they are like if we are saying a threshold of 95% of state is not going to just be hit all at the beginning of a depot, that makes sense.

**Tyera Eulberg:** Yeah I think we might wanna think that through and I don't know, work it out because I think hypothetically you could have like 20,000 leader slots and they could all because the leader schedule is randomized, they could all be at the start of the epoch and again you kind of have this problem where that whole rest of the epoch is wasted in terms of your support signalling. 

**Liam Heeger:** That is true, that is true, yeah it's something to think about I think there is a trade-off between like having contention on an individual account or having or doing this process of having the leader take care it during their control of the bank.

**Tyera Eulberg:** And I think the, our thinking coming into this is that we are already doing those, those stake assessments on the epoch boundary and it's true that might become a problem in a future where we have 100,000 nodes in the network, but it is a problem that we will already have to deal with.

**Liam Heeger:** Right. I think just a few other things that I have noted down here, I think there's a lot of things that are done in the SIMD that are new and I don't want to be the person who like throws, you know a wrench in the process and it is mentionedd that the BPF, like kind of the process of like enshrining a BPF program which has, like you know some of these very specific properties. 

It's is an important change to get right so that we could do it in the future because we'd love eventually that all of the native programs be BPF too and I think that's an aspirational change that, atleast the people who are working on the time new program runtime V2 are really excited about it too and so it would be good to actually get a process down for that and have their input too so that we can kind of process down for that and have their input too so that we can kind of standardize this at the outset, because like it's not clear who should actually update these programes, who should be responsible and what kind of you know. 

If they are used frequently maybe and they need like maybe some priviledged access to things within the bank that aren't normally available like we should probably think a little bit about, about those too, as well prior to this an then. I think like, I'm happy to write this up but like, kind of a full road map of like how the feature road map or the like feature gate stuff eventually turns into more of a governance-based approach as far as activation where the I think the big concern that I think several people that I have spoken to on this have is that, just updating your software, though may be implicit, it's a kind of implicit, you know conciously doing that it, you are not consenting to all these other new featurees that may affect, you know maybe a validator, you know may not be as profitabale after some change just because of the software update, which now triggers a feature activation right. 

So it would be good to, you know give the community that opportunity to come in and say, hey actually you know, before you, you know I like the software updates because I get the security fixes but, I don't realy  want this other thing that kind of hurts me. 

**Tyera Eulberg:** Totally, totally and I posted a comment on Dischord just kind of at the 11th hour so you might not have had a chance to read it yet but we, we definitely envision that being part of the process and I think that we probably use some problematic language in the SIMD that suggest like,  I don't know if, I hope the word vote doesn't show up there, but it might and then the word support probably also kind of problematic maybe we should have said something more like recongnize, because this is the aspect that we are addressing in the SIMD, is purely the, like hey this feature exists in my validator software aspect of it and isn't intended to replace an actual governance decision for queing a feature. 

**Liam Heeger:** Yeah, I think we should make that really explicit and then as I said before like I think it's worth and  I'm happy to sit down and do it, but figuring out how that, that Lego piece plugs into a future governance system and making sure there's going to be compatibility with anything that wants to do. 

I don't think there's a lot to do there, it's more just that I think we should think about it. So that it's not something we have to then reacrhitect the system for.

**Tyera Eulberg:** Yeah, that sounds great, if you are willing to start on a draft of that, I am sure Joe and I can both chime in and try and get something that seems solid. I don't know if this is a, like overarching orgranizational SIMD or where the best place for it is but.

**Joe C:** Kind of seems like governance is going to be tied into this in multiple ways too, because we have also touched on how we manage the problem upgrades too, especially considering it's like elevated status and that will also tie in as you said Liam, into the other programs that are going to be BPF eventuatlly, so. 

**Liam Heeger:** Yeah, this SIMD kind of highlights some things that want to be done elsewhere too in many ways like and  it will be good to, to knock some of them out, that's all I have. 

**Tyera Eulberg:** Thank you very much.

**Jacob Creech:** There's also a few comments in the chat, both on the I think ...
was talking about, the governance discussions that have been going on and measuring based off the epoch stakes, as well as I think Galactus from Mango is making some comments on the rewards distribution as well as this I guess, is this require is this current SIMD like requiring that SIMD or something similar, So that you can measure the, what's on chain, the stake distribution. 

**Liam Heeger:**  I think it's worth going back and looking. 

**Tyera Eulberg:** I don''t think that we have that requirement because the way we proposed it the stake sort of tallies would be done in the runtime, which has direct access to the data.

**Jacob Creech:** Okay.

**Tyera Eulberg:** Rather than inside a BPF program but, it's certainly worth considering and as things develop, if we push more of the functionality into the program that might change. 

**Jacob Creech:** Got it, right. If there's any questions from the audience feel free to raise you hand, otherwise we might be ending this early and continuing the discussion on this, On Discord. 

Galactus, I will get you on. 

**Galactus | Mango Market:** Yeah, hey thanks Jacob . So actually I wan to disucss another topic and I want to know that If Firedancer will be implementing the Geyser interface, because we are looking into RPC architecture, like revised RPC architecture which will stream all the data or Geyser and.

**Liam Heeger:**  Sorry what was the original question?

**Galactus | Mango Market:** I was like, will Firedancer Implement Geyser Interface or no.

**Liam Heeger:**  I think we are happy to like discuss it, there's not a specific plan yet, but it is certainly something that like we would want internally as far as tooling goes.

**Josh Siegel:** We recognize that there is a need for a Geyser like interface.

**Galactus | Mango Market:** Okay.

**Liam Heeger:**  We will likely provide something that's not identical, but is analogous.

**Galactus | Mango Market:** Okay, yeah okay, it makes sense yeah, and I also want to point out that, there is not enough testing or Geyser and we are like thinking, like strategies to test or in Solana labs client.
So if anyone has some ideas you can share.

**Jacob Creech:** Well also make sure you ask in the channel on Solana tech, Galactus so that we can continue the discussion there. 

**Galactus | Mango Market:** Okay, Okay thanks.

**Jacob Creech:** Alright, if there are no other question, I think ...
you have a question about voting, I am not quite sure if this is the palace for that, but we can continue that in Discord, but otherwise thank you everyone for coming to this month's core community call, let's continue the discussion in Discord. 

Thank you.

**Tyera Eulberg:** Thanks everyone. 

**Joe C:** Thanks everyone.
