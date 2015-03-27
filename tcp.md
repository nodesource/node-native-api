## Class `TCP`

The `TCP` class allows creation of a TCP server.

**NOTE:** This spec does not contain the entire API that currently exists in
Node. Instead only those things that are new/critical/different.

#### `new TCP()`

* Returns `Object` TCP handle instance

**Usage:**
```javascript
var server = new TCP();
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


#### `server.close(callback[, ... vargs])`

* `callback` - `Function`
* Returns `Object` server instance


#### `server.onclose(callback[, ... vargs])`

* `callback` - `Function`
* Returns `Object` server instance

Call `callback` when the server has closed. If an error caused the server to
close will be passed as first argument to `callback`.


#### `server.onconnection(callback[, ... vargs])`

* `callback` - `Function`
* Returns `Object` server instance

Call `callback` when a new connection is received by the server. The connection
is the first argument of `callback`.
