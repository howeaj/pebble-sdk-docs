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

title: Animations
description: |
  Create smooth animations using Timeline and easing functions.
guide_group: alloy
order: 9
---

Alloy provides powerful animation capabilities through the Timeline class
and built-in easing functions. These work with both Piu UI components and
Poco graphics.

> **Note**: All code in this guide runs on the watch in `src/embeddedjs/main.js`.

## Timeline Basics

The Timeline class creates smooth, coordinated animations:

```javascript
import Timeline from "piu/Timeline";

class AnimatedBehavior extends Behavior {
    onDisplaying(content) {
        const timeline = this.timeline = new Timeline();

        // Animate 'y' property from current to 100 over 750ms
        timeline.to(content, { y: 100 }, 750, Math.quadEaseOut, 0);

        // Setup playback
        content.duration = timeline.duration;
        timeline.seekTo(0);
        content.time = 0;
        content.start();
    }
    onTimeChanged(content) {
        this.timeline.seekTo(content.time);
    }
    onFinished(content) {
        console.log("Animation complete!");
    }
}
```

### Timeline Methods

| Method | Description |
|--------|-------------|
| `to(target, properties, duration, easing, delay)` | Animate properties to new values |
| `from(target, properties, duration, easing, delay)` | Animate properties from values |
| `on(target, properties, duration, easing, delay)` | Keyframe animation |
| `seekTo(time)` | Jump to specific time in animation |

### Timeline Properties

| Property | Description |
|----------|-------------|
| `duration` | Total duration of all animations |

## Easing Functions

Easing functions control the rate of change during animations. All easing
functions are available on the `Math` object:

### Available Easing Functions

| Ease In | Ease Out | Description |
|---------|----------|-------------|
| `Math.backEaseIn` | `Math.backEaseOut` | Overshoots then returns |
| `Math.bounceEaseIn` | `Math.bounceEaseOut` | Bouncing effect |
| `Math.circularEaseIn` | `Math.circularEaseOut` | Circular curve |
| `Math.cubicEaseIn` | `Math.cubicEaseOut` | Cubic curve (smooth) |
| `Math.elasticEaseIn` | `Math.elasticEaseOut` | Spring/elastic effect |
| `Math.exponentialEaseIn` | `Math.exponentialEaseOut` | Exponential curve |
| `Math.quadEaseIn` | `Math.quadEaseOut` | Quadratic curve |
| `Math.quartEaseIn` | `Math.quartEaseOut` | Quartic curve |
| `Math.quintEaseIn` | `Math.quintEaseOut` | Quintic curve |
| `Math.sineEaseIn` | `Math.sineEaseOut` | Sinusoidal curve |

### Using Easing Functions

```javascript
// Linear (no easing) - define your own
function linearEase(fraction) {
    return fraction;
}

// Use built-in easing
timeline.to(content, { y: 200 }, 500, Math.bounceEaseOut, 0);

// Ease in then ease out
timeline.to(content, { x: 100 }, 300, Math.quadEaseIn, 0);
timeline.to(content, { x: 200 }, 300, Math.quadEaseOut, 300);
```

### Easing Function Characteristics

- **EaseIn**: Starts slow, ends fast
- **EaseOut**: Starts fast, ends slow
- **Bounce**: Creates a bouncing effect at the end
- **Elastic**: Creates a spring-like overshoot
- **Back**: Pulls back before moving forward

## Animating in Poco

For Poco graphics, use easing functions with `setInterval`:

```javascript
import Poco from "commodetto/Poco";

const render = new Poco(screen);
const gray = render.makeColor(128, 128, 128);
const blue = render.makeColor(0, 100, 255);

const start = Date.now();
const duration = 1500;

setInterval(() => {
    // Calculate animation progress (0 to 1)
    const elapsed = (Date.now() - start) % duration;
    const progress = elapsed / duration;

    // Apply easing function
    const eased = Math.elasticEaseOut(progress);

    // Animate size based on eased value
    const size = 20 + (60 * eased);
    const x = (render.width - size) / 2;
    const y = (render.height - size) / 2;

    render.begin();
        render.fillRectangle(gray, 0, 0, render.width, render.height);
        render.fillRectangle(blue, x, y, size, size);
    render.end();
}, 17);  // ~60fps
```

## Scaling PDC Images

Animate PDC image scaling with easing:

```javascript
import Poco from "commodetto/Poco";

const render = new Poco(screen);
const gray = render.makeColor(128, 128, 128);

const dci = new Poco.PebbleDrawCommandImage(2);
console.log("PDC size: " + dci.width + " x " + dci.height);

const start = Date.now();

setInterval(() => {
    // Calculate scale with elastic easing
    const progress = ((Date.now() - start) % 1500) / 1500;
    const scale = Math.elasticEaseOut(progress);

    // Calculate centered position for scaled image
    const cx = (render.width - dci.width * scale) / 2;
    const cy = (render.height - dci.height * scale) / 2;

    render.begin();
        render.fillRectangle(gray, 0, 0, render.width, render.height);
        render.drawDCI(dci.clone().scale(scale), cx, cy);
    render.end();
}, 17);
```

## Complete Piu Animation Example

Here's a complete example showing Timeline animations with multiple easing
functions:

```javascript
import {} from "piu/MC";
import Timeline from "piu/Timeline";

const WHITE = "white";
const BLACK = "black";
const GRAY = "gray";

const backgroundSkin = new Skin({ fill: BLACK });
const headerSkin = new Skin({ fill: WHITE });
const boxSkin = new Skin({ fill: GRAY });
const borderSkin = new Skin({
    fill: "transparent",
    stroke: WHITE,
    borders: { left: 2, right: 2, top: 2, bottom: 2 }
});

const headerStyle = new Style({
    font: "bold 18px Gothic",
    color: BLACK
});

// List of easing functions to demonstrate
const easingFunctions = [
    { name: "quadEase", out: Math.quadEaseOut, in: Math.quadEaseIn },
    { name: "bounceEase", out: Math.bounceEaseOut, in: Math.bounceEaseIn },
    { name: "elasticEase", out: Math.elasticEaseOut, in: Math.elasticEaseIn },
    { name: "backEase", out: Math.backEaseOut, in: Math.backEaseIn },
];

class AnimationBehavior extends Behavior {
    onCreate(container) {
        this.index = 0;
    }
    onDisplaying(container) {
        this.startAnimation(container);
    }
    startAnimation(container) {
        const easing = easingFunctions[this.index];
        const box = container.first;
        const bottom = container.height - box.height - 4;

        // Animate down
        const timeline = this.timeline = new Timeline();
        timeline.to(box, { y: bottom }, 750, easing.out, 0);
        timeline.to(box, { y: 4 }, 750, easing.in, 250);

        container.duration = timeline.duration + 500;
        timeline.seekTo(0);
        container.time = 0;
        container.start();
    }
    onTimeChanged(container) {
        this.timeline.seekTo(container.time);
    }
    onFinished(container) {
        // Move to next easing function
        this.index = (this.index + 1) % easingFunctions.length;
        this.startAnimation(container);
    }
}

const AnimationApp = Application.template($ => ({
    skin: backgroundSkin,
    contents: [
        Column($, {
            top: 0, bottom: 0, left: 0, right: 0,
            contents: [
                Label($, {
                    top: 0, height: 30, left: 0, right: 0,
                    skin: headerSkin,
                    style: headerStyle,
                    string: "Animation Demo"
                }),
                Container($, {
                    top: 10, bottom: 10, left: 10, right: 10,
                    skin: borderSkin,
                    contents: [
                        Content($, {
                            top: 4, height: 25, width: 25,
                            skin: boxSkin
                        })
                    ],
                    Behavior: AnimationBehavior
                })
            ]
        })
    ]
}));

export default new AnimationApp({}, {});
```

## Animation Best Practices

1. **Choose appropriate easing**: Use `quadEaseOut` for UI transitions,
   `bounceEaseOut` for playful effects, `elasticEaseOut` for attention-grabbing
   animations.

2. **Keep animations short**: 200-500ms for UI transitions, up to 1000ms for
   decorative animations.

3. **Use appropriate frame rates**: 30fps (33ms interval) is usually sufficient.
   60fps (17ms interval) for smoother animations at higher battery cost.

4. **Clean up animations**: Stop intervals and timelines when no longer needed.

5. **Consider battery life**: Frequent screen updates drain battery. Use
   animations sparingly on watchfaces.

## Combining Animations

Chain multiple animations using Timeline delays:

```javascript
const timeline = new Timeline();

// First animation: move right
timeline.to(content, { x: 100 }, 300, Math.quadEaseOut, 0);

// Second animation: move down (starts after 300ms)
timeline.to(content, { y: 100 }, 300, Math.quadEaseOut, 300);

// Third animation: fade (starts after 600ms)
timeline.to(content, { state: 1 }, 200, Math.quadEaseOut, 600);
```

## Looping Animations

Create looping animations by restarting in `onFinished`:

```javascript
class LoopingBehavior extends Behavior {
    onDisplaying(content) {
        this.setupAnimation(content);
    }
    setupAnimation(content) {
        const timeline = this.timeline = new Timeline();
        timeline.to(content, { y: 100 }, 500, Math.sineEaseOut, 0);
        timeline.to(content, { y: 0 }, 500, Math.sineEaseIn, 500);

        content.duration = timeline.duration;
        timeline.seekTo(0);
        content.time = 0;
        content.start();
    }
    onTimeChanged(content) {
        this.timeline.seekTo(content.time);
    }
    onFinished(content) {
        // Restart the animation
        this.setupAnimation(content);
    }
}
```

## Examples

The [Pebble Examples](https://github.com/Moddable-OpenSource/pebble-examples)
repository includes an animation example:

- [`hellopiu-timeline`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-timeline) — demonstrates various easing equations with Timeline animation
