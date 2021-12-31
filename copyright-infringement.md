# Copyright Infringement of HUSH code

This file exists to document the copyright infringement of HUSH code as clearly as possible
with as much supporting info as possible, so people can decide for themselves. Any project
that infringes on "The Hush Developers" copyright will be listed here. Currently there
is one project that severely infringes the copyright of Hush code, which is Pirate.


## What is Copyright Infringement?

Copyright infringement is when one project takes code from another project, and maliciously
or accidentally changes copyright information and/or mixes code of incompatible copyrights
together. In the case of HUSH, our code is GNU Public License Version 3 (GPLv3), which means
projects that use non-GPL code cannot (legally) take our code and add it to their project,
unless they:

    * Change their copyright to GPLv3 (or a higher version)
    * Add copyright disclaimers from "The Hush Developers"

## z\_getbalances RPC

Hush wrote a new RPC which does not exist in any other cryptocoin called `z_getbalances` first created on April 13th 2021 in commit
https://github.com/hushmirror/hush3/commit/52e14828c8366d28a47d16acff42f64e988669fe then less than a month later the same exact RPC name, with same functionality was added
in Pirate commit https://github.com/PirateNetwork/pirate/commit/be114569ec692c41b0af5d6ad84e6acf7da6305f . It adds a new optional argument and is a derived work:

https://github.com/hushmirror/hush3/blob/dev/src/wallet/rpcwallet.cpp#L3646

https://github.com/PirateNetwork/pirate/blob/master/src/wallet/rpcwallet.cpp#L4157

## hush-cli was renamed pirate-cli

Very interestingly, in https://github.com/PirateNetwork/pirate/commit/412f5a296165ed7601749383cb4b94a4d8a0a9c5 it can be seen
that the `hush-cli` script was renamed `pirate-cli`, and the Hush copyright line was removed and replaced with a Pirate copyright
that says "no rights reserved" :

```
#!/bin/bash
-# Copyright (c) 2019 Hush developers
+# Copyright (c) 2019 PirateChain - no rights reserved

# set working directory to the location of this script
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
cd $DIR

-NAME=HUSH3
+NAME=PIRATE
CLI=${KOMODOCLI:-./komodo-cli}
$CLI -ac_name=$NAME "$@"
```

This file no longer exists in their master branch, but is a good example of Pirate blatantly infringing on Hush code copyright.

## TLS Code

Pirate seems to have copied huge sections of the TLS code from Hush, including entire files
and detailed code comments in certain functions, while also not adding our Copyright disclaimer
to their files. Since Pirate is MIT and Hush is GPLv3, mixing this code is incompatible.

For instance: https://github.com/hushmirror/hush3/blob/dev/src/hush/tlsenums.h was created in
commit https://github.com/PirateNetwork/pirate/commit/a21bf4051094a8ca9c481efd32d706361e8003b5
on Oct 31st 2021 but https://github.com/hushmirror/hush3/blob/dev/src/hush/tlsenums.h was
created on Sep 29th 2020 in commit https://github.com/hushmirror/hush3/commit/62f67821ecd801cc26b9f3aa14de8735d6563a0a

The files are almost exactly the same, except Hush Copyright lines have been removed, the namespace
"hush" was changed to "tls" and whitespace has been added. The exact ordering of elements and exact
names of variables is exactly the same.

Another fact is that all the filenames in https://github.com/PirateNetwork/pirate/tree/master/src/tls
are exactly the same as the filenames in https://github.com/hushmirror/hush3/tree/dev/src/hush yet 
the Hush files were created over a year prior.

Additionally the function threadSocketHandler at https://github.com/PirateNetwork/pirate/blob/master/src/tls/tlsmanager.cpp#L347
is almost exactly the same. The differences are because Hush uses WolfSSL where Pirate uses OpenSSL and
logging to a different log category ("net" vs "tls") .

The `-tls` CLI option and `-tls=only` argument was added in commit https://github.com/hushmirror/hush3/commit/62f67821ecd801cc26b9f3aa14de8735d6563a0a on Sep 29th 2020
and then appears over a year later in Pirate source code in commit https://github.com/PirateNetwork/pirate/commit/a21bf4051094a8ca9c481efd32d706361e8003b5

Additionally, Hush added the field `tls_connections` to two RPCs: `getinfo` and `getnetworkinfo` and Pirate took this code,
and turned the oneliner to a for loop to make it look different, yet it is still a derived work:

```
pirate/src/rpc/net.cpp
515:            "  \"tls_connections\": xxxxx,              (numeric) the number of TLS connections\n"
557:    obj.push_back(Pair("tls_connections", sslNode));

pirate/src/rpc/misc.cpp
212:            "  \"tls_connections\": xxxxx,   (numeric) the number of TLS connections\n"
302:    obj.push_back(Pair("tls_connections", sslNode));

hush3/src/rpc/net.cpp
506:            "  \"tls_connections\": xxxxx,              (numeric) the number of TLS connections\n"
541:    obj.push_back(Pair("tls_connections", (int)std::count_if(vNodes.begin(), vNodes.end(), [](CNode* n) {return n->ssl != NULL;})));

hush3/src/rpc/misc.cpp
219:            "  \"tls_connections\": xxxxx,   (numeric) the number of encrypted TLS (SSL) connections\n"
293:    obj.push_back(Pair("tls_connections", (int)std::count_if(vNodes.begin(), vNodes.end(), [](CNode* n) {return n->ssl != NULL;})));
```

You can also notice they exactly copied the documentation of each RPC, with no changes.

In the file `src/net.h` you can see that Pirate copied the exact variable names of the TLS context objects:

```
pirate/src/net.h 
extern SSL_CTX *tls_ctx_server;
extern SSL_CTX *tls_ctx_client;

/hush3/src/net.h 
extern SSL_CTX *tls_ctx_server;
extern SSL_CTX *tls_ctx_client;
```

Additionally, these two objects were added at exactly the same place in the file, between the `strSubVersion` and `LocalServiceInfo` struct.

https://github.com/PirateNetwork/pirate/blob/master/src/net.h#L226

https://github.com/hushmirror/hush3/blob/dev/src/net.h#L196

