## Streams

Currently this is in a phase for creating usage examples. The actual spec will
be created after enough examples have been accumulated. Everything here is up
for debate.


------

### Examples

Create a TCP server, pipe the data to a file and once the data has been queued
to be written to the file system echo it back out to the client. The stream
will automatically terminate after 10 MB have been written.
```javascript
let server = new TCP();

server.onconnection(function onConnection(c) {
  // Generate random id for log file for each new connection.
  let id = Math.random().toString(36).substr(2);
  let fd = new File(`/tmp/${id}.log`);

  // Open log file for the new connection. Pass connection as argument
  // to fdOpen() callback.
  fd.open('r+', fdOpen, c);
});


function fdOpen(err, c) {
  if (err) { /* handle error */ }

  // Now that the log file is open, proceed reading from connection.
  c.onreadable(cOnReadable);

  // First pipe daata to the file descriptor, then pipe data back to the
  // connection to act as an echo server. Reason for chaining the pipes is so
  // each will be alerted to issues with the other.  Also setup close event
  // handlers for the pipes specifically.
  c.pipe(this).onclose(fdOnPClose).onwrite(fdOnPWrite)
    .pipe(c).onclose(cOnPClose).onwrite(cOnPWrite);
}


function cOnReadable() {
  // Echo data back out
  while (this.status() === 'readable' &&
         this.tx() <= 0xa00000)
    this.write(this.read());

  // If more than 1MB has been written then close the connection. This will
  // automatically run the pipe's onend callback so the fd will know it's time
  // to cleanup, if desired.
  if (this.tx() > 0xa00000)
    this.close();
}


// Called automatically when the connection calls .end().
function fdOnPClose(err) {
  if (err) { /* handle error */ }
  this.close();
}


function fdOnPWrite(err, data) {
  if (err) { /* handle error */ }
}


function cOnPClose(err) {
  if (err) { /* handle error */ }
}


function cOnPWrite(err, data) {
  if (err) { /* handle error */ }
}
```


In this example `queued()` will be used to check how much data is waiting to be
written to a connection. Note, this means there is no need for a high water
mark for reading data. It simply depends on the user to check that the amount
of queued data is within their okay parameters before writing:
```javascript
let server = new TCP();

server.onconnection(function onConnection(c) {
  let fh = new File('/some/random/path');
  fh.onreadable(fhOnReadable, c);
  fn.onend(fhOnEnd, c);
  fn.open();
}).listen(8001);

function fhOnReadable(c) {
  while (this.status() === 'readable') {
    // Make sure no more than 10 MB is queued to be written to the
    // connection. If there is then wait until all the data has flushed
    // before writing more.
    if (c.queued() > 0xa00000)
      return c.onflush(cOnFlush, this);

    c.write(this.read());
  }
}

function cOnFlush(fh) {
  // Clear the flush callback. Only need this called if writing out
  // can't keep up with the data being read.
  this.onflush(null);
  // For simplicity, hand logical control back over to the other cb.
  fhOnReadable.call(fh, this);
}

// File has been completely read, so close the connection.
function fhOnEnd(c) {
  this.close();
  c.close();
}
```
**Note:** This does have repercussions for `pipe()`. If there is no high water
mark then pipe will just push data in as fast as it can, or will pipe wait for
a write to complete before writing more?


This example shows using the write request object returned by `write()` to set
a timeout on how long that write can be pending before it's canceled.
```javascript
let server = new TCP();

server.onconnection(function onConnection(c) {
  let writeReq = c.write('bye!');
  let timeout = setTimeout(cancelWrite, 1000, writeReq);
  // Set oncomplete after so timeout can be passed as an argument.
  writeReq.oncomplete(writeComplete, timeout);
}).listen(8001);

// The write hasn't completed in 1 sec, so cancel the write.
function cancelWrite(writeReq) {
  writeReq.cancel();
}

// Write has completed so we can clear the timeout.
function writeComplete(timeout) {
  clearTimeout(timeout);
}
```


In this example the bubbled `server.queued()` is used to check the amount of
data queued to be sent to all connections. If the amount queued is higher than
a specific value then pause the connections until things are sorted out.
Possibly `abort()` connections that have too many bytes pending.
```javascript
let server = new TCP();

// Setup the default onreadable() callback for all connections.
server.onreadable(cOnReadable);

// Now start listening for new connections.
server.listen(8001);

function cOnReadable() {
  // Shorthand the reference to the connection server.
  let server = this.server;

  while (this.status() === 'readable') {
    // If more than 100 MB of data are pending, fix it.
    if (server.queued() > 0x6400000) {
      // Loop through and abort any connections that are pending more than
      // 1 MB to be written.
      for (let handle of server.lsoc()) {
        if (handle.queued() > 0x100000)
          handle.abort();
      }
      // Check the amount of queued data again. If still high then simply
      // pause all connections until all connections have flushed or are
      // aborted. So the longest time lapse before the flush callback will
      // run is 1 sec (based on timeout set below).
      if (server.queued() > 0x6400000) {
        server.pause();
        server.onflush(serverOnFlush);
      }
    }

    let writeReq = this.write(this.read());
    // Make sure data is written within 1 sec.
    let timer = setTimeout(abortConnection, 1000, this);
    // Clear the abort timer if the write completes in time.
    writeReq.oncomplete(clearTimeout, timer);
  }
}

// All data to all connections has been flushed.
function serverOnFlush() {
  // Clear the flush callback.
  this.onflush(null);
  // Resume all connections.
  this.resume();
}

// Write took too long, so abort the connection.
function abortConnection(c) {
  c.abort(new Error('write request exceeded timeout'));
}
```
**Note:** The `queued()` callback must be defined by the implementing stream.
Because for non byte streams it's up to the implementor to determine the value
that `queued()` should return.


This example shows usage of the `onclose()` error argument to determine what's
happening with the stream, and recovery of the pipeline.
```javascript
// Open log file to which all streams will be aggregated and logged.
let fh = new File('random_log_file');
// A custom stream that is meant to aggregate many different streams into a
// single stream. This way many new TCP connections can all pipe to the same
// stream and have their contents logged correctly.
let sag = new StreamAggregation();
let server = new TCP();


// Passing as arguments for portability sake. Don't like depending on variable
// scoping for asynchronous calls.
fh.open('r+', function fhOnOpen(err, sag, server) {
  // There was an error opening the log file. Cannot continue from here.
  if (err)
    throw err;

  // Pipe the stream aggregator to the file.
  sag.pipe(this);

  // Now start accepting connections to the server.
  server.listen(8001);
}, sag, server);


// The "onclose()" event lets the user know that the stream is able to be
// closed. The input sources from piping results in ref counting to the piped
// stream. When all connections have closed and ref count == 0 this callback is
// called. Though it does _not_ mean the stream _will_ be closed, simply that
// the stream is preparing to be closed. It is possible to keep this stream
// alive. The only time it is not possible to keep the stream alive is if an
// Error is passed to the stream. In that case something critically bad
// happened and the stream must close so it can cleanup all its internal
// resources.
//
// Since this callback is only suggestive, there's an internal close() callback
// that needs to be set by the implementor that's run as the final step. This
// callback cleans up all resources, and it's function should essentially be
// invisible to the user.
// TODO(trevnorris): What exactly are the semantics for keeping the stream
// alive? There should be a call that simply states "keep this stream alive and
// don't call the onclose callback again until the ref counter increases then
// reaches zero again". Right now using .holdOpen(), but would love a better
// name.
sag.onclose(function sagOnClose(err, server) {
  // Critical error on the stream itself. Nothing to do with the incoming
  // streams. In this case we can pause the attached streams and attempt to
  // resurrect the stream.
  if (err) {
    // Prevent the server from receiving any new connections until the stream
    // error has been handled.
    server.pause();

    // Now Walk the list of handles that have been attached to this stream via
    // .pipe() and pause them.
    // TODO(trevnorris): Not married to .handles(). Was thinking it would
    // return .values() from the internal Set() of handles.
    for (let handle of this.handles())
      handle.readStop();

    // Recover the stream. This is implementation defined, and nothing
    // specifically to do with streams.
    this.recover(sagOnRecover, server);

    return;
  }

  // There was no error. All the existing connections simply closed, so keep
  // this stream alive and don't run the internal close cleanup.
  this.holdOpen();
}, server);


function sagOnRecover(err, server) {
  // If there was an error recovering the stream then bring down the server.
  // Something is seriously wrong.
  if (err)
    throw err;

  // Stream has recovered. Resume the server and resume reading from all the
  // connections.
  for (let handle of this.handles())
    handle.readStart();
  server.resume();
}


// Setup the default onclose callback for all new connections
server.onclose(function cOnClose(err) {
  if (err) {
    // Server had a critical error. Should probably bring down the process.
  }
});


// TODO(trevnorris): Should a default pipe for all new connections be able to
// be set via server.pipe(dest)?
server.onconnection(function onConnection(c, sag) {
  // Pipe the stream aggregator to the log file.
  c.pipe(sag);
}, sag);
```
