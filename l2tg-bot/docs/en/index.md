# L2 Telegram Bot — User Guide

Welcome!

This documentation helps you install, configure, and customize the L2 Telegram Bot and its companion Agent.

Project distributed as compiled JARs and configuration files. The system consists of two parts:

- Hub — a standalone Telegram bot and RMI server that talks to Telegram and coordinates features.
- Agent — a game‑server side component that reports data to the Hub.

### Features

The primary goal of the service is to provide the ability to receive events happening with a character on the game server.

- Flexible configuration of displayed events
- Character status and game session statistics (received drop, adena)
- Multi-server support
- Interface localization

### Multi-Server Support:

The L2 Telegram Bot is designed with a Hub‑Agent architecture that supports **multiple game servers simultaneously**. 

- A single **Hub** can manage connections from multiple **Agents**.
- Each **Agent** corresponds to one game server (e.g., Main, X100, Craft‑PvP).
- In the Telegram bot, users will see which server they are interacting with, and admins can monitor all connected servers from one place.
- **Important**: Each game server must have a unique `server.id` in its configuration.


Use the navigation to the left to find what you need:

- [Installation](installation.md) — prerequisites, build, and run instructions for Hub and Agent.
- [Telegram Bot Creation](bot-creation.md) — how to create a bot with @BotFather and obtain a token.
- [Configuration](configuration.md) — all properties you can set for Hub and Agent.
- [Localization](localization.md) — where messages live and how to edit them.
- [Customization](customization.md) — ways to adapt scripts and behavior for your server.

### Quick start

1) Create a Telegram bot and get its token (see [Telegram Bot Creation](bot-creation.md)).

2) Edit `config/hub.properties` and set `tg.username`, `tg.token`, and `tg.admin_chat_ids`.

3) Start the Hub.

4) Deploy the Agent to your Lineage II server and point it to the Hub host/port.

5) Open Telegram, start your bot, and follow the menu to link your account.

If you are unsure about any step, each chapter below contains a safe, templated path you can follow and later adjust to your needs.