---
title: DoS Vulnerability (fixed in Node v0.8.26 and v0.10.21)
blogAuthors: ['node-js-website']
category: 'vulnerabilities'
---

Node.js is vulnerable to a denial of service attack when a client
sends many pipelined HTTP requests on a single connection, and the
client does not read the responses from the connection.

We recommend that anyone using Node.js v0.8 or v0.10 to run HTTP
servers in production please update as soon as possible.

* v0.10.21 <http://blog.nodejs.org/2013/10/18/node-v0-10-21-stable/>
* v0.8.26 <http://blog.nodejs.org/2013/10/18/node-v0-8-26-maintenance/>

This is fixed in Node.js by pausing both the socket and the HTTP
parser whenever the downstream writable side of the socket is awaiting
a drain event. In the attack scenario, the socket will eventually
time out, and be destroyed by the server. If the "attacker" is not
malicious, but merely sends a lot of requests and reacts to them
slowly, then the throughput on that connection will be reduced to what
the client can handle.

There is no change to program semantics, and except in the
pathological cases described, no changes to behavior.

If upgrading is not possible, then putting an HTTP proxy in front of
the Node.js server can mitigate the vulnerability, but only if the
proxy parses HTTP and is not itself vulnerable to a pipeline flood
DoS.

For example, nginx will prevent the attack (since it closes
connections after 100 pipelined requests by default), but HAProxy in
raw TCP mode will not (since it proxies the TCP connection without
regard for HTTP semantics).

This addresses CVE-2013-4450.
