## Class `TCP`

The `TCP` class creates a TCP handle.

**NOTE:** This spec does not contain the entire API that currently exists in
Node. Instead only those things that are new/critical/different.

#### `new TCP()`

* Returns `Object` TCP handle instance

**Usage:**
```javascript
var server = new TCP();
```

Is also used to create a TCP connection out to a remote service.
```javascript
var client = new TCP();
client.connect(8080);
```


------

### Class Functions

#### `TCP.rx()`

* Returns `Number` of bytes received over all TCP connections since the start
  of the application.

**Usage:**
```javascript
// Print number of MB received from all connections
console.log(TCP.rx() / 0x100000);
```


#### `TCP.tx()`

* Returns `Number` of bytes written to all TCP connections since the start of
  the application.

**Usage:**
```javascript
// Print number of MB written to all connections
console.log(TCP.tx() / 0x100000);
```


#### `TCP.stat()`

* Returns `Array` of all open server handles.


------

### Instance Properties

#### `server.port`

* Returns port `Number` the server is running on, or null if not running.


#### `server.hostname`

* Returns `String` of host name.


#### `server.type`

* Returns `String` of type of server (e.g. `'ipv4'`)


------

### Instance Methods

#### `server.rx()`

* Returns `Number` of bytes read from all connections.


#### `server.tx()`

* Returns `Number` of bytes written to all connections.


#### `server.lsoc()`

* Returns `Array` of all open connections to the server.


#### `server.listen(port[, hostname][, backlog])`

* `port` - `Number`
* `hostname` - `String`, Default `'localhost'`
* `backlog` - `Number`, Default `511`
* Returns `Object` server instance

Begin listening on `port`. Consider this call immediate.


#### `server.close([callback[, ... vargs]])`

* `callback` - `Function`
* Returns `Object` server instance

Passing `callback` is a shorthand for setting `.onclose()`.


#### `server.onconnection(callback[, ... vargs])`

* `callback` - `Function`
* Returns `Object` server instance

Call `callback` when a new connection is received by the server. The connection
is the first argument of `callback`.


#### `server.onclose(callback[, ... vargs])`

* `callback` - `Function`
* Returns `Object` server instance

Call `callback` when the server has closed. If an error caused the server to
close will be passed as first argument to `callback`.


#### `server.onreadable(callback[, ... vargs])`

* `callback` - `Function`
* Returns `undefined`

Sets the default `onreadable()` callback for all new connections.

**Usage:**
```javascript
var server = new TCP();

server.onreadable(function onReadable() {
  // Echo data back to client
  while (this.status() === 'readable') {
    this.write(this.read());
  }
});

server.onconnection(function onConnection(c) {
  // Nothing much to do here.
});

server.listen(8080);
```

**Reason:**
By setting a default callback for new instances of a connection construction
can be made more performant. Specifically if the `callback` is made to be a
`Persistent<Function>` on the class instance of the connection then a trip
to C++ and a call to `Persistent::New()` can be saved in the process.


#### `server.onend(callback[, ... vargs])`

* `callback` - `Function`
* Returns `undefined`

Set the default `onend()` callback for all new connections.

**Usage:**
```javascript
var server = new TCP();

server.onend(function onEnd(err) {
  if (err) { /* handle error */ }
});

server.onconnection(function onConnection(c) {
  c.end('bye!');
});

server.listen(8080);
```

**Reason:**
By setting a default callback for new instances of a connection construction
can be made more performant. Specifically if the `callback` is made to be a
`Persistent<Function>` on the class instance of the connection then a trip
to C++ and a call to `Persistent::New()` can be saved in the process.


------

TCP Client

#### `client.flush()`

* Returns client `Object` handle instance

All `write()` are automatically queued internally and then written at the end
of the synchronous executable block. This takes advantage of `writev` where
available, and can at least help minimize the number of syscalls where it's
not. If you want the data to be written immediately call `flush()` on the
connection.

**Usage:**
```javascript
server.onconnection(function onConnection(c) {
  c.write('hello ');
  // Now write it out immediately
  c.flush();
  // And continue writing more
  c.write('world!');
});
```
