# Core Community Call 10 Notes

Meeting Date/Time: Friday, November 17th, 2023 19:00 UTC

Meeting Duration: 30 minutes 26 seconds

[Video of the meeting](https://www.youtube.com/watch?v=V7P0nAZjCc0) 

## Speakers
1. [Jacob Creech](https://github.com/jacobcreech) 
2.  [Andrew](https://github.com/apfitzge)
3.  Buffalu | Jito
4.  Voice 1
5.  Voice 2
6.  Voice 3
7.  Voice 4

## Meeting Notes

### Introduction

Video| [00:00](https://youtu.be/V7P0nAZjCc0?t=1)
-|-

**Jacob Creech:** Welcome everyone to this months core community call. This month we have Andrew giving a presentation on two SIMDs that he proposed in the past two weeks, it's Relaxing Transaction and Entry Constraints.

Andrew you can go and take it away.

### Transaction and Entry Constraint Relaxation

#### Introduction

Video |  [00:25](https://youtu.be/V7P0nAZjCc0?t=25)
-|-

**Andrew:** Let me try and share my screen real quick. Sorry, I am trying to figure out how to share the, alright can you all see that? 

**Jacob Creech:** Yeah.

**Andrew:** There's two SIMDs, [82](https://github.com/solana-foundation/solana-improvement-documents/pull/82) and [83](https://github.com/solana-foundation/solana-improvement-documents/pull/83) that deal with transactions and entry constraints relaxation respectively. Mostly I am going to talk about 82 and if we've got time I think we can get 83 which is a lot simpler. 

I am going to start off by giving some background and motivation for this work and then I will get in to the details of each of the SIMDs.

I started looking at this a couple of months and realized that constraints and transactions aren't documented anywhere except the code, there is no document that says this is what a valid transaction is. That is something I have been working on, it's not completed yet, but as I was going through that, I realized that some of these aren't really necessary there's a whole host of checks that we do, to make sure that transactions can be executed and these are transactions that could pay fees and then we drop them very late in the processing pipeline. 

We could record them and take the fees from them except that, that breaks consensus right now. I started going through the process of what if we relax this constraint that transactions have to be executable, then we could potentially collect more fees if we do see these transactions, of course when I started relaxing constraints, which constraints do we actually need when I was looking at the other constraints. 


#### Block Validation and Consensus

Video |  [02:13](https://youtu.be/V7P0nAZjCc0?t=133)
-|-

As all of this is going on we've been having on going discussions about asynchronous execution and so that was noodling in the back of my mind as I was going through this exercise and some background async execution. There's two questions that we answer on the validator side of things which is does this block comply with all the constraints, if it does then it can go into the history and then the consensus side of things which is, what's the result of this block and then does the network agree with me.

Async execution is just separation these two questions but right now because of the way that the constraints are built, they're very much tied together. You can't answer this first question does the block comply with all constraints without actually executing the block. 

What's stopping us from separating these, at least logically, right now the SIMDs are not proposing we got towards async execution. It's just moving towards separating these two questions logically but not necessarily making them separated in time. T

The only thing that's stopping us is account state on the transaction level, this consists of mainly four different checks.


**Jacob Creech:** Hey Andrew, I think you are presenting the key note but I don't think it's actually going from each slide to slide just as a heads up 


**Andrew:** Okay, let me fix that 

**Jacob Creech:** I don't think it's showing the presentation mode when you're sharing just the keynote.

**Andrew:** Gotcha.

**Jacob Creech:** We could just watch this


**Andrew:** Okay, can you guys see if I switch slides here 

**Jacob Creech:** Yes

**Andrew:** I will not present, I don't know what I am doing, I guess.

#### Separating Validation from Results 
 
 Video |  [04:09](https://youtu.be/V7P0nAZjCc0?t=249)
-|-

**Andrew:** At the transaction level, this is four checks, do the ALTs resolve and count look up tables. If the transaction is a nonce transaction, is the nonce account valid, can the transaction pay the fees and then can the transaction be executed. The last ones probably like 10 different checks rolled into one, but the summary of it is, we can execute this transaction.

#### SIMD-0082 - Transaction Constraints

 Video |  [04:33](https://youtu.be/V7P0nAZjCc0?t=273)
-|-

**Andrew:** And SIMD 82 is the relaxation of three of these so the nonce fee check and the executable check. The SIMD is not a proposal that will remove all these checks or execution of these checks for execution, all of these have to be met in order for the transaction to be executed. What it is saying is that if a transaction appears in a block, it does not validate invalidates that breaks one of these constraints. 

It doesn't invalidate the entire block we can either just not execute the transaction and take no fees, or we charge the leader a penalty for including that transaction. These transactions also will count fully towards the block limits as if they were executed with the maximum number of compute units that is just there to prevent some denial of service attacks. 

I had four constraint on the last page and I have only got three here. I think the natural question is why leave the ALT resolution, that's a temporary thing, it is still making us dependent on account state for the question of is this block valid, but there are higher level constraints at the entry and block level that make removing this tricky at this time. I'd like to remove that one at some later point. 

Additionally, ALT resolution is slightly different than the rest of these three constraints. The rest of these three constraints the answer can change based on the execution of transactions within the block that you are validating, whereas the ALTs are resolved at the beginning of the slot your view of then is essentially frozen when the slot begins and transactions within the block that you're validating can't change the ALT resolution. 

That makes it so you cant validate a block and just answer the question, does this block meet all the constraints without executing the block as long as you've executed the previous blocks. I kind of already talked about these and I'm just going to skip a slide and go to the benefits and drawbacks. 


#####  Benefits and Drawbacks
 
 Video |  [06:44](https://youtu.be/V7P0nAZjCc0?t=404)
-|-

**Andrew:** Like I mentioned this lets us validate the block. Does this block meet all of the constraints and can it go into history without actually executing the block. It also simplifies the protocol in some sense, because now there are a lot of fewer constraints on what can go into a block. 

We still have to document the executable checks which is the most complicated part, but it separates that from what is a valid transaction and not. Gives a lot more freedom in Block packing as well, I'm not suggesting that the lab client do this, but it enables something like a probabilistic fee ...
where I see some accounts has 500sol and they've had 10,000 recent transactions all paying fees, maybe I just assume that they're going to pay their fees correctly and I just record their transactions without doing strict checks, whereas with the strict constraint that all transactions have to pay fees, you can't do something like that. 

In terms of drawbacks, it's consensus breaking, it needs a feature gate, it's consensus breaking, it's not a huge drawback but just something to note. With the nonce check it introduces the possibility that, ancient nonce transactions which are no longer in the status cache could be repeated in the history, if the leader isn't doing a good job. Now, we can still check that, that nonce account is valid before execution so the transaction would have no effect on state, but it introduces the possibility that we have duplicate transactions in the history.

Depending on how we decide to do punishment for non-fee paying transactions, you could have that pay non fees and then on the SIMD there have been discussions about potential denial of service attacks. I think the main thing on the SIMD that I missed and did not clarify was the fact that these transactions are still counting towards block limits, as if they were executed with all of their compute units, which prevents a lot of the attacks that have been mentioned.

The main thing I was hoping for input from you guys on is,  how should we handle this transactions that can't pay fees? Right now there have been sort of two competing perspectives on how to handle these. One is that the leader is running and they are selling you compute if they fail to collect the fees, the punishment is that they don't get that reward, but there's also the perspective that when transactions pay fees they are burning half of it and that burning implicitly compensates the rest of the network for validating these transactions. 

Which would lead to the idea that if we have these non-fee paying transactions then the leader has to make up for it in some way either by paying those fees or paying some other fee associated with a bad transaction.


### Questions

 Video |  [09:55](https://youtu.be/V7P0nAZjCc0?t=595)
-|-

I think we can pause here and open it up for questions and feedback, is anyone totally opposed to this what do you guys think about how we should handle these, non-fee paying transactions?

**Jacob Creech:** Go for it Ritchie.

**Ritchie:** I guess I'll start with, I think this is a great proposal. It puts us further into the direction of blacklist leaders. I think the concern of possible DOS by not charging fees at all a bit of a risk, I don't see that you would want to do this attack, but there is an attack where block producer could create a maximum size block that just contains transactions that always fail, they cannot pay any fees.


**Voice 1:** This would only happen if it's is a malicious leader right? because a normal leader is going to filter those kind of things that a possible mitigation would be to charge the leader the fees.

**Ritchie:** Yeah, that's what I am thinking too.

**Andrew:** I had not written that in the original proposal, but since some discussion has been ongoing there, that is definitely where I'm leaning. I haven't change the proposal because I wanted feedback on this call first.

**Voice 1:** I don't think we need to ... 
by this attack because if the block limits are enforced, the compute limits and everything else, it's pretty benign to have a bunch to have bunch of junk data like it's okay like execution should be faster I hope, ... 
you should be able to fail those quickly and replay, so you just wasted some of the bandwidth.

**Ritchie:** Well it's kind of annoying because it makes the ...
layer of Solana, so to say completely free anyone can use turbine to distribute cluster wide junk data and nobody pays.

**Voice 1:** You have to have stake to do it, you have to be staked right?

**Ritchie:** Yeah, that's a good point and you don't get any rewards if you don't have Identity key balance to vote, so you would lose money if you have zero balance and you know we can't withdraw the leader as a punishment if it has zero balance, but then the leader would lose out on these staking rewards there by punishing it if we do charge a fee from the leader for these transactions, I mean it's that seems like a very simple change to do right?

**Voice 1:** Yeah, there's a couple dependencies on that for the leader to be in the leader schedule their vote account would need to have a certain balance that would be like you would add the constraints there. 

**Ritchie:**  You wouldn't charge the vote account, you would charge the identity account, because this account also gets rewarded with transaction fees. 

**Andrew:** I did a quick calculation this morning of basically if the minimum number of slot you can get allocated in the leader schedule is four. If they packed the smallest transactions, they could with assuming one signature, it'd be just under 1sol as a minimum balance. I think there's already some ongoing changes to make one 1sol as the minimum balance. I don't think we need to make any changes there and if they were running an attack like this it's sort of self-solving problem, because they get removed from the next leader schedule

**Ritchie:** Why would they get removed? 

**Andrew:** If we charge, if we take away their stake for punishment on these transactions, we charge them the fees.

**Ritchie:** Oh, okay but it isn't a mechanism that does this punishment a bit more complicated than just charging the fees or I might be misunderstanding. 

**Andrew:** What I'm saying is that the punishment is being charged those fees

**Ritchie:** Oh yeah, awesome.

**Andrew:** Okay, I think we're saying the same thing.

**Ritchie:** Cool.

**Andrew:** Another question, i guess if we're leaning toward charging them fees, normally fees are paid 50% to them, what do you think that that's reasonable that they're just essentially paying half the fees or should they be basically burning 100% of the fees as punishment sounds like a bit of an implementation detail. 

**Ritchie:** Sounds like a bit of an implementation detail. 

**Voice 1:** Yeah, I think that's like we can figure that out later.

**Andrew:** Okay sounds good but sounds like everybody is leaning toward charging the leader fees for, if these transactions don't or sorry either validate invalidate the nonce check or the fee paying check.

**Voice 1:** It might be easier to charge the vote account, whatever implementation constraints those would need to be kind of pushed to when the leader schedule is generated.

**Andrew:** Gotcha, yeah.

**Ritchie:** Sorry if we want to charge the leader all of those fees and they get drained, let's say they get drained at the beginning of the epoch and they still have a lot of leader slots.

**Voice 1:** That's we will need to have basically a balance check for their constraint is to be in the leader schedule.

**Ritchie:** And that would be multiplied by the number of  leader slots they have.

**Voice 1:** Yeah.

**Ritchie:** Okay. 

**Andrew:** That doesn't strictly prevent them from being drained, because there's really, to my knowledge, there's no cap, no responsible cap on what fees could be charged. They could insert a bunch of max priority transactions, which I don't think we want to enforce that they can have that many max priority transaction in a block. 

**Voice 2:** What does it matter, what priority they're? 

**Andrew:** It doesn't for execution, I'm just saying like if they inserted max priority transaction that can't pay fees, they have to have that, that make introduces the possibility even if we have like a minimum balance they could still be drained 

**Voice 2:** You just check before you start executing the block if they have sufficient like greater than whatever threshold balance and then you just skip their block, if they do, they drain it then they're drained for one block. 

**Voice 1:** You can't, it's really hard to adjust the leader schedule outside of the epoch boundary.

**Voice 2:** Yeah, yeah, you would just skip then you would adjust ...

**Voice 1:** You don't have a guarantee that everyone that's observing the network is in the same state, where they decide that that, that leader should be skipped or not.

**Voice 2:** I guess you'd have to make it part of the fork choice execution, when you start executing the block. Do you think that they have fees, I don't know maybe it's more complicated. 

**Andrew:** I think adding that check would sort of mean that we are adding another constraint that sort of prevents us from async execution, because you need to check the fee, the leader balance before the block. 

**Voice 2:** Yeah, good point

**Andrew:** Dependent on previous blocks. 

**Voice 1:** The draining doesn't need to cover the fees that you would expect as a network, it just needs to cover the cost which in this case is just bandwidth. It's just pointless to do it as a spam attack.

**Voice 3:** Yeah, that's a very good point, would charge them proportional to the size and charge them the actual fee that this transaction specifies. 

**Andrew:** Gotcha, you're basically saying don't charge them the fees but charge them some separate penalty that's based on the size of the transaction. Okay, that seems reasonable some sort of implementation detail.

**Voice 3:**  Solana fees making a come back.

**Ritchie:** Sorry, I want to come back to this, because to me I think what we should optimize for is still getting people's transactions like liveness number one. If someone can't pay for these extra transactions but there are still valid transactions in that block. We should probably prioritize getting those executed right?

**Voice 2:** I don't follow, what do you mean?

**Voice 3:** If someone creates a block, some of the transaction are going to get rejected and they don't and they don't have enough, to potentially pay for whatever ... 
fees you want to.

**Voice 2:** Yeah,The whole block should get executed there's just a cost to the leader to include junk an to make sure that the leader can pay the cost they need like a minimum sold balance at the start of the epoch and their vote account or whatever account that is hard to withdraw for during the epoch. That make sense?

**Voice 2:** Yeah, okay as long as we're optimizing that we can still get transactions in that, that to me seems more important. 

**Voice 1:** Well it's like it's going to be like somebody has, some valid transactions, but then is also doing a denial of service in the network, unless there's a but in the implementation. 

**Andrew:** I think the main intention this is that you've got someone who is intentionally putting these transactions in a block rather than or the bug like mentioned. 

**Buffalu | Jito:** Just the SIMD include relaxing the entry constraints or is that separate? 

**Andrew:** That is what SIMD 83 is, we can move to that. Nobody else has any more comments? Alright.

### SIMD 83 - Entry Constraints 

Video |  [20:53](https://youtu.be/V7P0nAZjCc0?t=1253)
-|-

**Andrew:** This one's a lot shorter and a lot easier. Right now entries cannot contain conflicting transactions, that's not really too necessary. I think this is lived on as a legacy of the old implementation, it makes it easier to parallelize in the current labs code, but it's not strictly necessary and this check also depends on us resolving the ALTs, because we can't tell which transactions conflict with each other plus we look at the account lookup tables.

SIMD 83 just removes this constraint entirely and says you can put whatever transactions you want in the entry and this allows for more freedom in the block production, it also allows for more efficient networking. Right now if I wanted to record five or 10 transaction that conflict with each other I have to put them in separate entries and those get sent out over turbine individually. Whereas now if they are just in a single entry, they can be shredded together and sent out in one packet and then it gets us one step closer to async execution because now the entry constraints no longer have dependence on account state. 

In terms of drawbacks pretty minimal on this one it's basically just that we need a feature gate and I was talking Steve yesterday and I am not event sure if this is possible. Right now, the way that the labs client does replay is, we receive the entire entry and then we will process it, it might be technically possible where you receive a partial entry and begin executing. 

I'm not 100% sure it, but if that is possible, this SIMD would make that impossible, because unless you receive the partial entries like in the order, you don't know that previous transactions in the entry don't conflict with them. You sort of have to receive at least in order, but again I'm not convinced that that's even possible with like the way we did serialize these shreds into entries.

**Ritchie:** It's a cool idea, I think it's possible. 

**Voice 4:** Transactions are in fixed size right, you have to scan to find transaction boundary.

**Andrew:** Yeah, that was the part that was the part I wasn't sure about, is like identifying if I receive like the 9th shred of 10, I don't know how to identify where a transaction within that arbitrary slice begins and ends. 

**Voice 1:** Yeah, benign transactions to shreds is something that we talked about, but it doesn't really depend on removing this constraint but I think that would be a totally separate thing. The benefit there is that you could do signature verification of the shred and the transaction totally in a separate pipeline like as you are receiving shreds, but that would be complex change.

**Andrew:** Right.

**Voice 5:** I'd love to see this constraints go, because it kind of keeps the scheduler hand tight behind its back but there needs to be some upper boundary, for example if there are two types of clients I like Firedancer and Labs or whatever and one of them consistently has a lower limit on the complexity of a graph that it can execute, it could somewhat split the network. It's super easy to measure that, because if you have a sequential dependency chain so you know transactions that all right to the same account and they all only load in the same small set of accounts that is can completely fin in in each it's pretty much ...
but because it's only as fast as the VM can execute. 

If you have a lot of transaction writing to the same account, but all of them also reading a wide set of account, then it becomes data bound because you then have to wait for all of these accounts to arrive every single ...
Liam who can unfortunately not join the meeting for Zoom buggy reasons has also done a lot of research into this scheduling, but I think we might need to come up with some mechanism to have some heuristics on how expensive a block is going to be given some arbitrary graph. 


**Voice 6:** Hey can you guys hear me? Hey sorry excuse the background I'm in an airport right now. I was not able to join for a minute but I was able to figure it out. Basically we are working on something to actually help the network agree on what the actual cost of running a block is in real time, that's outside of the current CU model. 

We are still working on this and I'm still like passing the Idea around and workshopping it, but the basic idea is that it actually takes the network some amount of time for the network to process the whole, it's not just runtime, it's also like network distribution time and all these other things and there could be ... 
reasons why a block takes a lot of time to send around and execute.

We are working on a proposal to address the issue of like hey "this block takes a lot longer everybody is going to have to wait around and do this ..."
So leader should probably have to pay for the little bit. I think what you guys are talking about is SIMD 82 and 83 right? 

Yes, I think I'm pretty happy with these proposals, I think my one big thin is that I'd love to just define what the new constraints is now that we're going to do this thing. What are the now, the constraints on transactions and I know that's a bit of an exploratory thing to have to do. but I'm happy to help figure out what and spec out what those actually. Those new constraints now are on like on entries and transactions. 

**Andrew:** Yeah, I will work next week on rewriting some sections of the SIMD to be very explicit about what the new constraints are, for both the inclusion in a block and then for execution of a transaction . I'm planning to do that, I just haven't done it quite yet. 

**Voice 5:** It's okay, it's going to take a while to figure this all out.27:54 

**Andrew:** Yeah, in terms of the cost modeling, I might have misunderstood what Richard was saying, but I don't think that something new with removing the constraint on entries, feasibly I could just put those in separate entries and still create a block that's very expensive to execute, even if it complies with CU limits. 

**Voice 5:** Yeah, maybe. I don't know the constraints on the entry counts for example too well.

**Voice 6:** Well, another thing to think about and I brought this up with the call with Andrew and a few others which is that, you have to think about an entry or I may use the term micro-block because that's what we call them at Firedancer, but we may call an entry a, you'll see an entry come over the network and the entry is one packet coming over the network and sometimes they'll come very busty or you'll just see one of these and you want to process like it's very easy and efficient to process individual packet and packets of transactions and having this constraint that you can't have multiple rights, there of slows down fundamentally because sometimes it does take, like we are doing packet level processing at this point. So we have to be really careful about how many dependence ... 

**Jacob Creech:** Just being mindful of time because we are at time. Real quick if anybody wants to continue discussing, you're welcome to stay, keep discussing. For anybody that wants to propose an agenda item for next month, you can propose on the Core Community Call Repository [here]](https://github.com/solana-foundation/core-community-call). If anybody has any other questions go and ask otherwise we can continue the discussion in the SIMD and on Discord.

Okay thank you Andrew for presenting and thank you everyone for coming for the discussion thanks for giving me the time, we'll see you al next month.