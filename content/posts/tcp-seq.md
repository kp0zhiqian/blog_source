---
title: "From Simple Reliable Transport Protocol to TCP (1)"
date: 2021-05-14T00:11:23+08:00
draft: false
tags:
    - tcp
keywords:
    - tcp seq
---


In the first article of this series of articles, we'll discuss the concepts we need to implement to design a very basic reliable transport protocol.  How to implement these concepts will be covered in future articles.

If you found anything incorrect, please don't hesitate to reach me.

## Checksum
Imagine you're designing a reliable transport protocol. You've already implemented the basic send and receive features, including segmenting the whole data when it's too large. However, because of the unstable underlying media, maybe a broken cable or an unstable ISP network. Sometimes the data you sent is broken. But the receiver side didn't know it and continue to read the data, which will cause the information transmitted incorrectly.

Yes, we need something to ensure the information the same between the sender and receiver. The first thought in my mind is to hash the segment and put the hash result into the segment header. So that when the receiver checks the hash is different after receiving, the receiver will know the segment is broken.

That is the basic concept of **checksum**.

## Sequence number, ACK and NACK

We've solved the broken segment problem, but another problem raises!
When we send the packets through an underlying ISP network, different packets may travel through different paths, which may cause the packets you send cannot be received in the same order as you send them. In the current design, this will cause the receiver side impossible to understand the whole data. 

You're pissed off by the unstable ISP network, but you cannot do anything about it unless you own an ISP company. Then you start thinking about how do we achieve reliability on top of an untrustable underlayer. Adding a sequence number for all of the segments will be a solution for the receiver side to identify the packets and rebuild the whole data based on the sequence number. No matter if the segments arrived in order.

That is the basic idea of **sequence number**.

It looks fine to use sequence numbers. However, you're pissed off again by the unstable ISP network, because some of the segments just disappear on the network and can never be received by the receiver side. Again, the receiver side cannot understand the data, even if it knows the order. 

You want the sender re-send the segments when the segments are lost. But how does the sender know which segment should be resent? We need a mechanism to inform the sender, which is the concept of **ack number** and **nack number**.

When the receiver successfully receives a segment, it will send back an ack message to the sender. If the receiver cannot understand the segment, it will send a nack message. After the sender receives a nack message it will re-send the segment whose sequence number has already been contained in the nack message. After receiving ack/nack, we will know what's the next segment we should send/re-send.

## Timer, Timeout

It looks like we solve the packet loss problem by using the ack/nack mechanism, but please remember, the ack/nack messages are still running on the same unstable underlayer! So we also need another feature to ensure reliability when the ack/nack messages are also lost...

The easiest way to solve this problem is to set up some **timers** to wait for the ack/nack message after sending a segment. If the timer has a timeout, it will identify the segment as failed and resend the segment with the same sequence number as before.

Similarly, if the receiver didn't get a certain sequence number for a long time(it depends on the timer on the receiver side), the receiver will also send back a nack.

This is the basic **Timeout** concept in the reliable protocol.

OK...

Now, it seems we already have a system that can make sure the packets are being received successfully no matter how unreliable the transport layer is.

## Is that all?

However, if you think through the whole process, you will find that we must either wait for the ack/nack or the timer to timeout, which is highly ineffective when we use this kind of process in a production environment. This is also a so-called stop-wait protocol.

Here comes the most tricky part of a reliable protocol.

## Sliding Window

Instead of sending one segment every time, we could send many segments to expand the efficiency. The transportation channel of each segment is just like a band connected between the sender and receiver. By sending more segments at the same time, we expend the width of the band so that the band could deliver more stuff. That's why we call it bandwidth(you get it?).

But if we only send many segments at the same time, it's still a stop-wait protocol. The only difference is waiting for one ack or waiting many acks/nacks.

At the very beginning of our design, we added sequence numbers to every segment to inform the order of the segment to the receiver. Imagine, if we write down all the sequence numbers of a whole bunch of data. It will be like:

```
1 2 3 4 5 6 7 8 9 10 11
```

What if we use some kind of window to contain the sequence number and slowly slide the window?
It will be like:

```
(Time n)
|  Window |
| 1 2 3 4 | 5 6 7 8 9 10 11

(Time n+1)
    |  Window |
1 2 | 3 4 5 6 | 7 8 9 10 11
```
Now let's see what happens if we implement this sliding window to our reliable protocol design. We will still send many segments at the beginning at the same time, but instead of waiting until receiving all of the acks of the segments, we only need to wait for the very first one. After receiving the ack of the first segment(like 1 in the above diagram), we slide the window forward and send the segment that is newly added to the window (like 5, 6 in the above diagram). 

When the ack of the larger sequence number(like 4) segment arrives before the smaller sequence number ack(like 3), the system will store the segment that has a larger sequence number. After receiving all ack of previous segments(3), the system will then deliver all of them to the upper layer and slide the window(2 steps in the example).

In this case, we could keep sliding the window until all the segments sent out and ack received.

This is the basic concept of **sliding window**.

## Congestion control
Now we grow the efficiency of our reliable protocol by adding something called a sliding window. It's great, but we have involved a new problem: *window size*. Imagine a situation: the network is so limited that it can only forward two segments at the same time. Our default window size is 4, so we send 4 segments at the same time. 2 of the 4 segments will be dropped and resend after the timeout, which will run the network into a congestion status.

So we must not use a fixed window size as we will never know the underlayer network status. We must use a dynamic window size to adapt according to the different network situation. 

About dynamically setting the window size, we surely need an algorithm. However, the algorithm is another big topic that we won't cover here.

And this is how **congestion control** comes from.

## Conclusion
Let's review what we've done from the very beginning, we have added the fundamental elements that could make sure a basic reliable transport protocol. 
- Checksum for validating the segments
- Sequence number and ack/nack to deal with disordering and acknowledgment/non-acknowledgment of the segments
- Timer and timeout for dealing with the case we lose the ack/nack information packets
- Sliding window to improve the efficiency and "bandwidth"
- Congestion control to dynamically set up sliding window and avoid congestion.

Please keep in mind that this is more like a learning note to understand the TCP feature. In my next article, I'll start talking about how TCP implement these feature.

See you.