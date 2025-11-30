# **[GENDA 7 [Core Community Call - June 16, 2023 - Light Clients](https://www.youtube.com/watch?v=9m_M8zEw1cE&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=7)]{.underline}**

-   Time: June 16, 2023

-   Video:
    > [[https://www.youtube.com/watch?v=9m_M8zEw1cE&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=7]{.underline}](https://www.youtube.com/watch?v=9m_M8zEw1cE&list=PLilwLeBwGuK7e_mH_sFwTytYQxalh7xd5&index=7)

-   **Speakers profiles**

```{=html}
<!-- -->
```
-   [[https://www.tinydancer.io/]{.underline}](https://www.tinydancer.io/)
    > (Tiny Dance)

```{=html}
<!-- -->
```
-   **Key concepts mentioned:**

```{=html}
<!-- -->
```
-   SIMD-0052: Add Transaction Proof and Block Merkle for Light
    > Clients -
    > [[https://github.com/solana-foundation/...]{.underline}](https://github.com/solana-foundation/%E2%80%A6)

Welcome, everyone, to this month\'s Core Community call. Today, we have
for discussion light clients, and we have the Tiny Dancer team, Anoushk
and Hirsch, talking about different ways that they have been thinking
about implementing it. The agenda can be found, as usual, on the Core
Community call repository, and the specific Cindy is 52, which I have
just added to the chat. Anoushk, you\'re welcome to take it away.

Thanks for the intro, Jacob. Let me just share my screen. Thanks,
everyone, for joining in and giving your time. I\'m Anoushk from the
Tiny Dance team, and we\'ve been working on implementing the flight plan
for Solana. It\'s draft 7052 for adding transaction proof verification.
So today, we\'re going to give a brief overview of our research.

So, I\'d like to start with why we\'re doing this, the motivation behind
assembly is primarily to implement light plans for Solana. The reason we
need light plans is that users need to verify queries that they make to
the RPC, and right now, they cannot do that, which is why they have to
trust the RPC that they give them the correct, you know, these like
lines need to be low Hardware pieces of software, and they need to be
able to run on a phone or browser. So, this is really important for
security for the blockchain Network, as we already know, more mature
networks like Ethereum that have been there for longer already have
tight lines, and this was a glaring problem in Solana. So, the crux of
this MD is adding something called transaction proof. So, on a high
level, it\'s just a multi-proof saying that from your transaction that
you sent, was included in the block and it succeeded or failed, however,
it should have. It did like awe the change that we\'re making here
requires adding requires some way of making sure that a particular
signature and a status, whether it\'s success or failure, was included
in that transaction in the block. The user needs to be able to verify
the inclusion of the transaction and the execution status. We would also
add an RPC method that would allow anyone to just call that method and
get its proof for the particular transaction. So, as I mentioned, for
the reason that we need this is that RPCs could give you incorrect
information that the transaction actually succeeded or it was included
in the blog, but it actually did and that\'s an attacker. You can still
verify this data if you have a snapshot, but obviously snapshots are,
you know, 30, 40 gigabits.

And that\'s a lot of data for the empty user. Also, relating to a
different simile about stakewade attestations, you can actually use
transaction proofs to verify that a particular value that has a certain
stake off-chain, as mentioned in the SPV proposal, a validator could
technically use reduced transaction proofs to use them as a checkpoint
to verify a certain state and as part of a different proposal that uses
SPV, it can also be used for interchange verification. Here\'s a list of
important resources that you could take a look at. This is the CMD that
we wrote, DSP proposal. There\'s an open issue that talks about adding
statuses to the bank cache, and that\'s the interchange and SBP
proposal. So, this is the part that we really there\'s been a lot of
discussion on and we think that there needs to be more con-like more
discussion on from different parts of the code of the community on
different ways to implement it in our CMD right now.

We are really implementing the first way, which is modifying the block
has to include statuses, but over the last week after discussing with
some members of the jp team and other members of Surround Labs, we also
found different ways to implement it to tackle different issues that may
arise while implementing the first method. So, I\'m gonna dive into each
of these very beautifully. So, modifying the block hash is pretty
straightforward and again part of the history proposal. So, all it does
is you add the transaction status along with the transaction signature
into the Merkle tree of the transactions, which is part of each entry,
and that gets hashed into that gets mobilized into the block hash and
becomes part of the bank cache. Currently, the bank cache is a
sequential hash of all the entries, but making it a virtuality would be
better in terms of verification size, and obviously having the status is
important for verifying if the transaction actually succeeded. Here\'s a
quick overview of the pros and cons. So, the pros being that on the
client side, the verification is less computationally heavy, so that\'s
good for low Hardware devices, and it\'s part of the core consensus
protocol, which means that all validators have to write it, you know,
overall or with. The downside of this being that it\'s quite a major
string general require-like feature flag activation, which means that
the round trip from implementing this to actually going live on Mainnet
would be quite long, and also there is a computational overhead of
memorializing entries that was initially taken into account.

Moving on to the second method, which is adding a separate transaction
tree, which would basically be a tree of all the receipts of each
transaction. This would be part of the bank hash, so basically just a
tree or a monkey tree of all these interaction signatures and statuses,
and would just be hashed into the bank cache. The pros of this are
mainly that it doesn\'t come the way of planting seeders, so a leader
could basically just create the entries, create the block, propagate the
block, and then asynchronously update this data bank cache with the
statuses, so they don\'t need to have it before propagating the block.
Again, this is also a major change, so comes with the challenges of the
first way to implement it, and so this is part of the second method,
which is we were told that implementing changing the bank hash would
somehow come in the way of backless leaders. So if we implement this,
which basically have the state of block n in block n plus 10, you
actually don\'t need to execute the transactions of log n to get the
statuses and then know you have to compute the bank cache, so you could
just do that in an async way. But this also would be a pretty major
change protocol, and that is also something that needs to be heavily
considered if you\'re deciding to go over this. The last one is probably
the most flexible and easy to work with because we are not really
changing any core part of the validator like the consensus. So we are
just using gossip or could be a different network.

We are even to asynchronously push commercial commitment of all the
transaction receipts, and this could be done, let\'s say every 10 blocks
or so, so you know, it\'s not even doing it every block. And you could
then have validators that pull this change, website against their state
and then push another station saying that hey this is this checks out
with the data that I have now or likely I could just you know pull that
data pull those attestations and just verify if X percent of the stake
has actually confirmed that this is sticker attached and that their
transaction was included in that. The pros of this are that there\'s no
overhead to block production because it\'s done asynchronously and is
not part of the bank cache. There\'s a low risk of liveness failures
because it\'s not part of the core consensus protocol, and it\'s also
easier to implement.

\"Because it\'s once cause it and not consensus, the only downside would
be that it\'s not making the code protocol, so it would be optional to
implement, and the validators aren\'t really forced to make those
attestations or commitments. So this is where we end this presentation
and want to hear more about the thoughts of the code community and open
to any critics or questions. Thank you, Richie.

Hey, so not to say thanks a lot for working on this. I think this is a
really nice initiative, and it seems like a rather easy win complexity
and wise for enabling this functionality. I would just like to voice my
preference for going the gossip route or any related method of doing it
this way. And while we don\'t modify the core data structures to support
this feature, I think my main problem with modifying the proof of
History hash would be that it\'s quite a breaking change of the
definition. What the proof of History has currently is where it would
basically go from committing to the block data contents of the current
block and all blocks before that to going to committing to the state
changes that this block induces. And this would matter for fire dancers,
for example, because we might use the POA hash to identify the chain
that fire dancer and Salonlabs are currently on. And let\'s say, for
example, we have a temporary mismatch in the runtime where we derive a
slightly different state on both clients. By redefining the proof of
History hash to potentially differ on both clients, I think it would be
much harder to tolerate such runtime mismatches or even detect them, as
that would basically stop validators from synchronizing entirely rather
than continuing replay with a slightly different state.

I think going the like Fast delay route doesn\'t seem too elegant to me
because that would basically introduce minimal latency, finally like the
client, where they would need to equip these, you know, n block speed 10
or so. I think regardless of which way we choose, maybe I thought of
splitting up the proposal into a few separate parts, and then it would
be much easier to vote on each one specifically because it feels like if
we try to incorporate this entire feature into one, there\'s going to be
a bit of discussion on it for quite a while.

So the first one that I thought of would be just agreeing on how we
actually compute the commitment for the transaction statuses only. So,
you know, given a vector of transaction statuses, what goes into the
hash? Do we just commit the transaction result code, or do we also hash
logs in some way? And then, you know, define what the actual root of the
transaction statuses is. And then we can do a separate sim that says
here\'s how you would propagate it over gossip. And then we might go
back and say, well, in hindsight, gossip was a bad idea. So here\'s
another sim deal of how we would propagate the transaction statuses over
whatever other protocol. And one other way would be of cross-posting it
on-chain itself, which, of course, has the benefit that this kind of
incentivizes validators to actually participate in this transaction
status calculation. Whereas, if I wanted to be mean, I could say, hey, I
don\'t want to implement this feature and fire that, so I\'m not just
gonna propagate it over gossip, which would, I think, decrease the
quality of service. I think we kind of want to incentivize validators to
participate in this feature. Again, thank you so much. I\'d love to hear
your thoughts and on these, and I\'m also gonna post the same feedback
on assembly itself.

Likewise, I also respect your interest in this, indeed. I think that
gossip is definitely a favorable route because it allows us to test a
lot of the client UX of the electronic, and because it\'s actually
simpler and less of a breaking change, it\'s easier to revert back if we
messed up. If we think that\'s not the right way, then doing that from,
you know, making a change to the bank hash and then like, you know,
thinking that okay, this is not the right way and going back to go
outside.

I think that regarding like validators being incentivized to participate
in the network, that\'s something that, again, eventually be figured out
and might even be another Sunday. And I think we can, I\'m leading more
towards what Richard said, we\'re going with gossip. So I totally agree
regarding the incentives. I\'d be also curious whether Solana Labs has
any thoughts on this proposal, but would be really cool to see progress
moving on this rather soon.

I just want to say, first of all, this is a great effort. It\'s gonna
have a lot, I think, to the community. One of the issues I saw regarding
the transaction status is we\'ve generally been moving away from
detailed transaction statuses more into very broad categories of
failure. And the reason for that is, for example, originally we even had
the runtime errors as part of the consensus, luckily that\'s not the
case anymore. So there\'s now only one case for the entire runtime in
the consensus. And the reason why we moved away from that is that the
success cases are already complex, but the failure modes are so much
more complex, and it really narrows down the implementation you can have
to get the exact precise error response of every transaction rise. And
that makes it almost impossible to change anything or to re-implement it
any other way. So just be careful about the transaction results, don\'t
include too much error state or logging into them, otherwise, we will
all be stuck with exactly one implementation. So we really just want to
include like success or failure. There was a suggestion to include like
transaction logs, but we can actually avoid that by just re-executing
the transaction on the client side by fetching the inputs. So we just
want successful failure. I heard some feedback suggesting that we also
add logs. I think logs are pretty scary because right now the truncation
of logs is not well-defined. But maybe that\'s actually an opportunity
to say that there\'s a recommendation to, for example, only do 256-byte
long log lines. The concern I had there was how this would affect
performance if we hash two or more sharp blocks for each program
execution or so. Would that limit the TPS in the future? I don\'t
remember who it was, I think it was Mango.

\"I don\'t know. I think the only concern was like with the overhead
added by logs, but I think you mentioned that with fire dancers like
implementation of, I think that might not be a problem. And once you
have your Henry\'s go-ahead and start, no, I just wanted to clarify,
Richard, when you actually, for both Richard and Anushk, when you say
the gossip route, you mean that there are going to be no consensus
changes, right? Like, even the validators are voting on regarding the
state or whatever state is necessary would be in a separate smart
contract. It wouldn\'t be part of the block. Yes, correct. It wouldn\'t
even be in a smart contract. It\'s basically just a structure, okay? So
the contact info is a structure that every validator publishes regularly
onto gossip. As far as I know, there\'s already a snapshot hash and then
the accounts hash, so it seems like it would be fairly trivial to fit in
another hash there. So that\'s actually a bit more discussion around
whether we should have an entirely separate hash for this, because
usually if you\'re getting the transaction status commitments, you
probably also want the account sashes. And the problem with doing
separate fields for this is that you\'d basically have all validators
accessing them separately. I think if we fit them all into the same
contact info block, that might not be a problem. They also don\'t need
to be at the same frequency, right? How frequently is this contact info
updated or how frequently do validators publish it? The question for
Solana Labs, I think that\'s an implementation detail anyway, but I just
wanted to be clear that in the gossip route, there are no consensus
changes, right? So even the transaction status or anything, the block
structure doesn\'t really change. So Tiny Dancer won\'t be blocked in
any way on that, right? Yes, okay, just wanted to clarify that. Are
there any other questions for Anushk and Hirsch? Carol, I think you
mentioned that the Mango was\... They suggested using a bi-directional
connection. Max, I think you\'re here. Do you want to speak a little bit
more on that?\"

\"I\'m sorry, could you repeat on which one should I speak? Corral
mentioned that y\'all suggested you use a bi-directional connection. I
believe this was originally like an experimental feature that y\'all are
implementing on the labs client, and it\'s like a client-specific.
Y\'all want to talk about whether it\'s relevant anymore.\"

\"I can give some background for the people who are interested. So what
we did is when we run benchmarks on a local network, we\'ll enable a
patch on the TPU side to give us back, basically, a status that
summarizes what happened with the transaction in the scheduler. This is
very similar to, I would say, like request tracing environment that you
would see in a commercial microservice architecture deployed in a
private company. You send something in from a load balancer, right? You
want to trace for a limited amount of the requests in the network, why
aren\'t they getting scheduled right? So you get a per-request
measurement that is actually complete because right now, the
measurements we have are statistical and broad, and it\'s very hard to
say in particular which transaction, like we just see all five out of
100 transactions had this issue, right? But we don\'t know what\'s wrong
with these five transactions, why didn\'t they get into the block?
Right, what were the five transactions that hit the CU limit or what
were the five transactions that hit the 2000 packets per 10 per 100
milliseconds on a quick connection, right? There are different limits in
a stack, and it\'s very hard to identify why a certain transaction
didn\'t pass. That was the original intention there. But I think
there\'s a lot of pushback against this kind of, I would say,
nice-to-have feature. We really enjoyed having it for local performance
testing because we get more insights, but that\'s it. I\'ve looked at
this a bit, and it wouldn\'t seem that hard to support and finance or
Solana labs, and I think it\'d also be pretty nice to have clients to
get explicit feedback about why the transaction might fail or not. And
it seems like, in financial kind of applications, that would seem like a
basic feature to have, especially if they\'re user-facing. Like if I go
in my wallet and send a transaction and that gets dropped somewhere
along the path, since this is an issue that would seem to directly
affect the user experience of our clients, I think it would make sense
to report that. But it requires a bit of plumbing to get that data back
to the networking layer because usually, at the point where you know
where your transaction gets dropped, you probably already anonymized the
traffic flows, where you don\'t have the IP addresses or quick
connections anymore.\"

\"So I feel like this should be\... I don\'t see why we wouldn\'t move
to bi-directional connections, just have the ability to do this kind of
reverse flow feedback in the future. And then we could just maybe
publish this in the specifying what the protocol should be for reverse
flow feedback, and then, you know, I think as time goes on, we\'d see
more Solana Labs clients adopt this and finance the clients.\"

\"What was the specific feedback why this wouldn\'t be a good idea?\"

\"I think there\'s a couple of performance questions there, right? So I
think the main issue is that it kind of creates risk on the security,
DDoS protection side. So, oh, I think I know where this is going. So
currently, every transaction is a separate unidirectional streaming.
After you send it, it gets closed. So if there\'s reversal feedback,
maybe that can force the client to keep open these streams for longer by
just saying, \'Hey, I dropped these packets. Please send them to me
again.\' But that should be easy to fix by, for example, delivering this
feedback with datagrams or with, you know, a persistent stream. I still
don\'t really see why we shouldn\'t do it if the state is easily
available. I see Galactus from Mango Prices.\"

\"So it\'s actually what we implemented is more like\... You have this,
like, with Solana client, we cannot really implement this bi-directional
stuff because once you get the\... you need that like connection, it\'s
dropped immediately after reading. So what we implemented is a separate
service where a validator can connect, and then it can say, like, a list
of transactions that it wants feedback for. And so actually, whatever is
executed for X reason is dropped, then we just send the, like, why it
was dropped actually. So it\'s more like a separate service. It\'s not
even in the same\... like, TPU client TPU server.\"

\"I see. I mean, it would seem a bit cleaner to just deliver
acknowledgments\... So I also would, like, I would also like to have,
like, a bi-directional channel where you just send a transaction and get
feedback like why it was dropped. But, like, on the\... this validator
side, there are, like, a lot of limits, and the connection can be
dropped for, like, a lot of reasons. And it was quite impossible just to
keep the connection up. So we said, like, okay, forget it. We\'ll just
have a new separate service. But,of course, I agree. Like, we can have
this per-directional connection of Quick, and then we\'ll just get
feedback for our transactions. I think if you install a mechanism to
signal optional features in the Quick handshake itself, it should be
pretty easy to trial out features like this on a real cluster without
affecting reliability too much or without introducing breaking changes.
But it seems like it needs a bit more time on finding out what the
actual right protocol is for delivering this feedback. I agree because
Quick right now, like, we have\... We don\'t have, like, a lot of bits
remaining just to have, like, give accurate acknowledgment. I think
while closing or even in the acknowledgment packets, maybe we have to
find maybe another mechanism to get this acknowledgment. I agree with
you.\"

\"We\'re a few minutes over time, so I think we\'ll have to do as well
to continue the discussion, probably open up the PR that\'s on this as
well as the discussion on a news the 7d52 earlier. But thank you all for
coming this month, and I\'ll see you all next month for the next Cindy
call. Reminder, if you have any agenda items, make sure you do a PR to
the next agenda so that we get it earlier rather than later, and people
can read any additional information beforehand so we have a better
discussion.\"
