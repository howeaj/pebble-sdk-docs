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
tutorial_part: 1

title: Build a Watchface in JavaScript using Alloy
description: A guide to making a new Pebble watchface with Alloy
permalink: /tutorials/alloy-watchface-tutorial/part1/
menu_section: tutorials
generate_toc: true
---

In this tutorial we'll cover the basics of writing a watchface with Alloy,
Pebble's modern JavaScript framework based on [Moddable](https://www.moddable.com/).
Alloy enables developers to create beautiful and feature-rich watchfaces using
modern JavaScript (ES6+).

We're going to start with some basics, then create a colorful digital watchface
and finally create an analog clock.

> **Platform Support**: Alloy currently supports Emery (Pebble Time 2) and
> Gabbro (Pebble Round 2).

![digital](/images/tutorials/alloy-watchface-tutorial/digital-watchface.png)

## First Steps

If you haven't already, head over to the [SDK Page](/sdk/install/) to learn how
to download and install the latest version of the Pebble Tool and SDK.

Once you have the Pebble Tool installed, create a new Alloy watchface project
with the following command:

```nc|text
$ pebble new-project --alloy mywatchface
```

This will create a new folder called `mywatchface` with the basic structure
required for an Alloy watchface application. The most important file is
`src/embeddedjs/main.js` — this is where you'll write your watchface code.
(See the [Appendix](#appendix-project-structure) for the full project
structure.)

## Understanding the Default Watchface

Open `src/embeddedjs/main.js` and you'll see the default watchface code:

```js
import Poco from "commodetto/Poco";

console.log("Hello, Watchface.");

let render = new Poco(screen);

const font = new render.Font("Bitham-Black", 30);
const black = render.makeColor(0, 0, 0);
const white = render.makeColor(255, 255, 255);

function draw(event) {
    const now = event.date;

    render.begin();
    render.fillRectangle(white, 0, 0, render.width, render.height);

    const msg = now.toTimeString().slice(0, 8);
    const width = render.getTextWidth(msg, font);

    render.drawText(msg, font, black,
        (render.width - width) / 2, (render.height - font.height) / 2);

    render.end();
}

Pebble.addEventListener('secondchange', draw);
```

Let's break down what's happening:

1. **Import Poco**: We import the Poco graphics library for drawing
2. **Create a renderer**: `new Poco(screen)` creates a renderer for the display
3. **Set up colors and fonts**: We define our colors and load a font
4. **Draw function**: The `draw` function receives an `event` parameter with an
   `event.date` property containing the current time — no need to call
   `new Date()`
5. **Event listener**: We register for `secondchange` events. Time events fire
   immediately when registered, so the watchface draws right away without
   needing an explicit startup call

## Watchface Basics

Watchfaces are long-running applications that update the display at regular
intervals. By minimizing how often the screen is updated, we conserve battery
life.

### Time Events

Alloy provides several time-related events through the `Pebble` global:

| Event | Description |
|-------|-------------|
| `secondchange` | Fires every second |
| `minutechange` | Fires every minute |
| `hourchange` | Fires every hour |
| `daychange` | Fires every day |

For most watchfaces, use `minutechange` to save battery:

```js
Pebble.addEventListener('minutechange', draw);
```

Only use `secondchange` if you need to display seconds.

### The Poco Renderer

Poco is a low-level graphics library that gives you precise control over
drawing. All drawing happens between `begin()` and `end()` calls:

```js
render.begin();
    // Drawing commands go here
render.end();
```

## Using Regular JavaScript

Alloy runs standard JavaScript (ES6+) on the XS engine. You can use modern
features like `const`/`let`, arrow functions, template literals, destructuring,
`async`/`await`, classes, and ES modules.

A few things to keep in mind:

- **No `eval()` or `new Function()`** — dynamic code generation is not
  supported
- **Frozen primordials** — built-in prototypes (like `Array.prototype`) are
  frozen and cannot be modified
- **No arbitrary npm packages** — the XS engine is not Node.js. Stick to
  plain JavaScript and Alloy/Moddable modules
- **Strict mode by default** — all code runs in strict mode

For more details, see the
[Getting Started guide](/guides/alloy/getting-started/).

## Creating a Digital Watchface

Let's create a colorful digital watchface. Replace the contents of `main.js`
with:

```js
import Poco from "commodetto/Poco";

const render = new Poco(screen);

// Colors - let's make it vibrant!
const darkBlue = render.makeColor(25, 25, 112);
const white = render.makeColor(255, 255, 255);
const cyan = render.makeColor(0, 255, 255);
const orange = render.makeColor(255, 165, 0);

// Fonts - Leco-Regular 42 is perfect for big watchface digits
const timeFont = new render.Font("Leco-Regular", 42);
const dateFont = new render.Font("Gothic-Bold", 24);

// Day and month names
const DAYS = ["Sunday", "Monday", "Tuesday", "Wednesday",
              "Thursday", "Friday", "Saturday"];
const MONTHS = ["January", "February", "March", "April", "May", "June",
                "July", "August", "September", "October", "November", "December"];

function draw(event) {
    const now = event.date;

    render.begin();

    // Dark blue background
    render.fillRectangle(darkBlue, 0, 0, render.width, render.height);

    // Format time as HH:MM
    const hours = now.getHours().toString().padStart(2, "0");
    const minutes = now.getMinutes().toString().padStart(2, "0");
    const timeStr = `${hours}:${minutes}`;

    // Draw time in cyan - centered
    let width = render.getTextWidth(timeStr, timeFont);
    render.drawText(timeStr, timeFont, cyan,
        (render.width - width) / 2,
        (render.height - timeFont.height) / 2 - 30);

    // Draw day of week in white
    const dayName = DAYS[now.getDay()];
    width = render.getTextWidth(dayName, dateFont);
    render.drawText(dayName, dateFont, white,
        (render.width - width) / 2,
        (render.height - timeFont.height) / 2 + 30);

    // Draw date in orange
    const dateStr = `${MONTHS[now.getMonth()]} ${now.getDate()}`;
    width = render.getTextWidth(dateStr, dateFont);
    render.drawText(dateStr, dateFont, orange,
        (render.width - width) / 2,
        (render.height - timeFont.height) / 2 + 60);

    render.end();
}

// Update every minute (saves battery compared to every second)
// Time events fire immediately when registered, so no explicit startup draw is needed
Pebble.addEventListener('minutechange', draw);
```

## First Compilation and Installation

To compile the watchface, save your file and run:

```nc|text
$ pebble build
```

After a successful build, you'll see `'build' finished successfully`.

Install and run in the emulator:

```nc|text
$ pebble install --emulator emery --logs
```

You should see your colorful digital watchface running!

## Creating an Analog Watchface

Now let's create an analog watchface with hour and minute hands.

![analog](/images/tutorials/alloy-watchface-tutorial/analog-watchface.png)

Replace `main.js` with:

```js
import Poco from "commodetto/Poco";

const render = new Poco(screen);

// Colors
const darkGray = render.makeColor(40, 40, 40);
const white = render.makeColor(255, 255, 255);
const red = render.makeColor(255, 60, 60);
const gold = render.makeColor(255, 215, 0);
const lightBlue = render.makeColor(100, 149, 237);

// Helper: Convert time fraction to radians
function fractionToRadians(fraction) {
    return fraction * 2 * Math.PI;
}

// Draw a clock hand from center outward
function drawHand(cx, cy, angle, length, color, thickness) {
    const x2 = cx + Math.sin(angle) * length;
    const y2 = cy - Math.cos(angle) * length;
    render.drawLine(cx, cy, x2, y2, color, thickness);
}

function draw(event) {
    const now = event.date;
    const hours = now.getHours() % 12;
    const minutes = now.getMinutes();

    // Calculate center and hand length
    const cx = render.width / 2;
    const cy = render.height / 2;
    const maxLength = (Math.min(render.width, render.height) - 30) / 2;

    render.begin();

    // Dark background
    render.fillRectangle(darkGray, 0, 0, render.width, render.height);

    // Draw hour markers
    for (let i = 0; i < 12; i++) {
        const angle = fractionToRadians(i / 12);
        const isMainHour = (i % 3 === 0);
        const innerRadius = isMainHour ? maxLength - 15 : maxLength - 8;
        const outerRadius = maxLength;
        const color = isMainHour ? gold : white;
        const thickness = isMainHour ? 3 : 2;

        // Cache trig values to avoid redundant computation
        const sinAngle = Math.sin(angle);
        const cosAngle = Math.cos(angle);

        const x1 = cx + sinAngle * innerRadius;
        const y1 = cy - cosAngle * innerRadius;
        const x2 = cx + sinAngle * outerRadius;
        const y2 = cy - cosAngle * outerRadius;

        render.drawLine(x1, y1, x2, y2, color, thickness);
    }

    // Calculate hand angles
    const minuteFraction = minutes / 60;
    const hourFraction = (hours + minuteFraction) / 12;
    const minuteAngle = fractionToRadians(minuteFraction);
    const hourAngle = fractionToRadians(hourFraction);

    // Draw hands - gold hour, light blue minute
    drawHand(cx, cy, hourAngle, maxLength * 0.5, gold, 6);
    drawHand(cx, cy, minuteAngle, maxLength * 0.75, lightBlue, 4);

    // Center dot
    render.drawCircle(red, cx, cy, 6, 0, 360);
    render.drawCircle(white, cx, cy, 3, 0, 360);

    render.end();
}

// Update every minute
// Time events fire immediately when registered, so no explicit startup draw is needed
Pebble.addEventListener('minutechange', draw);
```

Build and run to see your analog watchface!

> **Performance Tips**: On an embedded device, every bit of efficiency helps
> battery life. A few techniques to keep in mind:
>
> - **Cache trig results**: `Math.sin()` and `Math.cos()` are expensive. If
>   you use the same angle multiple times, store the result in a local
>   variable (as we did in the hour markers loop above).
> - **Integer division**: JavaScript division returns floating-point values,
>   which forces all subsequent math to floating-point. To stay in the integer
>   domain, use `Math.idiv(a, b)` (a Moddable extension), `(a / b) | 0`, or
>   `a >> 1` for dividing by 2.
> - **Minimize redraws**: Use `minutechange` instead of `secondchange` unless
>   you truly need per-second updates.

## Adding a Second Hand

If you want a second hand, add this to the `draw()` function and change the
event listener:

```js
// In draw(), after minute hand:
const seconds = now.getSeconds();
const secondFraction = seconds / 60;
const secondAngle = fractionToRadians(secondFraction);
drawHand(cx, cy, secondAngle, maxLength * 0.85, red, 2);

// Change event listener to secondchange:
Pebble.addEventListener('secondchange', draw);
```

> **Note**: Using `secondchange` will increase battery consumption since the
> screen updates 60 times more frequently.

## Troubleshooting

### Build Errors

If your build fails, check the error output for line numbers and descriptions.
Common issues:

- **Syntax errors**: Missing semicolons, brackets, or typos
- **Import errors**: Make sure module names are correct
- **Undefined variables**: Check spelling and scope

### Debugging with Logs

Add `console.log()` statements to trace execution:

```js
function draw() {
    console.log("Drawing at " + new Date().toTimeString());
    // ...
}
```

View logs with:

```nc|text
$ pebble logs --emulator emery
```

### Getting Help

If you're stuck, check out these resources:

- [Pebble Forums](https://forum.repebble.com/c/developers-ask-questions-and-get-help)
- [Discord Server]({{ site.links.discord_invite }})

## Conclusion

You've learned how to:

1. Create a new Alloy watchface project with `pebble new-project --alloy`
2. Use the Poco renderer for drawing graphics
3. Subscribe to time events like `minutechange` and `secondchange`
4. Create both digital and analog watchface displays
5. Build and install your watchface

## Resources

- [Complete source code for this tutorial](https://github.com/coredevices/alloy-watchface-part1)
- [Poco Graphics Guide](/guides/alloy/poco-guide/) — full reference for the
  Poco drawing API
- [Available Fonts](/guides/alloy/poco-guide/#using-pebble-built-in-fonts) —
  fonts bundled with the Pebble SDK

## What's Next

In the next part of this tutorial, we'll add weather information to the
watchface by fetching data from the internet using the `fetch()` API.

[Go to Part 2 &rarr; >{wide,bg-dark-red,fg-white}](/tutorials/alloy-watchface-tutorial/part2/)

---

## Appendix: Project Structure

Here are the files created by `pebble new-project --alloy`:

```nc|text
mywatchface/
  src/
    embeddedjs/
      main.js           # Your watchface code
      manifest.json     # Module configuration
    c/
      mdbl.c            # C bootstrap code (don't modify)
  package.json          # App metadata and configuration
  wscript               # Build script
```
