<!-- YAML
added: v8.4.0
changes:
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/17105
    description: Added the `maxOutstandingPings` option with a default limit of
                 10.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/16676
    description: Added the `maxHeaderListPairs` option with a default limit of
                 128 header pairs.
-->

* `options` {Object}
  * `allowHTTP1` {boolean} Incoming client connections that do not support
    HTTP/2 will be downgraded to HTTP/1.x when set to `true`.
    See the [`'unknownProtocol'`][] event. See [ALPN negotiation][].
    **Default:** `false`.
  * `maxDeflateDynamicTableSize` {number} Sets the maximum dynamic table size
    for deflating header fields. **Default:** `4Kib`.
  * `maxSessionMemory`{number} Sets the maximum memory that the `Http2Session`
    is permitted to use. The value is expressed in terms of number of megabytes,
    e.g. `1` equal 1 megabyte. The minimum value allowed is `1`. This is a
    credit based limit, existing `Http2Stream`s may cause this
    limit to be exceeded, but new `Http2Stream` instances will be rejected
    while this limit is exceeded. The current number of `Http2Stream` sessions,
    the current memory use of the header compression tables, current data
    queued to be sent, and unacknowledged `PING` and `SETTINGS` frames are all
    counted towards the current limit. **Default:** `10`.
  * `maxHeaderListPairs` {number} Sets the maximum number of header entries.
    The minimum value is `4`. **Default:** `128`.
  * `maxOutstandingPings` {number} Sets the maximum number of outstanding,
    unacknowledged pings. **Default:** `10`.
  * `maxSendHeaderBlockLength` {number} Sets the maximum allowed size for a
    serialized, compressed block of headers. Attempts to send headers that
    exceed this limit will result in a `'frameError'` event being emitted
    and the stream being closed and destroyed.
  * `paddingStrategy` {number} Identifies the strategy used for determining the
     amount of padding to use for `HEADERS` and `DATA` frames. **Default:**
     `http2.constants.PADDING_STRATEGY_NONE`. Value may be one of:
     * `http2.constants.PADDING_STRATEGY_NONE` - Specifies that no padding is
       to be applied.
     * `http2.constants.PADDING_STRATEGY_MAX` - Specifies that the maximum
       amount of padding, as determined by the internal implementation, is to
       be applied.
     * `http2.constants.PADDING_STRATEGY_CALLBACK` - Specifies that the user
       provided `options.selectPadding()` callback is to be used to determine
       the amount of padding.
     * `http2.constants.PADDING_STRATEGY_ALIGNED` - Will *attempt* to apply
       enough padding to ensure that the total frame length, including the
       9-byte header, is a multiple of 8. For each frame, however, there is a
       maximum allowed number of padding bytes that is determined by current
       flow control state and settings. If this maximum is less than the
       calculated amount needed to ensure alignment, the maximum will be used
       and the total frame length will *not* necessarily be aligned at 8 bytes.
  * `peerMaxConcurrentStreams` {number} Sets the maximum number of concurrent
    streams for the remote peer as if a `SETTINGS` frame had been received. Will
    be overridden if the remote peer sets its own value for
    `maxConcurrentStreams`. **Default:** `100`.
  * `selectPadding` {Function} When `options.paddingStrategy` is equal to
    `http2.constants.PADDING_STRATEGY_CALLBACK`, provides the callback function
    used to determine the padding. See [Using `options.selectPadding()`][].
  * `settings` {HTTP/2 Settings Object} The initial settings to send to the
    remote peer upon connection.
  * ...: Any [`tls.createServer()`][] options can be provided. For
    servers, the identity options (`pfx` or `key`/`cert`) are usually required.
* `onRequestHandler` {Function} See [Compatibility API][]
* Returns: {Http2SecureServer}

Returns a `tls.Server` instance that creates and manages `Http2Session`
instances.

```js
const http2 = require('http2');

const options = {
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem')
};

// Create a secure HTTP/2 server
const server = http2.createSecureServer(options);

server.on('stream', (stream, headers) => {
  stream.respond({
    'content-type': 'text/html',
    ':status': 200
  });
  stream.end('<h1>Hello World</h1>');
});

server.listen(80);
```

