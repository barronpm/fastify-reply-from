# fastify-reply-from

fastify plugin to forward the current http request to another server.

## Install

```
npm i fastify-reply-from
```

## Usagej

The following example set up two fastify servers and forward the request
from one to the other.

```
'use strict'

const Fastify = require('fastify')

const target = Fastify({
  logger: true
})

target.get('/', (request, reply) => {
  reply.send('hello world')
})

const proxy = Fastify({
  logger: true
})

proxy.register(require('fastify-reply-from'), {
  base: 'http://localhost:3001/'
})

proxy.get('/', (request, reply) => {
  reply.from('/')
})

target.listen(3001, (err) => {
  if (err) {
    throw err
  }

  proxy.listen(3000, (err) => {
    if (err) {
      throw err
    }
  })
})
```

### plugin options

#### `base`

Set the base URL for all the forwarded requests.

#### cacheURLs

The number of parsed URLs that will be cached. Default: 100.

#### keepAliveMsecs

Defaults to 1 minute, passed down to [`http.Agent`][http-agent] and
[`https.Agent`][https-agent] instances.

#### maxSockets

Defaults to 2048 sockets, passed down to [`http.Agent`][http-agent] and
[`https.Agent`][https-agent] instances.

#### rejectUnauthorized

Defaults to `true`, passed down to [`https.Agent`][https-agent] instances.
This needs to be set to `false` to reply from https servers with
self-signed certificates.

### reply.from(source, [opts])

The plugin decores the
[`Reply`](https://github.com/fastify/fastify/blob/master/docs/Reply.md)
instance with a `from` method, which will reply to the original request
__from the desired source__. The options allows to override any part of
the request or response being sent or received to/from the source.

#### onResponse(res)

Called when an http response is received from the source.
The default behavior is `reply.send(res)`, which will be disabled if the
option is specified.

#### rewriteHeaders(res)

Called to rewrite the headers of the response, before them being copied
over to the outer response.
It must return the new headers object.

#### queryString

Replaces the original querystring of the request with what is specified.
This will get passed to
[`querystring.stringify`](https://nodejs.org/api/querystring.html#querystring_querystring_stringify_obj_sep_eq_options).

#### body

Replaces the original request body with what is specified. Unless
[`contentType`][contentType] is specified, the content will be passed
through `JSON.stringify()`.
Setting this option will not verify if the http method allows for a body.

#### contentType

Override the `'Content-Type'` header of the forwarded request, if we are
already overriding the [`body`][body].

## TODO

* [ ] support overriding the body with a stream
* [ ] forward the request id to the other peer might require some
      refacotring because we have to make the `req.id` unique
      (see [hyperid](http://npm.im/hyperid)).
* [ ] support http2

## License

MIT

[http-agent]: https://nodejs.org/api/http.html#http_new_agent_options
[https-agent]: https://nodejs.org/api/https.html#https_class_https_agent
