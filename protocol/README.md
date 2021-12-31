# Hush Protocol Documentation

This documentation covers low-level internals of how Hush Protocol is different
from the Bitcoin Protocol and Zcash Protocol which Hush Protocol builds upon.
This documentation is for Hush (v3) as opposed to the original Hush codebase, which
was based on an older version of Zcash Protocol.

This documentation assumes some familiarity with how Bitcoin Protocol works, but
does not assume knowledge of Zcash Protocol or other privacy coins.

# Hush Addresses

Hush addresses work in essentially the exact same way as Bitcoin Protocol addresses,
except that Hush has both transparent addresses (taddrs) and shielded addresses (zaddrs).
Taddrs in Hush work exactly the same as their counterparts in Bitcoin Protocol. There is
a public key which when hashed with Base58Check algorithm, gives the public address of
that public key. Hush also has "compressed" or "uncompressed" public keys just like BTC.
Hush taddrs are CASE-SENSITIVE, just like BTC.

## Hush taddrs

Hush Protocol forked from Bitcoin Protocol before Segwit existed, so it has no complications
of "segwit-ish" things. Hush essentially has the Bitcoin 0.11.2 protocol frozen deep inside it,
with small improvements and adjustments. Just like originally all BTC mainnet addresses started with a 1 or 3,
Hush has taddrs that start with either R or b. This is related to HUSH having it's own base58 prefix.
All Hush Smart Chains will have compatible addresses by using the same base58 prefix, which aids
integration.

## Hush zaddrs

Hush zaddrs are Sapling Shielded addresses start with `zs1` in Bech32 format, which means they
are CASE-INSENSITIVE. This makes zaddrs more efficient when stored in QR codes. Hush Smart Chains
also will have addresses starting with `zs1`.

# Hush Transactions

Originally in 2016 Hush started as a fork of Zcash 1.x and had *optional privacy*. We decided that was a bad
idea and transitioned to *enforced privacy* as of Block 340000 of the HUSH v3 chain in November 2020. So originally
a transaction could have a taddr as an output (receiving funds) but that is no longer possible. We call this "z2z"
because it means normal transactions must be from a zaddr and to a zaddr. Mining new funds and DPoW still use taddrs
since that data must be public.

When a new block is mined on HUSH, 90% of the funds go to the taddr of the miner and 10% goes to the address of
the Founder Reward (or Developer Tax, however you like to think of it) which is `RHushEyeDm7XwtaTWtyCbjGQumYyV8vMjn` .

Zcash has never and seemingly never will mandate zaddr usage, so z2z transactions are rare on their network. In addition
to z2z, Hush has something called Sietch which obscures even more metadata about shielded transactions, specifically,
the ordering of the outputs and change and the number of outputs.

By default Zcash z2z transactions will have two outputs, where Hush will have eight on average. Individual transactions
will have between seven and nine shielded outputs which is non-deterministically chosen at transaction creation time.

# Hush Peer-to-Peer (P2P) layer

Hush has mandatory TLS 1.3 encryption of all network connections, the first cryptocoin to accomplish this. What it means
is that your ISP, network administrator and many low-level internet operators no longer can easily see exactly what you
are doing, the details of your transactions and which IP addresses make them at which times is blinded to them, amongst
lots of other metadata. Bitcoin, Zcash and Monero all broadcast all network data in plaintext, which leaks massive amounts
of data to network eavesdroppers. Since everything is public already on Bitcoin, it makes sense that Satoshi did not use
TLS. But for privacy coins, encrypted connections is a must to lessen metadata leakage.

Mandatory TLS means that people recording the internet don't get to see all your data,
and that is very good. To this date, all privacy coins known to the author use plaintext or optionally use TLS, no coin
except Hush requires privacy for all network connections!

What if the attacker is willing to run one or more nodes?  In both Bitcoin/Zcash protocol, it is simple to have one or many nodes on the network
that listen for which IP address is first relaying a transaction, which is a very good indication of that node having created that transaction.
Monero uses the (imperfect, but better than nothing) Dandelion Protocol method of improving network privacy, and Hush was inspired by that to
make improvements against so-called "Sybil Attacks". 

A "Sybil Attack" is a form of Denial-of-Service where the attacker is willing and able to run many nodes. It is often used to attack the Tor Network,
where a large percentage of Tor Exit Nodes will become malicious and reduce the privacy of the network. Zcash and Monero both have privacy failures
when a Sybil Attack is able to run a small percentage of the network, such as 10%. If a Sybil Attack would be able to run 50% or even 99% of the network,
the privacy failures just get worse.

The way the Hush p2p layer is designed, even if the Sybil Attacker is running 99% of the nodes on the network, they still would only have, less than a 50% chance of being able to tell which node created a transaction. This also means that under normal circumstances, it is impossible for a network eavesdropper or malicious node(s) on the network to identify which node created which transaction. While Bitcoin and Zcash Protocol are optimized to reduce bandwidth of the network, which makes it vulnerable to Sybil Attacks, Hush chooses to use more bandwidth and slightly more network latency to gain privacy.

## P2P Details

The number of maximum outbound peers in 8 in Bitcoin, 16 in Zcash and 64 with Hush. This necessarily means more network traffic and higher bandwidth usage
in Hush versus Zcash or Bitcoin, which is increased even more by our use of TLS 1.3 instead of plaintext. Privacy is worth it to us, so we are happy to trade bandwidth
for privacy.

In Bitcoin and Zcash, when a node makes a brand new transaction, it relays it to all known outbound peers that it knows about. This is very good for resiliency,
since it spreads the new transaction to many nodes at once. It is very bad for privacy, because of something called the "First Timestamp Estimator" from the paper
"Deanonymization in the bitcoin P2P network" https://papers.nips.cc/paper/2017/file/6c3cf77d52820cd0fe646d38bc2145ca-Paper.pdf .


Nodes can listen
to P2P traffic, which is never written to the public "ledger" data viewable on an explorer, and estimate which node relayed a transaction first. This is very
likely the node that created it, or a service that helped create the transaction, such as a lite wallet server. The more nodes you are willing to have listen,
the more likely you are to guess correctly. If you have two nodes that say two different IP addresses made a transaction, the one with earlier timestamp is the
better guess.

It is trivial on Bitcoin and Zcash networks to run nodes to perform this "First Timestamp Estimator" and the Hush community assumes this already happens and is
embedded into blockchain analysis softare already. This means that blockchain analysis companies can likely already tell the IP address of Bitcoin and Zcash
transactions with high likelihood.

## Defeating "First Timestamp Estimator" (FTE)

The idea of a FTE comes from "Anonymity Properties of the Bitcoin P2P Network" at https://arxiv.org/pdf/1703.08761.pdf by Giulia Fanti and Pramod Viswanath.
We thank Guilia Fanti for their feedback on this section.

Dandelion is one way to try to defeat it, but the internals implementation is complex and can potentially add new attacks, which is why it was never merged
into Bitcoin. Monero does have Dandelion, but attacks such as BADCACA show it is still vulnerable to Sybil Attacks. Dandelion++ is an improvement to Dandelion
that fixes many issues with the original Dandelion algorithm, but so far it seems no cryptocoin fully implements it, likely because of the complexity of how
it changes network and mempool internals.

Hush was motivated by the question "What is the simplest way to defeat the first timestamp estimator?". We also wanted to optimize for relatively bad conditions,
of Sybil Attacks that run up to 49% of nodes. Sybil attacks of 50% or more of the network are very powerful and no cryptocoin, to our knowledge, has come up with
a method to protect against them. Implementing Dandelion
requires changing how the mempool works and we also wanted to avoid that, since it's very hard to know if new vulnerabilities are being introduced.

Hush's method to defeat the FTE is choosing a *random subset of peers* at transaction relay time, and only relaying the transaction to those peers.
This means that it takes longer for Hush transactions to fully propagate across the network. Hush is happy to trade some network latency for privacy and since this
latency is on the order of seconds, it does not slow down anything noticeable to end users. Currently Hush relays transactions to half of it's outbound nodes, and rounds
down in case of odd numbers. So for instance, if a Hush node has 5 outbound connections, it will relay to a random subset of 2 of them, which is 40%. Rounding down
helps make things harder on the attacker for nodes with small numbers of outbound connections, such as those who have only recently joined the network.

Imagine a scenario where the network is 25% Sybil nodes, which are using the FTE to correlate which nodes created which transactions, i.e. creating 
a mapping between IP address and transaction id. With the Hush transaction relay algorithm, it's possible for a Sybil node to be directly connected to the node
making a transaction, yet that node might not relay the transaction to the Sybil node, since that outbound node is not one of the randomly 50% of peers that is chosen
when the transaction is made.

If we assume that 25% of our nodes are Sybil nodes and we only relay our transaction to 50% of our connections, then on average, the FTE will only be correct
12.5% of the time. As we assume the Sybil Attacker has a larger percentage of network nodes, the FTE will increase in accuracy but never be more than 50% accurate, unless
it has Sybile nodes which comprise more than 50% of the network.

Nodes with small numbers of connections also do well against the attacker. If a node has only 3 outbound connections and makes a transaction, it will only relay it to 1
node, which is 33.3% of it's peers. If we assume a Sybil Attacker with 25% of network nodes, the FTE will only be accurate about `25% * 33% =~ 8.3%` of the time in trying to decide which node made the
transaction. In the same situation, a Hush node with 15 connections will result in an FTE with `25% * 7/15 =~ 11.6%` accuracy.

We note that, as mentioned by Guilia Fanti, if a Sybil Attacker comprises say, 99% of nodes on the network, then they will almost always be one of the peers of every
other node on the network, and the above defense fails.

## Tor P2P Support

Tor v2 was deprecated by the Tor network and they have changed to v3, and Hush is currently in the process of merging Tor v3 support from BTC Core on the `p2p` branch of hush3.git.
