Iquidus Explorer - 1.7.4
================

An open source block explorer written in node.js.

### See it in action

*  [List of live explorers running Iquidus](https://github.com/iquidus/explorer/wiki/Live-Explorers)


*Note: If you would like your instance mentioned here contact me*

### Requires

*  node.js >= 8.17.0 (12.14.0 is advised for updated dependencies)
*  mongodb 4.2.x
*  *coind

### Create database

Enter MongoDB cli:

    $ mongo

Create databse:

    > use explorerdb

Create user with read/write access:

    > db.createUser( { user: "iquidus", pwd: "3xp!0reR", roles: [ "readWrite" ] } )

*Note: If you're using mongo shell 4.2.x, use the following to create your user:

    > db.addUser( { user: "username", pwd: "password", roles: [ "readWrite"] })

### Get the source

    git clone https://github.com/iquidus/explorer explorer

### Install node modules

    cd explorer && npm install --production

### Configure

    cp ./settings.json.template ./settings.json

*Make required changes in settings.json*

### Start Explorer

    npm start

*Note: mongod must be running to start the explorer*

As of version 1.4.0 the explorer defaults to cluster mode, forking an instance of its process to each cpu core. This results in increased performance and stability. Load balancing gets automatically taken care of and any instances that for some reason die, will be restarted automatically. For testing/development (or if you just wish to) a single instance can be launched with

    node --stack-size=10000 bin/instance

To stop the cluster you can use

    npm stop

### Syncing databases with the blockchain

sync.js (located in scripts/) is used for updating the local databases. This script must be called from the explorers root directory.

    Usage: node scripts/sync.js [database] [mode]

    database: (required)
    index [mode] Main index: coin info/stats, transactions & addresses
    market       Market data: summaries, orderbooks, trade history & chartdata

    mode: (required for index database only)
    update       Updates index from last sync to current block
    check        checks index for (and adds) any missing transactions/addresses
    reindex      Clears index then resyncs from genesis to current block

    notes:
    * 'current block' is the latest created block when script is executed.
    * The market database only supports (& defaults to) reindex mode.
    * If check mode finds missing data(ignoring new data since last sync),
      index_timeout in settings.json is set too low.


*It is recommended to have this script launched via a cronjob at 1+ min intervals.*

**crontab**

*Example crontab; update index every minute and market data every 2 minutes*

    */1 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js index update > /dev/null 2>&1
    */2 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js market > /dev/null 2>&1
    */5 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/peers.js > /dev/null 2>&1

### Wallet

Iquidus Explorer is intended to be generic, so it can be used with any wallet following the usual standards. The wallet must be running with atleast the following flags

    -daemon -txindex
    
### Security

Ensure mongodb is not exposed to the outside world via your mongo config or a firewall to prevent outside tampering of the indexed chain data. 

### Known Issues

**script is already running.**

If you receive this message when launching the sync script either a) a sync is currently in progress, or b) a previous sync was killed before it completed. If you are certian a sync is not in progress remove the index.pid and db_index.pid from the tmp folder in the explorer root directory.

    rm tmp/index.pid
    rm tmp/db_index.pid

**exceeding stack size**

    RangeError: Maximum call stack size exceeded

Nodes default stack size may be too small to index addresses with many tx's. If you experience the above error while running sync.js the stack size needs to be increased.

To determine the default setting run

    node --v8-options | grep -B0 -A1 stack_size

To run sync.js with a larger stack size launch with

    node --stack-size=[SIZE] scripts/sync.js index update

Where [SIZE] is an integer higher than the default.

*note: SIZE will depend on which blockchain you are using, you may need to play around a bit to find an optimal setting*





API Documentation

The block explorer provides an API allowing users and/or applications to retrieve information from the network without the need for a local wallet.
API Calls

Return data from coind

    getdifficulty
    Returns the current difficulty.
    radioblockchain.info:80/api/getdifficulty

    getconnectioncount
    Returns the number of connections the block explorer has to other nodes.
    radioblockchain.info:80/api/getconnectioncount

    getblockcount
    Returns the current block index.
    radioblockchain.info:80/api/getblockcount

    getblockhash [index]
    Returns the hash of the block at ; index 0 is the genesis block.
    radioblockchain.info:80/api/getblockhash?index=1337

    getblock [hash]
    Returns information about the block with the given hash.
    radioblockchain.info:80/api/getblock?hash=00000000002db22bd47bd7440fcad99b4af5f3261b7e6bd23b7be911e98724f7

    getrawtransaction [txid] [decrypt]
    Returns raw transaction representation for given transaction id. decrypt can be set to 0(false) or 1(true).
    radioblockchain.info:80/api/getrawtransaction?txid=c251b0f894193dd55664037cbf4a11fcd018ae3796697b79f5097570d7de95ae&decrypt=0
    radioblockchain.info:80/api/getrawtransaction?txid=c251b0f894193dd55664037cbf4a11fcd018ae3796697b79f5097570d7de95ae&decrypt=1

    getnetworkhashps
    Returns the current network hashrate. (hash/s)
    radioblockchain.info:80/api/getnetworkhashps



Extended API

Return data from local indexes

    getmoneysupply
    Returns current money supply
    radioblockchain.info:80/ext/getmoneysupply

    getdistribution
    Returns wealth distribution stats
    radioblockchain.info:80/ext/getdistribution

    getaddress (/ext/getaddress/hash)
    Returns information for given address
    radioblockchain.info:80/ext/getaddress/RSDucSViM4EBqRbf4U6sBNDQEWs7Eqgf6w

    gettx (/ext/gettx/hash)
    Returns information for given tx hash
    radioblockchain.info:80/ext/gettx/c251b0f894193dd55664037cbf4a11fcd018ae3796697b79f5097570d7de95ae

    getbalance (/ext/getbalance/hash)
    Returns current balance of given address
    radioblockchain.info:80/ext/getbalance/RSDucSViM4EBqRbf4U6sBNDQEWs7Eqgf6w

    getlasttxsajax (/ext/getlasttxsajax/min)
    Returns last transactions greater than [min]
    Note: returned values are in satoshis
    radioblockchain.info:80/ext/getlasttxsajax/100 



Links to the block explorer (regular web ui use)

    transaction (/tx/txid)
    radioblockchain.info:80/tx/c251b0f894193dd55664037cbf4a11fcd018ae3796697b79f5097570d7de95ae

    block (/block/hash)
    radioblockchain.info:80/block/00000000002db22bd47bd7440fcad99b4af5f3261b7e6bd23b7be911e98724f7

    address (/address/hash)
    radioblockchain.info:80/address/RSDucSViM4EBqRbf4U6sBNDQEWs7Eqgf6w

    qrcode (/qr/hash)
    radioblockchain.info:80/qr/RSDucSViM4EBqRbf4U6sBNDQEWs7Eqgf6w





### License

Copyright (c) 2015, Iquidus Technology  
Copyright (c) 2015, Luke Williams  
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Iquidus Technology nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
