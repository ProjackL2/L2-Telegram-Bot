# Customization

This chapter shows safe, templated ways to adapt the bot and agent to your server.

## Customize menus and commands (Hub)

- Menu texts and help messages live in `config/localization/messages/` (see Localization chapter).
- For deeper behavior changes, explore the Hub code under `java/l2tg/hub/`:
    - Bot layer: `java/l2tg/hub/bot/` and `java/l2tg/hub/bot/command/**`
    - Services: `java/l2tg/hub/service/**`
- Typical customizations:
    - Add or hide menu items for admins/players.
    - Change how server stats are formatted before sending to Telegram.
- After code changes:
    - Rebuild the Hub: `gradlew.bat obfuscateHub` (or `distHub` for a ready folder).
    - Replace `l2tg-hub-1.0.jar` on your server and restart.

## Customize Agent integration script

- The Agent integrates with your game server via `AgentScript.java` located per target:
    - `target/asgard/script/services/AgentScript.java`
    - `target/kain/script/services/AgentScript.java`
    - `target/eternity-hf/script/services/AgentScript.java`
- These files are copied into the distribution as `game/data/scripts/services/AgentScript.java`.
- What you can change:
    - Which in‑game events to watch and send to the Hub (kills, levels, logins, etc.).
    - How player names/IDs are resolved.
    - Throttling or filtering noisy events.
  - Reload scripts per your server pack documentation or restart the server.

## Customize in‑game HTML (Agent)

- HTML templates live under `data/html/en/mods/tgbot/` and are packaged into each Agent distribution under `game/data/html/...`.
- You can edit these to match your server branding and language.
- After edits, rebuild and redeploy the Agent package.

## Logging and diagnostics

- Adjust `config/log.properties` to increase verbosity while testing.
- Watch Hub logs for RMI registration messages when an Agent connects.
- If messages are delayed, review `tg.global.rate_limit`, `event.sender.pool_size`, and network latency.
