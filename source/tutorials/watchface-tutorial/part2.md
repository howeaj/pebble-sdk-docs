---
# Copyright 2026 Core Devices LLC
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
tutorial: watchface
tutorial_part: 2

title: Customizing Your Watchface
description: |
  How to add custom fonts to give your watchface a unique look.
permalink: /tutorials/watchface-tutorial/part2/
generate_toc: true
---

In the previous part we created a basic watchface that displays the time and
date using system fonts. It works, but it looks like every other watchface out
there. Let's fix that by adding a custom font.

By the end of this part, your watchface will look something like this:

{% screenshot_viewer %}
{
  "image": "/images/tutorials/watchface-tutorial/part2.png",
  "platforms": [
    {"hw": "aplite", "wrapper": "steel-black"},
    {"hw": "basalt", "wrapper": "time-red"},
    {"hw": "chalk", "wrapper": "time-round-rosegold-14"},
    {"hw": "diorite", "wrapper": "pebble2-black"},
    {"hw": "emery", "wrapper": ""}
  ]
}
{% endscreenshot_viewer %}


## How Resources Work

App resources - fonts, images, and other data files - are managed through the
`media` array in `package.json`. Each entry specifies the resource type, a name
to reference it in code, and the path to the file.

All resource files must be placed inside the `resources/` directory of your
project.


## Adding a Custom Font

A custom font must be a [TrueType](http://en.wikipedia.org/wiki/TrueType) font
in the `.ttf` file format. For this tutorial we will use
[Jersey 10](https://fonts.google.com/specimen/Jersey+10) from Google Fonts, but
you can use any `.ttf` font you like.

Place your font file in `resources/fonts/` and add entries to the `media` array
in `package.json`. We will register the same font twice at different sizes -
one large size for the time, and a smaller one for the date:

```json
"resources": {
  "media": [
    {
      "type": "font",
      "name": "FONT_JERSEY_56",
      "file": "fonts/Jersey10-Regular.ttf",
      "compatibility": "2.7"
    },
    {
      "type": "font",
      "name": "FONT_JERSEY_24",
      "file": "fonts/Jersey10-Regular.ttf",
      "compatibility": "2.7"
    }
  ]
}
```

The `name` field becomes a constant you can reference in C code, prefixed with
`RESOURCE_ID_`. The number at the end of the name (56, 24) is just part of the
name you chose - it serves as a reminder of the intended font size.


## Loading Custom Fonts in C

Back in `main.c`, declare two ``GFont`` variables at the top of the file to
hold our loaded fonts:

```c
static GFont s_time_font;
static GFont s_date_font;
```

Load them in `main_window_load()` using ``fonts_load_custom_font()`` and
``resource_get_handle()``:

```c
// Load custom fonts
s_time_font = fonts_load_custom_font(resource_get_handle(RESOURCE_ID_FONT_JERSEY_56));
s_date_font = fonts_load_custom_font(resource_get_handle(RESOURCE_ID_FONT_JERSEY_24));
```

## Centering the Layout

While we're here, let's properly center the time and date on screen. The date
starts 56 pixels below the time and its layer is 30 pixels tall, giving us a
total block height. We center the block by subtracting half its height from the
screen center:

```c
// Center the time + date block vertically
int date_height = 30;
int block_height = 56 + date_height;
int time_y = (bounds.size.h / 2) - (block_height / 2) - 10;
int date_y = time_y + 56;
```

Notice the `- 10` offset at the end of the `time_y` calculation. Custom fonts
often include internal padding (ascent space above the tallest glyph) that
shifts the rendered text lower than the calculated position. Subtracting a small
offset compensates for this and keeps the block visually centered on screen. You
may need to adjust this value depending on the font you choose.

Use `time_y` and `date_y` in the ``text_layer_create()`` calls instead of the
old ``PBL_IF_ROUND_ELSE()`` values:

```c
s_time_layer = text_layer_create(
    GRect(0, time_y, bounds.size.w, 60));
```

Now replace the system font calls with our custom fonts. Change the
``text_layer_set_font()`` calls for both layers:

```c
// For the time layer (was FONT_KEY_BITHAM_42_BOLD)
text_layer_set_font(s_time_layer, s_time_font);

// For the date layer (was FONT_KEY_GOTHIC_24_BOLD)
text_layer_set_font(s_date_layer, s_date_font);
```

## Cleaning Up

Custom fonts must be unloaded when no longer needed. Add the cleanup calls to
`main_window_unload()`, after destroying the ``TextLayer``s:

```c
// Unload custom fonts
fonts_unload_custom_font(s_time_font);
fonts_unload_custom_font(s_date_font);
```

> Always destroy layers before unloading the fonts they use. The layer may try
> to access the font during destruction.

Compile and install with `pebble build && pebble install`. You should see your
watchface now uses the custom font, giving it a much more distinctive look.


## Experimenting

Here are some things you can try:

- Use a different font file. There are many free `.ttf` fonts available online
  at sites like [dafont.com](http://www.dafont.com) and
  [Google Fonts](https://fonts.google.com).
- Adjust the font sizes by changing the `name` values in `package.json`.
- Try different Y-positions in ``text_layer_create()`` to adjust the layout.

> **Tip**: Not all fonts render well at small sizes on the Pebble display.
> Pixel-style and bitmap fonts tend to look the sharpest.


## Conclusion

In this part we learned how to:

1. Register font resources in `package.json`.
2. Load custom fonts with ``fonts_load_custom_font()`` and
   ``resource_get_handle()``.
3. Apply fonts to ``TextLayer``s.
4. Clean up font resources properly.

Your watchface now has a unique visual identity. Check your code against the
source provided in this part's project folder if you run into any issues.


## What's Next?

In the next part we will add a battery meter and Bluetooth disconnect alerts
to give users useful information at a glance.

[Go to Part 3 &rarr; >{wide,bg-dark-red,fg-white}](/tutorials/watchface-tutorial/part3/)
