# Configuration

This chapter explains all key options for both the Hub and the Agent. Default config files are provided and can be edited directly.

### Hub configuration — `config/hub.properties`

```
# Bot username (without the leading @)
tg.username=

# Telegram API token obtained from @BotFather
tg.token=

# Comma‑separated list of Telegram chat IDs with admin permissions.
# Example: 123456789,987654321
tg.admin_chat_ids=

# Hostname/IP the Hub will advertise for RMI. Must be reachable from Agents.
hub.host=127.0.0.1
hub.port=1099

# Shared secret. If set on Hub, every Agent must set the same token.
rmi.auth.token=

# Localization
# Default language key (see Localization chapter)
translation.default.lang=en
# Whether to cache localized messages in memory (true/false)
translation.cache=true

# Telegram rate limits and event processing
# Global outgoing messages per second allowed by the Hub.
tg.global.rate_limit=30
# Internal event queue capacity
event.queue.capacity=3000
# Thread pool size used to send messages to Telegram
event.sender.pool_size=4
# Queue capacity for the sender pool
event.sender.queue.capacity=1000

# Code generation seed (do not change unless you know what you are doing)
code.salt=1420070400000

# Database settings (SQLite by default)
db.driver=org.sqlite.JDBC
db.url=jdbc:sqlite:config/hub.db
db.user=
db.password=
db.pool.max_size=10
```

Notes

- `tg.username`, `tg.token`, `tg.admin_chat_ids` can be obtained from [Telegram Bot Creation](bot-creation.md).
- `hub.host` should be the public or internal address Agents can reach. If Hub runs behind NAT, forward and publish the correct reachable address.
- `rmi.auth.token` must be consistent across all Hub and Agent instances.
- If you change `db.url`, ensure the JDBC driver matches and the path is writable. By default, SQLite is installed with the distribution.
- Rate limit should stay within Telegram global limits.

### Agent configuration — `game/config/agent.properties`

```
# Identity of this game server in the Hub registry
server.id=1
server.name=Server Name

# Address of the Hub RMI endpoint (the one Agents connect to)
hub.host=127.0.0.1
hub.port=1099

# Address that the Agent advertises for callbacks from the Hub
agent.host=127.0.0.1
agent.port=1100

# Must match Hub if set
rmi.auth.token=

# Polling & heartbeat (milliseconds)
ping.period=5000
events.period=250
```

Notes

- `server.id` must be unique across all Agents connected to the same Hub. This is critical for the system to distinguish between different game servers.
- `agent.host` must be reachable by the Hub (consider NAT/port‑forwarding if needed).
- Keep `rmi.auth.token` secret and consistent across Hub and all Agents.

### Logging configuration — `config/log.properties`

- Controls log formats and levels. You can increase verbosity during troubleshooting.
- A typical way to raise verbosity is to switch the root or specific packages to `DEBUG` level.

### How to open ports

Opening ports is essential because the Hub and Agent communicate using RMI (Remote Method Invocation). For the system to function correctly, the Hub must be able to accept incoming connections from Agents (default: `1099`), and each Agent must be able to accept callback connections from the Hub (default: `1100`). If these ports are blocked by a firewall, the components will not be able to "see" or talk to each other, resulting in connection errors.

- [Opening ports on windows server](https://www.kamatera.com/knowledgebase/how-to-open-a-port-in-windows-firewall/)
- [Opening ports on linux server](https://www.digitalocean.com/community/tutorials/opening-a-port-on-linux)

### Applying configuration changes

- For most settings, restart the Hub/Agent to apply changes.
- Safe practice: back up the original files before editing, and use version control to track your changes.