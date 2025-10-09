---
layout: sdk/markdown
title: Use the Pebble SDK in the Cloud
permalink: /sdk/cloud
menu_section: sdk
menu_subsection: cloud
generate_toc: true
scripts:
  - sdk/index
---

This is the fastest way to get started building Pebble apps and watchfaces! Seriously. Press the button and in 1 minute you'll be up and running.

## Get started

Develop Pebble apps using a browser-based version of VS Code with the Pebble SDK pre-installed.

Press to launch a cloud development environment:

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/coredevices/codespaces-pebble?quickstart=1)

To resume your Codespace later, visit [cloud.repebble.com](https://cloud.repebble.com).

## Usage

Your Codespace will launch into an example project with the C file already open. 

**Step 1**
![Codespaces Step 1](/images/sdk/codespaces1.png)
**Step 2**
![Codespaces Step 2](/images/sdk/codespaces2.png)
**Step 3**
![Codespaces Step 3](/images/sdk/codespaces3.png)
**Step 4**
![Codespaces Step 4](/images/sdk/codespaces4.png)

Copy an example or write your own app, then click Emulator on the left to rebuild. When you're ready to install the app on your watch, grab the new Pebble mobile app (install at [rePebble.com/app](https://repebble.com/app)) -> go to Devices -> tap 3 dots -> Enable Dev Connect -> Sign into GitHub. Then back on your computer, click Run -> Phone. That's it! 


#### Learn more

The best way to learn is by checking out our examples apps: [weather](https://github.com/pebble-examples/pebblekit-js-weather/), [simple game](https://github.com/pebble-hacks/pandas-and-bananas/), [concentricity watchface](https://github.com/pebble-examples/concentricity/), and [many more](/examples)! Or try [tutorials](/tutorials/) for a step-by-step guide on how
to write a simple C Pebble application.

#### Save to GitHub

You should connect your Codespace to a GitHub repository to ensure permanent storage of your project code. 

To do this, press the Source Control icon in the left sidebar of VS Code, then press "Publish Branch." Approve the pop-up asking to sign in with GitHub, select your GitHub account, then select "Publish to GitHub private repository." When you make changes, you can commit and push to that GitHub repo. All projects in your codespace will be saved in one repo.

#### Import an existing project

In your terminal, paste `code /workspaces/codespaces-pebble`. VS Code will reload and open to that folder.

If there's a "build" folder in the project you want to import, delete it to speed up the upload.

Drag and drop your project into the file list pane on the left. Wait for the upload to complete. Then, press the Pebble icon in the left sidebar, select "Open Project", and select the project you just uploaded. You can now continue developing it!