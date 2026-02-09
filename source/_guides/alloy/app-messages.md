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

title: App Messages
description: |
  Send and receive messages between the watch and the phone.
guide_group: alloy
order: 7
---

The Message API provides direct communication between watch code and PebbleKit JS
(PKJS) on the phone. Unlike [networking](/guides/alloy/networking/), app messages
don't require the `@moddable/pebbleproxy` package — they use Pebble's built-in
messaging system.

Code examples in this guide are labeled with **📱 PKJS** or **⌚ Watch** to
indicate where they run.

## Watch-Phone Messaging

Use the `Message` class to send and receive structured data between the watch
and PKJS:

**⌚ Watch** (src/embeddedjs/main.js):

```js
import Message from "pebble/message";

const message = new Message({
    input: 256,
    output: 256,
    keys: new Map([
        ["COMMAND", 0],
        ["DATA", 1],
        ["RESULT", 2]
    ]),
    onReadable() {
        const msg = this.read();
        if (!msg) return;

        msg.forEach((value, key) => {
            console.log(key + ": " + value);
        });
    },
    onWritable() {
        console.log("Ready to send messages");
    }
});
```

## Sending Messages from Watch

**⌚ Watch** (src/embeddedjs/main.js):

```js
message.write(new Map([
    ["COMMAND", 1],
    ["DATA", 42]
]));
```

## Message Keys in package.json

Message keys must be defined in `package.json`:

```json
{
  "pebble": {
    "messageKeys": {
      "COMMAND": 0,
      "DATA": 1,
      "RESULT": 2
    }
  }
}
```

## Receiving Messages in PKJS

**📱 PKJS** (src/pkjs/index.js):

```js
const moddableProxy = require("@moddable/pebbleproxy");

Pebble.addEventListener('appmessage', function(e) {
    if (moddableProxy.eventReceived(e))
        return;

    console.log("Received from watch: " + JSON.stringify(e.payload));

    // Send a response back to watch
    Pebble.sendAppMessage({
        'RESULT': 123
    });
});
```

## Getting GPS Location

Location comes from the phone, so it requires messaging between watch and PKJS.

### PKJS Location Handler

**📱 PKJS** (src/pkjs/index.js):

```js
const moddableProxy = require("@moddable/pebbleproxy");

Pebble.addEventListener('appmessage', function(e) {
    if (moddableProxy.eventReceived(e))
        return;

    // Handle location request
    if (e.payload['REQUEST_LOCATION'] !== undefined) {
        console.log("Location requested");

        navigator.geolocation.getCurrentPosition(
            function(pos) {
                console.log("Location: " + pos.coords.latitude + ", " + pos.coords.longitude);
                Pebble.sendAppMessage({
                    'LATITUDE': Math.round(pos.coords.latitude * 10000),
                    'LONGITUDE': Math.round(pos.coords.longitude * 10000)
                });
            },
            function(err) {
                console.log("Location error: " + err.message);
            },
            { timeout: 15000, maximumAge: 60000 }
        );
    }
});
```

### Watch Code to Request Location

**⌚ Watch** (src/embeddedjs/main.js):

```js
import Message from "pebble/message";

let latitude = null;
let longitude = null;

const message = new Message({
    input: 256,
    output: 256,
    keys: new Map([
        ["LATITUDE", 0],
        ["LONGITUDE", 1],
        ["REQUEST_LOCATION", 2]
    ]),
    onReadable() {
        const msg = this.read();
        if (!msg) return;

        if (msg.has("LATITUDE") && msg.has("LONGITUDE")) {
            latitude = msg.get("LATITUDE") / 10000;
            longitude = msg.get("LONGITUDE") / 10000;
            console.log("Location: " + latitude + ", " + longitude);
        }
    },
    onWritable() {
        // Request location when connection is ready
        this.write(new Map([["REQUEST_LOCATION", true]]));
    }
});
```

### package.json for Location

```json
{
  "pebble": {
    "messageKeys": {
      "LATITUDE": 0,
      "LONGITUDE": 1,
      "REQUEST_LOCATION": 2
    }
  }
}
```

## Examples

The [Pebble Examples](https://github.com/Moddable-OpenSource/pebble-examples)
repository includes app message examples:

- [`hellomessage`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellomessage) — sending and receiving messages between watch and phone
- [`helloconnected`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/helloconnected) — monitoring watch-phone connection status
