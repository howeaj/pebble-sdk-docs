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
tutorial_part: 2

title: Adding Weather to an Alloy Watchface
description: A guide to adding web content to an Alloy JavaScript watchface
permalink: /tutorials/alloy-watchface-tutorial/part2/
menu_section: tutorials
generate_toc: true
---

In the [previous tutorial](/tutorials/alloy-watchface-tutorial/part1), we
created a basic digital and analog watchface using Alloy. In this tutorial,
we'll extend our watchface to display weather information fetched from the
internet.

## How Network Requests Work in Alloy

Alloy apps can make HTTP requests using the standard `fetch()` API. HTTP
requests are proxied through the connected phone by PebbleKit JS (PKJS). This
means you need:

1. A phone connected to your Pebble
2. The phone to have internet access
3. **The `@moddable/pebbleproxy` package installed in your project**

Here's how the pieces fit together:

```text
┌─────────────┐          ┌──────────────────┐          ┌──────────────┐
│   Watch     │          │  Phone (PKJS)    │          │   Internet   │
│             │          │                  │          │              │
│  fetch()  ──┼── msg ──>│  pebbleproxy   ──┼── HTTP ─>│  API server  │
│             │          │                  │          │              │
│  <── msg ───┼──────────┤  <── response ───┼──────────┤              │
│             │          │                  │          │              │
│  Message  ──┼── msg ──>│  appmessage      │          │              │
│  (location) │          │  → GPS lookup  ──┼──────────┤              │
│  <── msg ───┼──────────┤  → sendAppMsg    │          │              │
└─────────────┘          └──────────────────┘          └──────────────┘
```

## Setting Up the HTTP Proxy

Install the `@moddable/pebbleproxy` package in your project directory:

```nc|text
$ pebble package install @moddable/pebbleproxy
```

Then add the proxy to your `src/pkjs/index.js`:

```js
const moddableProxy = require("@moddable/pebbleproxy");
Pebble.addEventListener("ready", moddableProxy.readyReceived);
Pebble.addEventListener('appmessage', moddableProxy.appMessageReceived);
```

That's it! The proxy package hooks into `appmessage` events to forward
`fetch()` requests from your watch code through the phone's internet
connection.

## Using the fetch() API

Alloy supports the `fetch()` API for making HTTP requests. Use the `URL` and
`URLSearchParams` objects to build your request:

```js
async function getData() {
    const url = new URL("http://api.example.com/data");
    url.search = new URLSearchParams({
        param1: "value1",
        param2: "value2"
    });

    const response = await fetch(url);
    const json = await response.json();
    console.log("Got data: " + JSON.stringify(json));
}
```

> **Note**: The phone connection must be established before network requests
> can succeed. You can listen for connection state changes with
> `Pebble.addEventListener("connected", ...)`.

## Building a Weather Watchface

Let's create a watchface that displays the current time and weather conditions.
We'll use the [Open-Meteo API](https://open-meteo.com/) which provides free
weather data with no API key required!

![weather](/images/tutorials/alloy-watchface-tutorial/weather-watchface.png)

### Step 1: Add Weather to the Digital Watchface

Start from the digital watchface we built in
[Part 1](/tutorials/alloy-watchface-tutorial/part1/). We need to add three
things: a `weather` variable, a `drawWeather()` function, and a weather
conditional in `draw()`.

Add the weather variable near the top of `main.js`, after the font
declarations:

```js
// Weather data (will be populated by fetch)
let weather = null;
```

Add a `drawWeather()` function:

```js
function drawWeather() {
    // Temperature in yellow at top
    const tempStr = `${weather.temp}°C`;
    let width = render.getTextWidth(tempStr, weatherFont);
    render.drawText(tempStr, weatherFont, yellow,
        (render.width - width) / 2, 20);

    // Conditions in orange at bottom
    width = render.getTextWidth(weather.conditions, conditionsFont);
    render.drawText(weather.conditions, conditionsFont, orange,
        (render.width - width) / 2, render.height - conditionsFont.height - 20);
}
```

Then update `draw()` to show weather when available. Since `draw()` may also
be called manually (e.g., after fetching weather), we use `event?.date` with a
fallback:

```js
function draw(event) {
    const now = event?.date ?? new Date();

    render.begin();
    render.fillRectangle(teal, 0, 0, render.width, render.height);

    // Draw time in white
    const hours = now.getHours().toString().padStart(2, "0");
    const minutes = now.getMinutes().toString().padStart(2, "0");
    const timeStr = `${hours}:${minutes}`;

    let width = render.getTextWidth(timeStr, timeFont);
    render.drawText(timeStr, timeFont, white,
        (render.width - width) / 2,
        (render.height - timeFont.height) / 2);

    // Draw weather if available
    if (weather) {
        drawWeather();
    } else {
        // Show loading message
        const msg = "Loading...";
        width = render.getTextWidth(msg, conditionsFont);
        render.drawText(msg, conditionsFont, white,
            (render.width - width) / 2, 20);
    }

    render.end();
}
```

You'll also need two additional fonts — add these with your other font
declarations:

```js
const weatherFont = new render.Font("Gothic-Bold", 28);
const conditionsFont = new render.Font("Gothic-Regular", 24);
```

### Step 2: Understanding the Open-Meteo API

Open-Meteo provides weather data through a simple URL structure:

```
http://api.open-meteo.com/v1/forecast?latitude=LAT&longitude=LON&current=temperature_2m,weather_code
```

> **Note**: Use `http://` for API requests — the PebbleKit JS proxy handles
> the connection.

The response looks like:

```json
{
  "current": {
    "temperature_2m": 18.5,
    "weather_code": 1
  }
}
```

Weather codes indicate conditions (0 = Clear, 1-3 = Cloudy, 61-67 = Rain, etc.).

### Step 3: Get Real Location

To use the phone's GPS, we need to set up messaging between the watch and phone.
First, add message keys to your `package.json` in the `pebble` section:

```json
"messageKeys": {
    "LATITUDE": 0,
    "LONGITUDE": 1,
    "REQUEST_LOCATION": 2
}
```

Then update `src/pkjs/index.js` to handle location requests. Since we now have
our own app messages alongside the proxy, we use `appMessageReceived()` to let the
proxy handle its messages first:

```js
const moddableProxy = require("@moddable/pebbleproxy");

Pebble.addEventListener("ready", moddableProxy.readyReceived);

Pebble.addEventListener('appmessage', function(e) {
    if (moddableProxy.appMessageReceived(e))
        return;

    // Check if this is a location request
    if (e.payload['REQUEST_LOCATION'] !== undefined) {
        console.log("Location requested");

        navigator.geolocation.getCurrentPosition(
            function(pos) {
                console.log("Got location: " + pos.coords.latitude + ", " + pos.coords.longitude);
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

### Step 4: Fetch Weather with Location

Now update `main.js` to request location and fetch weather:

```js
import Message from "pebble/message";

// Store location when received
let latitude = null;
let longitude = null;

// Set up messaging to receive location
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
            // Coordinates are multiplied by 10000 to preserve precision
            latitude = msg.get("LATITUDE") / 10000;
            longitude = msg.get("LONGITUDE") / 10000;
            console.log("Got location: " + latitude + ", " + longitude);
            fetchWeather();
        }
    },
    onWritable() {
        // Request location when connection is ready
        if (!this.requested) {
            this.requested = true;
            console.log("Requesting location...");
            this.write(new Map([["REQUEST_LOCATION", true]]));
        }
    }
});

// Map Open-Meteo weather codes to descriptions
function getWeatherDescription(code) {
    if (code === 0) return "Clear";
    if (code <= 3) return "Cloudy";
    if (code <= 49) return "Fog";
    if (code <= 59) return "Drizzle";
    if (code <= 69) return "Rain";
    if (code <= 79) return "Snow";
    if (code <= 99) return "Thunderstorm";
    return "Unknown";
}

async function fetchWeather() {
    if (latitude === null || longitude === null) {
        console.log("No location yet");
        return;
    }

    try {
        const url = new URL("http://api.open-meteo.com/v1/forecast");
        url.search = new URLSearchParams({
            latitude,
            longitude,
            current: "temperature_2m,weather_code"
        });

        console.log("Fetching weather for " + latitude + ", " + longitude);
        const response = await fetch(url);
        const data = await response.json();

        weather = {
            temp: Math.round(data.current.temperature_2m),
            conditions: getWeatherDescription(data.current.weather_code)
        };

        console.log("Weather: " + weather.temp + "C, " + weather.conditions);
        draw();

    } catch (e) {
        console.log("Weather fetch error: " + e);
    }
}
```

### Step 5: Schedule Weather Updates

Add code to refresh weather periodically:

```js
// Refresh weather every hour
Pebble.addEventListener('hourchange', fetchWeather);
```

### Complete Code

Here's the complete `main.js` with real location support:

```js
import Poco from "commodetto/Poco";
import Message from "pebble/message";

const render = new Poco(screen);

// Colors - teal and orange theme
const teal = render.makeColor(0, 128, 128);
const white = render.makeColor(255, 255, 255);
const yellow = render.makeColor(255, 215, 0);
const orange = render.makeColor(255, 140, 0);

// Fonts - Leco for big digits, Gothic-Bold for weather
const timeFont = new render.Font("Leco-Regular", 42);
const weatherFont = new render.Font("Gothic-Bold", 28);
const conditionsFont = new render.Font("Gothic-Regular", 24);

// Weather and location data
let weather = null;
let latitude = null;
let longitude = null;

// Set up messaging to receive location from phone
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
            console.log("Got location: " + latitude + ", " + longitude);
            fetchWeather();
        }
    },
    onWritable() {
        if (!this.requested) {
            this.requested = true;
            console.log("Requesting location...");
            this.write(new Map([["REQUEST_LOCATION", true]]));
        }
    }
});

// Map Open-Meteo weather codes to descriptions
function getWeatherDescription(code) {
    if (code === 0) return "Clear";
    if (code <= 3) return "Cloudy";
    if (code <= 49) return "Fog";
    if (code <= 59) return "Drizzle";
    if (code <= 69) return "Rain";
    if (code <= 79) return "Snow";
    if (code <= 99) return "Thunderstorm";
    return "Unknown";
}

function drawWeather() {
    const tempStr = `${weather.temp}°C`;
    let width = render.getTextWidth(tempStr, weatherFont);
    render.drawText(tempStr, weatherFont, yellow,
        (render.width - width) / 2, 20);

    width = render.getTextWidth(weather.conditions, conditionsFont);
    render.drawText(weather.conditions, conditionsFont, orange,
        (render.width - width) / 2, render.height - conditionsFont.height - 20);
}

function draw(event) {
    const now = event?.date ?? new Date();

    render.begin();
    render.fillRectangle(teal, 0, 0, render.width, render.height);

    // Draw time in white
    const hours = now.getHours().toString().padStart(2, "0");
    const minutes = now.getMinutes().toString().padStart(2, "0");
    const timeStr = `${hours}:${minutes}`;

    let width = render.getTextWidth(timeStr, timeFont);
    render.drawText(timeStr, timeFont, white,
        (render.width - width) / 2,
        (render.height - timeFont.height) / 2);

    // Draw weather if available
    if (weather) {
        drawWeather();
    } else {
        const msg = "Loading...";
        width = render.getTextWidth(msg, conditionsFont);
        render.drawText(msg, conditionsFont, white,
            (render.width - width) / 2, 20);
    }

    render.end();
}

async function fetchWeather() {
    if (latitude === null || longitude === null) {
        console.log("No location yet");
        return;
    }

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
        draw();

    } catch (e) {
        console.log("Weather fetch error: " + e);
    }
}

// Time updates (fires immediately when registered — no startup draw needed)
Pebble.addEventListener('minutechange', draw);

// Refresh weather every hour
Pebble.addEventListener('hourchange', fetchWeather);
```

And the complete `src/pkjs/index.js`:

```js
const moddableProxy = require("@moddable/pebbleproxy");

Pebble.addEventListener("ready", moddableProxy.readyReceived);

Pebble.addEventListener('appmessage', function(e) {
    if (moddableProxy.appMessageReceived(e))
        return;

    if (e.payload.REQUEST_LOCATION !== undefined) {
        console.log("Location requested");

        navigator.geolocation.getCurrentPosition(
            function(pos) {
                console.log("Got location: " + pos.coords.latitude + ", " + pos.coords.longitude);
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

## Storing Weather Data

You might want to persist weather data so it's available immediately when the
watchface starts. Use `localStorage`:

```js
// Save weather when fetched
function saveWeather() {
    if (weather) {
        localStorage.setItem("weather", JSON.stringify(weather));
        localStorage.setItem("weatherTime", Date.now());
    }
}

// Load cached weather on startup
function loadCachedWeather() {
    const cached = localStorage.getItem("weather");
    const cachedTime = localStorage.getItem("weatherTime");

    if (cached && cachedTime) {
        const age = Date.now() - Number(cachedTime);
        // Use cache if less than 1 hour old
        if (age < 60 * 60 * 1000) {
            weather = JSON.parse(cached);
            console.log("Using cached weather");
            return true;
        }
    }
    return false;
}

// Call on startup
if (!loadCachedWeather()) {
    fetchWeather();
}
```

Don't forget to call `saveWeather()` after successfully fetching new data!

## Error Handling Best Practices

Always handle network errors gracefully:

```js
async function fetchWeather() {
    if (latitude === null || longitude === null) {
        console.log("No location yet");
        return;
    }

    try {
        const url = new URL("http://api.open-meteo.com/v1/forecast");
        url.search = new URLSearchParams({
            latitude,
            longitude,
            current: "temperature_2m,weather_code"
        });

        const response = await fetch(url);
        const data = await response.json();
        // Process data...

    } catch (e) {
        console.log("Error: " + e);

        // Show error to user
        weather = {
            temp: "--",
            conditions: "No data"
        };
        draw();
    }
}
```

## Adding Battery Status

Let's also add battery status to make a more complete watchface:

```js
import Battery from "embedded:sensor/Battery";

let batteryPercent = 100;

const battery = new Battery({
    onSample() {
        batteryPercent = this.sample().percent;
        draw();
    }
});
batteryPercent = battery.sample().percent;

// In draw(), add:
function drawBattery() {
    const batteryStr = `${batteryPercent}%`;
    const width = render.getTextWidth(batteryStr, conditionsFont);
    render.drawText(batteryStr, conditionsFont, white,
        render.width - width - 10, 15);
}
```

## Using Fahrenheit

If you prefer Fahrenheit, add `temperature_unit` to the URL parameters:

```js
const url = new URL("http://api.open-meteo.com/v1/forecast");
url.search = new URLSearchParams({
    latitude,
    longitude,
    current: "temperature_2m,weather_code",
    temperature_unit: "fahrenheit"
});
```

Then update the display string:

```js
const tempStr = `${weather.temp}°F`;
```

## Resources

- [Complete source code for this tutorial](https://github.com/coredevices/alloy-watchface-part2)
- [Poco Graphics Guide](/guides/alloy/poco-guide/) — full reference for the
  Poco drawing API

## Conclusion

You've learned how to:

1. Use the `fetch()` API to make HTTP requests
2. Parse JSON responses from the Open-Meteo API
3. Display dynamic weather data on your watchface
4. Cache data using `localStorage`
5. Handle network errors gracefully
6. Add battery status monitoring

Your watchface now displays the current time and weather conditions!

## What's Next

Now that you've completed this tutorial, here are some ideas for extending your
watchface:

- Add configurable locations
- Display weather icons
- Show sunrise/sunset times (Open-Meteo provides these!)
- Add step count from Pebble Health
- Create multiple watchface themes

Check out the [Alloy Guides](/guides/alloy/) for more information on advanced
features.

## Get Help

If you have questions or want to share what you've built:

- [Pebble Forums](https://forum.repebble.com/c/developers-ask-questions-and-get-help)
- [Discord Server]({{ site.links.discord_invite }})
