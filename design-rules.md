## Design Rules

This document outlines the rules of the overarching design behind the API. Any
API decision anywhere else should first consult this document. This document
should be expanded as the other APIs do.

All APIs should have a similar feel. Basically it should be intuitive to pick
up an API of a subsystem not used if another is already known.


------

### Class Structure

**Class functions** don't require an instance to run. Shouldn't have special
cases where the instance is passed as an argument. Also meant to be a
generalized call that can address all instances (e.g. `FS.rx()`).

**Instance properties** should be used with values that are set set once (e.g.
retrieving the port a server is running on) or when user operation explicitly
changes the property (e.g. popping an item off an array).

**Instance methods** also used as "getters" for values that can be changed
internally (e.g. the "state" of a handle).

It is also possible to set a default callback for new instances. Reason for
this is mainly for performance. For example, when an incoming connection is
handled the callbacks set on the object either have to be made `Persistent`, or
have to go through a property lookup from C++. Both those are slow. By setting
a default this can be bypassed.


------

### Callback Usage

Callbacks use the "error back" style of callback. As many errors are passed to
the callback as possible.

All callbacks are called asynchronously. If an error is immediately detected
then the error is immediately created, stored and set to be called
asynchronously.

The calling context (e.g. `this`) of all callbacks should be the same as the
handle making the call.

Depending on variable scope in outer function closures is prohibited. If a
variable needs to reach a callback then it should be passed as an argument.


------

### Events

There is no fancy event emitter. Instead all methods use similar `.on*(` syntax
to which a single callback can be passed. An abstraction layer on top of that
can be made to handle multiple callbacks for a single event.

The event callback can receive any number of arguments. This is greatly
simplified by there being no multi-event handler. For example:
```javascript
var e = new Event();
e.onevent(function(err) {
  // handle the event
});

//// internally ////
// can pass N arguments to the callback
this.event_cb(err);
```


------

### Exceptions

Throwing should be considered last resort. It should truly mean that there's an
unrecoverable issue and the application needs to be brought down. When the API
has no idea what to do with user's code. For example, when expecting a callback
but none is passed there is then no place for any other errors to be passed.
But say another argument is incorrect, that error should be created immediately
then passed to the callback.


------

### Misc

Items that don't yet have any particular place to go.
