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
tutorial_part: 5

title: User Settings with localStorage
description: |
  How to persist user preferences using localStorage so users can
  customize their watchface.
permalink: /tutorials/alloy-watchface-tutorial/part5/
generate_toc: true
---

The finishing touch for any great watchface is letting users make it their own.
In this final part we will add user settings using `localStorage` — a simple
key-value store built right into Alloy. Users will be able to choose background
and text colors, toggle the date display, and pick temperature units.

This section continues from
[*Part 4*](/tutorials/alloy-watchface-tutorial/part4/).


## How localStorage Works in Alloy

Alloy provides the standard Web `localStorage` API for persistent storage.
Data is saved to flash and survives app restarts. The API is straightforward:

- `localStorage.setItem(key, value)` — store a string value
- `localStorage.getItem(key)` — retrieve a value (or `null` if missing)
- `localStorage.removeItem(key)` — delete a value

Since `localStorage` only stores strings, we use `JSON.stringify()` and
`JSON.parse()` to store structured data like our settings object.


## Defining Default Settings

Add a defaults object near the top of `main.js`, after the font declarations:

```js
const DEFAULT_SETTINGS = {
    backgroundColor: { r: 0, g: 0, b: 0 },
    textColor: { r: 255, g: 255, b: 255 },
    useFahrenheit: false,
    showDate: true
};
```

We store colors as `{ r, g, b }` objects so they serialize cleanly to JSON.


## Loading and Saving Settings

Add two helper functions:

```js
function loadSettings() {
    const stored = localStorage.getItem("settings");
    if (stored) {
        try {
            return { ...DEFAULT_SETTINGS, ...JSON.parse(stored) };
        } catch (e) {
            console.log("Failed to parse settings");
        }
    }
    return { ...DEFAULT_SETTINGS };
}

function saveSettings() {
    localStorage.setItem("settings", JSON.stringify(settings));
}

let settings = loadSettings();
```

The spread pattern `{ ...DEFAULT_SETTINGS, ...JSON.parse(stored) }` is
important. It ensures that if we add new settings in a future version, they
automatically get default values even when loading old saved data. This is the
Alloy equivalent of the C pattern of calling `prv_default_settings()` before
`persist_read_data()`.


## Creating Colors from Settings

Replace the hardcoded `black` and `white` color declarations with
settings-driven ones:

```js
let bgColor = render.makeColor(settings.backgroundColor.r,
    settings.backgroundColor.g, settings.backgroundColor.b);
let textColor = render.makeColor(settings.textColor.r,
    settings.textColor.g, settings.textColor.b);

function updateColors() {
    bgColor = render.makeColor(settings.backgroundColor.r,
        settings.backgroundColor.g, settings.backgroundColor.b);
    textColor = render.makeColor(settings.textColor.r,
        settings.textColor.g, settings.textColor.b);
}
```

The `updateColors()` function recreates the Poco color values whenever settings
change. Call it after loading new settings.


## Applying Settings to the Display

Update `drawScreen()` to use the settings-driven colors and respect the
`showDate` toggle:

```js
function drawScreen(event) {
    const now = event?.date ?? lastDate;
    if (event?.date) lastDate = event.date;

    render.begin();
    render.fillRectangle(bgColor, 0, 0, render.width, render.height);

    // ... battery bar and disconnect indicator ...

    // Draw time
    let width = render.getTextWidth(timeStr, timeFont);
    render.drawText(timeStr, timeFont, textColor,
        (render.width - width) / 2, timeY);

    // Draw date if setting is enabled
    if (settings.showDate) {
        // ... date drawing code ...
        render.drawText(dateStr, dateFont, textColor,
            (render.width - width) / 2, dateY);
    }

    // ... weather display ...

    render.end();
}
```

The key changes:
- `bgColor` replaces `black` for the background fill
- `textColor` replaces `white` for all text drawing
- The date block is wrapped in `if (settings.showDate)`

Also update the battery bar border to use `textColor` instead of hardcoded
white, so it matches the user's color choice:

```js
function drawBatteryBar() {
    // ...
    render.fillRectangle(textColor, barX, barY, barWidth, barHeight);
    render.fillRectangle(bgColor, barX + 1, barY + 1, barWidth - 2, barHeight - 2);
    // ...
}
```


## Temperature Unit Support

Update the weather display to show the correct unit:

```js
    if (weather) {
        const unit = settings.useFahrenheit ? "F" : "C";
        const weatherStr = `${weather.temp}°${unit} ${weather.conditions}`;
        // ...
    }
```

In `fetchWeather()`, pass the unit preference to Open-Meteo so the API returns
the temperature in the correct unit directly:

```js
async function fetchWeather() {
    // ...
    const params = {
        latitude,
        longitude,
        current: "temperature_2m,weather_code"
    };

    if (settings.useFahrenheit) {
        params.temperature_unit = "fahrenheit";
    }

    const url = new URL("http://api.open-meteo.com/v1/forecast");
    url.search = new URLSearchParams(params);
    // ...
}
```

Open-Meteo supports a `temperature_unit` parameter natively, so we do not need
to convert on the watch — the API handles it for us.


## Caching Weather Data

Since weather data is expensive to fetch (it requires a phone connection and
network request), we cache it in `localStorage` so the watchface can show
recent weather immediately on startup:

```js
function loadCachedWeather() {
    const cached = localStorage.getItem("weather");
    const cachedTime = localStorage.getItem("weatherTime");

    if (cached && cachedTime) {
        const age = Date.now() - Number(cachedTime);
        // Use cache if less than 1 hour old
        if (age < 60 * 60 * 1000) {
            try {
                weather = JSON.parse(cached);
                console.log("Using cached weather");
                return true;
            } catch (e) {
                console.log("Failed to parse cached weather");
            }
        }
    }
    return false;
}

function saveWeather() {
    if (weather) {
        localStorage.setItem("weather", JSON.stringify(weather));
        localStorage.setItem("weatherTime", String(Date.now()));
    }
}
```

Call `loadCachedWeather()` at startup, and call `saveWeather()` at the end of
a successful `fetchWeather()`:

```js
// At module level
loadCachedWeather();

// Inside fetchWeather(), after setting weather:
saveWeather();
drawScreen();
```

This gives users instant weather display on app launch instead of "Loading..."
while waiting for the phone connection and API response.


## Alloy vs. C Settings

In the C tutorial, settings are handled with Clay — a phone-side configuration
page that generates a settings UI and sends values to the watch via AppMessage.
The C watch stores them with `persist_write_data()`.

In Alloy, `localStorage` gives us a simpler approach:

| | C (Clay) | Alloy (localStorage) |
|---|---|---|
| **Settings UI** | Clay JSON config on phone | Not covered (can be done with a companion app) |
| **Storage** | `persist_write_data()` / `persist_read_data()` | `localStorage.setItem()` / `getItem()` |
| **Serialization** | Raw struct bytes | JSON strings |
| **New fields** | Must call defaults before load | Spread operator merges defaults |

> **Note:** Alloy does not currently have a built-in equivalent of Clay for
> generating a settings UI on the phone. The settings in this tutorial are
> stored and applied on the watch side. A phone-side settings page can be built
> as a companion web app in a future step.


## Conclusion

Congratulations! You have built a complete, feature-rich Pebble watchface in
Alloy. Here is everything it includes:

1. **Digital time display** with the Jersey font.
2. **Date display** that can be toggled on/off.
3. **Live weather** from Open-Meteo (no API key needed).
4. **Battery meter** with color-coded levels.
5. **Bluetooth disconnect** indicator.
6. **User settings** persisted with `localStorage`.
7. **Weather caching** for instant startup display.

In this final part we learned how to:

- Use `localStorage` for persistent key-value storage.
- Define default settings with a spread-merge pattern.
- Apply color and display preferences dynamically.
- Pass unit preferences to the Open-Meteo API.
- Cache API responses for faster startup.

Check your code against
[the source for this part](https://github.com/coredevices/alloy-watchface-tutorial/tree/main/part5).
Now it is time to
[publish your watchface](/guides/appstore-publishing/publishing-an-app/)
and share it with the world!
