# FDP-GraphSync

![](https://img.shields.io/badge/made%20by-Myel-blue)
![](https://img.shields.io/github/license/myelnet/js-graphsync?color=green)

> JS implementation of the GraphSync v2 wire protocol using FDP as transport

## Background

GraphSync is an IPFS data transfer protocol used across the IPFS and Web3 ecosystem for exchanging
IPLD data. It is used by Filecoin for syncing the blockchain and transfering DAGified content
in a trustless fashion.

Requires a [libp2p-circuit](https://github.com/libp2p/js-libp2p/blob/633d4a9740ea02e32c0bb290c0a3958b68f181e9/src/circuit/README.md) [relayer](https://github.com/libp2p/js-libp2p/blob/633d4a9740ea02e32c0bb290c0a3958b68f181e9/doc/CONFIGURATION.md#setup-with-relay)
```

## Usage

```ts
import { multiaddr } from '@multiformats/multiaddr'
import Libp2p from 'libp2p'
import { TCP } from '@libp2p/tcp'


import {create, getPeer} from "libp2p";
import {Noise} from "@chainsafe/libp2p-noise";
import WebSockets from "libp2p-websockets";
import Mplex from "libp2p-mplex";
import {MemoryBlockstore} from "blockstore-core/memory";
import {GraphSync, unixfsPathSelector} from "@dcdn/graphsync";
import { FdpStorage } from '@fairdatasociety/fdp-storage'

const batchId = 'GET_BATCH_ID_FROM_YOUR_NODE' // fill it with batch id from your Bee node
const fdp = new FdpStorage('http://localhost:1633', batchId)

// TODO: Implement SwarmBeeBlockstore
const blocks = new SwarmBeeBlockstore(fdp);
const relayer = await SwarmBeeBlockstore.createLibp2pCircuitRelayer(`<multiaddr>`)
const node = await createLibp2p({
  addresses: {
    listen: [relayer]
  },
  transports: [
    new TCP()
  ],
  streamMuxers: [
    new Mplex()
    ],
  connectionEncryption: [
    new Noise()
  ]
  },
  config: {
    relay: {                   // Circuit Relay options (this config is part of libp2p core configurations)
      enabled: true           // Allows you to dial and accept relayed connections. Does not make you a relay.
    }
  }
})
await node.start();
    
const exchange = new GraphSync(node, blocks);


// Connect to an IPFS node
const provider = getPeer("/ip4/127.0.0.1/tcp/41505/ws/p2p/12D3KooWCYiNWNDoprcW74NVCEKaMhSbrfMvY4JEMfWrV1JamSsA");

node.peerStore.addressBook.add(provider.id, provider.multiaddrs);

// Request block
const [cid, selector] = unixfsPathSelector("bafyreiakhbtbs4tducqx5tcw36kdwodl6fdg43wnqaxmm64acckxhakeua/Cat.jpg");

// TODO: Implement BeesonMessageHooks to convert CID blocks to Beeson/SoCs or feeds
const request = exchange.request(cid, selector);
request.open(provider.id);

// Save the blocks into the store;
await request.drain();

```


