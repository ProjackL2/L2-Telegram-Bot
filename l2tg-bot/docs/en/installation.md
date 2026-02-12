# Installation

This chapter guides you through installing the Hub (telegram bot server) and the Agent (game‑server side component).
 
### Prerequisites

- Java 21 or newer installed and on PATH (`java -version`).
- Network connectivity between the Hub host and each Agent host (default ports: Hub `1099`, Agent `1100`).
- A game server with a working integration script a.k.a `AgentScript.java`.

### Hub package

   - Copy the contents of `hub/` to your target server.
   - Files of interest:
     - `l2tg-hub-1.0.jar` — executable Hub JAR.
     - `config/hub.properties` — hub configuration.
     - `config/log.properties` — logging configuration.
     - `start.bat` / `start.sh` — convenience launch scripts.
   - Edit `config/hub.properties` and set at least:
     - `tg.username`, `tg.token`, `tg.admin_chat_ids` (get these values from [Telegram Bot Creation](bot-creation.md)).
     - `hub.host` (public or internal IP/hostname visible to Agents).
   - Start Hub:
     - Windows: double‑click `start.bat` or run `start.bat` in a console.
     - Linux/macOS: `chmod +x ./start.sh && ./start.sh`
     - Or run directly: `java -jar l2tg-hub-1.0.jar`

### Agent package

   - Copy the resulting folder from `temp/agent-<target>/` to your game server machine.
   - Files of interest inside Agent package:
     - `libs/l2tg-agent-1.0.jar` — Agent binary.
     - `game/config/agent.properties` — Agent configuration.
     - `game/data/scripts/services/AgentScript.java` — integration script for your game server.
     - `game/data/...` — additional data/HTML used by the bot.
   - Edit `game/config/agent.properties`:
     - Point `hub.host`/`hub.port` to your Hub instance.
     - Set unique `server.id` and friendly `server.name`.
     - Ensure `rmi.auth.token` matches Hub (if used).
   - Deploy `AgentScript.java` to your game server scripts and reload scripts following your server’s procedure, or restart the server.

### Deploying Multiple Agents

If you have more than one game server:

1. Repeat the "Agent package" steps for each game server.
2. In each Agent's `game/config/agent.properties`, assign a **unique** `server.id` (e.g., `1`, `2`, `3`) and a descriptive `server.name`.
3. If multiple Agents run on the **same physical machine**, they must use different `agent.port` values (e.g., `1100`, `1101`, etc.) to avoid port conflicts.

### Firewall and networking

- Open Hub port `1099/tcp` on the Hub host for inbound connections from Agents.
- Open Agent port `1100/tcp` on each Agent host for inbound connections from the Hub (for RMI callbacks).
- Ensure that `hub.host` and `agent.host` values are reachable from the opposite side.

### Upgrading

- Stop the Hub/Agent processes.
- Replace the JAR(s) with newer versions.
- Review and merge changes of allowed configuration/localization files.
- Start the services again.