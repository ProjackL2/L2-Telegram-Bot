# Localization

This chapter explains how to change user‑visible messages and add translations.

### Where messages live

- Hub messages/buttons texts are stored as Markdown files under `config/localization/messages/`.
- Messages: `msg_start.md`, `msg_menu.md`, `msg_error.md`.
- Menu labels: `menu_settings.md`, `menu_admin.md`, etc.
- Files are loaded by the Hub at runtime and rendered as Telegram messages.
- You can disable translation caching by setting `translation.cache=false` in `config/hub.properties`.

#### Editing existing texts

1) If the translation cache is enabled, then stop the Hub (recommended to avoid partial reloads).

2) Open the file you want to change in `config/localization/messages/`.

3) Edit the Markdown content.

4) Start the Hub if it was stopped before and verify the result in Telegram.


### Events and Class Formats (XML)

While general messages are stored in Markdown files, specific data like event notifications and character class names are managed in XML files located directly in `config/localization/`:

- `events.xml`: Defines how game events (Logins, Deaths, Kills, etc.) are formatted in Telegram.
- `classes.xml`: Contains translations for Lineage II character classes.

#### Modifying XML translations
1. Open the desired XML file (`events.xml` or `classes.xml`).
2. Locate the `<tr name="...">` tag for the entry you want to edit.
3. Modify the attributes within the language-specific tag (e.g., `<en ... />` for English or `<ru ... />` for Russian).
   - **name**: Short name used in menus or headers.
   - **message**: The actual text sent to Telegram (supports Markdown).
   - **description**: Internal description of the event.

Key points

- Markdown Support: Use `*bold*`, `_italic_`, and other Telegram-supported Markdown in the `message` attribute.
- Placeholders: Keep placeholders like `%playerName%` or `%serverName%` exactly as they are. They are dynamically replaced by the Hub.
- XML Structure: Do not change the `name` attribute of the `<tr ...>` tag, as it is used by the Hub to identify the event or class.

### Languages and defaults

- Default language is controlled by `translation.default.lang` in `config/hub.properties` (for example, `en`).
- The simplest setup is a single message set (the files listed above) which acts as the default.
- Templated approach for multiple languages:
  - Create a parallel set of message files per language, mirroring keys.
  - Example structure (you can adapt it to your needs):
    - `config/localization/messages/` — default (English)
    - `config/localization/messages/ru/` — Russian
    - `config/localization/messages/es/` — Spanish
  - Then set `translation.default.lang=<your_lang>` if you want to switch the default.
  - If the code requests a specific `lang` for a user, it should load from the matching folder (fallback to default if missing).

### Related in‑game HTML (Agent side)

- Some in‑game content templates live under `data/html/en/mods/tgbot/` and are copied into game server packages.
- You can localize/adjust these HTML files similarly and redeploy the Agent package where needed.