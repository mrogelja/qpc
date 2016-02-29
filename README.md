# qpc

RPC library based on AMQP protocol.

[![Dependency Status](https://david-dm.org/rpcjs/qpc/status.svg?style=flat)](https://david-dm.org/rpcjs/qpc)
[![Build Status](https://travis-ci.org/rpcjs/qpc.svg?branch=master)](https://travis-ci.org/rpcjs/qpc)
[![npm version](https://badge.fury.io/js/qpc.svg)](http://badge.fury.io/js/qpc)
[![Coverage Status](https://coveralls.io/repos/github/rpcjs/qpc/badge.svg?branch=master)](https://coveralls.io/github/rpcjs/qpc?branch=master)


## Install RabbitMQ

```
apt-get install rabbitmq-server
```


## Install library

```
npm install qpc
```


## round-robin

Example: Call remote function.
Run multiple servers.js for round-robin shared.


### `server.js` example

```js
const qpc = require('qpc')

qpc.consumer({
  uri: 'amqp://guest:guest@localhost:5672',
})
.then(consumer => {
  consumer.on('inc', function (param, cb) {
    let prevVal = param
    let nextVal = param + 2
    cb(++param, prevVal, nextVal)
  })

  consumer.on('say.*', function (param, cb, inf) {
    let arr = inf.cmd.split('.')

    let name = (param && param.name) ? param.name : 'world'

    cb(arr[1] + ' ' + name + '!')
  })

  consumer.on('withoutCB', function (param, cb, inf) {
    if (cb) {
      cb('please run function without cb parameter')
      return
    }

    console.log('this is function withoutCB')
  })
})
```


### `client.js` example

```js
const qpc = require('qpc')

qpc.publisher({
  uri: 'amqp://guest:guest@localhost:5672',
})
.then(publisher => {
  publisher.call('inc', 5, function () {
    console.log('results of inc:', arguments)  //output: [6,4,7]
  })

  publisher.call('say.Hello', { name: 'John' }, function (msg) {
    console.log('results of say.Hello:', msg)  //output: Hello John!
  })

  publisher.call('withoutCB', {}, function (msg) {
    console.log('withoutCB results:', msg)
    //output: please run function without cb parameter
  })

  publisher.call('withoutCB', {}) //output message on server side console
})
```


## License

MIT © [Zoltan Kochan](https://www.kochan.io)
