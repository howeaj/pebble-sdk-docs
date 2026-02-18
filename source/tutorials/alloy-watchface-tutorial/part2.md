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

title: Customizing Your Watchface
description: |
  How to use a custom font and properly center the layout to give your
  watchface a unique look.
permalink: /tutorials/alloy-watchface-tutorial/part2/
generate_toc: true
---

In the previous part we created a basic watchface that displays the time and
date using `Bitham-Bold`. It works, but it looks like every other watchface out
there. Let's fix that by switching to a custom font - the same Jersey font used
in the C watchface tutorial - and improving the layout.

By the end of this part, your watchface will look something like this:

{% screenshot_viewer %}
{
  "image": "/images/tutorials/alloy-watchface-tutorial/part2.png",
  "default": "emery",
  "platforms": [
    {"hw": "emery", "wrapper": "core-time2-red"},
    {"hw": "gabbro", "wrapper": "core-time-round2-black-20"}
  ]
}
{% endscreenshot_viewer %}

This section continues from
[*Part 1*](/tutorials/alloy-watchface-tutorial/part1/), so be sure to re-use
your code or start with that finished project.


## Adding a Custom Font

Alloy supports custom TrueType fonts through the Moddable SDK's font pipeline.
The build system converts `.ttf` files into optimized bitmap resources that Poco
can render efficiently.

### Getting the Font File

Download
[Jersey10-Regular.ttf](https://fonts.google.com/specimen/Jersey+10) (it's a
free Google Font) and place it in your project:

```text
src/
  embeddedjs/
    assets/
      Jersey10-Regular.ttf
    main.js
    manifest.json
```

### Declaring Font Resources

Create `src/embeddedjs/manifest.json` (or update it if it already exists). This
tells the build system to convert the font into bitmap resources at the sizes
we need:

```json
{
    "include": [
        "$(MODDABLE)/examples/manifest_mod.json"
    ],
    "modules": {
        "*": "./main.js"
    },
    "resources": {
        "*-alpha": [
            {
                "source": "./assets/Jersey10-Regular",
                "size": 56,
                "monochrome": true,
                "blocks": ["Basic Latin"]
            },
            {
                "source": "./assets/Jersey10-Regular",
                "size": 24,
                "monochrome": true,
                "blocks": ["Basic Latin"]
            }
        ]
    }
}
```

Key properties:

- **`source`** - path to the `.ttf` file (without the extension), relative to
  the manifest
- **`size`** - font size in pixels to render at
- **`monochrome`** - `true` for crisp 1-bit rendering (ideal for Pebble)
- **`blocks`** - which Unicode character blocks to include. `"Basic Latin"`
  covers the digits, letters, and punctuation we need. Including only the
  characters you need saves memory.

We declare the font at two sizes: 56 for the time display (same as the C
tutorial's `FONT_JERSEY_56`) and 24 for the date.


### Loading Custom Fonts in Code

In the C SDK, you load a custom font with `fonts_load_custom_font()`. In Alloy,
you load the two generated resource files (`.fnt` for metrics, `-alpha.bm4` for
pixel data) and combine them:

```js
import parseBMF from "commodetto/parseBMF";
import parseRLE from "commodetto/parseRLE";

function getFont(name, size) {
    const font = parseBMF(new Resource(`${name}-${size}.fnt`));
    font.bitmap = parseRLE(new Resource(`${name}-${size}-alpha.bm4`));
    return font;
}
```

Now replace the built-in font declarations with custom font loading:

```js
// Was: const timeFont = new render.Font("Bitham-Bold", 42);
// Was: const dateFont = new render.Font("Gothic-Bold", 24);
const timeFont = getFont("Jersey10-Regular", 56);
const dateFont = getFont("Jersey10-Regular", 24);
```

The returned font object works exactly like a built-in font - you can pass it
to `render.drawText()`, `render.getTextWidth()`, and read its `.height`
property.


## Centering the Layout

In Part 1 we positioned the time and date using hardcoded offsets. Let's
properly center the time+date block vertically by calculating positions based on
the font metrics.

Add these calculations at the top level, after the font declarations:

```js
// Precompute layout positions
const blockHeight = timeFont.height + dateFont.height;
const timeY = (render.height - blockHeight) / 2;
const dateY = timeY + timeFont.height;
```

The `height` property on a font gives the line height in pixels. We center the
combined block on screen by computing half the total height.

Since these values only depend on the font sizes and screen dimensions, we
compute them once at startup rather than every frame. This is a good habit for
embedded development - precompute what you can.

Now update the `draw()` function to use `timeY` and `dateY`:

```js
function draw(event) {
    const now = event.date;

    render.begin();
    render.fillRectangle(black, 0, 0, render.width, render.height);

    // Format time as HH:MM
    const hours = String(now.getHours()).padStart(2, "0");
    const minutes = String(now.getMinutes()).padStart(2, "0");
    const timeStr = `${hours}:${minutes}`;

    // Draw time centered
    let width = render.getTextWidth(timeStr, timeFont);
    render.drawText(timeStr, timeFont, white,
        (render.width - width) / 2, timeY);

    // Format date as "Mon Jan 01"
    const dayName = DAYS[now.getDay()];
    const monthName = MONTHS[now.getMonth()];
    const dateStr = `${dayName} ${monthName} ${String(now.getDate()).padStart(2, "0")}`;

    // Draw date centered below time
    width = render.getTextWidth(dateStr, dateFont);
    render.drawText(dateStr, dateFont, white,
        (render.width - width) / 2, dateY);

    render.end();
}
```

Compile and install with `pebble build && pebble install`. You should see the
watchface now uses Jersey - the same distinctive font from the C tutorial - with
the time and date properly centered as a block.


## C vs. Alloy Custom Fonts

| | C SDK | Alloy |
|---|---|---|
| **Font file** | `.ttf` in `resources/fonts/` | `.ttf` in `src/embeddedjs/assets/` |
| **Declaration** | `package.json` resources array | `manifest.json` `*-alpha` resources |
| **Loading** | `fonts_load_custom_font()` | `parseBMF()` + `parseRLE()` |
| **Character subsetting** | All glyphs included | Specify `blocks` or `characters` |
| **Cleanup** | `fonts_unload_custom_font()` | Automatic (garbage collected) |


## Experimenting

Here are some things you can try:

- Switch to a different TTF font - any TrueType font works.
- Change the font sizes to see how the layout adapts.
- Add a third font size for a different text element.
- Try different `blocks` values like `"Latin Extended-A"` for accented
  characters.

> **Tip**: Use `"monochrome": true` for sharp text on Pebble's display. Omit
> it for anti-aliased rendering on higher-color displays.


## Conclusion

In this part we learned how to:

1. Add a custom TrueType font to an Alloy project.
2. Declare font resources in `manifest.json` with character subsetting.
3. Load custom fonts with `parseBMF` and `parseRLE`.
4. Center the time+date block vertically using font metrics.
5. Precompute layout positions for better performance.

Your watchface now has the same distinctive Jersey look as the C tutorial.
Check your code against
[the source for this part](https://github.com/coredevices/alloy-watchface-tutorial/tree/main/part2)
if you run into any issues.


## What's Next?

In the next part we will add a battery meter and connection disconnect alerts
to give users useful information at a glance.

[Go to Part 3 &rarr; >{wide,bg-dark-red,fg-white}](/tutorials/alloy-watchface-tutorial/part3/)
