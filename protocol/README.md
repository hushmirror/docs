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

# Hush Peer-to-Peer (P2P) layer

Hush has mandatory TLS 1.3 encryption of all network connections, the first cryptocoin to accomplish this. What it means
is that your ISP, network administrator and many low-level internet operators no longer can easily see exactly what you
are doing, the details of your transactions and which IP addresses make them at which times is blinded to them, amongst
lots of other metadata. Bitcoin, Zcash and Monero all broadcast all network data in plaintext, which leaks massive amounts
of data to network eavesdroppers. Since everything is public already on Bitcoin, it makes sense that Satoshi did not use
TLS. But for privacy coins, encrypted connections is a must to lessen metadata leakage.

Mandatory TLS means that people recording the internet don't get to see all your data,
and that is very good.

What if the attacker is willing to run one or more nodes?  In both Bitcoin/Zcash protocol, it is simple to have one or many nodes on the network
that listen for which IP address is first relaying a transaction, which is a very good indication of that node having created that transaction.
Monero uses the (imperfect, but better than nothing) Dandelion Protocol method of improving network privacy, and Hush was inspired by that to
make improvements against so-called "Sybil Attacks". 

A "Sybil Attack" is a form of Denial-of-Service where the attacker is willing and able to run many nodes. It is often used to attack the Tor Network,
where a large percentage of Tor Exit Nodes will become malicious and reduce the privacy of the network. Zcash and Monero both have privacy failures
when a Sybil Attack is able to run a small percentage of the network, such as 10%. If a Sybil Attack would be able to run 50% or even 99% of the network,
the privacy failures just get worse.

The way the Hush p2p layer is designed, even if the Sybil Attacker is running 99% of the nodes on the network, they still would only have, less than a 50% chance of being able to tell which node created a transaction. This also means that under normal circumstances, it is impossible for a network eavesdropper or malicious node(s) on the network to identify which node created which transaction. While Bitcoin and Zcash Protocol are optimized to reduce bandwidth of the network, which makes it vulnerable to Sybil Attacks, Hush chooses to use more bandwidth and gain privacy.

