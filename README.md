[![JSON check](https://github.com/mempool/mining-pools/actions/workflows/validate-json.yml/badge.svg)](https://github.com/mempool/mining-pools/actions/workflows/validate-json.yml)


# Bitcoin Mining Pools

Mining pools definition used on https://mempool.space/graphs/mining/pools

# Contributing

Contributions welcome. All changes must be applied in `pools-v2.json` file.

## Adding a new mining pool

Regardless of the choosen method, we recommend adding a appropriate slug to each
new mining pool you add to `pools-v2.json`. The slug will be used as a unique tag for
the mining pool, for example in the public facing urls like https://mempool.space/graphs/mining/pools (here `btcpool` is the slug).

You can specify mining pool slugs in the `slugs` object in `pools-v2.json`. If you
don't specify one, we will automatically generate one [as such](https://github.com/mempool/mempool/blob/02820b0e6836c4202c2e346195e8aace357e3483/backend/src/api/pools-parser.ts#L106-L110).

```javascript
if (slug === undefined) {
  // Only keep alphanumerical
  slug = poolNames[i].replace(/[^a-z0-9]/gi, '').toLowerCase()
  logger.warn(`No slug found for '${poolNames[i]}', generating it => '${slug}'`)
}
```

### Add a new mining pool by `coinbase_tags`

You can add a new mining pool by specifying the coinbase tag they're using in
the coinbase transaction.

To add a new pool, you must add a new JSON object in the `coinbase_tags` object.
Note that you can add multiple tags for the same mining pool, but you _must_ use
the exact same values for `name` and `link` in each new entry.
For example:

```json
"Foundry USA Pool" : {
  "name" : "Foundry USA",
  "link" : "https://foundrydigital.com/"
},
"Foundry USA Pool another tag" : {
  "name" : "Foundry USA",
  "link" : "https://foundrydigital.com/"
},
```

Each coinbase tag will be use as a regex to match blocks with their mining pool.
This is how we use it in mempool application. You can see the code [here](https://github.com/mempool/mempool/blob/02820b0e6836c4202c2e346195e8aace357e3483/backend/src/api/blocks.ts#L238-L246).

```javascript
const regexes: string[] = JSON.parse(pools[i].regexes)
for (let y = 0; y < regexes.length; ++y) {
  const regex = new RegExp(regexes[y], 'i')
  const match = asciiScriptSig.match(regex)
  if (match !== null) {
    return pools[i]
  }
}
```

### Add a new mining pool by `payout_addresses`

You can add a new mining pool by specifying the receiving address they're using in
the coinbase transaction to receive the miner reward.

To add a new pool, you must add a new JSON object in the `payout_addresses` object.
Note that you can add multiple addresses for the same mining pool, but you _must_ use
the exact same values for `name` and `link` in each new entry.
For example:

```json
"1Hb7iC63bqxtt7X9oYr1VtDTE2Yk5xLQAU" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
"1Myy4QCu9zWESRHrVZBusN6g9bS5G7L5UK" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
```

Each address will be use to match blocks with their mining pool by matching the
coinbase transaction output address.
This is how we use it in mempool application. You can see the code [here](https://github.com/mempool/mempool/blob/02820b0e6836c4202c2e346195e8aace357e3483/backend/src/api/blocks.ts#L230-L236).

```javascript
const address = txMinerInfo.vout[0].scriptpubkey_address;
for (let i = 0; i < pools.length; ++i) {
  if (address !== undefined) {
    const addresses: string[] = JSON.parse(pools[i].addresses);
    if (addresses.indexOf(address) !== -1) {
      return pools[i];
    }
  }
```

## Change an existing mining pool metadata

You can also change an existing mining pool's name, link and slug. In order to
do so properly, you must update all existing entry in the `pools-v2.json` file.

For example, if you'd like to rename `Foundry USA` to `Foundry Pool`, you must replace
all occurences of the old string with the new one in `pools-v2.json` file, with no
exception, otherwise you'll end with two mining pools. The samme idea applies if
you want to change the link or the slug.

For example, to rename `Foundry USA` to `Foundry Pool` you'd need to update the
following (using today's `pools-v2.json` as reference):

```json
// Original
"Foundry USA Pool" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
  "/2cDw/" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
// Renamed
"Foundry USA Pool" : {
    "name" : "Foundry Pool",
    "link" : "https://foundrydigital.com/"
},
  "/2cDw/" : {
    "name" : "Foundry Pool",
    "link" : "https://foundrydigital.com/"
},
```

```json
// Original
"1Myy4QCu9zWESRHrVZBusN6g9bS5G7L5UK" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
"1Hb7iC63bqxtt7X9oYr1VtDTE2Yk5xLQAU" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
// Renamed
"1Myy4QCu9zWESRHrVZBusN6g9bS5G7L5UK" : {
    "name" : "Foundry Pool",
    "link" : "https://foundrydigital.com/"
},
"1Hb7iC63bqxtt7X9oYr1VtDTE2Yk5xLQAU" : {
    "name" : "Foundry Pool",
    "link" : "https://foundrydigital.com/"
},
```

```json
// Original
"Foundry USA": "foundryusa",
// Renamed - Be aware, this will also change the mining pool page link from
mempool.space/mining/pool/foundryusa to mempool.space/mining/pool/foundrypool
"Foundry Pool": "foundrypool",
```

## Block re-indexing

When a mining pool's coinbase tag or addresses is updated in `pools.jon`,
mempool can automatically re-index the appropriate blocks in order to re-assign
them to the correct mining pool.
"Appropriate" blocks here concern all blocks which are not yet assigned to a
mining pool (`unknown` pool), from block 0 (first known mining pool block)
as well as all blocks from the update mining pool.
You can find the re-indexing logic [here](https://github.com/mempool/mempool/blob/02820b0e6836c4202c2e346195e8aace357e3483/backend/src/api/pools-parser.ts#L224-L249)

You can enable/disable this behavior using by setting the following backend
configuration variable:

```
{
  "MEMPOOL": {
    "AUTOMATIC_BLOCK_REINDEXING": true
  }
}
```

# The Mempool Open Source Project® [![mempool](https://img.shields.io/endpoint?url=https://dashboard.cypress.io/badge/simple/ry4br7/master&style=flat-square)](https://dashboard.cypress.io/projects/ry4br7/runs)

https://user-images.githubusercontent.com/93150691/226236121-375ea64f-b4a1-4cc0-8fad-a6fb33226840.mp4

When the mempool backend starts, we automatically fetch the latest `pools-v2.json`
version from github. By default the url points to https://github.com/kimbaobao/mining-pools/blob/master/pools-v2.json but you can configure it to points to another repo by setting
the following backend variables:

```
{
  "MEMPOOL": {
    'POOLS_JSON_URL': 'https://raw.githubusercontent.com/kimbaobao/mining-pools/master/pools-v2.json',
    'POOLS_JSON_TREE_URL': 'https://api.github.com/repos/kimbaobao/mining-pools/git/trees/master'
  }
}
```
<br>

Mempool is the fully-featured mempool visualizer, explorer, and API service running at [mempool.space](https://mempool.space/). 

It is an open-source project developed and operated for the benefit of the Bitcoin community, with a focus on the emerging transaction fee market that is evolving Bitcoin into a multi-layer ecosystem.

# Installation Methods

Mempool can be self-hosted on a wide variety of your own hardware, ranging from a simple one-click installation on a Raspberry Pi full-node distro all the way to a robust production instance on a powerful FreeBSD server. 

Most people should use a <a href="#one-click-installation">one-click install method</a>.

Other install methods are meant for developers and others with experience managing servers. If you want support for your own production instance of Mempool, or if you'd like to have your own instance of Mempool run by the mempool.space team on their own global ISP infrastructure—check out <a href="https://mempool.space/enterprise" target="_blank">Mempool Enterprise®</a>.

<a id="one-click-installation"></a>
## One-Click Installation

Mempool can be conveniently installed on the following full-node distros: 
- [Umbrel](https://github.com/getumbrel/umbrel)
- [RaspiBlitz](https://github.com/rootzoll/raspiblitz)
- [RoninDojo](https://code.samourai.io/ronindojo/RoninDojo)
- [myNode](https://github.com/mynodebtc/mynode)
- [StartOS](https://github.com/Start9Labs/start-os)
- [nix-bitcoin](https://github.com/fort-nix/nix-bitcoin/blob/a1eacce6768ca4894f365af8f79be5bbd594e1c3/examples/configuration.nix#L129)

**We highly recommend you deploy your own Mempool instance this way.** No matter which option you pick, you'll be able to get your own fully-sovereign instance of Mempool up quickly without needing to fiddle with any settings.

## Advanced Installation Methods

Mempool can be installed in other ways too, but we only recommend doing so if you're a developer, have experience managing servers, or otherwise know what you're doing.

- See the [`docker/`](./docker/) directory for instructions on deploying Mempool with Docker.
- See the [`backend/`](./backend/) and [`frontend/`](./frontend/) directories for manual install instructions oriented for developers.
- See the [`production/`](./production/) directory for guidance on setting up a more serious Mempool instance designed for high performance at scale.
- 
