# Hush DevOps

This documentation is for services that run a Hush full node, such as mining pools, exchanges
and lite wallet node operators. It covers advanced things to know which can help reduce downtime.

## Minimize Rescan Downtime

Sometimes a Hush node must rescan the history of the blockchain, to look for incoming and outgoing
transactions. It normally happens when importing a new private key, but can also happen when some
type of crash happens. For instance:

  * The physical server or VM runs out of memory
  * There is a power outage
  * A bug in hushd itself causes a coredump and unclean exit
  * kill -9 hushd is used

If the above circumstances happen, hushd will not be able to do it's normal "bookkeeping" when
doing a clean exit. Part of this work is updating metadata about shielded transactions, called "witnesses".
When hushd starts and notices this data is out of date, it performs a *full rescan of all time* to update
the data, which works, but it's very slow and will cause downtime of potentially hours or longer.

To avoid this downtime, first make sure to be using at least Hush 3.8.0 which has two new features

  * `-rescanheight` is an added option to `-rescan` that gives a specific height to rescan from instead of the genesis block
  * `-keepnotewitnesscache` which means keep the partial ztx metadata and update it, instead of recreating it from scratch

For example, lets say from the debug.log that a system admin knows that the last block height the node processed
was 12345. This can be found by looking for the very last UpdateTip line in the debug.log. That means witness data is correct
up to that height, and that is what we can use as a starting point. So the sysadmin would do

    hushd -rescan -rescanheight=12345 -keepnotewitnesscache

and then hushd will only rescan the history from block 12345 until the current block, which saves huge amount of time.

### What can go wrong?

Lets say that the sysadmin uses the wrong `-rescanheight` on accident, what can go wrong? Is it possible to lose funds?
No, it is not possible to lose access to funds with this method, especially if you have backups of the wallet.dat and private keys.
If the sysadmin uses a block height further back in history, say 12300, that is perfectly fine. The node will do some extra
work and take slightly longer. If the sysadmin uses a block height that is *too high*, such as 12500, it
could cause hushd to be unaware of any transactions between block 12345 and 12500. If incoming transactions occurred, they will not
be reflected in wallet balances and if outgoing transactions occured, wallet errors may happen when trying to spend shielded funds
which hushd things are *unspent* but were actually spent between blocks 12345 and 12500.

The fix is simple, the sysadmin can just stop and restart hushd with a `-rescanheight` further back in the past, such as block 12345
or earlier. If the sysadmin does not know an exact block height to use for `-rescanheight` than a conservative estimate in the past
can be used, such as a block height from one month or even one year ago. This will still help avoid much of blockchain history during
the rescan and will still result in drastically less downtime while the node is rescanning.

## My RPC interface stops responding to requests!

It is sometimes possible for the RPC interface to stop answering requests, due to a bug inherited from BTC/ZEC and made worse
by the privacy additions inside of HUSH. There are two different mitigations Hush has added to hushd to deal with this, while
the underlying problem is researched more.

### PLZ STOP

If the RPC interface stops answering requests, it becomes impossible to run `hush-cli stop` and sysadmins are forced to use `kill`
to stop the node, which can lead to problems described above which hushd crashing. So Hush developers made a purely disk-based method
to ask hushd to stop. A sysadmin simply creates a file called `plz_stop` in the same directory as `wallet.dat`. Every 120 seconds, `hushd`
looks for this file and will stop the node if it is found. The file can be of zero size, it's contents do not matter. So for instance:

```
cd ~/.hush/HUSH3 # or cd ~/.komodo/HUSH3 for legacy locations
touch plz_stop
# hushd will stop within 2 minutes
```

With the above "trick", you can avoid using `kill` and hushd will do it's ztx bookkeeping just before it shuts down, which avoids long rescans.


### RPC Work Queue

One way to avoid the "RPC deadlock bug" or make it much rarer is to increase the size of the "RPC work queue" in HUSH3.conf like this:

```
rpcworkqueue=8192
```

This makes it take much longer and less likely for all RPC "slots" to be deadlocked at the same time, which prevents issuing RPC commands to hushd.
Larger numbers can be used, and in general, the more zaddrs and transactions a wallet has, the more likely it will run into a deadlock and need a higher
`rpcworkqueue` value. This method uses a few more kilobytes of RAM to have more slots, which is a good trade to avoid downtime and node maintenance.

