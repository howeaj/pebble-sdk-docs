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

title: Piu UI Framework
description: |
  Build declarative user interfaces with the Piu framework.
guide_group: alloy
order: 2
---

Piu is a declarative UI framework for building user interfaces in Alloy apps.
It provides a component-based architecture with automatic layout, styling, and
animation support.

> **Note**: All code in this guide runs on the watch in `src/embeddedjs/main.js`.

## Getting Started with Piu

For apps using advanced Piu features (behaviors, templates, containers), include
the Piu runtime:

```javascript
import {} from "piu/MC";
```

A basic Piu application creates an `Application` object that fills the screen:

```javascript
const application = new Application(null, {
    skin: new Skin({ fill: "white" })
});
```

## Core Concepts

### Content Objects

Content objects are the basic building blocks of Piu UIs. The main types are:

| Type | Description |
|------|-------------|
| `Content` | Basic rectangular element (base class for most UI objects) |
| `Container` | Holds other content objects |
| `Column` | Vertical layout container |
| `Row` | Horizontal layout container |
| `Label` | Single line of text |
| `Text` | Multi-line formatted text |
| `Application` | Root container (one per app) |

### Skins

Skins define the visual appearance of content objects:

```javascript
// Solid color skin
const whiteSkin = new Skin({ fill: "white" });

// Skin with stroke/border
const borderedSkin = new Skin({
    fill: "transparent",
    stroke: "black",
    borders: { left: 2, right: 2, top: 2, bottom: 2 }
});

// Texture-based skin (for images)
const ballTexture = new Texture("balls.png");
const ballSkin = new Skin({
    texture: ballTexture,
    x: 0, y: 0,
    width: 30, height: 30,
    variants: 30  // for sprite sheets
});
```

### Styles

Styles define text appearance. Styles cascade, similar to CSS - you can create a
base style and let child content objects inherit from it, making it easy to
maintain a consistent look:

```javascript
const textStyle = new Style({
    font: "OpenSans-Regular-15",
    color: "black",
    horizontal: "center",  // "left", "right", "center", "justify"
    left: 10, right: 10,
    top: 15, bottom: 15
});

// Style with state colors (normal, active)
const buttonStyle = new Style({
    font: "bold 18px Gothic",
    color: ["black", "gray"]  // [normal, active]
});
```

## Positioning and Layout

Content objects are positioned using constraints:

```javascript
// Absolute positioning
Content($, { left: 20, top: 20, width: 80, height: 80 });

// Anchored to edges
Content($, { right: 20, bottom: 20, width: 80, height: 80 });

// Fill available space
Content($, { left: 0, right: 0, top: 0, bottom: 0 });

// Centered (no position constraints)
Content($, { width: 80, height: 80 });
```

## Behaviors

Behaviors add interactivity and logic to content objects. They are essential
for handling events and updating the UI:

```javascript
class BallBehavior extends Behavior {
    onCreate(ball, delta) {
        // delta is the data passed to the content
        this.dx = delta;
        this.dy = delta;
    }
    onDisplaying(ball) {
        // Store initial position and bounds
        this.x = ball.x;
        this.y = ball.y;
        this.width = ball.container.width - ball.width;
        this.height = ball.container.height - ball.height;
        ball.start();  // Start time-based updates
    }
    onTimeChanged(ball) {
        // Move the ball
        ball.moveBy(this.dx, this.dy);

        // Update position and check bounds
        this.x += this.dx;
        this.y += this.dy;

        // Bounce off walls
        if (this.x < 0 || this.x > this.width) this.dx = -this.dx;
        if (this.y < 0 || this.y > this.height) this.dy = -this.dy;
    }
}

// Attach behavior to content - the first argument (6) is passed to onCreate
Content(6, {
    left: 0, top: 0,
    width: 30, height: 30,
    skin: ballSkin,
    variant: 0,
    Behavior: BallBehavior
});
```

### Common Behavior Events

| Event | Description |
|-------|-------------|
| `onCreate(content, data)` | Content created with data |
| `onDisplaying(content)` | Content added to display |
| `onTimeChanged(content)` | Animation frame (after `start()`) |
| `onFinished(content)` | Animation/duration completed |

Pebble button presses are handled using the `Button` class rather than touch
events. See the [Sensors and Input](/guides/alloy/sensors-and-input/) guide for
details on button handling. Piu also uses the `Button` class internally for
button input.

## Templates

Templates create reusable component definitions. They are most useful when you
need to create multiple instances of the same component:

```javascript
// Define a template
const Square = Content.template($ => ({
    width: 80, height: 80,
    skin: new Skin({ fill: $ })  // $ is the data passed to template
}));

// Use the template
const redSquare = new Square("red", { left: 20, top: 20 });
const blueSquare = new Square("blue", { right: 20, bottom: 20 });

application.add(redSquare);
application.add(blueSquare);
```

> **Note**: For one-off content, create instances directly rather than defining
> a template first. Templates use more code and RAM than direct instantiation
> when you only need a single instance.

## Building an Application

Here's a complete example with multiple components:

```javascript
const backgroundSkin = new Skin({ fill: "silver" });
const headerSkin = new Skin({ fill: "white" });
const headerStyle = new Style({ font: "bold 18px Gothic", color: "black" });

class HeaderBehavior extends Behavior {
    onDisplaying(label) {
        label.string = "My App";
    }
}

const application = new Application(null, {
    skin: backgroundSkin,
    contents: [
        new Column(null, {
            top: 0, bottom: 0, left: 0, right: 0,
            contents: [
                new Label(null, {
                    top: 0, height: 30, left: 0, right: 0,
                    skin: headerSkin,
                    style: headerStyle,
                    Behavior: HeaderBehavior
                }),
                new Content(null, {
                    top: 10, bottom: 10, left: 10, right: 10,
                    skin: new Skin({ fill: "gray" })
                })
            ]
        })
    ]
});
```

## Anchors and References

Use anchors to reference content objects from behaviors:

```javascript
const MyApp = Application.template($ => ({
    contents: [
        Label($, {
            anchor: "TITLE",  // Creates $.TITLE reference
            string: "Hello"
        })
    ],
    Behavior: class extends Behavior {
        onCreate(app, data) {
            this.data = data;
        }
        updateTitle(app, newTitle) {
            this.data.TITLE.string = newTitle;
        }
    }
}));
```

## Animations with Timeline

The Timeline class creates smooth animations:

```javascript
import Timeline from "piu/Timeline";

class AnimatedBehavior extends Behavior {
    onDisplaying(content) {
        let timeline = new Timeline();

        // Animate 'y' property over 750ms
        timeline.to(content, { y: 100 }, 750, Math.quadEaseOut, 0);

        content.duration = timeline.duration;
        timeline.seekTo(0);
        content.time = 0;
        content.start();
    }
    onTimeChanged(content) {
        this.timeline.seekTo(content.time);
    }
}
```

### Easing Functions

Available easing functions on the `Math` object:

- `Math.backEaseIn` / `Math.backEaseOut`
- `Math.bounceEaseIn` / `Math.bounceEaseOut`
- `Math.circularEaseIn` / `Math.circularEaseOut`
- `Math.cubicEaseIn` / `Math.cubicEaseOut`
- `Math.elasticEaseIn` / `Math.elasticEaseOut`
- `Math.exponentialEaseIn` / `Math.exponentialEaseOut`
- `Math.quadEaseIn` / `Math.quadEaseOut`
- `Math.quartEaseIn` / `Math.quartEaseOut`
- `Math.quintEaseIn` / `Math.quintEaseOut`
- `Math.sineEaseIn` / `Math.sineEaseOut`

For more animation techniques including chaining and looping, see the
[Animations](/guides/alloy/animations/) guide.

## Displaying Images

Load and display images using Texture and Skin:

```javascript
// Load a texture from resources
const logoTexture = new Texture("logo.png");

// Create a skin from the texture
const logoSkin = new Skin({
    texture: logoTexture,
    x: 0, y: 0,
    width: 64, height: 64
});

// Display the image
application.add(new Content(null, {
    skin: logoSkin
}));
```

## Text Display

### Single Line (Label)

```javascript
application.add(new Label(null, {
    left: 0, right: 0, top: 50,
    style: new Style({ font: "bold 24px Gothic", color: "black" }),
    string: "Hello, World!"
}));
```

### Multi-line (Text)

```javascript
application.add(new Text(null, {
    left: 10, right: 10, top: 10,
    style: textStyle,
    blocks: [
        { spans: "First paragraph of text." },
        { spans: [
            "Second paragraph with ",
            { style: boldStyle, spans: "bold text" },
            " inline."
        ]}
    ]
}));
```

## Using Pebble Fonts

Access built-in Pebble fonts with CSS-like syntax:

```javascript
// Format: "[style] [size]px [family]"
const headerStyle = new Style({
    font: "bold 14px Gothic",
    color: "black"
});

const titleStyle = new Style({
    font: "black 30px Bitham",
    color: "blue"
});

const bodyStyle = new Style({
    font: "24px Leco",
    color: "white"
});
```

Available Pebble fonts include:
- `Gothic` - Regular and Bold weights
- `Bitham` - Black and Bold weights
- `Roboto` - Condensed weight
- `Leco` - Regular weight (great for numbers)
- `Droid` - Serif style

The available sizes are limited for each font family. See the
[System Fonts](/guides/app-resources/system-fonts/) reference for the full table
of all fonts, styles, and sizes.

## Loading Bitmaps from Resources

Load images from Pebble resources by ID:

```javascript
// Load texture by resource ID
const backgroundTexture = new Texture(2);  // Resource ID 2

// Create a tiled background skin
const backgroundSkin = new Skin({
    texture: backgroundTexture,
    x: 0, y: 0,
    width: 40, height: 40,
    tiles: { left: 0, right: 0, top: 0, bottom: 0 }
});

// Load and display an image
const iconTexture = new Texture(3);
application.add(new Content(null, {
    skin: new Skin({
        texture: iconTexture,
        width: iconTexture.width,
        height: iconTexture.height
    })
}));
```

## Complete Example: Bouncing Balls

Here's a complete animated app with multiple bouncing balls:

```javascript
import {} from "piu/MC";

const backgroundSkin = new Skin({ fill: "silver" });
const ballTexture = new Texture("balls.png");
const ballSkin = new Skin({
    texture: ballTexture,
    x: 0, y: 0,
    width: 30, height: 30,
    variants: 30  // sprite sheet with 30px variants
});

class BallBehavior extends Behavior {
    onCreate(ball, delta) {
        this.dx = delta;
        this.dy = delta;
    }
    onDisplaying(ball) {
        this.x = ball.x;
        this.y = ball.y;
        this.width = ball.container.width - ball.width;
        this.height = ball.container.height - ball.height;
        ball.start();
    }
    onTimeChanged(ball) {
        ball.moveBy(this.dx, this.dy);
        this.x += this.dx;
        this.y += this.dy;
        if (this.x < 0 || this.x > this.width) this.dx = -this.dx;
        if (this.y < 0 || this.y > this.height) this.dy = -this.dy;
    }
}

const BallApplication = Application.template($ => ({
    skin: backgroundSkin,
    contents: [
        Content(6, { left: 0, top: 0, skin: ballSkin, variant: 0, Behavior: BallBehavior }),
        Content(5, { right: 0, top: 0, skin: ballSkin, variant: 1, Behavior: BallBehavior }),
        Content(4, { right: 0, bottom: 0, skin: ballSkin, variant: 2, Behavior: BallBehavior }),
        Content(3, { left: 0, bottom: 0, skin: ballSkin, variant: 3, Behavior: BallBehavior }),
    ]
}));

export default new BallApplication(null, { pixels: screen.width * 4 });
```

## Related Guides

- [Animations](/guides/alloy/animations/) - Advanced animation techniques with
  Timeline and easing functions
- [Port (Custom Drawing)](/guides/alloy/port-drawing/) - Combine Piu layout with
  custom Poco-style drawing
- [Poco Graphics](/guides/alloy/poco-guide/) - Low-level graphics alternative
  to Piu

## Examples

The [Pebble Examples](https://github.com/Moddable-OpenSource/pebble-examples)
repository includes several Piu examples:

- [`hellopiu-balls`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-balls) - animated bouncing balls with Behavior and texture variants
- [`hellopiu-coloredsquares`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-coloredsquares) - basic layout with colored skins
- [`hellopiu-gbitmap`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-gbitmap) - displaying Pebble GBitmap PNG images
- [`hellopiu-jsicon`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-jsicon) - displaying a Moddable SDK bitmap from a PNG resource
- [`hellopiu-text`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-text) - dynamic text layout with different fonts and alignment
- [`hellopiu-pebbletext`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-pebbletext) - text rendering with Pebble's built-in fonts
