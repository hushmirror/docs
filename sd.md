# Silent Dragon

This documentation is how you setup Silent Dragon, which is the Hush full node wallet.

## 1) Setup hushd

First you want to setup the Hush daemon (hushd). This enables your computer to download the complete Hush blockchain.

- If you have Linux, [click here](hushd-desktop-linux.md)
- If you have Windows, [click here](hushd-desktop-windows.md)
- If you have Mac, <_please contribute_>

## 2) Install Silent Dragon

**Make sure you make a paper backup of your seed phrases!**

Pick either the [binary release](https://git.hush.is/hush/SilentDragon/releases) or [compile it yourself](https://git.hush.is/hush/SilentDragon/src/branch/master/README.md). I personally compile but some prefer binaries.

# Advanced

## Sweeping funds from taddrs

Situation: You have funds in a taddr but when you send to a zaddr you get an
`16: bad-txns-oversize` or `bad-txns-acpublic-chain` error. What to do:

First,  try to send the total amount in the taddr minus a transaction fee. For example, if your taddr has 500 HUSH, then try to send 499.9999. As long as you don't have too many payments, this should work for most people.

If you get `16: bad-txns-oversize`, then you have too many inputs to send it all in one transaction. Since block 340,000 sending to a HUSH taddr is not allowed, including change outputs.

You must spend your inputs from largest to smallest, minus the transaction fee. Here is an example:

If you have received payments of 100 HUSH, 50 HUSH and then 1000 payments of 1 HUSH, then this is what you do:

* First send a transaction of 99.9999 HUSH. There will be no change with a 0.0001 HUSH fee.
* Then send a transaction of 49.9999 HUSH.
* Then you can send payments of 0.9999 HUSH. Do it repeatedly and occasionally try to send the full amount (minus transaction fee). Once the number of unspent payments is reduced enough, they will all fit in one transaction.
* For small amounts, you can try a fee=0 transaction.

You must spend payments in order from largest to smallest, exactly, with no change. 


