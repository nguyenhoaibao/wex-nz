wex-nz
=====

An node.js client for the [wex.nz trade api](https://wex.nz/api/3/docs)

## Fort from (https://github.com/pskupinski/node-btc-e)

## Thanks Preston Skupinski <preston.skupinski@gmail.com> about (Preston Skupinski <preston.skupinski@gmail.com>)

## Installation

wex-nz is available as `wex-nz` on npm.

```
npm install wex-nz
```

## Usage

```javascript
var WEXNZ = require('wex-nz'),
    wexnzTrade = new WEXNZ("YourApiKey", "YourSecret"),
    // No need to provide keys if you're only using the public api methods.
    wexnzPublic = new WEXNZ();

// Public API method call.
// Note: Could use "wexnzTrade" here as well.
wexnzPublic.ticker("ltc_btc", function(err, data) {
  console.log(err, data);
});

// Trade API method call.
wexnzTrade.getInfo(function(err, info) {
  console.log(err, info);
});
```

## Options

The constructor supports an optional third parameter for passing in various options to override defaults, which can either be a hash of the overrides or a nonce generation function if that is the only override required.

When passed as a hash, the following options are supported:
* agent - The HTTPS agent to use when making requests
* timeout - The timeout to use when making requests, defaults to 5 seconds
* nonce - A nonce generation function ([Custom nonce generation](#custom-nonce-generation))
* tapi_url - The base url to use when making trade api requests, defaults to `https://wex.nz/tapi`
* public_url - The base url to use when making public api requests, defaults to `https://wex.nz/api/3/`
* strict_ssl - `true` by default, but can be set to `false` if desired, such as if wex.nz has problems with their SSL certificate again.

```javascript
var WEXNZ = require('wex-nz'),
    HttpsAgent = require('agentkeepalive').HttpsAgent,
    wexnzTrade = new WEXNZ("YourApiKey", "YourSecret", {
      agent: new HttpsAgent()
    });
```

## Custom nonce generation

By default the module generates a nonce based on the current timestamp in seconds(can't use anything smaller than seconds since wex.nz is capped at 4294967294 for nonces) as a means of providing a consistently increasing number, but for traders who want to possibly get in more than one trade api request per second per api key there is a way to do so by providing a nonce generation function as the `nonce` option in an options hash provided as the third parameter to the constructor.  Please don't abuse the service wex.nz is providing though.

wex.nz expects every nonce given to be greater than the previous one for each api key you have, this presents a big problem when trying to do multiple async calls with the same api key since there is no guarantee that the first api call will be processed before the second one and so on.  Chaining calls synchronously(take a look at promises with [q.js](https://github.com/kriskowal/q) for help with that) or using multiple clients, each with their own API key are the only way around that problem.

```javascript
var WEXNZ = require('wex-nz'),
    fs = require('fs'),
    currentNonce = fs.existsSync("nonce.json") ? JSON.parse(fs.readFileSync("nonce.json")) : 0,
    // Provide a nonce generation function as the third parameter if desired.
    // The function must provide a number that is larger than the one before and must not
    // be larger than the 32-bit unsigned integer max value of 4294967294.
    wexnz = new WEXNZ("YourApiKey", "YourSecret", {
      nonce: function() {
        return ++currentNonce;
      }
    });

process.on('exit', function(code){
  fs.writeFileSync("nonce.json", currentNonce);
  process.exit();
});
process.on('SIGINT', process.exit);

wexnz.getInfo(function(err, info) {
  console.log(err, info);
});
```

## License

This module is [ISC licensed](https://github.com/tannvdts/wex-nz/blob/master/LICENSE.txt).
