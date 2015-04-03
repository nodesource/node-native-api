## LL-API Interface

The purpose of these documents is to provide a uniform API across Node. It is
meant to be more low-level than the current API, has been tuned for performance
and flexibility. This API has the following goals:

1) Be so light weight that it offers no observable impact on users' code.

2) Be flexible enough that any other desired interface could be written using
   it.

3) Be resilient and allow the developer to recover.


### Documents

Most of the documents address API, but there are a couple that explain
generalities.

* `README.md` - You're reading it.
* `design-rules.md` - List of general rules for creating new APIs. It is to
  help guide the design process so all APIs can feel as uniform as possible.
* `fs.md` - Some revised API for the file system module.
* `tcp.md` - A TCP API, instead of unified `'net'` module.
* `udp.md` - A UDP API
* `stream.md` - Some streams examples that will be used to write the spec.
