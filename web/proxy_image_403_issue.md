## PROBLEM

I had an ejs template that used data from the server.

```html
<div class="preview-card"
     style="background-image: var(--darken-gradient), url('<%= result.image %>')">
    <div class="grid-container">
        <div class="info-chip episode-chip"> Episode <%= result.episode || 'Unknown' %> </div>
        <div class="info-chip timestamp-chip">
            <%= formatTime(result.from) %> - <%= formatTime(result.to) %>
        </div>
        <div class="info-chip similarity-chip">
            <%= result.similarity ? ${Math.round(result.similarity * 100)}% Match : 'Unknown' %>
        </div>
        <div class="title-wrapper">
            <p class="title"> <%= result.filename.split('.').slice(0, -1).join('.') %> </p>
        </div>
    </div>
</div>
```

However, the image would not load when the card was rendered, the video played properly though.
I checked the debug console and there was no error, so I checked the network tab and saw that the image fetch
had the message `(failed)net::ERR_BLOCKED_BY_ORB`.

My solution was to use a proxy, which I hosted on the same server as the ejs template.

```js
router.get('/proxy-image', async (req, res) => {
    let imageUrl = req.query.url;
    if (!imageUrl) {
        console.error('No image url query parameter found.');
        return res.status(400).send('Image URL is required');
    }
    
    await queue.add(async () => {
        try {
            const response = await axios.get(imageUrl, {responseType: 'arraybuffer'},
            );

            const contentType = response.headers['content-type'];

            res.set('Content-Type', contentType);
            return res.send(response.data);
        } catch (error) {
            console.error('Error Occurred', error.response?.data?.error || error);
            return res.status(500).send('Error fetching image');
        }
    });
});
```

Then I updated the ejs template to use the proxy.

```html
<div class="preview-card"
     style="background-image: var(--darken-gradient), url('/proxy-image?url=<%= result.image %>')">
    <div class="grid-container">
        <div class="info-chip episode-chip"> Episode <%= result.episode || 'Unknown' %> </div>
        <div class="info-chip timestamp-chip">
            <%= formatTime(result.from) %> - <%= formatTime(result.to) %>
        </div>
        <div class="info-chip similarity-chip">
            <%= result.similarity ? ${Math.round(result.similarity * 100)}% Match : 'Unknown' %>
        </div>
        <div class="title-wrapper">
            <p class="title"> <%= result.filename.split('.').slice(0, -1).join('.') %> </p>
        </div>
    </div>
</div>
```

This threw a 403 error.

```shell
GET /proxy-image?url=https://api.trace.moe/image/141902/(2022.08.06)ONE%20PIECE%20FILM%20RED-%5B1080p%5D%5BBDRIP%5D%5Bx265.FLAC%5D.mp4.jpg?t=6753.04&now=1731765600&token=2M5EECc0UhyCevLRqeEW1u5neg 500 1255.296 ms - 20
Error Occurred AxiosError: Request failed with status code 403
    at settle (file:///home/akundadababalei/WebProjects/AppBrewery%20Web/Capstones/Capstone%203%20AniScene/node_modules/axios/lib/core/settle.js:19:12)
    at BrotliDecompress.handleStreamEnd (file:///home/akundadababalei/WebProjects/AppBrewery%20Web/Capstones/Capstone%203%20AniScene/node_modules/axios/lib/adapters/http.js:599:11)
    at BrotliDecompress.emit (node:events:531:35)
    at endReadableNT (node:internal/streams/readable:1696:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:82:21)
    at Axios.request (file:///home/akundadababalei/WebProjects/AppBrewery%20Web/Capstones/Capstone%203%20AniScene/node_modules/axios/lib/core/Axios.js:45:41)
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)
    at async file:///home/akundadababalei/WebProjects/AppBrewery%20Web/Capstones/Capstone%203%20AniScene/routes/index.js:66:30
    at async file:///home/akundadababalei/WebProjects/AppBrewery%20Web/Capstones/Capstone%203%20AniScene/node_modules/p-queue/dist/index.js:187:36 {
  code: 'ERR_BAD_REQUEST',
  config: {
    transitional: {
      silentJSONParsing: true,
      forcedJSONParsing: true,
      clarifyTimeoutError: false
    },
    adapter: [ 'xhr', 'http', 'fetch' ],
    transformRequest: [ [Function: transformRequest] ],
    transformResponse: [ [Function: transformResponse] ],
    timeout: 0,
    xsrfCookieName: 'XSRF-TOKEN',
    xsrfHeaderName: 'X-XSRF-TOKEN',
    maxContentLength: -1,
    maxBodyLength: -1,
    env: { FormData: [Function], Blob: [class Blob] },
    validateStatus: [Function: validateStatus],
    headers: Object [AxiosHeaders] {
      Accept: 'application/json, text/plain, */*',
      'Content-Type': undefined,
      'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36',
      'Accept-Encoding': 'gzip, compress, deflate, br'
    },
    responseType: 'arraybuffer',
    method: 'get',
    url: 'https://api.trace.moe/image/140596/%5BOhys-Raws%5D%20Ijiranaide,%20Nagatoro-san%202nd%20-%2012%20(BS11%201280x720%20x264%20AAC).mp4.jpg?t=368.33500000000004',
    data: undefined
  },
  request: <ref *1> ClientRequest {
    _events: [Object: null prototype] {
      abort: [Function (anonymous)],
      aborted: [Function (anonymous)],
      connect: [Function (anonymous)],
      error: [Function (anonymous)],
      socket: [Function (anonymous)],
      timeout: [Function (anonymous)],
      finish: [Function: requestOnFinish]
    },
    _eventsCount: 7,
    _maxListeners: undefined,
    outputData: [],
    outputSize: 0,
    writable: true,
    destroyed: true,
    _last: true,
    chunkedEncoding: false,
    shouldKeepAlive: true,
    maxRequestsOnConnectionReached: false,
    _defaultKeepAlive: true,
    useChunkedEncodingByDefault: false,
    sendDate: false,
    _removedConnection: false,
    _removedContLen: false,
    _removedTE: false,
    strictContentLength: false,
    _contentLength: 0,
    _hasBody: true,
    _trailer: '',
    finished: true,
    _headerSent: true,
    _closed: true,
    socket: TLSSocket {
      _tlsOptions: [Object],
      _secureEstablished: true,
      _securePending: false,
      _newSessionPending: false,
      _controlReleased: true,
      secureConnecting: false,
      _SNICallback: null,
      servername: 'api.trace.moe',
      alpnProtocol: false,
      authorized: true,
      authorizationError: null,
      encrypted: true,
      _events: [Object: null prototype],
      _eventsCount: 9,
      connecting: false,
      _hadError: false,
      _parent: null,
      _host: 'api.trace.moe',
      _closeAfterHandlingError: false,
      _readableState: [ReadableState],
      _writableState: [WritableState],
      allowHalfOpen: false,
      _maxListeners: undefined,
      _sockname: null,
      _pendingData: null,
      _pendingEncoding: '',
      server: undefined,
      _server: null,
      ssl: [TLSWrap],
      _requestCert: true,
      _rejectUnauthorized: true,
      timeout: 5000,
      parser: null,
      _httpMessage: null,
      autoSelectFamilyAttemptedAddresses: [Array],
      [Symbol(alpncallback)]: null,
      [Symbol(res)]: [TLSWrap],
      [Symbol(verified)]: true,
      [Symbol(pendingSession)]: null,
      [Symbol(async_id_symbol)]: -1,
      [Symbol(kHandle)]: [TLSWrap],
      [Symbol(lastWriteQueueSize)]: 0,
      [Symbol(timeout)]: Timeout {
        _idleTimeout: 5000,
        _idlePrev: [TimersList],
        _idleNext: [TimersList],
        _idleStart: 1160334,
        _onTimeout: [Function: bound ],
        _timerArgs: undefined,
        _repeat: null,
        _destroyed: false,
        [Symbol(refed)]: false,
        [Symbol(kHasPrimitive)]: false,
        [Symbol(asyncId)]: 793,
        [Symbol(triggerId)]: 791
      },
      [Symbol(kBuffer)]: null,
      [Symbol(kBufferCb)]: null,
      [Symbol(kBufferGen)]: null,
      [Symbol(shapeMode)]: true,
      [Symbol(kCapture)]: false,
      [Symbol(kSetNoDelay)]: false,
      [Symbol(kSetKeepAlive)]: true,
      [Symbol(kSetKeepAliveInitialDelay)]: 1,
      [Symbol(kBytesRead)]: 0,
      [Symbol(kBytesWritten)]: 0,
      [Symbol(connect-options)]: [Object]
    },
    _header: 'GET /image/140596/%5BOhys-Raws%5D%20Ijiranaide,%20Nagatoro-san%202nd%20-%2012%20(BS11%201280x720%20x264%20AAC).mp4.jpg?t=368.33500000000004 HTTP/1.1\r\n' +
      'Accept: application/json, text/plain, */*\r\n' +
      'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36\r\n' +
      'Accept-Encoding: gzip, compress, deflate, br\r\n' +
      'Host: api.trace.moe\r\n' +
      'Connection: keep-alive\r\n' +
      '\r\n',
    _keepAliveTimeout: 0,
    _onPendingData: [Function: nop],
    agent: Agent {
      _events: [Object: null prototype],
      _eventsCount: 2,
      _maxListeners: undefined,
      defaultPort: 443,
      protocol: 'https:',
      options: [Object: null prototype],
      requests: [Object: null prototype] {},
      sockets: [Object: null prototype] {},
      freeSockets: [Object: null prototype],
      keepAliveMsecs: 1000,
      keepAlive: true,
      maxSockets: Infinity,
      maxFreeSockets: 256,
      scheduling: 'lifo',
      maxTotalSockets: Infinity,
      totalSocketCount: 1,
      maxCachedSessions: 100,
      _sessionCache: [Object],
      [Symbol(shapeMode)]: false,
      [Symbol(kCapture)]: false
    },
    socketPath: undefined,
    method: 'GET',
    maxHeaderSize: undefined,
    insecureHTTPParser: undefined,
    joinDuplicateHeaders: undefined,
    path: '/image/140596/%5BOhys-Raws%5D%20Ijiranaide,%20Nagatoro-san%202nd%20-%2012%20(BS11%201280x720%20x264%20AAC).mp4.jpg?t=368.33500000000004',
    _ended: true,
    res: IncomingMessage {
      _events: [Object],
      _readableState: [ReadableState],
      _maxListeners: undefined,
      socket: null,
      httpVersionMajor: 1,
      httpVersionMinor: 1,
      httpVersion: '1.1',
      complete: true,
      rawHeaders: [Array],
      rawTrailers: [],
      joinDuplicateHeaders: undefined,
      aborted: false,
      upgrade: false,
      url: '',
      method: null,
      statusCode: 403,
      statusMessage: 'Forbidden',
      client: [TLSSocket],
      _consuming: true,
      _dumped: false,
      req: [Circular *1],
      _eventsCount: 4,
      responseUrl: 'https://api.trace.moe/image/140596/%5BOhys-Raws%5D%20Ijiranaide,%20Nagatoro-san%202nd%20-%2012%20(BS11%201280x720%20x264%20AAC).mp4.jpg?t=368.33500000000004',
      redirects: [],
      [Symbol(shapeMode)]: true,
      [Symbol(kCapture)]: false,
      [Symbol(kHeaders)]: [Object],
      [Symbol(kHeadersCount)]: 44,
      [Symbol(kTrailers)]: null,
      [Symbol(kTrailersCount)]: 0
    },
    aborted: false,
    timeoutCb: null,
    upgradeOrConnect: false,
    parser: null,
    maxHeadersCount: null,
    reusedSocket: true,
    host: 'api.trace.moe',
    protocol: 'https:',
    _redirectable: Writable {
      _events: [Object],
      _writableState: [WritableState],
      _maxListeners: undefined,
      _options: [Object],
      _ended: true,
      _ending: true,
      _redirectCount: 0,
      _redirects: [],
      _requestBodyLength: 0,
      _requestBodyBuffers: [],
      _eventsCount: 3,
      _onNativeResponse: [Function (anonymous)],
      _currentRequest: [Circular *1],
      _currentUrl: 'https://api.trace.moe/image/140596/%5BOhys-Raws%5D%20Ijiranaide,%20Nagatoro-san%202nd%20-%2012%20(BS11%201280x720%20x264%20AAC).mp4.jpg?t=368.33500000000004',
      [Symbol(shapeMode)]: true,
      [Symbol(kCapture)]: false
    },
    [Symbol(shapeMode)]: false,
    [Symbol(kCapture)]: false,
    [Symbol(kBytesWritten)]: 0,
    [Symbol(kNeedDrain)]: false,
    [Symbol(corked)]: 0,
    [Symbol(kOutHeaders)]: [Object: null prototype] {
      accept: [Array],
      'user-agent': [Array],
      'accept-encoding': [Array],
      host: [Array]
    },
    [Symbol(errored)]: null,
    [Symbol(kHighWaterMark)]: 16384,
    [Symbol(kRejectNonStandardBodyWrites)]: false,
    [Symbol(kUniqueHeaders)]: null
  },
  response: {
    status: 403,
    statusText: 'Forbidden',
    headers: Object [AxiosHeaders] {
      date: 'Sat, 16 Nov 2024 13:28:24 GMT',
      'content-type': 'text/html; charset=utf-8',
      'transfer-encoding': 'chunked',
      connection: 'keep-alive',
      'access-control-allow-credentials': 'true',
      'access-control-allow-headers': 'Content-Type, x-trace-secret',
      'access-control-allow-methods': 'GET, POST, OPTIONS',
      'access-control-allow-origin': '*',
      'alt-svc': 'h3=":443"; ma=86400',
      vary: 'Origin, Accept-Encoding',
      'x-ratelimit-limit': '100',
      'x-ratelimit-remaining': '86',
      'x-ratelimit-reset': '1731763761',
      'cf-cache-status': 'BYPASS',
      'report-to': '{"endpoints":[{"url":"https:\\/\\/a.nel.cloudflare.com\\/report\\/v4?s=VwZ2t2GqV2n71qdFdOG%2Be6OFErgX%2BbEy%2FBnxB1pTM00D1umKDyfWBHSqYc5lvMwS9JODC4cZwb3CYQjAsEOrQycW1ojKGlwZ5pvh5OLJ73X5UzPahZsqpw%2Fb1t0YRC8%3D"}],"group":"cf-nel","max_age":604800}',
      nel: '{"success_fraction":0,"report_to":"cf-nel","max_age":604800}',
      'strict-transport-security': 'max-age=31536000; includeSubDomains; preload',
      'x-content-type-options': 'nosniff',
      server: 'cloudflare',
      'cf-ray': '8e37d2f1b9db7193-LHR',
      'server-timing': 'cfL4;desc="?proto=TCP&rtt=132292&sent=501&recv=816&lost=0&retrans=0&sent_bytes=8972&recv_bytes=1117336&delivery_rate=41737&cwnd=251&unsent_bytes=0&cid=74f88f72ce590541&ts=7402&x=0"'
    },
    config: {
      transitional: [Object],
      adapter: [Array],
      transformRequest: [Array],
      transformResponse: [Array],
      timeout: 0,
      xsrfCookieName: 'XSRF-TOKEN',
      xsrfHeaderName: 'X-XSRF-TOKEN',
      maxContentLength: -1,
      maxBodyLength: -1,
      env: [Object],
      validateStatus: [Function: validateStatus],
      headers: [Object [AxiosHeaders]],
      responseType: 'arraybuffer',
      method: 'get',
      url: 'https://api.trace.moe/image/140596/%5BOhys-Raws%5D%20Ijiranaide,%20Nagatoro-san%202nd%20-%2012%20(BS11%201280x720%20x264%20AAC).mp4.jpg?t=368.33500000000004',
      data: undefined
    },
    request: <ref *1> ClientRequest {
      _events: [Object: null prototype],
      _eventsCount: 7,
      _maxListeners: undefined,
      outputData: [],
      outputSize: 0,
      writable: true,
      destroyed: true,
      _last: true,
      chunkedEncoding: false,
      shouldKeepAlive: true,
      maxRequestsOnConnectionReached: false,
      _defaultKeepAlive: true,
      useChunkedEncodingByDefault: false,
      sendDate: false,
      _removedConnection: false,
      _removedContLen: false,
      _removedTE: false,
      strictContentLength: false,
      _contentLength: 0,
      _hasBody: true,
      _trailer: '',
      finished: true,
      _headerSent: true,
      _closed: true,
      socket: [TLSSocket],
      _header: 'GET /image/140596/%5BOhys-Raws%5D%20Ijiranaide,%20Nagatoro-san%202nd%20-%2012%20(BS11%201280x720%20x264%20AAC).mp4.jpg?t=368.33500000000004 HTTP/1.1\r\n' +
        'Accept: application/json, text/plain, */*\r\n' +
        'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36\r\n' +
        'Accept-Encoding: gzip, compress, deflate, br\r\n' +
        'Host: api.trace.moe\r\n' +
        'Connection: keep-alive\r\n' +
        '\r\n',
      _keepAliveTimeout: 0,
      _onPendingData: [Function: nop],
      agent: [Agent],
      socketPath: undefined,
      method: 'GET',
      maxHeaderSize: undefined,
      insecureHTTPParser: undefined,
      joinDuplicateHeaders: undefined,
      path: '/image/140596/%5BOhys-Raws%5D%20Ijiranaide,%20Nagatoro-san%202nd%20-%2012%20(BS11%201280x720%20x264%20AAC).mp4.jpg?t=368.33500000000004',
      _ended: true,
      res: [IncomingMessage],
      aborted: false,
      timeoutCb: null,
      upgradeOrConnect: false,
      parser: null,
      maxHeadersCount: null,
      reusedSocket: true,
      host: 'api.trace.moe',
      protocol: 'https:',
      _redirectable: [Writable],
      [Symbol(shapeMode)]: false,
      [Symbol(kCapture)]: false,
      [Symbol(kBytesWritten)]: 0,
      [Symbol(kNeedDrain)]: false,
      [Symbol(corked)]: 0,
      [Symbol(kOutHeaders)]: [Object: null prototype],
      [Symbol(errored)]: null,
      [Symbol(kHighWaterMark)]: 16384,
      [Symbol(kRejectNonStandardBodyWrites)]: false,
      [Symbol(kUniqueHeaders)]: null
    },
    data: <Buffer 46 6f 72 62 69 64 64 65 6e>
  },
  status: 403
}
```

## SOLUTION

After trying several things like using `encodeURI` on the server before making the request, I found
out that the issue was with the url from the client side.

You see, When you build a URL for the proxy-image route, any special characters (e.g., ?, &, =) 
in the original image URL need to be encoded.

For example:

```
/proxy-image?url=https://example.com/image.jpg?foo=bar&baz=qux
```
The server interprets foo=bar&baz=qux as part of the proxy-image route's query string, rather than part of the image URL.

The solution is to encode the original image URL while building the url for the proxy-image route right
there on the client side.

```html
<div class="preview-card"
     style="background-image: var(--darken-gradient), url('/proxy-image?url=<%= encodeURIComponent(result.image) %>')">
    <div class="grid-container">
        <div class="info-chip episode-chip"> Episode <%= result.episode || 'Unknown' %> </div>
        <div class="info-chip timestamp-chip">
            <%= formatTime(result.from) %> - <%= formatTime(result.to) %>
        </div>
        <div class="info-chip similarity-chip">
            <%= result.similarity ? ${Math.round(result.similarity * 100)}% Match : 'Unknown' %>
        </div>
        <div class="title-wrapper">
            <p class="title"> <%= result.filename.split('.').slice(0, -1).join('.') %> </p>
        </div>
    </div>
</div>
```