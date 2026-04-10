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

layout: tutorials/tutorial
tutorial: alloy-watchface
tutorial_part: 3

title: Battery Meter and Connection Alerts
description: |
  How to add a battery level meter and connection disconnect alerts
  to your watchface.
permalink: /tutorials/alloy-watchface-tutorial/part3/
generate_toc: true
platform_choice: true
---

Our watchface tells the time with style, but a great watchface also gives
useful information at a glance. In this part we will add two popular features:
a battery meter and a connection disconnect alert.

By the end of this part, your watchface will look something like this:

{% screenshot_viewer %}
{
  "image": "/images/tutorials/alloy-watchface-tutorial/part3.png",
  "default": "emery",
  "platforms": [
    {"hw": "emery", "wrapper": "core-time2-red"},
    {"hw": "gabbro", "wrapper": "core-time-round2-black-20"}
  ]
}
{% endscreenshot_viewer %}

This section continues from
[*Part 2*](/tutorials/alloy-watchface-tutorial/part2/), so be sure to re-use
your code or start with that finished project.


## The Battery Meter

### Reading Battery State

In Alloy, battery information comes from the `Battery` sensor. Import it at the
top of your file:

```js
import Battery from "embedded:sensor/Battery";
```

Create a `Battery` instance with an `onSample` callback that fires whenever the
battery state changes:

```js
let batteryPercent = 100;

const battery = new Battery({
    onSample() {
        batteryPercent = this.sample().percent;
        drawScreen();
    }
});
batteryPercent = battery.sample().percent;
```

`battery.sample()` returns an object with `percent` (0–100), `charging`
(boolean), and `plugged` (boolean). We call `sample()` once at startup to get
the initial value, and the `onSample` callback handles updates. Each update
triggers a full redraw.

### Drawing the Battery Bar

We will draw the battery bar near the top of the screen. It has a white border
with a filled portion that changes color based on the charge level - green when
healthy, yellow when getting low, red when critical.

Add a `drawBatteryBar()` function:

```js
const green = render.makeColor(0, 170, 0);
const yellow = render.makeColor(255, 170, 0);
const red = render.makeColor(255, 0, 0);

function drawBatteryBar() {
    const barWidth = (render.width / 2) | 0;
    const barX = ((render.width - barWidth) / 2) | 0;
    const barY = render.height < 180 ? 6 : 20;
    const barHeight = 8;

    // Draw border
    render.fillRectangle(white, barX, barY, barWidth, barHeight);
    render.fillRectangle(black, barX + 1, barY + 1, barWidth - 2, barHeight - 2);

    // Choose color based on battery level
    let barColor;
    if (batteryPercent <= 20) {
        barColor = red;
    } else if (batteryPercent <= 40) {
        barColor = yellow;
    } else {
        barColor = green;
    }

    // Draw filled portion
    const fillWidth = ((batteryPercent * (barWidth - 4)) / 100) | 0;
    render.fillRectangle(barColor, barX + 2, barY + 2, fillWidth, barHeight - 4);
}
```

The `| 0` trick truncates floating-point results to integers, which is faster
than `Math.floor()` on an embedded device.

We position the bar differently based on screen height
(`render.height < 180`) so it looks good on both Emery and Gabbro.

Call `drawBatteryBar()` inside `draw()`, after clearing the background.


## Connection Disconnect Alert

### Monitoring Connection State

Alloy provides connection status through the `watch.connected` object. We can
listen for changes with the `connected` event:

```js
let isConnected = true;

function checkConnection() {
    isConnected = watch.connected.app;
    drawScreen();
}
watch.addEventListener("connected", checkConnection);
checkConnection();
```

`watch.connected.app` is `true` when the watch is connected to the phone app,
`false` when disconnected. We check immediately at startup and on every change.


### Showing the Disconnect Indicator

When the connection is lost, we draw a red "X" below the battery bar. Add this
to your draw function after the battery bar:

```js
    // Draw disconnect indicator below battery bar
    if (!isConnected) {
        const btStr = "X";
        const btWidth = render.getTextWidth(btStr, smallFont);
        const btY = render.height < 180 ? 16 : 30;
        render.drawText(btStr, smallFont, red,
            (render.width - btWidth) / 2, btY);
    }
```

You will need the `smallFont` for this - add it with your other font
declarations:

```js
const smallFont = new render.Font("Gothic-Regular", 18);
```


## Updating the Draw Function

Since battery and connection changes now trigger redraws outside of time events,
we need to handle the case where `draw()` is called without an event. Rename
it to `drawScreen()` and add a fallback for the date:

```js
let lastDate = new Date();

function drawScreen(event) {
    const now = event?.date ?? lastDate;
    if (event?.date) lastDate = event.date;

    // ... rest of draw code
}
```

The `event?.date` optional chaining returns `undefined` if `event` is missing
or has no `date`. The `??` nullish coalescing operator falls back to `lastDate`.
We save the latest date so non-time redraws still show the correct time.

Update the event listener to use the new name:

```js
watch.addEventListener("minutechange", drawScreen);
```


## Testing in the Emulator

^CP^ Click the **play** button to compile and install your watchface in the
CloudPebble emulator.

^LC^ Build and install your watchface:

{% platform local %}
```text
$ pebble build && pebble install --emulator emery
```
{% endplatform %}

### Setting the Battery Level

^CP^ In the CloudPebble emulator, use the gear menu to adjust the battery level.

^LC^ Use `pebble emu-battery` to change the simulated battery level:

{% platform local %}
```nc|text
$ pebble emu-battery --percent 80
$ pebble emu-battery --percent 30
$ pebble emu-battery --percent 10
```
{% endplatform %}

You should see the bar go from green to yellow to red as the level decreases.

### Toggling the Connection

^CP^ In the CloudPebble emulator, use the gear menu to toggle the connection on
and off.

^LC^ Use `pebble emu-bt-connection` to simulate a disconnect:

{% platform local %}
```nc|text
$ pebble emu-bt-connection --connected no
$ pebble emu-bt-connection --connected yes
```
{% endplatform %}

When disconnected, the red "X" should appear below the battery bar.


## Conclusion

In this part we learned how to:

1. Import and use the `Battery` sensor for charge level updates.
2. Draw a color-coded battery bar using Poco primitives.
3. Monitor connection status with `watch.connected` and the `connected` event.
4. Show a disconnect indicator.
5. Handle redraws triggered by non-time events.

Your watchface now shows the battery level and alerts you when the phone
disconnects. Check your code against
[the source for this part](https://github.com/coredevices/alloy-watchface-tutorial/tree/main/part3).


## What's Next?

In the next part we will add weather information by fetching data from the
Open-Meteo API - our first foray into network communication.

[Go to Part 4 &rarr; >{wide,bg-dark-red,fg-white}](/tutorials/alloy-watchface-tutorial/part4/)
