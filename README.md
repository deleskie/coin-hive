# Coin-Hive [![Build Status](https://travis-ci.org/cazala/coin-hive.svg?branch=master)](https://travis-ci.org/cazala/coin-hive)

Mine cryptocurrency [Monero (XMR)](https://getmonero.org/) using [Coin-Hive](https://coin-hive.com/) from node.js

## Disclaimer

This project has nothing to do with `coin-hive.com`

## Install

```
npm install -g coin-hive
```

## Usage

```js
const CoinHive = require('coin-hive');
(async () => {

  // Options are not mandatory, defaults values:
  const options = {
    interval: 1000, // interval for "update"
    port: 3002, // puppeteer port
    host: 'localhost', // puppeteer host,
    threads: -1 // number of threads to start with, defaults to navigator.hardwareConcurrency see https://coin-hive.com/documentation/miner#constructor-options
  }

  // Create miner
  const miner = await CoinHive('ZM4gjqQ0jh0jbZ3tZDByOXAjyotDbo00', options); // Coin-Hive's Site Key

  // Start miner
  await miner.start();

  // Listen on events
  miner.on('found', () => console.log('Found!'))
  miner.on('accepted', () => console.log('Accepted!'))
  miner.on('update', data => console.log(`
    Hashes per second: ${data.hashesPerSecond}
    Total hashes: ${data.totalHashes}
    Accepted hashes: ${data.acceptedHashes}
  `));

  // Stop miner
  setTimeout(async () => await miner.stop(), 60000);
})();
```

## CLI

```
Usage: coin-hive <site-key>

<site-key>: You CoinHive Site Key

Options:

  --interval  Interval between updates (logs)
  --port      Port for the miner server
  --host      Host for the miner server
  --threads   Number of threads for the miner
```

## API

- `CoinHive(siteKey)`: Returns a promise of a `Miner` instance. It requires a [Coin-Hive Site Key](https://coin-hive.com/settings/sites).

- `miner.start()`: Connect to the pool and start mining. Returns a promise that will resolve once the miner is started.

- `miner.stop()`: Stop mining and disconnect from the pool. Returns a promise that will resolve once the miner is stopped.

- `miner.kill()`: Stop mining, disconnect from the pool, shutdown the server and close the headless browser. Returns a promise that will resolve once the miner is dead.

- `miner.on(event, callback)`: Specify a callback for an event. The event types are:

  - `update`: Informs `hashesPerSecond`, `totalHashes` and `acceptedHashes`.

  - `open`:	The connection to our mining pool was opened. Usually happens shortly after miner.start() was called.

  - `authed`:	The miner successfully authed with the mining pool and the siteKey was verified. Usually happens right after open.

  - `close`:	The connection to the pool was closed. Usually happens when miner.stop() was called.

  - `error`:	An error occured. In case of a connection error, the miner will automatically try to reconnect to the pool.

  - `job`:	A new mining job was received from the pool.

  - `found`:	A hash meeting the pool's difficulty (currently 256) was found and will be send to the pool.

  - `accepted`:	A hash that was sent to the pool was accepted.

- `miner.rpc(methodName, argsArray)`: This method allows you to interact with the Coin-Hive miner instance. It returns a Promise that resolves the the value of the remote method that was called. The miner intance API can be [found here](https://coin-hive.com/documentation/miner#miner-is-running). Here's an example:

```js
var miner = await CoinHive('SITE_KEY');
await miner.rpc('isRunning'); // false
await miner.start();
await miner.rpc('isRunning'); // true
await miner.rpc('getThrottle'); // 0
await miner.rpc('setThrottle', [0.5]);
await miner.rpc('getThrottle'); // 0.5
```

## ENVIRONMENT VARIABLES

All the following environment variables can be used to configure the miner from the outside:

- `COINHIVE_SITE_KEY`: Coin-Hive's Site Key

- `COINHIVE_INTERVAL`: The interval on which the miner reports an update

- `COINHIVE_PORT`: The port that will be used to launch the server, and where puppeteer will point to

- `COINHIVE_HOST`: The host that will be used to launch the server, and where puppeteer will point to

- `COINHIVE_PUPPETEER_URL`: In case you don't want to point puppeteer to the local server, you can use this to make it point somewhere else where the miner is served (ie: `PUPPETEER_URL=http://coin-hive.herokuapp.com`)

- `COINHIVE_THREADS`: Number of threads

## Requisites

+ Node v8+
