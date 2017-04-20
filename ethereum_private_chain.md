# Why did it fail?!

## Ethereum

### Goal
Let's build a private ethereum chain which only our defined nodes connect to

### Setup
- geth-alltools-linux-386-1.6.0-facc47cb
- DigitalOcean 5$ debian boxes and a local machine

### Private Chain

#### Nodes don't connect

##### Never let your nodes connect to the public ethereum network!
- Run geth with *--nodiscover* flag.   
- If a node does connect to the public network, it's database is most likeley 'polluted.   
- Delelte it and start clean. (rm -r ~/.ethereum)   

##### The private chain is defined by the so called genesis block.
- Every node has to be initialized with this genesis block, so it knows where the chain starts.
- The genesis block can be generated using a genesis.json file and the command `geth init genesis.json`
- Make sure that the genesis.json file you use on your nodes is EXACTLY the same. If it isn't, your nodes won't connect!
- Make up some random number for your networkid. E.g. 1234
- Verify by checking the genesis block hash and network id from the geth javascript console by logging in to the console:
`geth --networkid 1234 --nodiscover console` and then running: `admin.nodeInfo.protocols.eth`
- (You can exit the console with 'exit' ... who would have thought ;) )

*genesis.json*   
```
  {
    "config": {
        "chainId": 2217,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "difficulty": "0x00400",
    "gasLimit": "2100000",
    "alloc": {
        "4b86b43b798c527234998992165200f993d3e228": { "balance": "10000000000000000000" },
        "f41c74c9ae680c1aa78f42e5647a62f353b7bdde": { "balance": "400000" }
    }
}
```

##### Autodiscovery of the nodes using the `bootnode` tool didn't work
- Start simple and manually connect the nodes
- Find out the enode id of the node you want to connect to:
- Run `admin.nodeInfo.enode` inside geth's javascript console:

`"enode://750ade65e0f208aa48d9314306fe6d66ec26f83cff39ee9fbf710c1f8398b5357e2c272f02b0df2072041fe9c3b6017cf11121f4169fa2718b3801b441e056ae@[::]:30303?discport=0"`

- Before adding it to the static-nodes.json file we remove the ?discport=0 part at the end, and replace [::] with the nodes external ip address:

`"enode://750ade65e0f208aa48d9314306fe6d66ec26f83cff39ee9fbf710c1f8398b5357e2c272f02b0df2072041fe9c3b6017cf11121f4169fa2718b3801b441e056ae@[207.154.123.234]:30303"`

- We can define the nodes (peers) to connect to for every node in a config file called *static-nodes.json*.
- It has to be placed in the data directory (default: ~/.ethereum/static-nodes.json)
- It's syntax is a simple array containing the enode id's to connect to.

*static-nodes.json*   
```
[
  "enode://750ade65e0f208aa48d9314306fe6d66ec26f83cff39ee9fbf710c1f8398b5357e2c272f02b0df2072041fe9c3b6017cf11121f4169fa2718b3801b441e056ae@[207.154.123.234]:30303"
]
```

- Check that the enode line contains the target nodes IP-Address and port at the end. Verify that it is correct.

- Start up the first node (the one without a static-nodes.json)
  `./geth --networkid 1234 --nodiscover --verbosity 9 console`

- Start up the second node (with the static-nodes.json)
  `./geth --networkid 1234 --nodiscover --verbosity 9 console`

- We see log output on the first node like:   
```
  TRACE[04-20|14:52:57] Accepted connection                      addr=207.154.123.65:48582
  DEBUG[04-20|14:52:57] Adding p2p peer                          id=53511b4a8eb2ffe7 name=Geth/v1.6.0-stable-f...                         addr=207.154.123.65:48582 peers=1
  TRACE[04-20|14:52:57] Starting protocol eth/63                 id=53511b4a8eb2ffe7 conn=inbound
  DEBUG[04-20|14:52:57] Ethereum peer connected                  id=53511b4a8eb2ffe7 conn=inbound name=Geth/v1.6.0-stable-facc47cb/linux-amd64/go1.8.1
  TRACE[04-20|14:52:57] Registering sync peer                    peer=53511b4a8eb2ffe7
```

- Verify by running `admin.peers` in the console of both nodes.   
```
  [{
    caps: ["eth/62", "eth/63"],
    id: "53511b4a8eb2ffe72e8431b73a55e165a73fd9a8ad89bebc798ffdcac471b2ee08645a96c1961630bcaa45d2d3ca3325770cf4029f44fbd51806bcfedfe7c559",
    name: "Geth/v1.6.0-stable-facc47cb/linux-amd64/go1.8.1",
    network: {
      localAddress: "207.154.123.234:30303",
      remoteAddress: "207.154.123.65:48582"
    },
    protocols: {
      eth: {
        difficulty: 5269312,
        head: "0xd5faedc447bce76ebb94fd6f90b8ca4c236d314b06993789d2af919163604ca2",
        version: 63
      }
    }
}]
```

#### Nodes are connected! Let's try mining

- Exit the console of the second node. (exit)
- Start it again with added `--mine` flag. (You can also add `--minerthreads 1` to get a clean logflow for now)
`./geth --networkid 1234 --nodiscover --verbosity 9 --mine console`

- the mining node is only writing single lines containing:
```
INFO [04-20|15:01:50] Generating DAG in progress               epoch=1 percentage=0  elapsed=32.719s
DEBUG[04-20|15:02:06] Recalculated downloader QoS values       rtt=20s confidence=1.000 ttl=1m0s
INFO [04-20|15:02:26] Generating DAG in progress               epoch=1 percentage=1  elapsed=1m8.585s
```
- Check the server you are running on. Probably it's too low on memory. Try mining on a faster computer first to remove this problem.

mining node (running on fast pc):   
```
INFO [04-20|16:58:00] Generating ethash verification cache     epoch=1 percentage=78 elapsed=3.016s
INFO [04-20|16:58:01] Generated ethash verification cache      epoch=1 elapsed=3.894s
TRACE[04-20|16:58:04] Ethash nonce found and reported          miner=0 attempts=249439 nonce=5383070798337378377
INFO [04-20|16:58:04] Successfully sealed new block            number=197 hash=85f23f…6218aa
DEBUG[04-20|16:58:04] Trie cache stats after commit            misses=14 unloads=0
INFO [04-20|16:58:04] 🔨 mined potential block                  number=197 hash=85f23f…6218aa
INFO [04-20|16:58:04] Commit new mining work                   number=198 txs=0 uncles=0 elapsed=274.331µs
TRACE[04-20|16:58:04] Propagated block                         hash=85f23f…6218aa recipients=1 duration=2562047h47m16.854s
TRACE[04-20|16:58:04] Announced block                          hash=85f23f…6218aa recipients=1 duration=2562047h47m16.854s
TRACE[04-20|16:58:04] Announced block                          hash=85f23f…6218aa recipients=0 duration=2562047h47m16.854s
TRACE[04-20|16:58:04] Started ethash search for new nonces     miner=0 seed=8692281432698145952
```

node1 receiving the mined blocks:   
```
DEBUG[04-20|14:58:06] Importing propagated block               peer=750ade65e0f208aa number=198 hash=f85541…2cfee4
TRACE[04-20|14:58:06] Propagated block                         hash=f85541…2cfee4                                                                                                              recipients=0 duration=10.410ms
DEBUG[04-20|14:58:06] Trie cache stats after commit            misses=186 unloads=4
DEBUG[04-20|14:58:06] Inserted new block                       number=198 hash=f85541…2cfee4                                                                                                              uncles=0 txs=0  gas=0      elapsed=10.110ms
INFO [04-20|14:58:06] Imported new chain segment               blocks=1   txs=0  mgas=0.000 elapsed=10.614ms     mgasps=0.000 number=198 hash=f85541…2cfee4
TRACE[04-20|14:58:06] Announced block                          hash=f85541…2cfee4                                           
```

### We have a working, private ethereum blockchain with 2 connected nodes, one of which is mining blocks!
