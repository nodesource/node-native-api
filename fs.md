## Class `File`

The `File` class allows operations on files by creating and returning a handle
to the given file. The handle only serves as a way to perform operations.
Creating the handle doesn't have any affect on the passed `path`.

**NOTE:** This spec does not contain the entire API that currently exists in
Node. Instead only those things that are new/critical/different.

#### `new File(path)`

* Returns `Object` file handle instance

Use the returned handle to operate on a file.

**Usage:**
```javascript
var handle = new File('/some/path/to/file');
```

**Reason:**
The reason the file path is passed to the `File()` constructor is because many
operations can then be taken from the returned handle. e.g. `stat()`,
`access()`, `open()`.


------

### Class Functions


#### `File.lsof()`

* Returns `Array` of file handles opened by the `File` module in this
  application instance

Useful for iterating over all file handles that have been opened by the `File`
module.

**Usage:**
```javascript
var openHandles = File.lsof();
// Loop through and print the status of each file handle.
for (var handle of openHandles)
  console.log(handle.path, handle.status());
```


#### `File.rx()`

* Returns `Number` of bytes read from the `File` module by the application
  since the start of the application

**Usage:**
```javascript
// Print number of MB read to disk by File
console.log(File.rx() / 0x100000);
```


#### `File.tx()`

* Returns `Number` of bytes written by the `File` module by the application
  since the start of the application

**Usage:**
```javascript
// Print number of MB written from disk by File
console.log(File.tx() / 0x100000);
```


------

### Instance Properties

#### `fh.path`

* Returns `String` of the path for the file handle


------

### Instance Methods

#### `fh.status()`

* Returns `String` of current file handle status

**NOTE:** The basic statuses would be `'open'`/`'closed'`, but others like
`'writing'`/`'reading'` could be helpful. Looking for feedback.


#### `fh.rx()`

* Returns `Number` of bytes read by the file handle.


#### `fh.tx()`

* Returns `Number` of bytes written to the file handle.


#### `fh.open(flags[, mode], callback[, ... vargs])`

* `flags` - `String`
* `mode` - `Number`, Default `0o666`
* `callback` - `Function`
* `vargs` - Variable number of `Value`s passed that will reach the callback
* Returns `Object` file handle instance

Asynchronous file open. See open(2).

`flags` can be:

* `'r'` - Open file for reading. An `Error` is passed to `callback` if file
  does not exist.
* `'r+'` - Open file for reading and writing. An `Error` is passed to
  `callback` if file does not exist.
* `'rs'` - Open file for reading in synchronous mode. Instructs the operating
  system to bypass the local file system cache. An `Error` is passed to
  `callback` if file does not exist.
  This is mainly useful for opening files on NFS mounts as it allows you to
  skip the potentially stale local cache. It has a very real impact on I/O
  performance.
* `'rs+'` - Open file synchronously for reading and writing. An `Error` is
  passed to `callback` if file does not exist. See notes on `'rs'` for usage.
* `'w'` - Open file for writing. The file is created if it does not exist or is
  truncated otherwise.
* `'w+'` - Open file for reading and writing. The file is created if it does
  not exist or is truncated otherwise.
* `'a'` - Open file for appending. File is created if it does not exist.
* `'ax'` - Open file for appending. An `Error` is passed to `callback` if file
  does not exist.
* `'a+'` - Open file for reading and appending. File is created if it does not
  exist.
* `'ax+'` - Open file for reading and appending. An `Error` is passed to
  `callback` if file does not exist.

`mode` sets the file mode if the file was created.

`callback` receives one argument. `this` context is the file handle `Object`
returned by `fh.open()`

**Usage:**
```javascript
var handle = new File('/tmp/test.out');
handle.open('r', function(err) {
  if (err) { /* handle error */ }

  // use now file handle
});
```


#### `fh.close(callback[, ... vargs])`

* `callback` - `Function`
* `vargs` - Variable number of `Value`s passed that will reach the callback
* Returns `Object` file handle instance

Asynchronous close(2).

`callback` receives one argument. `this` context is the file handle that has
been closed.

**Usage:**
```javascript
var handle = new File('/tmp/test.out');
handle.open('r', function(err) {
  if (err) { /* handle error */ }

  this.close(handleClosed);
});

function handleClosed(err) {
  if (err) { /* handle error */ }

  // file handle has now been closed
}
```


#### `fh.read(callback[, ... vargs])`

* `callback` - `Function`
* `vargs` - Variable number of `Value`s passed that will reach the callback
* Returns `Object file handle instance

Read the entire contents of file into a new `Buffer`.

**Usage:**
```javascript
var handle = new File('/tmp/test.out');
handle.open('r', function(err) {
  if (err) { /* handle error */ }
  this.read(doneReading);
});

function doneReading(err, bytesRead, buffer) {
  if (err) { /* handle error */ }
  this.close(handleClosed);
}

function handleClosed(err) {
  if (err) { /* handle closed */ }
}
```


#### `fh.read(buffer, offset, length[, position], callback[, ... vargs])`

* `buffer` - `Buffer`
* `offset` - `Number` from beginning of buffer to write into
* `length` - `Number` of bytes to read from file
* `position` - `Number` of bytes offset from beginning of file
* `callback` - `Function`
* `vargs` - Variable number of `Value`s passed that will reach the callback
* Returns `Object file handle instance

Read contents of file into supplied `buffer`.

**Usage:**
```javascript
var handle = new File('/tmp/test.out');
handle.open('r', function(err) {
  if (err) { /* handle error */ }
  var b = new Buffer(1024);
  this.read(b, 0, 1024, doneReading);
});

function doneReading(err, bytesRead, buffer) {
  if (err) { /* handle error */ }
  this.close(handleClosed);
}

function handleClosed(err) {
  if (err) { /* handle error */ }
}
```


#### `fh.write(buffer, offset, length[, position], callback[, ... vargs])`

* `buffer` - `Buffer` containing data to be written to handle
* `offset` - `Number` offset from beginning of buffer
* `length` - `Number` how many bytes to write
* `position` - `Number` where in file to write, Default: `0`
* `callback` - `Function` called when write is complete
* `vargs` - Variable number of `Value`s passed that will reach the callback
* Returns `Object` file handle instance

`callback` receives 3 arguments `(err, written, buffer)` where `written`
specifies how many bytes from `buffer` were written.

Note that it is unsafe to use `fh.write()` multiple times on the same file
without waiting for the callback.

On Linux, positional writes don't work when the file is opened in append mode.
The kernel ignores the position argument and always appends the data to the end
of the file.

**Usage:**
```javascript
var handle = new File('/tmp/test.out');
handle.open('r', function(err) {
  if (err) { /* handle error */ }
  var b = new Buffer(1024).fill('abc');
  this.write(b, 0, b.length, 0, doneWriting);
});

function doneWriting(err, written, buffer) {
  if (err) { /* handle error */ }
  this.close(handleClosed);
}

function handleClosed(err) {
  if (err) { /* handle error */ }
}
```


#### `fh.write(data[, position][, encoding], callback[, ... vargs])`

* `data` - `Value`
* `position` - `Number`
* `encoding` - `String` expected string encoding
* `callback` - `Function`
* `vargs` - Variable number of `Value`s passed that will reach the callback
* Returns `Object` file handle instance

If `data` is not a `Buffer` then it will be coerced to a `String`.

`position` refers to the offset from the beginning of the file where this data
should be written.

`callback` receives 3 arguments `(err, written, buffer)` where `written`
specifies how many bytes from `buffer` were written.

If only a partial string needs to be written then please use `substr()` to grab
the needed characters. Reason this is not handled in the API call is because
the substring and the number of bytes it takes may not be equal.

Note that it is unsafe to use `fh.write()` multiple times on the same file
without waiting for the callback.

On Linux, positional writes don't work when the file is opened in append mode.
The kernel ignores the position argument and always appends the data to the end
of the file.

**Usage:**
```javascript
var handle = new File('/tmp/test.out');
handle.open('r', function(err) {
  if (err) { /* handle error */ }
  this.write('hello world', 'latin1', doneWriting);
});

function doneWriting(err, written, buffer) {
  if (err) { /* handle error */ }
  this.close(handleClosed);
}

function handleClosed(err) {
  if (err) { /* handle error */ }
}
```

**Reason:**
The reason for explicitly having an API that would allow writing strings
instead of requiring all strings be first converted to a `Buffer` is for
performance.  When the `String` is written into memory resources can be saved
by not needing to attach that memory to an `Object`, then place the GC under
extra strain to clean it up afterwards.




------

### Misc Examples:

The following examples are to help bring new APIs under consideration, or to
help hammer out existing APIs.


Attempt to write a large file to disk, then cancel the write in progress if
it's taking more than a given time.
```javascript
var handle = new File('file');
handle.open('w', function(err) {
  if (err) { /* handle error */ }

  var b = new Buffer(1024 * 1024 * 500).fill('abc');  // 500 MB file
  // Queue up the buffer to be written to disk
  this.write(b, 0, b.length, finishedWriting);
});

// Close the file handle after 100ms regardless of whether the file
// has completed writing or not.
setTimeout(function(h) {
  if (!this.closed())
    h.close(fdClosed);
}, 100, handle);

function finishedWriting(err, length, buffer) {
  if (err) {
    // check "length" for amount of data written before error occurred.
    // close handle if still open
    if (!this.closed())
      this.close(fdClosed);
  }
}

function fdClosed(err) {
  if (err) { /* handle error */ }
}
```


Check the progress of writing a large file to disk.
```javascript
var handle = new File('file');
handle.open('w', function(err) {
  if (err) { /* handle error */ }

  var b = new Buffer(1024 * 1024 * 500).fill('abc');
  this.write(b, 0, b.length, finishedWriting);
});

setInterval(function(h) {
  // print progress of queue currently set to be written
  console.log(h.progress());
}, 1000, handle);

function finishedWriting(err, length, buffer) {
  if (err) { /* handle error */ }

  this.close(fdClosed);
}

function fdClosed(err) {
  if (err) { /* handle error */ }
}
```


Alternative to `.progress()` would be to simply return the number of bytes
remaining the queue to be written.
```javascript
var handle = new File('file');
handle.open('w', function(err) {
  if (err) { /* handle error */

  var b = new Buffer(1024 * 1024 * 500).fill('abc');
  this.write(b, 0, b.length, finishedWriting);
});

setInterval(function(h) {
  // print number of bytes remaining to be written
  console.log(h.pending());
}, 1000, handle);

function finishedWriting(err, length, buffer) {
  if (err) { /* handle error */ }

  this.close(fdClosed);
}

function fdClosed(err) {
  if (err) { /* handle error */ }
}
```
