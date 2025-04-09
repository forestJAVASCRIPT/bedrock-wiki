---
title: Basic Chat Commands
description: Custom commands using scripts, before the CustomCommandRegistry comes out.
category: Tutorials
tags:
    - experimental
mentions:
    - cda94581
    - FrankyRay
    - destruc7ion
    - jannik-de
    - riesters
    - Fabrimat
    - SmokeyStack
    - CrackedMatter
    - JaylyDev
    - Herobrine643928
    - conmaster2112
    - kumja1
    - modmaker101
    - SimpleDevMCBE
    - QuazChick
    - forestJAVASCRIPT
---

::: warning
The Script API is currently in active development, and breaking changes are frequent. This page assumes the format of Minecraft 1.21.70
:::

Who doesn't want cool custom commands? With the Script API, you can create your own. In this article, we will be creating them using the Script API.

## Setup Pack

:::tip
Before creating a script, it is recommended to learn the basics of JavaScript, Add-Ons, and the Script API. To see what the Script API can do, see the [Microsoft Docs](https://learn.microsoft.com/en-us/minecraft/creator/scriptapi/)
:::

Assuming you have understood the basics of scripting, let's start creating the pack.

<CodeHeader>BP/manifest.json</CodeHeader>

```json
{
    "format_version": 2,
    "header": {
        "name": "Custom Commands",
        "description": "Custom Commands using the Script API",
        "uuid": "c8c3239f-027f-4e80-890f-880eba65027d",
        "min_engine_version": [1, 19, 40],
        "version": [1, 0, 0]
    },
    "modules": [
        {
            "description": "Behavior Pack Module",
            "type": "data",
            "uuid": "cd2cd41a-1849-410e-8f0a-5d30fde4bd9a",
            "version": [1, 0, 0]
        },
        {
            "description": "Gametest Module",
            "type": "script",
            "language": "javascript",
            "entry": "scripts/main.js",
            "uuid": "f626740d-50a6-49f1-a24a-834983b72134",
            "version": [1, 0, 0]
        }
    ],
    "dependencies": [
        {
            "module_name": "@minecraft/server",
            "version": "2.0.0-beta" // needs to be the latest or it will break ( latest as of 1.21.70 )
        }
    ]
}
```

In our manifest, we have added script module. The `entry` is where our script file is stored. This is typically within the `scripts` folder of the behavior pack. The dependency allows us to use that script module in our script.

<FolderView :paths="[
    'BP/manifest.json',
    'BP/pack_icon.png',
    'BP/scripts/main.js'
]" />

## Creating Custom Commands

Now comes the fun part - creating our custom commands. First, we will add the module.

<CodeHeader>BP/scripts/main.js</CodeHeader>

```js
import { world } from "@minecraft/server";
```

Next, we will make an array of commands that can be used and add a prefix that can later be changed
In this case, we'll use gamemode commands

<CodeHeader>BP/scripts/main.js</CodeHeader>

```js
world.beforeEvents.chatSend.subscribe((res) => {
    const { sender: player, message: msg } = res;
    let commands = [
        "gmc",
        "gma",
        "gms",
        "gmspect"
    ];

    const prefix = "!";

    try {
        res.cancel = true; // Makes the message not appear in chat
        if (!msg.startsWith(prefix)) return; // If the command doesn't start with our prefix, stop the code.

        const command = msg.slice(1).trim(); // This becomes the command, the prefix is sliced out.
        if (command.length === 0) throw "Please enter a command!";
        if (!commands.includes(command)) throw "Invalid command.";
        
        switch (command) {
            case "gmc":
                player.setGameMode(GameMode.creative);
                break;
            case "gma":
                player.setGameMode(GameMode.adventure);
                break;
            case "gms":
                player.setGameMode(GameMode.survival);
                break;
            case "gmspect":
                player.setGameMode(GameMode.spectator);
                break;
        } // We can do better than this, but for readability and understanding the code, I will stick with a switch statement.


    } catch(err) {
        player.sendMessage(err)
    }
});
```

This is the main function to execute our commands. `world.beforeEvents.chatSend.subscribe()` will run before chat messages get sent.

-   A `switch` statement runs through the possible options for the value, and if it matches, runs the code until the next `break` statement.
-   `res.cancel = true` will cancel the chat message that will be sent- similar to how vanilla commands work.
-   `const player = data.sender` declares the variable `player` to be used later.
-   `player.setGameMode` is a method that allows us to set the gamemode of a player without commands.

-   `try{} catch(err){}` Attempts code and if there are errors, catches them.

## Limited Command Usage by Tags

This function will always be checking if the player types the special message to activate the command, even if the player shouldn't have access. To prevent this, we can use tags to limit these commands to specific people.

For example, let's make our commands usable only to players that have the `Admin` tag.

<CodeHeader>BP/scripts/main.js</CodeHeader>

```js
world.beforeEvents.chatSend.subscribe((res) => {
    const { sender: player, message: msg } = res;
    let commands = [
        "gmc",
        "gma",
        "gms",
        "gmspect"
    ];

    const prefix = "!";

    try {
        res.cancel = true; // Makes the message not appear in chat
        if(!player.hasTag('Admin') return;
        if (!msg.startsWith(prefix)) return; // If the command doesn't start with our prefix, stop the code.

        const command = msg.slice(1).trim(); // This becomes the command, the prefix is sliced out.
        if (command.length === 0) throw "Please enter a command!";
        if (!commands.includes(command)) throw "Invalid command.";
        
        switch (command) {
            case "gmc":
                player.setGameMode(GameMode.creative);
                break;
            case "gma":
                player.setGameMode(GameMode.adventure);
                break;
            case "gms":
                player.setGameMode(GameMode.survival);
                break;
            case "gmspect":
                player.setGameMode(GameMode.spectator);
                break;
        } // We can do better than this, but for readability and understanding the code, I will stick with a switch statement.


    } catch(err) {
        player.sendMessage(err)
    }
});
```

In plain text, `if (!eventData.sender.hasTag('Admin')) return;` means: "If the player does NOT (`!`) have the 'Admin' tag, stop the script from running past here (`return`)"

For more information about the Script API, you can reference the [wiki](/scripting/scripting-intro) or the [Microsoft Docs](https://docs.microsoft.com/en-us/minecraft/creator/documents/gametestgettingstarted)
