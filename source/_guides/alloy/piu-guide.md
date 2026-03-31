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

## PDC (Pebble Draw Command) Images

PDC images are vector graphics converted from SVG. In Piu, use the `SVGImage`
class to display and transform PDC files. Place `.pdc` files in your
`src/embeddedjs/assets/` directory and reference them by filename:

```javascript
const application = new Application(null, {
    skin: new Skin({ fill: "gray" }),
    contents: [
        SVGImage(null, { path: "icon.pdc" })
    ]
});
```

The `path` property specifies the `.pdc` filename from your assets directory.

### SVGImage Transforms

SVGImage supports rotation, scaling, and translation through built-in
properties:

| Property | Description |
|----------|-------------|
| `r` | Rotation angle in radians |
| `s` | Uniform scale factor |
| `sx`, `sy` | Non-uniform scale (horizontal, vertical) |
| `tx`, `ty` | Translation offset (x, y) |
| `cx`, `cy` | Center of rotation |

In the [gravity example](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/piu/apps/gravity),
a PDC star is continuously rotated and moved based on accelerometer readings.
The rotation portion of its Behavior updates the `r` property each frame:

```javascript
let degree = this.degree + 1;
if (degree > 360)
    degree -= 360;
this.degree = degree;
star.r = (this.degree / 180) * Math.PI;
```

Unlike Poco's `clone().rotate()` approach, Piu transforms the SVGImage in place
through its properties - no cloning needed.

### Animated SVGImage with Timeline

Use `duration`, `fraction`, and `start()` to animate SVGImage transforms with
easing:

```javascript
const graySkin = new Skin({ fill: "gray" });

class ImageBehavior extends Behavior {
    onAnimate(image) {
        image.duration = 1500;
        image.time = 0;
        image.start();
    }
    onDisplaying(image) {
        this.delta = (image.container.width >> 1) + image.width;
        image.tx = this.delta;
    }
    onFinished(image) {
        image.r = 0;
        image.tx = this.delta;
        let next = image.next ?? image.container.first;
        next.delegate("onAnimate");
    }
    onTimeChanged(image) {
        let fraction = image.fraction;
        if (fraction < 0.4) {
            fraction = Math.quadEaseOut(0.4 - fraction);
            image.r = fraction * Math.PI;
            image.tx = this.delta * fraction;
        }
        else if (fraction > 0.6) {
            fraction = Math.quadEaseOut(0.6 - fraction);
            image.r = fraction * Math.PI;
            image.tx = this.delta * fraction;
        }
        else {
            image.r = 0;
            image.tx = 0;
        }
    }
}

class TestApplicationBehavior {
    onDisplaying(application) {
        application.duration = 1000;
        application.first.delegate("onAnimate");
    }
}

const TestApplication = Application.template($ => ({
    Behavior: TestApplicationBehavior, skin: graySkin,
    contents: [
        SVGImage(6, { path: "Pebble_80x80_Incoming_call_centered.pdc", Behavior: ImageBehavior }),
        SVGImage(5, { path: "Pebble_80x80_Scheduled_event.pdc", Behavior: ImageBehavior }),
        SVGImage(4, { path: "Pebble_80x80_Outgoing_call.pdc", Behavior: ImageBehavior }),
    ]
}));

export default new TestApplication({}, {
    displayListLength: 4096, touchCount: 0, pixels: screen.width * 4,
});
```

The `fraction` property returns a value from 0 to 1 representing progress
through the `duration`. Combine it with easing functions from `Math` for smooth
motion.

### PDC Sequences

PDC sequences are animated vector graphics containing multiple frames. Use
`SVGImage` with a sequence `.pdc` file and control playback through Behavior
events:

```javascript
const graySkin = new Skin({ fill: "gray" });

class ClockBehavior extends Behavior {
    onDisplaying(image) {
        image.start();
    }
    onFinished(image) {
        image.time = 0;
        image.start();
    }
    onTimeChanged(image) {
        let fraction = image.fraction;
        if (fraction < 0.5)
            fraction = 1 + fraction;
        else
            fraction = 2 - fraction;
        image.sx = image.sy = fraction;
    }
}

class TestApplicationBehavior {
}

const TestApplication = Application.template($ => ({
    Behavior: TestApplicationBehavior, skin: graySkin,
    contents: [
        SVGImage($, { bottom: 20, path: "clock_sequence.pdc", Behavior: ClockBehavior })
    ]
}));

export default new TestApplication({}, {
    displayListLength: 2048, touchCount: 0, pixels: screen.width * 4,
});
```

### Clock Hands with SVGImage

A common pattern is using SVGImage for watchface clock hands. Each hand is a
separate PDC file, rotated based on the current time:

```javascript
const scale = Math.min(screen.width, screen.height) / 240;

class FaceHandBehavior {
    onFractionChanged(content, fraction) {
        const angle = ((-fraction * 2) - 1) * Math.PI;
        content.r = angle;
    }
    onClockResized(content) {
        const container = content.container;
        content.x = (container.width >> 1) - content.cx;
        content.y = (container.height >> 1) - content.cy;
    }
}

class FaceHoursBehavior extends FaceHandBehavior {
    onDisplaying(content) {
        content.cx = 7;
        content.cy = 14;
        content.s = scale;
        this.onClockResized(content);
    }
    onClockChanged(content, clock) {
        this.onFractionChanged(content,
            (clock.hours % 12 + clock.minutes / 60) / 12);
    }
}

class FaceMinutesBehavior extends FaceHandBehavior {
    onDisplaying(content) {
        content.cx = 5;
        content.cy = 20;
        content.s = scale;
        this.onClockResized(content);
    }
    onClockChanged(content, clock) {
        this.onFractionChanged(content, clock.minutes / 60);
    }
}
```

The `cx` and `cy` properties set the rotation center, which should match the
pivot point of the clock hand in the original SVG. The `scale` factor adapts
the hand size to the screen.

In the application template, layer the hands from bottom to top:

```javascript
class FaceApplicationBehavior {
    onDisplaying(application) {
        watch.addEventListener("secondchange", (clock) => {
            const date = clock.date;
            application.distribute("onClockChanged", {
                date,
                hours: date.getHours(),
                minutes: date.getMinutes(),
                seconds: date.getSeconds(),
            });
        });
    }
    onResize(application) {
        application.distribute("onClockResized");
    }
}

const FaceApplication = Application.template($ => ({
    Behavior: FaceApplicationBehavior,
    contents: [
        Content($, { skin: new Skin({ texture: new Texture("dial.png"),
            width: screen.width, height: screen.height }) }),
        SVGImage($, { left: 0, width: 14, top: 0, height: 79,
            path: "hours.pdc", Behavior: FaceHoursBehavior }),
        SVGImage($, { left: 0, width: 10, top: 0, height: 100,
            path: "minutes.pdc", Behavior: FaceMinutesBehavior }),
    ]
}));

export default new FaceApplication(null, {
    displayListLength: 2048, touchCount: 0, pixels: screen.width * 4,
});
```

Use `distribute()` to send clock updates to all hands simultaneously.

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
- [Converting SVG to PDC](/guides/app-resources/converting-svg-to-pdc/) - How to
  create PDC files from SVG
- [Vector Graphics](/guides/graphics-and-animations/vector-graphics/) - Overview
  of vector graphics on Pebble

## Examples

The [Pebble Examples](https://github.com/Moddable-OpenSource/pebble-examples)
repository includes several Piu examples:

- [`hellopiu-balls`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-balls) - animated bouncing balls with Behavior and texture variants
- [`hellopiu-coloredsquares`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-coloredsquares) - basic layout with colored skins
- [`hellopiu-gbitmap`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-gbitmap) - displaying Pebble GBitmap PNG images
- [`hellopiu-jsicon`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-jsicon) - displaying a Moddable SDK bitmap from a PNG resource
- [`hellopiu-text`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-text) - dynamic text layout with different fonts and alignment
- [`hellopiu-pebbletext`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-pebbletext) - text rendering with Pebble's built-in fonts
- [`piu/apps/pdc-images`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/piu/apps/pdc-images) - moving and rotating PDC (SVG) images
- [`piu/apps/pdc-sequence`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/piu/apps/pdc-sequence) - playback of animated PDC sequences
- [`piu/apps/compass`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/piu/apps/compass) - compass visualization with rotating PDC image
- [`piu/apps/gravity`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/piu/apps/gravity) - accelerometer-driven PDC star animation
- [`piu/watchfaces/minato`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/piu/watchfaces/minato) - watchface with PDC clock hands
