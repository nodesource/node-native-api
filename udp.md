## Class `UDP`

The `UDP` class creates a UDP handle.

#### `new UDP([type])`

* `type` - `String` of handle type, Default `'udp4'`
* Returns new `Object` UDP handle instance

**Usage:**
```javascript
let handle = new UDP();
```


------

### Class Functions

#### `UDP.rx()`

* Returns `Number` of bytes received over all UDP connections since the start
  of the application.

**Usage:**
```javascript
// Print number of MB received from all connections
console.log(UDP.rx() / 0x100000);
```


#### `UDP.tx()`

* Returns `Number` of bytes written to all UDP connections since the start of
  the application.

**Usage:**
```javascript
// Print number of MB written to all connections
console.log(UDP.tx() / 0x100000);
```


#### `UDP.stat()`

* Returns `Array` of all open handles.

**Usage:**
```javascript
let list = UDP.stat();

for (let handle of list)
  // Print how many bytes each connection has received
  console.log(handle.rx());
```


------

### Instance Properties

#### `handle.port`

* Returns port `Number`.


#### `handle.address`

* Returns `String` of `address` passed to `.bind()`.


#### `handle.type`

* Returns `String` of handle type (e.g. `'udp4'`)


------

### Instance Methods

#### `handle.bind(port[, address][, exclusive][, callback[, ... vargs]])`

* `port` - `Number`
* `address` - `String`, Default `'localhost'`
* `exclusive` - `Boolean`, Default `false`
* `callback` - `Function`
* Returns UDP handle `Object`

Bind the handle to a `port` and optional `address`. If `address` is not
specified the OS will try to listen on all addresses. `callback` is used to
receive any errors that may have occurred while binding.

**Usage:**
```javascript
let handle = new UDP('udp4');

handle.bind(8080, function onBind(err) {
  if (err) { /* handle error */ }
  console.log(`Listening on ${this.address}:${this.port}`);
});
```


#### `handle.close([callback[, ... vargs]])`

* `callback` - `Function`
* Returns UDP handle `Object` instance

Stop listening for new data and close the underlying socket. Passing a
`callback` is just short hand for setting the `onclose()` callback and will
override any callback passed to `onclose()`.


#### `handle.onclose(callback[, ... vargs])`

* `callback` - `Function`
* Returns UDP handle `Object` instance

`callback` will receive an `Error` as the first argument if something went
awry.

**Usage:**
```javascript
let handle = new UDP('udp6');

handle.onclose(function onClose(err) {
  if (err) { /* handle error */ }
});

handle.bind(8080);
```


#### `handle.write(data, offset, length, port[, address][, callback[, ... vargs]])`

* `data` - `Buffer` or `String`
* `offset` - `Number`
* `length` - `Number`
* `port` - `Number`
* `address` - `String`
* `callback` - `Function`
* Returns handle `Object` request

For UDP sockets, the destination `port` and `address` must be specified. A
hostname may be supplied for the `address` parameter, and it will be resolved
with DNS.

If the address is omitted or is an empty string, `'0.0.0.0'` or `'::0'` is used
instead. Depending on the network configuration, those defaults may or may not
work; it's best to be explicit about the destination address.

If the socket has not been previously bound with a call to bind, it gets
assigned a random port number and is bound to the "all interfaces" address
(`'0.0.0.0'` for udp4 sockets, `'::0'` for udp6 sockets.)

An optional callback may be specified to detect DNS errors or for determining
when it's safe to reuse the buf object. Note that DNS lookups delay the time to
send for at least one tick. The only way to know for sure that the datagram has
been sent is by using a callback.

With consideration for multi-byte characters, `offset` and `length` will be
calculated with respect to byte length and not the character position.

**Usage:**
```javascript
let handle = new UDP('udp4');
let msg = new Buffer('hello world!');
handle.write(msg, 0, msg.length, 41234, function onWrite(err) {
  if (err) { /* handle error */ }
  this.close();
});
```

**Notes:**
The maximum size of an IPv4/v6 datagram depends on the MTU (_Maximum
Transmission Unit_) and on the Payload Length field size.

* The Payload Length field is `16 bits` wide, which means that a normal payload
  cannot be larger than 64K octets including internet header and data (65,507
  bytes = 65,535 − 8 bytes UDP header − 20 bytes IP header); this is generally
  true for loopback interfaces, but such long datagrams are impractical for
  most hosts and networks.

* The `MTU` is the largest size a given link layer technology can support for
  datagrams. For any link, `IPv4` mandates a minimum `MTU` of 68 octets, while
  the recommended `MTU` for `IPv4` is 576 (typically recommended as the `MTU`
  for dial-up type applications), whether they arrive whole or in fragments.

  For IPv6, the minimum `MTU` is 1280 octets, however, the mandatory minimum
  fragment reassembly buffer size is 1500 octets. The value of 68 octets is
  very small, since most current link layer technologies have a minimum `MTU`
  of 1500 (like Ethernet).

Note that it's impossible to know in advance the `MTU` of each link through
which a packet might travel, and that generally sending a datagram greater than
the (receiver) `MTU` won't work (the packet gets silently dropped, without
informing the source that the data did not reach its intended recipient).


#### `handle.setBroadcast(flag)`

* `flag` - `Boolean`
* Returns UDP handle `Object` instance

Sets or clears the `SO_BROADCAST` socket option. When this option is set, UDP
packets may be sent to a local interface's broadcast address.


#### `handle.setTTL(ttl)`

* `ttl` - `Number`, between `1` and `255`
* Returns UDP handle `Object` instance

Sets the `IP_TTL` socket option. TTL stands for "Time to Live," but in this
context it specifies the number of IP hops that a packet is allowed to go
through. Each router or gateway that forwards a packet decrements the TTL. If
the TTL is decremented to 0 by a router, it will not be forwarded. Changing TTL
values is typically done for network probes or when multicasting.

The argument to `setTTL()` is a number of hops between 1 and 255. The default
on most systems is 64.


#### `handle.setMulticastTTL(ttl)`

* `ttl` - `Number`, between `1` and `255`
* Returns UDP handle `Object` instance

Sets the `IP_MULTICAST_TTL` socket option. TTL stands for "Time to Live," but
in this context it specifies the number of IP hops that a packet is allowed to
go through, specifically for multicast traffic. Each router or gateway that
forwards a packet decrements the TTL. If the TTL is decremented to 0 by a
router, it will not be forwarded.

The argument to `setMulticastTTL()` is a number of hops between 0 and 255. The
default on most systems is 1.


#### `handle.setMulticastLoopback(flag)`

* `flag` - `Boolean`
* Returns UDP handle `Object` instance

Sets or clears the `IP_MULTICAST_LOOP` socket option. When this option is set,
multicast packets will also be received on the local interface.


#### `handle.addMembership(multicastAddress[, multicastInterface])`

* `multicastAddress` - `String`
* `multicastInterface` - `String`
* Returns UDP handle `Object` instance

Tells the kernel to join a multicast group with `IP_ADD_MEMBERSHIP` socket
option.

If `multicastInterface` is not specified, the OS will try to add membership to
all valid interfaces.


#### `handle.dropMembership(multicastAddress[, multicastInterface])`

* `multicastAddress` - `String`
* `multicastInterface` - `String`
* Returns UDP handle `Object` instance

Tells the kernel to leave a multicast group with `IP_DROP_MEMBERSHIP` socket
option. This is automatically called by the kernel when the socket is closed or
process terminates, so most apps will never need to call this.

If `multicastInterface` is not specified, the OS will try to drop membership to
all valid interfaces.


#### `handle.unref()`

* Returns UDP handle `Object` instance

Calling `unref()` on a socket will allow the program to exit if this is the
only active socket in the event system. If the socket is already unrefd calling
`unref()` again will have no effect.


#### `handle.ref()`

* Returns UDP handle `Object` instance

Calling ref on a previously unrefd socket will not let the program exit if it's
the only socket left (the default behavior). If the socket is refd calling ref
again will have no effect.
