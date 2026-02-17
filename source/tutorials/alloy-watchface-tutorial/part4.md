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
tutorial_part: 4

title: Adding Weather with Open-Meteo
description: |
  How to fetch weather data from the web using fetch() and display
  it on your watchface.
permalink: /tutorials/alloy-watchface-tutorial/part4/
generate_toc: true
---

Up until now, everything in our watchface has been running entirely on the
watch. In this part we take a big step: communicating with the phone to fetch
live weather data from the web.

We will use the `Location` sensor to get the user's GPS coordinates, then the
standard `fetch()` API to get weather data from the free
[Open-Meteo](https://open-meteo.com) API (no API key needed!).

This section continues from
[*Part 3*](/tutorials/alloy-watchface-tutorial/part3/).


## How Networking Works in Alloy

Alloy apps make HTTP requests using the `fetch()` API. Requests are proxied
through PebbleKit JS (PKJS) running on the phone. This means you need:

1. The `@moddable/pebbleproxy` package installed
2. A phone connected to your Pebble with internet access

```text
┌─────────────┐          ┌──────────────────┐          ┌──────────────┐
│   Watch     │          │  Phone (PKJS)    │          │   Internet   │
│             │          │                  │          │              │
│  fetch()  ──┼── msg ──>│  pebbleproxy   ──┼── HTTP ─>│  API server  │
│             │          │                  │          │              │
│  <── msg ───┼──────────┤  <── response ───┼──────────┤              │
│             │          │                  │          │              │
│  Location ──┼── msg ──>│  → GPS lookup    │          │              │
│  sensor     │          │  → sends coords  │          │              │
│  <── msg ───┼──────────┤  back to watch   │          │              │
└─────────────┘          └──────────────────┘          └──────────────┘
```


## Setting Up the Network Proxy

Install the `@moddable/pebbleproxy` package:

```text
$ pebble package install @moddable/pebbleproxy
```

Create (or update) `src/pkjs/index.js` to set up the proxy:

```js
const moddableProxy = require("@moddable/pebbleproxy");
Pebble.addEventListener('ready', moddableProxy.readyReceived);
Pebble.addEventListener('appmessage', moddableProxy.appMessageReceived);
```

That's it! The proxy handles forwarding `fetch()` requests and Location sensor
data between the watch and phone. No custom location code is needed in PKJS.


## Adding the Location Capability

Add the `location` capability to `package.json` so the phone is allowed to
access GPS:

```json
"capabilities": [
  "location"
]
```


## Preparing the Layout

We need a weather display at the bottom of the screen. Add a weather variable
near the top of `main.js`:

```js
let weather = null;
```

In `drawScreen()`, add the weather display after the date, before
`render.end()`:

```js
    // Draw weather at bottom
    const weatherY = render.height - smallFont.height -
        (render.height < 180 ? 6 : 20);
    if (weather) {
        const weatherStr = `${weather.temp}°C ${weather.conditions}`;
        width = render.getTextWidth(weatherStr, smallFont);
        render.drawText(weatherStr, smallFont, white,
            (render.width - width) / 2, weatherY);
    } else {
        const msg = "Loading...";
        width = render.getTextWidth(msg, smallFont);
        render.drawText(msg, smallFont, white,
            (render.width - width) / 2, weatherY);
    }
```


## Getting the User's Location

Alloy provides a `Location` sensor that follows the same ECMA-419 pattern as
the Battery sensor we used in Part 3. Import it at the top of `main.js`:

```js
import Location from "embedded:sensor/Location";
```

Create a `Location` instance. The `onSample` callback fires once the phone
has obtained the GPS coordinates:

```js
const location = new Location({
    onSample() {
        const sample = this.sample();
        console.log("Got location: " + sample.latitude + ", " + sample.longitude);
        this.close();
        fetchWeather(sample.latitude, sample.longitude);
    }
});
```

A few important things to note:

- **`this.close()`** — unlike Battery, we close the Location sensor after
  getting the first reading. Location is a one-shot request, not a continuous
  monitor.
- **Coordinates come as floats** — `sample.latitude` and `sample.longitude`
  are standard decimal degree values (e.g., `37.7749`, `-122.4194`). No need
  to multiply or divide by 10000 like in the C tutorial.
- **The proxy handles everything** — the `@moddable/pebbleproxy` package
  handles the phone-side GPS lookup automatically. You don't need any custom
  PKJS code for location.

Compare this to the C approach where you need custom PKJS code with
`navigator.geolocation`, `sendAppMessage`, integer coordinate encoding,
`AppMessage` handlers, and `messageKeys` in `package.json`. With the Location
sensor, it's just a few lines.


## Fetching Weather Data

Add a function to convert Open-Meteo weather codes:

```js
function getWeatherDescription(code) {
    if (code === 0) return "Clear";
    if (code <= 3) return "Cloudy";
    if (code <= 48) return "Fog";
    if (code <= 55) return "Drizzle";
    if (code <= 57) return "Fz. Drizzle";
    if (code <= 65) return "Rain";
    if (code <= 67) return "Fz. Rain";
    if (code <= 75) return "Snow";
    if (code <= 77) return "Snow Grains";
    if (code <= 82) return "Showers";
    if (code <= 86) return "Snow Shwrs";
    if (code === 95) return "T-Storm";
    if (code <= 99) return "T-Storm";
    return "Unknown";
}
```

Now the fetch function itself. Notice it takes `latitude` and `longitude` as
parameters — the Location sensor passes them directly:

```js
async function fetchWeather(latitude, longitude) {
    try {
        const url = new URL("http://api.open-meteo.com/v1/forecast");
        url.search = new URLSearchParams({
            latitude,
            longitude,
            current: "temperature_2m,weather_code"
        });

        console.log("Fetching weather...");
        const response = await fetch(url);
        const data = await response.json();

        weather = {
            temp: Math.round(data.current.temperature_2m),
            conditions: getWeatherDescription(data.current.weather_code)
        };

        console.log("Weather: " + weather.temp + "C, " + weather.conditions);
        drawScreen();

    } catch (e) {
        console.log("Weather fetch error: " + e);
    }
}
```

> **Why Open-Meteo?** Unlike many weather APIs, Open-Meteo is completely free
> and requires no API key. The URL is simple and the response is clean JSON.

Notice how clean this is compared to the C approach: no `XMLHttpRequest`, no
manual JSON parsing callbacks. Alloy's `fetch()` with `async`/`await` makes
the code straightforward.


## Automatic Refresh

To keep the weather current, trigger a refresh every 30 minutes. We create a
new `Location` instance each time, which requests fresh GPS coordinates and
chains into a weather fetch:

```js
watch.addEventListener("minutechange", e => {
    if (e.date.getMinutes() % 30 === 0) {
        new Location({
            onSample() {
                const sample = this.sample();
                this.close();
                fetchWeather(sample.latitude, sample.longitude);
            }
        });
    }
});
```


## Conclusion

In this part we learned how to:

1. Set up the `@moddable/pebbleproxy` package for network access.
2. Use the `Location` sensor to get GPS coordinates from the phone.
3. Use `fetch()` with `async`/`await` to call a web API.
4. Parse the Open-Meteo JSON response.
5. Display weather data on the watchface.
6. Set up automatic refresh via a time event.

Your watchface now shows live weather data! Check your code against
[the source for this part](https://github.com/coredevices/alloy-watchface-tutorial/tree/main/part4).


## What's Next?

In the next part we will add user settings — letting users choose colors, toggle
the date display, and pick temperature units — all persisted with
`localStorage`.

[Go to Part 5 &rarr; >{wide,bg-dark-red,fg-white}](/tutorials/alloy-watchface-tutorial/part5/)
