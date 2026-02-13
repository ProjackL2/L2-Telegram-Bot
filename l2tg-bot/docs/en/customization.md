# Customization

This chapter shows safe, templated ways to adapt the bot and agent to your server.

## Customize menus and commands (Hub)

- Menu texts and help messages live in `config/localization/messages/` (see [Localization](localization.md) chapter).

## Customize Agent integration script

- The Agent integrates with your game server via `AgentScript.java`.
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

## Agent integration guide

The Agent acts as a bridge between your game server and the Hub. Integration is typically done via a script (e.g., `AgentScript.java`) that initializes the Agent and hooks into your server's event system.

### 1. Implement Server and Character APIs

You must provide implementations for `IGameServerApi` and `ICharacterApi` so the Agent can query server status and character details.

- [IGameServerApi](game-server-api.md): Provides methods for uptime, memory usage, online player count, and character resolution (ID to Name, Name to ID). It also includes `reportLinkSuccess(characterId, username)` which is called when a character is successfully linked to a Telegram user.
- [ICharacterApi](character-api.md): A simple wrapper for character data (Name, Level, Class ID).

```java
public class MyGameServerApi implements IGameServerApi {
    @Override
    public int getOnlinePlayers() {
        return GameObjectsStorage.getAllPlayersCount();
    }
    
    @Override
    public void reportLinkSuccess(int characterId, String username) {
        // Optional: Notify the player in-game about successful linking
    }
    // ... implement other methods
}
```

### 2. Initialize the Agent

The Agent is a singleton. You should configure it with your API implementation and then start it.

```java
final Agent agent = Agent.getInstance();
agent.setGameServerApi(new MyGameServerApi());
// ... register events and listeners (see below)
agent.start();
```

### 3. Define and Register Events

Events are data packets sent to the Hub. You should create classes that inherit from `l2tg.agent.event.Event`.

- **Event Name**: Must be unique and match the `name` attribute of a `<tr />` tag in `events.xml`.
- **Fields**: All fields in your event class are automatically available as placeholders in the localization message (e.g., a field `String playerName` can be used as `%playerName%` in `events.xml`).

```java
public static class LevelUpEvent extends Event {
    public static String NAME = "level_up";
    private final String playerName;
    private final int level;

    public LevelUpEvent(int playerId, String playerName, int level) {
        super(NAME, playerId);
        this.playerName = playerName;
        this.level = level;
    }
}

// Registration (call this before agent.start()):
agent.registerEvent(LevelUpEvent.NAME);
```

### 4. Define special Admin Events

Agent allows sending special events directly to admin users. You should create classes that inherit from `l2tg.agent.event.AdminEvent`.

- **Event Name**: Must be unique and match the `name` attribute of a `<tr />` tag in `events.xml`.
- **Fields**: All fields in your event class are automatically available as placeholders in the localization message (e.g., a field `String playerName` can be used as `%playerName%` in `events.xml`).

```java
public static class ReachedLevelEvent extends AdminEvent {
    public static String NAME = "level_up";
    private final String playerName;

    public ReachedLevelEvent(String playerName, int level) {
        super(NAME);
        this.playerName = playerName;
    }
}

// Registration (call this before agent.start()):
agent.registerEvent(ReachedLevelEvent.NAME);
```

### 5. Implement and Register Listeners

Listeners catch in-game events and submit them to the Agent. Inherit from `l2tg.agent.event.EventListener` and implement your game server's listener interfaces.

```java
public static class LevelChangeListener extends EventListener implements OnLevelChangeListener {
    @Override
    public void onLevelChange(Player player, int oldLvl, int newLvl) {
        if (oldLvl < newLvl) {
            Agent.getInstance().submitEvent(new LevelUpEvent(player.getObjectId(), player.getName(), newLvl));
        }
    }

    @Override
    public void register() {
        PlayerListenerList.addGlobal(this);
    }

    @Override
    public void unregister() {
        PlayerListenerList.removeGlobal(this);
    }
}

// Registration:
agent.registerListener(new LevelChangeListener());
```

### 6. Character Linking

To allow players to link their in-game character to Telegram, you must provide a way for them to obtain a unique linking URL.

- Call `Agent.getInstance().getCharacterLink(playerId)` to get the URL.
- If it returns an empty string, the character is already linked.
- You can show this link to the player via a bypass command or an HTML window.
- For a better user experience, you can generate a QR code from this link using `QrCodeService.getInstance().getOrCreateDdsQrCodeImage(link)` and send it to the client (e.g., as a pledge crest).

```java
public boolean useBypass(String command, Player player) {
    if (command.startsWith("linktg")) {
        String link = Agent.getInstance().getCharacterLink(player.getObjectId());
        if (link != null) {
            if (link.isEmpty()) {
                // Character already linked
            } else {
                // Show link or QR code to player
            }
        }
        return true;
    }
    return false;
}
```
