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

## Get started

You can develop Pebble apps using a browser-based version of VS Code with the Pebble SDK pre-installed.

Press to launch a cloud development environment:

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/coredevices/codespaces-pebble?quickstart=1)

To resume your Codespace later, visit [cloud.repebble.com](https://cloud.repebble.com).

## Usage

Your Codespace will launch into an example project with the C file already open. Press the button at the top right of the C file called "Run on Emulator." It will take a minute for this button to appear.

Once you're ready to create your own Pebble app, press the smartwatch icon on the left sidebar of VS Code and select **New Project**.

#### Save to GitHub

You should connect your Codespace to a GitHub repository to ensure permanent storage of your project code. 

To do this, press the Source Control icon in the left sidebar of VS Code, then press "Publish Branch." Approve the pop-up asking to sign in with GitHub, select your GitHub account, then select "Publish to GitHub private repository." When you make changes, you can commit and push to that GitHub repo. All projects in your codespace will be saved in one repo.

#### Import an existing project

In your terminal, paste `code /workspaces/codespaces-pebble`. VS Code will reload and open to that folder.

If there's a "build" folder in the project you want to import, delete it to speed up the upload.

Drag and drop your project into the file list pane on the left. Wait for the upload to complete. Then, press the Pebble icon in the left sidebar, select "Open Project", and select the project you just uploaded. You can now continue developing it!