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

So, `Max` from [Mango](https://youtu.be/AEnkivbha0k?t=1) will be presenting on the [SIMD-0047](https://solana.com/ecosystem/mango), on the new sysvar for the last voted on the slot. And then, he will also want to talk about how the different tuning parameters were chosen for [QUIC](https://blog.changehero.io/what-is-quic-in-solana/). Go ahead Max.

**Maximilian Schneider:** Sorry for being late.

**Jacob Creech:** So the floor is yours. `Max`, you can go ahead and get started with whichever one you want to get started with. Either the QUIC tuning parameters or your SIMD that you have been working on.

**Maximilian Schneider:** Yeah, so for the quick tuning parameters, that was like my request which was that we have someone who worked on it lead the discussion. I don't know if someone is here that worked on it. And I can only give the outside perspective of reverse engineering some of those things. But you know, reverse engineering requirements is very difficult sometimes and that was the main request that we get someone who knows all the requirements, maybe to present them. There is no one here, then maybe we put it for next time.

**Jacob Creech:** `Stephen Akridge`, could you talk a little bit how you all choose tuning for QUIC.

**Stephen Akridge:** Yeah, I can talk a little bit. Yeah, we had a somewhat specific performance idea in mind right. We want it to be in that like 50 to 100K ish TPS range in the frontend or like at most try to you know split the streams that way. And allow right enough streams for each client to be able to fit within that. So, you know it was mainly from like benchmarking in the test environment and seeing kind of what bandwidth we could obtain unlike a somewhat optimal connection right which would be like the worst case in terms of TPS in like the real validator and trying to calibrate that to what we thought like the backend behind Quic could sustain or handle and then also you know like keeping the memory Women Within something that was you know feasible for like a normal validator.
