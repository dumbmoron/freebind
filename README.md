# freebind.js
[npm](https://www.npmjs.com/package/freebind)

inspired by [freebind](https://github.com/blechschmidt/freebind) (the C project), this library allows you to bind sockets to random IP addresses from specified prefixes. freebind.js is only supported on Linux -- it **won't** work on other platforms (since they don't have a FREEBIND socket option).

note that it is still **very experimental**, potential bug reports and pull requests are welcome.

## usage
freebind supports creating TCP sockets, and also provides a dispatcher wrapper for use with `undici` (aka native node >=18 fetch)

### fetch
```js
import { randomDispatcher } from './index.js'

const dispatcher = randomDispatcher('fc00:dead:beef::/48')

const example1 = async () => console.log(
    await fetch(
        'https://icanhazip.com',
        { dispatcher }
    ).then(a => a.text())
)

// connections on IPs can be held open
// unless you explicitly close the connection
await example1(); // ~> fc00:dead:beef:4465:f5e0:26d4:2921:6891
await example1(); // ~> fc00:dead:beef:4465:f5e0:26d4:2921:6891
await example1(); // ~> fc00:dead:beef:4465:f5e0:26d4:2921:6891
await example1(); // ~> fc00:dead:beef:4465:f5e0:26d4:2921:6891

const example2 = async () => console.log(
    await fetch(
        'https://icanhazip.com',
        { dispatcher,
          headers: { connection: 'close' }
        }
    ).then(a => a.text())
)
await example2(); // ~> fc00:dead:beef:7d1e:b71e:1c77:1cc6:8a07
await example2(); // ~> fc00:dead:beef:31cd:a6e7:be0b:6872:b51a
await example2(); // ~> fc00:dead:beef:8f8e:c97f:2ba2:3a6:9d7b
await example2(); // ~> fc00:dead:beef:3a48:bb50:d193:471c:f553

```

### tcp
```js
import { tcp } from './index.js'

// a random address will be bound to this socket
const sock = await tcp.createRandomSocket('::1', 9999, 'fc00:dead:beef::/48')
// returns a net.Socket: https://nodejs.org/api/net.html#class-netsocket

// you can also specify a specific address
const sock2 = await tcp.createSocket('hostname.com.example', 9999, 'fc00:dead:beef::b00b')

// you can also specify how many bits to
// randomize (the remainder is zeroed out)
const sock3 = await tcp.createRandomSocket('::1', 9999, 'fc00:dead:beef::/48', undefined, 8)
// (starting from most significant bits, e.g.
// the could be bound to IPs like:
// fc00:dead:beef:2c00::, fc00:dead:beef:3b00::
// fc00:dead:beef:f500::, and so on
```