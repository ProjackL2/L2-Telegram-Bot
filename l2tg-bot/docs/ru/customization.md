# Кастомизация

Этот раздел описывает надежные способы адаптации бота и агента под ваш сервер с использованием шаблонов.

## Настройка меню и команд (Hub)

- Тексты меню и справочные сообщения находятся в `config/localization/messages/` (см. раздел [Локализация](localization.md)).

## Настройка скрипта интеграции Agent

- Agent интегрируется с вашим игровым сервером через `AgentScript.java`.
- Что можно изменить:
    - Какие внутриигровые события отслеживать и отправлять в Hub (убийства, уровни, входы и т. д.).
    - Как разрешаются имена/ID игроков.
    - Ограничение или фильтрация часто повторяющихся событий.
- Перезагрузите скрипты в соответствии с документацией вашей серверной сборки или перезагрузите сервер.

## Настройка внутриигрового HTML (Agent)

- Шаблоны HTML находятся в `data/html/en/mods/tgbot/` и упаковываются в каждый дистрибутив Agent в `game/data/html/...`.
- Вы можете редактировать их в соответствии с брендингом и языком вашего сервера.
- После редактирования пересоберите и заново разверните пакет Agent.

## Логирование и диагностика

- Настройте `config/log.properties` для повышения детализации во время тестирования.
- Следите за логами Hub на предмет сообщений о регистрации RMI при подключении Agent.
- Если сообщения задерживаются, проверьте параметры `tg.global.rate_limit`, `event.sender.pool_size` и задержку сети.

## Руководство по интеграции Agent

Agent выступает в качестве моста между вашим игровым сервером и Hub. Интеграция обычно выполняется через скрипт (например, `AgentScript.java`), который инициализирует Agent и подключается к системе событий вашего сервера.

### 1. Реализация Server и Character API

Вы должны предоставить реализации для `IGameServerApi` и `ICharacterApi`, чтобы Agent мог запрашивать статус сервера и данные персонажей.

- [IGameServerApi](game-server-api.md): Предоставляет методы для получения времени аптайма, использования памяти, количества игроков онлайн и разрешения имен/ID персонажей. Также включает метод `reportLinkSuccess(characterId, username)`, который вызывается при успешной привязке персонажа к пользователю Telegram.
- [ICharacterApi](character-api.md): Простая обертка для данных персонажа (имя, уровень, ID класса).

```java
public class MyGameServerApi implements IGameServerApi {
    @Override
    public int getOnlinePlayers() {
        return GameObjectsStorage.getAllPlayersCount();
    }
    
    @Override
    public void reportLinkSuccess(int characterId, String username) {
        // Опционально: Уведомите игрока в игре об успешной привязке
    }
    // ... реализуйте остальные методы
}
```

### 2. Инициализация Agent

Agent является синглтоном. Вам необходимо настроить его, передав вашу реализацию API, а затем запустить.

```java
final Agent agent = Agent.getInstance();
agent.setGameServerApi(new MyGameServerApi());
// ... регистрация событий и слушателей (см. ниже)
agent.start();
```

### 3. Определение и регистрация событий

События — это пакеты данных, отправляемые в Hub. Вам необходимо создать классы, наследующие `l2tg.agent.event.Event`.

- **Имя события (Event Name)**: Должно быть уникальным и соответствовать атрибуту `name` тега `<tr />` в файле `events.xml`.
- **Поля (Fields)**: Все поля вашего класса события автоматически становятся доступными как плейсхолдеры в локализованном сообщении (например, поле `String playerName` можно использовать как `%playerName%` в `events.xml`).

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

// Регистрация (вызывается до agent.start()):
agent.registerEvent(LevelUpEvent.NAME);
```

### 4. Определение специальных админ событий

Agent позволяет отправлять события напрямую администраторам. Вам необходимо создать классы, наследующие `l2tg.agent.event.AdminEvent`.

- **Имя события (Event Name)**: Должно быть уникальным и соответствовать атрибуту `name` тега `<tr />` в файле `events.xml`.
- **Поля (Fields)**: Все поля вашего класса события автоматически становятся доступными как плейсхолдеры в локализованном сообщении (например, поле `String playerName` можно использовать как `%playerName%` в `events.xml`).

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

### 5. Реализация и регистрация слушателей

Слушатели перехватывают внутриигровые события и передают их в Agent. Наследуйте `l2tg.agent.event.EventListener` и реализуйте интерфейсы слушателей вашего игрового сервера.

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

// Регистрация:
agent.registerListener(new LevelChangeListener());
```

### 6. Привязка персонажа

Чтобы игроки могли привязать своего игрового персонажа к Telegram, вы должны предоставить им способ получить уникальную ссылку для привязки.

- Вызовите `Agent.getInstance().getCharacterLink(playerId)`, чтобы получить ссылку.
- Если метод возвращает пустую строку, персонаж уже привязан.
- Вы можете показать эту ссылку игроку через команду bypass или в окне HTML.
- Для лучшего пользовательского опыта вы можете сгенерировать QR-код из этой ссылки с помощью `QrCodeService.getInstance().getOrCreateDdsQrCodeImage(link)` и отправить его клиенту (например, как эмблему клана/pledge crest).

```java
public boolean useBypass(String command, Player player) {
    if (command.startsWith("linktg")) {
        String link = Agent.getInstance().getCharacterLink(player.getObjectId());
        if (link != null) {
            if (link.isEmpty()) {
                // Персонаж уже привязан
            } else {
                // Показать ссылку или QR-код игроку
            }
        }
        return true;
    }
    return false;
}
```
