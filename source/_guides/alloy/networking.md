---
# Copyright 2025 Core Devices LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

title: Networking
description: |
  Make HTTP requests and use WebSockets via the phone proxy.
guide_group: alloy
order: 6
---

Alloy apps can communicate with the internet by proxying network requests
through PebbleKit JS (PKJS) running on the phone.

## Understanding Watch vs Phone Code

Alloy apps have two JavaScript environments:

| Location | File Path | Runs On | Purpose |
|----------|-----------|---------|---------|
| **embeddedjs** | `src/embeddedjs/main.js` | Watch | Your app UI and logic |
| **PKJS** | `src/pkjs/index.js` | Phone | Network proxy, location, config |

Code examples in this guide are labeled with **📱 PKJS** or **⌚ Watch** to
indicate where they run.

## Setting Up the Network Proxy

To use `fetch()` or `WebSocket` on the watch, install the
`@moddable/pebbleproxy` package:

```nc|text
$ pebble package install @moddable/pebbleproxy
```

Then set up your `src/pkjs/index.js`:

**📱 PKJS** (src/pkjs/index.js):

```js
const moddableProxy = require("@moddable/pebbleproxy");
Pebble.addEventListener('ready', moddableProxy.readyReceived);
Pebble.addEventListener('appmessage', moddableProxy.appMessageReceived);

```

If your app also needs to handle its own events, call the proxy functions from your handlers:

```js
const moddableProxy = require("@moddable/pebbleproxy");

Pebble.addEventListener('ready', function(e) {
    moddableProxy.readyReceived(e);
    // Handle your own ready event here
});

Pebble.addEventListener('appmessage', function(e) {
    if (moddableProxy.appMessageReceived(e))
        return;

    // Handle your own app messages here
});
```

## HTTP Requests with fetch()

Once the proxy is set up, use `fetch()` in your watch code:

**⌚ Watch** (src/embeddedjs/main.js):

```js
async function fetchData() {
    const url = new URL("http://api.open-meteo.com/v1/forecast");
    url.search = new URLSearchParams({
        latitude: 37.7749,
        longitude: -122.4194,
        current: "temperature_2m"
    });

    const response = await fetch(url);
    const data = await response.json();
    console.log("Temperature: " + data.current.temperature_2m);
}
```

### Important Notes

- **Wait for the proxy** - network requests only work after the proxy signals
  it is ready. Listen for `watch.addEventListener("connected", ...)` before
  calling `fetch()` or opening a `WebSocket`

### Response Methods

| Method | Description |
|--------|-------------|
| `response.json()` | Parse response as JSON |
| `response.text()` | Get response as string |
| `response.ok` | Boolean: true if status 200-299 |
| `response.status` | HTTP status code |

## WebSockets

WebSockets are also handled by the `@moddable/pebbleproxy` package - no
additional proxy setup needed.

**⌚ Watch** (src/embeddedjs/main.js):

```js
const ws = new WebSocket("ws://websockets.chilkat.io/wsChilkatEcho.ashx");
ws.binaryType = "arraybuffer";

ws.addEventListener("open", event => {
    console.log("WebSocket connected");
    ws.send("Hello from Pebble!");
    ws.send(Uint8Array.of(0, 1, 2, 3, 4, 5));
});

ws.addEventListener("message", event => {
    let data = event.data;
    if (data instanceof ArrayBuffer) {
        console.log("Received binary data");
        data = new Uint8Array(data);
    } else {
        console.log("Received: " + data);
        if (data === "Goodbye") ws.close(1000, "Done");
    }
});

ws.addEventListener("close", event => {
    console.log("Closed: " + event.code + " " + event.reason);
});
```

## Connection Status

Check if the phone is connected from watch code:

**⌚ Watch** (src/embeddedjs/main.js):

```js
function logConnected() {
    console.log("App connected: " + watch.connected.app);
    console.log("PebbleKitJS connected: " + watch.connected.pebblekit);
}

watch.addEventListener('connected', logConnected);
logConnected();
```

Network requests only work once the proxy is ready. Wait until
`watch.connected.pebblekit` is `true` before calling `fetch()` or opening a
`WebSocket`.

## Best Practices

1. **Install `@moddable/pebbleproxy`** - required for `fetch()` and `WebSocket`
2. **Wait for the proxy** - network requests only work after the proxy is ready;
   listen for the `connected` event or check `watch.connected.pebblekit`
3. **Handle errors** - network requests can fail
4. **Minimize data** - request only what you need
5. **Cache responses** - use `localStorage` to reduce requests

## Advanced Networking

For more control over HTTP and WebSocket connections, Alloy also provides
low-level ECMA-419 networking APIs:

- `HTTPClient` - streaming HTTP client with fine-grained header control
- `WebSocketClient` - low-level WebSocket with callback-based API, provides streaming support

These are also handled by `@moddable/pebbleproxy`. See the
[Advanced Networking](/guides/alloy/advanced-networking/) guide for details.

## Examples

The [Pebble Examples](https://github.com/Moddable-OpenSource/pebble-examples)
repository includes networking examples:

- [`hellofetch`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellofetch) - HTTP requests using the `fetch()` API
- [`hellowebsocket`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellowebsocket) - WebSocket connections using the `WebSocket` API
