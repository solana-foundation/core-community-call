# Core Community Call 1 Notes

Meeting Date/Time: Friday, December 16th, 2022 19:00 UTC

Meeting Duration: 30 mins

[Video of the meeting](https://www.youtube.com/watch?v=XZhy9VFGKZc)

## Decisions

- Access control on SIMDs should be clearly defined
- Every feature should have an associated SIMD

## Action Required

- Outline process for critical issues when there's multiple validator clients

## Meeting Notes

### Introduction 
Video | [00:00](https://www.youtube.com/watch?v=XZhy9VFGKZc)
-|-

**Jacob Creech**: This is what we're considering as the first Core Community call or the first Solana Core Developer Sync. What we're hoping that this turns into is a way for developers on the Solana protocol to sync on changes that are being made,  feature developments that they're thinking of, and proposing changes on Solana so that we can more collaborate together as a whole. 

Ultimately it's up to you all as the engineers to make up as you best wish of what this call looks like. In future calls we'll call for an agenda if people want to put forth what they want to talk about on the agenda side. The first item that we want to discuss is like what do you all think that this call should look like and then later we'll also talk about SIMD and what that process looks like so that we can make proposals and actually use it to make changes, and keep aligned on changes on Solana.
Go ahead, Richie 


**Richie Patel**: All right! I think generally this would be most useful to have an online forum where client teams can discuss changes that affect everyone, mainly protocol breaking changes that Solana labs, Jito and Firedancer, and so on would have to implement. I think we should probably start with a process where we first specify something in the SIMD and then go forward. Other things will probably be things that actively concern mainnet and require call intervention to be fixed, I guess that that would be the Quality of Service related incidents and so on.  Was there a written agenda or a proposal for the agenda for this call? I think I saw something, I don't fully remember.


**Jacob Creech**: For this specific call it's to talk about what we both most want out of this as well as talk about the SIMDs, and then anything else that you all want to bring up.

### Read and Write Access for the Solana Protocol 
Video | [02:40](https://youtu.be/XZhy9VFGKZc?t=160)
-|-

**Dan Albert**: Just kind of reflecting on something I've been sharing with the team internally. There's been a lot of calls for action from inside of the community of what I see is more people wanting, I'll say read access and then write access for the Solana protocol. Read access being there's a lot of teams core devs and non-core devs that want better visibility into what's coming down the line, how is this going to impact me, particularly like RPC operators, or Dapp
developers that are going to be impacted by any breaking changes.

That I think maybe doesn't need to be in scope for this call, you know either today's or in
general, but I mean it's something that as the core developer Community, I think we can do a better job of communicating. Obviously engineering roadmaps for
technology like this are incredibly difficult to nail down and I think people understand
that, but figuring out a way as a community or like as an organization, labs or foundation to put out more visibility to the broader community. Then as far as write access: who can make contributions, or rather how does one
go about making contributions by engaging here, by making an informal proposal on Discord or on some other forum, followed by getting feedback here or in SIMD comment threads, and then kind of trying to figure out what does that sort of approval process look like, and should anyone really be owning an approval process when anyone can build from source, and run a client with whatever changes they desire.

So just trying to feel out what people think is best there and then as we go on like to be able to kind of dig into the actual details of like: Okay, what do people actually feel about this particular proposal or that code change, etc.


**Jacob Creech**: Cool, and then to kind of like further jump-start the conversation, I'll drop in the chat the SIMDs that we currently have or that we're working on. We recently kind of launched this process of how we can propose protocol changes to Solana that would be breaking changes something that's a big change economic wise, and what we can do is everybody here can create a proposal that will change how Solana works and then we can use this to discuss what is different, how different changes work, how it affects all of us, especially between Firedancer, Jito and working with the Mango folks. How we can create or keep in line of the big changes coming on Solana in the future, especially when
there are feature activations. How do we do that across teams etc.

### Challenges in making public repositories and their possible solutions 
Video | [06:08](https://youtu.be/XZhy9VFGKZc?t=368)
-|-

**Kevin Bowers**: So, I'm just going to say a bunch of things that are kind of in the space and
some of those are things that we also discussed with Dan a little bit earlier, and I think a lot of this is just related to work going on internally and how we've thought about a lot of these
problems. None of this is anything that I have really prepared remarks on or anything but one of the things that already exists in this. We talk about specs is part of our job is to essentially come up with specs for what the existing protocol is from the current code and there have been some repos set up in places like the Solana Foundation. I know there's a lot of kind of comments on that and that's also somewhat a prerequisite for work we're doing
internally where we've engaged with various experts in formal verification and whatnot, and it's very difficult to formally verify if we don't actually have a spec for doing that. 

So, at least from the kind of specs repo read-write access and whatnot, I’d kind of direct things in that direction unless there's a broad consensus in the community we don't want to have those. I don't really see that as not being an option for the long term if we want to have multiple validators or whatnot and, you know, kind of put people in that direction. I think when it comes down to those that’s not the only repo, like there are the internal repos that we're developing, there's obviously the Solana Labs repos, other people's repos that are kind of sitting around here. We spent a lot of time in the initial discussion when we were asked to look in this area as to what was the right model for doing the engagement and we were deathly afraid that we had two different organizations, different cultures, different development styles, different languages and all that, that if we were to try to do stuff in the same repo it'd be disastrous. It wasn't just that kind of abstract concern internally, Jump has a lot of internal communities with the various trading groups and so on and so forth, and you know doing everything in a common repo just tends not to work. So we've done a lot of stuff internally just for you know non-crypto-related things where we essentially develop stuff at boundaries between repos and come up with interfaces, be they shared memory interfaces or whatnot, and a lot of the stuff that we've been planning on here is to essentially iterate independently in our own repo. Let Solana iterate in their repo. Use the specs that we're doing to essentially define boundaries both from the protocol works but also ways that we can divide stuff up and componentize the existing validator and then swap out various parts as we go and do that. A lot of that is being done at process boundaries, so we don't have to have a whole lot of friction or tight integration between organizations and you know those kinds of logistical problems that can do.

With respect to protocol-breaking changes as we've been going through a lot of stuff and checking more stuff in, we had a lot of ideas and we've spent a lot of time thinking about how we might, you know, bring them to the community. I think one of the things that already exists that we were planning on is we do have some scope in our development plan for essentially doing proof of concept designs and like some of the stuff that we were thinking about doing is just like, you know, we'll set up a proof of concept over here to essentially come up with reference implementation and then bring that to a community as opposed to just having an open-ended abstract discussion to the effect like we don't like this, maybe we could do it this way then have everyone just be kind of like well it's not obvious that'll be better or not, and so at least as far as protocol breaking changes are, at least intent is, we would make recommendations but only make those recommendations after we had some working code to show how those things work.

Sorry for a bunch of rants, I don't know if that necessarily addresses the points that you mentioned but you know I think it's things of making notes on as people were talking

### SIMD merge criteria 
Video | [10:00](https://youtu.be/XZhy9VFGKZc?t=600)
-|-

**Richie Patel**: I think we can take a lot of inspiration from the Ethereum process because that has proven to work quite well at scale and the way they do it is fairly similar. So for any non-trivial change to these specifications and whatnot, usually when you check in an EIP, or
in our case in SIMD, you also have a proof of concept in your own repo and then at least start to
think ahead about how another team might Implement that.


**Dan Albert**: Yeah, I think in my mind I see the bar for actually getting a SIMD merged should be quite high. I see that as the tail end of the sloppier, less formal sort of debate process people testing things trying to figure out what makes the most sense, do the prototyping, and get some sort of initial social consensus from the validators, the developer community that not only do we
think it's a good idea but someone on the team is willing to go ahead and implement it, right. If Solana Labs thinks some SIMD is great but the Firedancer team thinks some SIMD isn't, the same SIMD is a terrible idea, and each client, in the future has 50% of the stake on the network, well both teams need to implement it in order for it to actually right come to fruition. I  agree with your proposal Richie on having the proven concept and I would just want to call out as we're kind of discovering it not to like jump to a SIMD for any random new idea, right. I think this kind of stuff in Discord, GitHub issues like is more appropriate, where people are trying to suss out the interest and feasibility of a new process.


**Richie Patel**: As much as I don't belittle the point, the Ethereum community does check in a lot of EIPs never actually deployed on mainnet, it's nice to have some sort of proposed standard when you get to implementing the idea but often, you have something sitting around for three years and then you might implement it at some point in the future. There's lots of different
branches of these implementations but I think if we restrict ourselves to saying anything that's an accepted SIMD will eventually be implemented and activated then, a lot of this political discussion I think will actually prevent the engineering efforts of standing up a proof of concept, so it's, I would maybe have less restrictions and then say well this is not even an accepted standard but at least it's checked in and it's there someone wants to implement it.


**Dan Albert**: Okay. Yeah, I guess I'm not crazy about the idea of having a SIMD repo become like a Boneyard of unimplemented proposals, right? So there needs to be some kind of filtering mechanism but I agree like we can have a spec or a proposal that everyone agrees is a good idea and is going to take 12 months of work or there's just more urgent issues to be addressed sooner.


**Anatoly Yakovenko**: The problem with those long ideas is that they usually die or by the time you get around to building them, something has changed to where they're no longer valid. So like I think that I would prefer that the specs that do get checked in have like a path to implementation within six months.


**Jacob Creech**: To take from Ethereum as well I believe what they have is if there's a certain point where it just can't be implemented due to a number of changes that happened previously, for example, if it took six months to implement and the things changed is no longer a viable option, they do change in the status to stagnant and they throw it out at that point.


**Kevin Galler**: Probably like marking statuses, is there like a mechanism for this?


**Jacob Creech**: Yeah. There's kind of like a process as outlined in like the first SIMD of what you do is you draft it and then you work with people trying to get consensus on should we move forward with this or should we not. Then once you get that consensus you can move into accepted. To get to implementation phase requires process around feature activation or who's owning the different feature activation in each client. It's who's building the feature, what's the status of it, and how do we get it to actual activation across the network.

### Write Access to SIMDs and Specifications 
Video | [15:11](https://youtu.be/XZhy9VFGKZc?t=911)
-|-

**Richie Patel**: I guess this begs the question of who actually gets to decide to press the merge button and define what consensus is. I think for now it's perfectly valid if Solana labs and
Solana Foundation has this authority since, with the exception of Jito, we're just starting out hitting mainnet. In the long term, it would be nice to maybe define a set of people. I know this always tends to exclude people but I would maybe define consensus like that there is no big agreement, there's like general you know acceptance of a change even if it's not a direct confirmation from each team.


**Jacob Creech**: Yeah, as part of the first SIMD it's there's no obvious outspoken disagreement and that if there is agreement from the majority of core contributors, you can move forward by who has access to do write, triage, or to maintain the repo of SIMDs. I wrote up an access policy that is very similar to how Mozilla does it and how they give access to write or to triage different issues, so that is under review. We can create that list of possible merge-access people. It's really it's both a social consensus as well as like a technical consensus. If someone abuses that power it can easily be taken away as well.


**Dan Albert**: I think there's maybe also been some questions, and tell me if you think this often the weeds are out of scope here, but I think it's good for us to kind of feel out what these kind of processes feel right here. I think there's also a lot of less process heavy and more just sort of navigating the space questions some folks have come up with just like, Hey I wanna like I have this PR open against say the Solana Labs repo right because I'd like to fix a bug or I want to make a change whether it's a small thing or a bigger thing but they're a community member or maybe it's someone
from Jito wants to push something back to Labs. 

Maybe it doesn't require a SIMD because it's not a protocol thing. I'm curious maybe Anatoly or someone from the Labs team who's on the call, do you guys have a sense, either written or informal, I know they're like the quality has been very mixed right over time because anyone can open a PR against the repo like, how does the Labs team view, you know, receiving and merging changes?


**Anatoly Yakovenko**: I think if it has like a feature activation it should probably have an associated proposal because the feature activation required implies that there is something breaking in the client that it's not like a change you can easily roll in and it should probably be, or it definitely needs to be communicated to all the other, to the Firedancer client and we should use like the proposal process for that.


**Dan Albert**: Meaning anything that requires feature activation should have a corresponding SIMD?


**Anatoly Yakovenko**: Yep 


**Dan Albert**: Okay. Yeah, I think that's good.

### What to do about implementation details that are not protocol defining 
Video | [18:52](https://youtu.be/XZhy9VFGKZc?t=1132)
-|-

**Buffalu**: What do you think about like the changes that we've made to the validator where they're not necessarily related to consensus but they are like pretty significant changes?


**Anatoly Yakovenko**: I would love to have a proposal for these are the API hooks that you guys need that for all conforming clients to implement and then the Firedancer team and us, we can just conform to that standard like these are the hooks that Jito needs and this is how we load like, I really want to load a dynamic shared object instead of like downloading a separate Jito client.


**Buffalu**: Yeah, same. Okay, cool.


**Dan Albert**: I think Richie and Kevin on Firedancer had a kind of similar plans, is that right? To basically kind of design or spec out these interfaces that aren't necessarily protocol defining but it's just more of like an implementation detail.


**Kevin Bowers**: Those are definitely fresh on our mind as we're getting very very close to getting the last bits of plumbing put together on our end and I think the big things you know far. I'm going to answer this may be more mechanically because there are different ways you could approach this from a process standpoint or what it means to a repo standpoint but our intent is to take the existing validator at Solana and do all the plumbing that's needed on that side to interface with our component one and we already have basically those points where we want to go and how to do those boundaries and whatnot, and then just like proof of concept like work it out and say okay we worked it out, and then essentially kick that to you guys to then say what is the proper way to do that, and I think part of the call here is getting some idea what those processes are. Now, since that wouldn't be something that'd be a protocol-breaking change or whatnot doesn't sound like to have a SIMD involved but we'd say here are some changes we'd like these to go in, we'd like to be respected kind of going forward that defines those internal API hooks, those API hooks then you know any other organization Jito or whatnot could go ahead and then be using those for doing their development against too and they could interface at those points.

We also are trying to minimize the complexity and the surface area of those interfaces so we aren't too particular about it so if we get it wrong or you know somebody wants some other kind of change on that it's not that big a change. We're not trying to find the max cut and create the most complex interfaces across or trying to find very very minimal points to simplify this kind of organizational interfacing that we're talking about here.

### Validator Specification and Internal API 
Video | [21:30](https://youtu.be/XZhy9VFGKZc?t=1290)
-|-

**Richie Patel**: I remember as part of the three repos we created for these inter clients, local APIs there were first of all these specs, the SIMD, but there's also a validated APIs repo that Solana Foundation created, and originally that was the plan to have all the C structure definitions and so on posted there but I think this can quickly get quite complicated if that needs to be compatible with like three different CI processes and build systems and so on. So I'm not too sure what the best platform would be, maybe we just copy headers between clients and socially ensure that they don't get out of sync instead of using any complicated tools.


**Dan Albert**: Yeah, I'm happy to remove that validator internal API repo. It hasn't seen any activity since we kind of kicked that idea around a while ago, because it's really not, like you said, it's not a protocol definition thing, it's more of an implementation for convenience between multiple teams.

**Jacob Creech**: And then a question for the specs: whenever we have a SIMD proposed and it's accepted, do you all want to wait until after like it's actually implemented across the different clients or do you, like when do the specs change in accordance to like upgrades the network?


**Richie Patel**: I mean as far as I can tell, SIMDs are proposed changes and specs are authoritative sources of what the protocol actually is and merging that before and anything is activated on mainnet is I guess technically wrong, but I don't know if we want to split hairs here. It's hard to tell but I would like when it's clear that a feature is about to be activated I think that's a good point when we want to merge things into the specs repo.


**Jacob Creech**: Okay, so we're coming up a little bit on time for this call anyways. So I guess for future calls what we'll do is we will kind of like put out calls for agenda items, future could be including like a SIMD that someone proposed or like Hey, we're kind of talking about changing
this in the validator client this restructure what do you all think about?

So we'll do that for the new year, but is there anything else that people want to bring up before the end here?


**Zano**: Yeah, another thing that would be good on these calls, let me know if you guys agree, is like having a spotlight for like large changes that have been merged in and kind of so yeah spotlighting that may be just like giving a brief overview of what happened there.


**Jacob Creech**: Yeah, that's a really good one.


**Dan Albert**: Yeah, I wonder,  I'm just thinking because like that's a topic that I think a large audience would be, a larger audience than the speakers on this call might be interested in,
right? So, this is sort of what I was going back to earlier right there are more people than the core devs that are interested in you know I'll say read access to how new changes are coming in. This would certainly be a good forum to spotlight I'll say like a debate or discussion on said content, but I'm wondering if like the initial presentation could be the separate thing that's put out to the public and then we say, Hey I'm you know this quarterly call we're gonna you know to pick it apart or provide feedback or whatever.

I'm just thinking based on the number of people here and the number of opinions that I'm sure could be voiced, we will want to make space for the debate right, because a presentation could just be like, Hey I like recorded this 30-minute thing, deliver it to the public, and then we have a review period after.

### Process for security patches 
Video | [23:49](https://youtu.be/XZhy9VFGKZc?t=1429)
-|-

**Alexander Meißner**: I have a point properly for the next call and it's to design a process for security patches, which I think is quite tricky.


**Anatoly Yakovenko**: What do you mean by security patches? Like zero days?


**Alexander Meißner**: Yeah, at the moment we don't have a released structured way to deal with GitHub security issues. It's always situational, sometimes you catch them immediately, sometimes if you check it, sometimes you try to hide them, and sometimes we are really obvious about them. Let's get started, yes.


**Anatoly Yakovenko**: Yeah, the top validators should all be building from source and we should be able to send them the patch directly in an emergency like, so it doesn't need to be revealed in the GitHub repo.

There’s like two kinds of attacks right like one is a safety issue and those you can get out as a patch to the top 33 of the validators. So if that has been exploited the network holds. The liveness ones are harder because you need two-thirds to upgrade and that requires like basically a general release so that it's really hard for us to get two-thirds of the validators to upgrade via like a secret patch right like, so that has to go through I think the general release process.


**Alexander Meißner**: Yeah, at that point it would be now in this category of SIMDs because that is probably the route for feature activation. That's why I brought it up.


**Anatoly Yakovenko**: Yeah, if there's a liveness issue that needs a feature activation, it's just like really unfortunate.


**Micheal Vines**: I do think for the first category though there's an opportunity to really improve the ad hoc process that we have. I think talking about that would be good at some point.


**Richie Patel**: I really do want to get the client to a point where it can continue the chain without having two-thirds vote. I think you can probably optimistically confirm without finalizing that would probably relax some of the like emergency patching that we hopefully don't have to do. I also think we should definitely fast-track the SIMD process and not be held up by approvals or things like that. If there's a very obvious change I think that can be both implemented on like the two or three clients we currently have without having to put the, you know, rubber stamp on it.


**Jacob Creech**: All right, cool. So what we'll do is for the future call, coming up in the new year, we'll grab all this information from you all and kind of work with y'all to figure out what the full agenda is and we'll just go through it each time. Feel free to give either me or anybody's feedback on the call like, Hey do we need this, do you want to continue it, like how could we make it better? We're more than happy to like help facilitate that and help you all work together as developers on the Solana protocol.

All right, cool. Thank you everyone for coming today.


**Dan Albert**: Thanks, everybody. Looking forward to figuring all this out. Hope everyone has
a great New Year.


**Kevin Bowers**: Happy New Year, thank you.


**Dan Albert**: Thanks, Jacob.