## Streams

Currently this is in a phase for creating usage examples. The actual spec will
be created after enough examples have been accumulated. Everything here is up
for debate.


------

### Examples

Create a TCP server, pipe the data to a file and once the data has been queued
to be written to the file system echo it back out to the client. The stream
will automatically terminate after 10MB have been written.
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
