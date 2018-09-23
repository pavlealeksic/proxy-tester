# proxy-tester

### Reliable proxy testing
##### This project is a fork from [check-proxy](https://www.npmjs.com/package/check-proxy) (un-maintained?)

Library consists of a **server**, and **client**. Server runs on a known IP address and client attempts to connect to server through the proxy under test. 

What it does:
 * Tests http, socks4 and socks5 proxies
 * Tests GET, POST, COOKIES, `referer` support
 * Tests https support
 * Provides geoIP result on proxy
 * Tests proxy speed - provides total time and connect time
 * Tests anonymity (binary checks - anonymous or not, 1 - anonymous, i.e. doesn't leak your IP address in any of the headers, 0 - not anonymous)
 * Allows custom connectivity tests to specific websites - by custom function, regex, or substring search

## Installation

````javascript
  npm i proxy-tester --save
````
## Server Setup:
#### server.js
````javascript
const express = require('express'),
  app = express(),
  url = require('url'),
  bodyParser = require('body-parser'),
  cookieParser = require('cookie-parser'),
  getProxyType = require('check-proxy').ping,
  ipAddress = process.env.MASTER_IP || '10.0.10.150',
  port = process.env.PORT || 8686;

app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
app.use(cookieParser());
app.get('/', ping);
app.post('/', ping);
app.listen(port, ipAddress, function() {
  console.log('%s:\nNode server started on %s:%d ...', Date(Date.now() ), ipAddress, port);
});

function ping(req, res) {
  console.log('ip', req.connection.remoteAddress);
  // console.log('headers', req.headers);
	// console.log('cookies', req.cookies);
  res.json(getProxyType(req.headers, req.query, req.body, req.cookies));
}
````
## Client Setup
#### client.js
````javascript
const checkProxy = require('proxy-tester').check;
checkProxy({ // Returns a Promise
  testHost: 'ping.mydomain.com', // put your ping server url here
  proxyIP: '107.151.152.218', // proxy ip to test
  proxyPort: 80, // proxy port to test
  localIP: '10.0.10.10', // local machine IP address to test
  connectTimeout: 6, // curl connect timeout, sec
  timeout: 10, // curl timeout, sec
  websites: [{
    name: 'google',
    url: 'https://www.google.com/',
    regex: function(html) { // expected result - custom function
      return html && html.indexOf('google') != -1;
  },{
    name: 'amazon',
    url: 'https://www.amazon.com/',
    regex: 'Amazon', // expected result - look for this string in the output
  },{
    name: 'ebay',
    url: 'https://www.ebay.com/',
    regex: /ebay/ig, // expected result - regex.test(result)
  }]
})
.then(function(res) {
	console.log('final result', res);
})
.catch(e => {
  console.log(e);
});
````

## Result (in a Promise):
```
[{
  get: true,
  post: true,
  cookies: true,
  referer: true,
  'user-agent': true,
  anonymityLevel: 1,
  supportsHttps: true,
  protocol: 'http',
  ip: '107.151.152.218',
  port: 80,
  country: 'MX',
  connectTime: 0.23, // Time in seconds it took to establish the connection
  totalTime: 1.1, // Total transaction time in seconds for last the transfer
  websites: {
    google: {
      "responseCode": 200,
      "connectTime": 0.648131, // seconds
      "totalTime": 0.890804, // seconds
      "receivedLength": 1270, // bytes
      "averageSpeed": 1425 // bytes per second
    },
    amazon: false,
    ebay: {
      "responseCode": 200,
      "connectTime": 0.648131, // seconds
      "totalTime": 0.890804, // seconds
      "receivedLength": 1270, // bytes
      "averageSpeed": 1425 // bytes per second
    }
  }
}]
```

## Tests
```
mocha
```