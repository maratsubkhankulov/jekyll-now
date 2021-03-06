---
published: true
---
Chubby is a distributed lock service. Its interface and design share similarities with file systems. This article summarizes [the paper](https://static.googleusercontent.com/media/research.google.com/en//archive/chubby-osdi06.pdf) about Chubby.

It maintains high reliability using typically 5 replicas. After release it's found its use as name server within Google. This paper went into details about low-level systems programming concepts for which I don't have much of a taste. Probably because I haven't been exposed to these concepts in the past. I may review and tidy up these notes in the future, in the mean time they'll serve as a summary of this paper. Which I found boring.  

Section 1:
Chubby is used by GFS and Bigtable for storing shared metadata. Interface similar to filesystem. Based on Paxos distributed concensus algorithm.

Section 2:...

2.1  Rationale - coarse-grained locking, cells, Paxos and consensus, not-time-based locking on a file.

2.2 System Structure - master server, replicas (usually 5), master lease - time during swich master stays elected, DNS for contacting members of the group, election protocol.

2.3 Files, directories and handles - file system similar but simpler than UNIX'. ACLs stored on ephemeral nodes as files.

2.4 Locks and sequencers - lock-delay disallows a faulty client from locking a resource for an arbitary time, locking requires write-permissions, lock checking is not mandatory. Exclusive writer-mode, shared reader-mode on lock files.

2.5 Events - clients can subscribe to events. E.g. permissions change, child node added/removed - can be used to implement mirroring.

2.6 API - opaque handle to file, open(), close(), delete(), getSequencer() - similar to a file system.

2.7 Caching - Chubby clients cache data in a consistent, write-through cache. Lease mechanism maintains cache. Cache is kept consistent by invalidations sent by master. When data is about to be modified, master invalidates client caches and waits for response or cache lease expires on all clients.

2.8 Sessions and Keep Alives - session is a relationship between Chubby client and cell. Keep Alives maintain session active. The session is deemed expired after if Keep Alives aren't sent for some time.

2.9 Fail-overs - new master recovers state from a mix of database via regular replication protocol, client state and state estimation and gives applications the illusion of distruption-free operation during fail-over.

2.10 Database implementation - replicated version of Berkeley DB was used by early Chubby implementations.

3.0 Mechanism for scaling - have seen 90,000 clients communicating with one master. Master may increase lease times during high load to deal with fewer Keep Alives as typical failure mode is failing to process them all. 

3.1 Proxies - Chubby's protocol can be proxied, but proxies aren't ideal

3.2 Partitioning - namespace of a cell can be partitioned between servers. Partitioning reduces read/write traffic on any one partition but not Keep Alive traffic.

4. Uses, surprises and design errors
4.1 Use and behavior - keep alives dominate network traffic

4.2 Java client - 7000 LOC similar to it the server - surprisingly little.

4.3 Use as a name server - Chubby's very commonly used as a name server within Google. It can scale well.

4.4 Problems with failover - original fail-over design requires master to write new sessions as they are created.

4.5 Abusive clients - authors didn't predict how Chubby will be used which made Chubby vulnerable to abuse-like usage despite the system being used internally. Several approaches were taken to mitigate the risk - reviews, storage quotas, making operations cheaper.

4.6 Lessons learned - developers don't often think about availability, fine-grained control could be ignored because it's usually discarded during system optimization, RPC use affects transport protocol choice.

5 Related work - Chubby was based on file systems such as AFS. Because it's a lock service it isn't intended to be used for storing large volumes of data.