# Common Concepts

## NodeStat

Some queries return `NodeStat` structure as a part of result. Naturally, this `NodeStat` is not-null only when a requested node exists.
Most notably, `NodeStat` contains creation time of node, versions of its data and children, session id of ephemeral node owner and so on. 
For details please refer to XML documentation in code.

## Node Data and Version

Any node can contain a small amount of binary data. This amount can not exceed 1024*1023 bytes.
Every request to set data would increase node version which can be found in `NodeStat.Version`. Re-creation of a node resets its version.

Many operations support specifying this version along with the request. If actual node version does not match with one in request, such request will be declined by server.
Of course, checking of a version and executing of an actual operation is atomic.
