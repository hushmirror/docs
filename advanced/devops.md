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
