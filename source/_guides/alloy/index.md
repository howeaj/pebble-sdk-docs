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

title: Alloy
description: |
  Build Pebble apps using modern JavaScript with the Alloy framework.
guide_group: alloy
menu: false
permalink: /guides/alloy/
generate_toc: false
hide_comments: true
---

Alloy is a JavaScript framework for building Pebble apps, based on the
[Moddable SDK](https://www.moddable.com/). Alloy provides developers a 
mix of standard Web APIs, standard Embedded JavaScript APIs from ECMA-419, 
and dedicated Pebble OS APIs. It allows you to write apps using
modern JavaScript (ES2025, ES6++) with powerful UI frameworks and access 
to Pebble hardware features.

> **Platform Support**: Alloy currently supports Emery (Pebble Time 2) and
> Gabbro (Pebble Round 2).

## Key Features

- **Modern JavaScript**: Write apps using ES modules, classes, async/await,
  and other modern JavaScript features
- **Two UI Frameworks**: Choose between Piu (declarative, component-based) or
  Poco (procedural, low-level graphics)
- **Hardware Access**: Full access to accelerometer, battery, buttons, compass,
  and other Pebble sensors
- **Network Communication**: Built-in support for HTTP requests, WebSockets,
  and watch-phone messaging
- **Persistent Storage**: Simple APIs for storing data locally using
  `localStorage`, key-value storage, or files

## Getting Started

To create a new Alloy project:

```text
$ pebble new-project --alloy my-app
```

This creates a new project with the standard Alloy structure:

```text
my-app/
  src/
    embeddedjs/
      main.js           # Watch code (runs on Pebble)
    pkjs/
      index.js          # Phone code (for networking, location)
    c/
      mdbl.c             # C code as entry point for embeddedjs (usually no need to modify)
  resources/            # App resources (images, fonts)
  package.json          # App manifest
```

Alloy apps have two JavaScript environments: **embeddedjs** runs on the watch,
while **pkjs** runs on the connected phone for network and location services.

## Hello World

The simplest Alloy app:

```javascript
console.log("Hello, Pebble!");
```

## Guides

{% include guides/contents-group.md group=site.data.guides.alloy %}

## Additional Resources

Alloy is built on the Moddable SDK. For more detailed documentation on the
underlying APIs:

- [Moddable SDK Documentation](https://github.com/Moddable-OpenSource/moddable/tree/public/documentation) -
  Comprehensive documentation for Piu, Poco, and other modules
- Moddable's [Pebble Examples](https://github.com/Moddable-OpenSource/pebble-examples) repository - Example apps that developers may find valuable in getting the most out of Alloy.
