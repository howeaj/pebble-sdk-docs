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
    keys: ["COMMAND", "DATA", "RESULT"],
    onReadable() {
        const msg = this.read();
        msg.forEach((value, key) => {
            console.log(key + ": " + value);
        });
    },
    onWritable() {
        console.log("Ready to send messages");
    }
});
```

The `keys` array lists the message key names used for communication. These must
match the keys defined in `package.json`.

## Sending Messages from Watch

**⌚ Watch** (src/embeddedjs/main.js):

```js
message.write(new Map([
    ["COMMAND", 1],
    ["DATA", 42]
]));
```

## Message Keys in package.json

Message keys must be defined in `package.json` as an array:

```json
{
  "pebble": {
    "messageKeys": [
      "COMMAND",
      "DATA",
      "RESULT"
    ]
  }
}
```

## Receiving Messages in PKJS

**📱 PKJS** (src/pkjs/index.js):

```js
Pebble.addEventListener('appmessage', function(e) {
    console.log("Received from watch: " + JSON.stringify(e.payload));

    // Send a response back to watch
    Pebble.sendAppMessage({
        'RESULT': 123
    });
});
```

## Combining with the Network Proxy

If your app uses both app messages and the network proxy, set up the PKJS to
handle both:

**📱 PKJS** (src/pkjs/index.js):

```js
const moddableProxy = require("@moddable/pebbleproxy");

Pebble.addEventListener('ready', moddableProxy.readyReceived);

Pebble.addEventListener('appmessage', function(e) {
    if (moddableProxy.appMessageReceived(e))
        return;

    // Handle your own app messages here
    console.log("Received from watch: " + JSON.stringify(e.payload));
});
```

The proxy's `appMessageReceived()` returns `true` if the message was for the
proxy (e.g., a `fetch()` request). If it returns `false`, the message is one of
your own and you can handle it.

## Getting GPS Location

For GPS location, use the `Location` sensor instead of app messages. See the
[Sensors and Input](/guides/alloy/sensors-and-input/) guide for details on the
Location sensor, or the
[watchface tutorial Part 4](/tutorials/alloy-watchface-tutorial/part4/) for a
complete weather example.

## Examples

The [Pebble Examples](https://github.com/Moddable-OpenSource/pebble-examples)
repository includes app message examples:

- [`hellomessage`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellomessage) — sending and receiving messages between watch and phone
- [`helloconnected`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/helloconnected) — monitoring watch-phone connection status
