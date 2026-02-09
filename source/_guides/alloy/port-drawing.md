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

title: Port (Custom Drawing)
description: |
  Combine Piu's declarative UI with custom drawing using Port.
guide_group: alloy
order: 10
---

The `Port` content type lets you do custom drawing within a Piu application.
This combines the best of both worlds: Piu's declarative layout and component
system with Poco-style direct drawing.

> **Note**: All code in this guide runs on the watch in `src/embeddedjs/main.js`.

## What is Port?

Port is a Piu content type that provides a canvas for custom drawing. Unlike
other Piu content types that are styled with skins, Port content uses an
`onDraw` callback where you can draw shapes, text, and graphics.

## Basic Port Usage

```javascript
class MyPortBehavior extends Behavior {
    onDraw(port, x, y, width, height) {
        // Draw a blue rectangle
        port.fillColor("#0066cc", 0, 0, width, height);

        // Draw some text
        const style = new Style({ font: "18px Gothic", color: "white" });
        port.drawString("Hello!", style, "white", 10, 10);
    }
}

const myPort = new Port(null, {
    top: 0, bottom: 0, left: 0, right: 0,
    Behavior: MyPortBehavior
});

const application = new Application(null, {
    skin: new Skin({ fill: "white" })
});

application.add(myPort);
```

## Port Drawing Methods

### Filling Shapes

```javascript
onDraw(port, x, y, width, height) {
    // Fill a rectangle with a color
    port.fillColor("blue", 10, 10, 50, 30);

    // Fill with hex color
    port.fillColor("#ff6600", 70, 10, 50, 30);
}
```

### Drawing Text

```javascript
onDraw(port, x, y, width, height) {
    const style = new Style({
        font: "bold 18px Gothic",
        color: "black"
    });

    // Draw text at position
    port.drawString("Hello World", style, "black", 10, 50);

    // Measure text width for alignment
    const text = "Centered";
    const measured = style.measure(text);
    const textX = (width - measured.width) / 2;
    port.drawString(text, style, "black", textX, 80);
}
```

### Drawing Images

```javascript
onDraw(port, x, y, width, height) {
    const texture = new Texture(2);  // Resource ID
    port.drawTexture(texture, "white", 10, 10, 0, 0, texture.width, texture.height);
}
```

## Updating Port Content

Call `invalidate()` to trigger a redraw:

```javascript
class AnimatedPortBehavior extends Behavior {
    onDisplaying(port) {
        this.value = 0;
        port.interval = 100;  // Update every 100ms
        port.start();
    }
    onTimeChanged(port) {
        this.value = (this.value + 5) % 100;
        port.invalidate();  // Request redraw
    }
    onDraw(port, x, y, width, height) {
        // Clear background
        port.fillColor("white", 0, 0, width, height);

        // Draw bar based on current value
        const barWidth = (width * this.value) / 100;
        port.fillColor("green", 0, height / 2 - 10, barWidth, 20);
    }
}
```

## Complete Example: Live Bar Chart

This example creates an animated bar chart with random data:

```javascript
const GRAY = "gray";
const BLUE = "#1932ab";

const textStyle = new Style({
    font: "bold 18px Gothic"
});

class GraphBehavior extends Behavior {
    onDisplaying(port) {
        // Initialize data array
        const barCount = Math.floor(screen.width / 16);
        this.values = new Array(barCount);
        this.values.fill(0);

        // Start animation
        port.interval = 100;
        port.start();
    }
    onTimeChanged(port) {
        // Shift data and add new random value
        this.values.shift();
        this.values.push(Math.floor(Math.random() * 100));
        port.invalidate();
    }
    onDraw(port, x, y, width, height) {
        // Draw Y-axis labels and grid lines
        for (let i = 100, yOffset = 0; i >= 0; i -= 20) {
            const label = String(i);
            const labelWidth = textStyle.measure(label).width;
            port.drawString(label, textStyle, "black", 30 - labelWidth, yOffset);
            port.fillColor(GRAY, 35, yOffset + 10, width - 35, 1);
            yOffset += height / 5;
        }

        // Draw bars
        let xOffset = 35;
        for (let i = 0; i < this.values.length; i++) {
            const value = this.values[i];
            const barHeight = Math.floor((value * (height - 10)) / 100);
            port.fillColor(BLUE, xOffset, height - barHeight, 12, barHeight);
            xOffset += 14;
        }
    }
}

const graph = new Port(null, {
    top: 0, bottom: 0, left: 0, right: 0,
    Behavior: GraphBehavior
});

const application = new Application(null, {
    skin: new Skin({ fill: "white" })
});

application.add(graph);
```

## Combining Port with Piu Layout

Port works within Piu's layout system, so you can combine custom drawing
with standard Piu components:

```javascript
import {} from "piu/MC";

const headerSkin = new Skin({ fill: "darkblue" });
const headerStyle = new Style({
    font: "bold 18px Gothic",
    color: "white"
});

class ChartBehavior extends Behavior {
    onDisplaying(port) {
        this.data = [30, 50, 80, 45, 90, 60, 75];
        port.invalidate();
    }
    onDraw(port, x, y, width, height) {
        port.fillColor("white", 0, 0, width, height);

        const barWidth = width / this.data.length - 4;
        let xPos = 2;

        for (const value of this.data) {
            const barHeight = (value / 100) * height;
            port.fillColor("#4CAF50", xPos, height - barHeight, barWidth, barHeight);
            xPos += barWidth + 4;
        }
    }
}

const DashboardApp = Application.template($ => ({
    skin: new Skin({ fill: "#f0f0f0" }),
    contents: [
        Column($, {
            top: 0, bottom: 0, left: 0, right: 0,
            contents: [
                // Header using standard Piu Label
                Label($, {
                    top: 0, height: 40, left: 0, right: 0,
                    skin: headerSkin,
                    style: headerStyle,
                    string: "Weekly Stats"
                }),
                // Custom chart using Port
                Port($, {
                    top: 10, bottom: 10, left: 10, right: 10,
                    Behavior: ChartBehavior
                })
            ]
        })
    ]
}));

export default new DashboardApp({}, {});
```

## Port vs Poco

| Feature | Port | Poco |
|---------|------|------|
| Integration | Works within Piu apps | Standalone rendering |
| Layout | Uses Piu's layout system | Manual positioning |
| Updates | Call `invalidate()` | Call `begin()`/`end()` |
| Use case | Custom drawing in Piu apps | Full-screen graphics |

Use Port when you want custom drawing as part of a larger Piu UI. Use Poco
when you need full control over the entire screen.

## Performance Tips

1. **Minimize invalidate() calls**: Only call when data actually changes.

2. **Draw only what's needed**: The `onDraw` callback receives bounds -
   use them to clip your drawing.

3. **Cache expensive calculations**: Calculate positions and sizes once,
   not on every draw.

4. **Use appropriate intervals**: 100ms intervals are usually sufficient
   for charts. Use shorter intervals only when smooth animation is needed.

## Common Patterns

### Progress Bar

```javascript
class ProgressBehavior extends Behavior {
    onCreate(port, data) {
        this.progress = data.progress || 0;
    }
    setProgress(port, value) {
        this.progress = Math.max(0, Math.min(100, value));
        port.invalidate();
    }
    onDraw(port, x, y, width, height) {
        // Background
        port.fillColor("#ddd", 0, 0, width, height);

        // Progress fill
        const fillWidth = (width * this.progress) / 100;
        port.fillColor("#4CAF50", 0, 0, fillWidth, height);
    }
}
```

### Gauge/Meter

```javascript
class GaugeBehavior extends Behavior {
    onCreate(port, data) {
        this.value = data.value || 0;
        this.max = data.max || 100;
    }
    onDraw(port, x, y, width, height) {
        const centerX = width / 2;
        const centerY = height / 2;
        const radius = Math.min(width, height) / 2 - 10;

        // Background circle (using rectangles as approximation)
        port.fillColor("#eee", centerX - radius, centerY - radius,
                       radius * 2, radius * 2);

        // Value indicator
        const percentage = this.value / this.max;
        const indicatorHeight = radius * 2 * percentage;
        port.fillColor("#2196F3",
                       centerX - radius, centerY + radius - indicatorHeight,
                       radius * 2, indicatorHeight);
    }
}
```

## Examples

The [Pebble Examples](https://github.com/Moddable-OpenSource/pebble-examples)
repository includes a Port example:

- [`hellopiu-port`](https://github.com/Moddable-OpenSource/pebble-examples/tree/main/hellopiu-port) — animated bar graph using a Piu Port with custom drawing
