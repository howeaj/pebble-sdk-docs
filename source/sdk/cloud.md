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

This lets you build Pebble apps without needing to install the Pebble SDK on your computer.

## Usage

Your GitHub Codespace will launch into an example project with the C file already open. Press the button at the top right of the C file called "Run on Emulator." It may take a minute for this button to appear.

You can play around with the example project. Then, once you're ready to create your own Pebble app, press the smartwatch icon on the left sidebar of VS Code and select **New Project**.

Save your changes as a repository in your GitHub account by pressing the Source Control icon in VS Code's left sidebar. All the projects in your codespace will be saved in one repository. 

#### Import an existing project

Press the menu icon on the top-left (three horizontal lines) and select File -> Open Folder. Make sure the pop-up prompts you to open "/workspaces/codespaces-pebble/". Press OK.

If there's a "build" folder in the project you want to import, delete it to speed up the upload.

Drag and drop your project into the pane on the left listing your files and folders. Wait for the upload to complete (you'll see a progress indicator in the bottom status bar). Then, press the Pebble icon in the left sidebar, select "Open Project", and select the project you just uploaded. You can now develop on it!

#### Troubleshooting

If you press "Run on Emulator" and see "Connection refused," press the button again.