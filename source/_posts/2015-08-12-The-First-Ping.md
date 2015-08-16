title: The First Ping
date: 2015-08-12 10:59:42
tags:
- socket
- ping
---

Few days ago I decided to write a basic PING program to practice my C programing skills and network stuff. And it turns out to be a 2 days work, I really have tasted the difficulities of C programing after I spent hours on location bugs of operator priority and wrongly regarding pointer as struct. Actually by retrospection, apart from these stupid mistakes, this work is not very hard. It's just that so many new concepts need to be learnt first.

The PING program, as [wiki][1] says,is used to measure the round trip time(RTT) of remote and local machines for network diagnosis purpose. This [page][2] vividly describes what it does and how it came out.

Basically PING runs on ICMP protocol which belongs to IP protocol family. And it utilizes the ECHO REQUEST, which by definition will trigger the remote to send back a ECHO REPLY packet that carries the exactly same payload sent, packet to measure the RTT. Usually it's achieved by using the current time as the payload and perform a substraction when we got the reply.

So to implement it, basic understandings of ICMP and Socket are certainly necessary. And most time I spent is to understand them. As I had not gotten a book around, I found a few example sources and studied by researching them. Though I can guess some lines of code literally, it indeed costed me lots of time to study the structs and constants used. If not counting these things, things were fine, I just need to create a right address and find the right protocol then assign them to a raw socket, and then set the proper header like checksum, send it and receive it at last.


C programming requires siginificant patience and carefulness.

[1]: https://en.wikipedia.org/wiki/Ping_(networking_utility)
[2]: http://www.ping127001.com/pingpage.htm
