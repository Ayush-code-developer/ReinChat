# ReinChat

**ReinChat 1.0.0** is a configurable chat plugin for Paper servers, providing chat formatting, channels, tags, chat colors, private messaging, moderation filters, broadcasts, AFK status, nicknames, and inventory/item previews in chat.

- **Author:** zRein
- **Main class:** `rein.me.chat.ReinChat`
- **API version:** Paper/Bukkit `1.21`
- **Build Java version:** 21
- **Optional integration:** PlaceholderAPI
- **Required library:** ReinCore (the project declares `rein.me:reincore:1` as a provided dependency)

> This README is based on the supplied ReinChat project ZIP. Check the server's exact Paper version and your installed ReinCore build before deploying.

## Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Commands](#commands)
- [Permissions](#permissions)
- [Configuration files](#configuration-files)
- [Chat channels](#chat-channels)
- [Chat colors and menus](#chat-colors-and-menus)
- [Tags](#tags)
- [Help menu](#help-menu)
- [Private messages and SocialSpy](#private-messages-and-socialspy)
- [Filters and moderation](#filters-and-moderation)
- [Broadcasts](#broadcasts)
- [Item and inventory previews](#item-and-inventory-previews)
- [PlaceholderAPI](#placeholderapi)
- [Reloading](#reloading)
- [Building from source](#building-from-source)
- [Troubleshooting](#troubleshooting)

## Features

- Configurable public chat formats and rank/group prefixes.
- Configurable channels, including global, staff, developer, and builder chat.
- Tag and chat-color selection menus, including solid colors, gradients, and custom hex colors.
- Private messages, replies, ignoring players, and permission-gated SocialSpy.
- Chat filters for spam, repeated messages, links, caps, and configurable rules.
- Scheduled and manual broadcasts.
- AFK status tracking.
- Nicknames and administrative `/sudo` actions.
- Clickable item, inventory, and ender-chest previews in chat, with stored snapshots.
- Configurable `/help` message or GUI menu.
- Optional PlaceholderAPI expansion.

## Requirements

1. A Paper server compatible with the plugin's `api-version: '1.21'`.
2. Java 21 to build this project; use a Java runtime supported by your specific Paper server version.
3. The matching ReinCore library/plugin. ReinChat's Maven project declares ReinCore as `provided`, so it is not shaded into the ReinChat JAR.
4. PlaceholderAPI is optional. ReinChat declares it as a soft dependency and only registers its expansion when PlaceholderAPI is installed.

## Installation

1. Build or obtain the ReinChat JAR.
2. Install the required ReinCore dependency compatible with this build.
3. Place `ReinChat.jar` in the server's `plugins/` directory.
4. Optionally install PlaceholderAPI if you want to use the ReinChat placeholders.
5. Start or restart the server.
6. Inspect `plugins/ReinChat/` and customize the generated YAML files.
7. Restart the server after making changes to plugin files unless you are deliberately using `/reinchat reload`.

**Important:** Do not publish your live `playerdata.yml` or private moderation webhook URL. Keep backups before editing configuration files.

## Commands

Commands are listed according to the command declarations and executors in this source version. Permission checks may also depend on the settings in YAML files.

### Player commands

| Command | Aliases | Usage / behavior |
|---|---|---|
| `/help` | — | Runs the help behavior configured in `help.yml`; may show a message or open a menu. |
| `/msg <player> <message>` | `/tell`, `/whisper`, `/w` | Send a private message. |
| `/r <message>` | `/reply` | Reply to the last private conversation. |
| `/ignore <player>` | — | Toggle ignoring a player's private messages. |
| `/tags` | `/tag` | Open the tag menu, equip a tag by ID, or use `/tag none`, `/tag off`, or `/tag reset` to clear it. |
| `/chatcolor` | `/chatcolors` | Open the chat-color menu, equip a configured color by name, or use `/chatcolor reset`, `/chatcolor none`, or `/chatcolor off`. |
| `/afk` | — | Toggle AFK status. Automatic AFK tracking is also enabled according to the AFK configuration and permission. |
| `/nick <nickname\|off>` | — | Set or reset your own nickname, subject to permissions. |
| `/global [message]` | `/g` | Global channel command, loaded from `channels/global.yml`. A bare command may toggle the active channel. |

The channel commands below are dynamically registered from the YAML files in `channels/`. The command name and aliases can be changed there.

| Default command | Alias | Permission to send/read |
|---|---|---|
| `/staffchat [message]` | `/sc` | `reinchat.channel.staff` |
| `/devchat [message]` | `/dc` | `reinchat.channel.developer` |
| `/buildchat [message]` | `/bch` | `reinchat.channel.builder` |

A channel command can send one message when given an argument or toggle the player's active channel when used without a message. `read-permission` controls who can see channel messages. If omitted, it falls back to `permission`.

### Staff and administration commands

| Command | Aliases | Usage / behavior |
|---|---|---|
| `/reinchat` | — | Shows the appropriate help output. |
| `/reinchat help` | — | Shows admin help to administrators; otherwise delegates to the configured player help. |
| `/reinchat reload` | — | Reloads ReinChat configuration. Requires operator status or `reinchat.admin`. |
| `/reinchat view <id>` | — | Opens a stored item/inventory snapshot by ID. Player-only. Used by clickable preview links. |
| `/socialspy` | `/spy` | Toggle private-message SocialSpy. Player-only and requires `reinchat.socialspy`. |
| `/broadcast <message>` | `/bc` | Send a manual broadcast if manual broadcasts are enabled and the configured permission is granted. |
| `/sudo <player> <command>` | — | Run a command as an online player; requires `reinchat.sudo`. |
| `/sudo <player> c:<message>` | — | Send a chat message as an online player; requires `reinchat.sudo`. |
| `/nick <player> <nickname\|off>` | — | Change or reset another player's nickname; requires the relevant other-player permission. |

**Alias note:** `/bc` is the alias of `/broadcast`. The default builder-channel alias is `/bch`, configured in `channels/builder.yml`, so the two defaults do not collide.

## Permissions

Permissions declared in `plugin.yml` are listed below. Some features use configurable permission strings or additional checks in code, so this table is not every possible permission used by a customized setup.

| Permission | Default | Purpose |
|---|---:|---|
| `reinchat.admin` | Operators | Full ReinChat access; grants the child permissions listed below. |
| `reinchat.view` | Operators | View inventory and ender-chest snapshots. |
| `reinchat.show.item` | Everyone | Use item display in chat. |
| `reinchat.show.inventory` | Everyone | Use inventory display in chat. |
| `reinchat.show.enderchest` | Everyone | Use ender-chest display in chat. |
| `reinchat.show.bypasscooldown` | Operators | Bypass the chat display cooldown. |
| `reinchat.socialspy` | Operators | Toggle and view private-message SocialSpy. |
| `reinchat.broadcast` | Operators | Permission used by the default manual broadcast configuration. |
| `reinchat.afk` | Everyone | Use `/afk` and receive automatic AFK tracking. |
| `reinchat.nick` | Operators | Set your own nickname. |
| `reinchat.nick.others` | Operators | Change or reset another player's nickname. |
| `reinchat.sudo` | Operators | Force a player to execute a command or send a chat message. |
| `reinchat.admin.help` | Operators | View the administration command list. |

`reinchat.admin` includes these child permissions: `reinchat.view`, `reinchat.show.item`, `reinchat.show.inventory`, `reinchat.show.enderchest`, `reinchat.show.bypasscooldown`, `reinchat.socialspy`, `reinchat.broadcast`, `reinchat.afk`, `reinchat.nick`, `reinchat.nick.others`, `reinchat.sudo`, and `reinchat.admin.help`.

### Additional permissions configured in YAML

These permissions are referenced by the default configuration or sample data; assign them if you enable the corresponding feature.

| Permission | Used for |
|---|---|
| `reinchat.channel.staff` | Sending and reading staff-channel messages by default. |
| `reinchat.channel.developer` | Sending and reading developer-channel messages by default. |
| `reinchat.channel.builder` | Sending and reading builder-channel messages by default. |
| `reinchat.filter.bypass` | Bypass chat filters. |
| `reinchat.alerts` | Receive filter alerts, according to `filters.yml`. |
| `reinchat.links` | Allow links that would otherwise be blocked by the links filter. |
| `reinchat.color.<name>` | Use a configured chat color when individual color permissions are enabled by the implementation/configuration. |
| `reinchat.color.hex` | Use custom hex colors through the chat-color menu. |
| `reinchat.chat.legacy` | Use legacy chat color formatting where supported. |
| `reinchat.chat.hex` | Use hex chat formatting where supported. |
| `reinchat.chat.gradient` | Use gradient chat formatting where supported. |
| `reinchat.bundle.<id>` | Use a configured permission bundle, such as `reinchat.bundle.vip` or `reinchat.bundle.mvp`. |
| `reinchat.tag.<id>` | Use the corresponding configured tag, such as `reinchat.tag.veteran`. |

Permission strings for broadcasts, channel use/read access, filter alerts, and tag/color access may be changed in the relevant YAML files. Always check those files if you customize them.

## Configuration files

ReinChat's default resource files are:

| File | Purpose |
|---|---|
| `config.yml` | Chat-color presets, color menu, tag menu, private-message formatting, AFK timeout, hover formatting, and item/inventory preview settings. |
| `messages.yml` | Player-facing messages, welcome lines, AFK/join/quit/death messages, and the administrator help list. |
| `help.yml` | Player help mode (`MESSAGE` or `MENU`), menu layout, item entries, actions, and permissions. |
| `tags.yml` | Available tag IDs, display text, permissions, and descriptions. |
| `permission-bundles.yml` | Bundles that group permissions for supported color/tag access checks. |
| `filters.yml` | Chat filter rules, spam/caps/link handling, bypass permissions, alerts, and optional Discord webhook settings. |
| `broadcasts.yml` | Scheduled broadcast cycle, formatting, messages, and manual broadcast settings. |
| `channels/global.yml` | Public chat format, default group format, group prefixes, and global channel settings. |
| `channels/staff.yml` | Staff channel command, permissions, format, filters, logging, and SocialSpy detection. |
| `channels/developer.yml` | Developer channel settings. |
| `channels/builder.yml` | Builder channel settings. |

### MiniMessage formatting

Chat and menu text uses MiniMessage-style formatting. Examples:

```yaml
prefix: "<dark_gray>[<#b00b69>ReinChat]<dark_gray> <gray>"
example-gradient: "<gradient:#b00b69:#ff69b4>Gradient text</gradient>"
example-bold: "<bold>Important</bold>"
```

Use valid MiniMessage tags and quote YAML strings containing special characters. When a setting expects an item material, use a valid Bukkit/Paper material name.

## Chat channels

Each channel YAML file defines a channel. The filename (without `.yml`) is its channel ID. For example, `channels/builder.yml` creates the `builder` channel.

Important channel settings:

```yaml
enabled: true
display-name: "<yellow><b>Builder"
command: "buildchat"
aliases: ["bch"]
permission: "reinchat.channel.builder"
read-permission: "reinchat.channel.builder"
format: "<dark_gray>[<channel><dark_gray>] <prefix> <yellow><name> <dark_gray>» <white><message>"
sound: ""
apply-filters: true
log-to-console: true
socialspy-detect: false
```

- `enabled`: whether the channel is enabled.
- `display-name`: text substituted for `<channel>` in the format.
- `command` / `aliases`: channel command and aliases.
- `permission`: permission to use/send in the channel.
- `read-permission`: permission to receive channel messages; if omitted, it defaults to `permission`.
- `format`: message format. The default channel implementation supports placeholders including `<channel>`, `<prefix>`, `<tag>`, `<name>`, `<message>`, `<player>`, `<world>`, and `<server>`.
- `sound`: optional sound played to recipients; blank disables it.
- `apply-filters`: whether configured chat filters apply.
- `log-to-console`: whether messages are printed to console.
- `socialspy-detect`: whether channel messages are included in SocialSpy detection.

To create another channel, copy a channel YAML file, give it a new filename, adjust the command, permissions, display name, and format, then reload or restart. Check command/alias collisions before enabling it. The `global.yml` file also contains the public chat and rank/group format configuration and has additional settings.

## Chat colors and menus

Use `/chatcolor` to open the color GUI. The default `config.yml` contains solid presets and gradient/rainbow presets, including `white`, `red`, `gold`, `yellow`, `green`, `aqua`, `blue`, `pink`, `purple`, `magenta`, `sunset`, `ocean`, `fire`, `magenta-glow`, and `rainbow`.

Example:

```yaml
chat-colors:
  magenta: "#b00b69"
  sunset: "<gradient:#ff5f6d:#ffc371>"
  rainbow: "<rainbow>"
```

- Solid colors can use hex values or MiniMessage color syntax supported by the plugin.
- Gradient/rainbow presets use MiniMessage tags.
- `/chatcolor <name>` equips a preset if the player can use it.
- `/chatcolor reset` (also `none` or `off`) restores the default.
- Custom hex selection is available through the GUI when permitted; enter a value such as `#FF55AA` in chat, or type `cancel`.
- `chat-color-icons` optionally overrides the menu icon material for a preset. Without a custom icon, the menu uses a dyed leather item as the color preview.

Color menu configuration lives under `chatcolor-menu` in `config.yml`. It includes GUI titles, item names/descriptions, navigation labels, equipped/locked labels, preview text, and hex-related messages.

## Tags

Configure tag entries in `tags.yml`. The key is the tag ID; the display text is shown in chat/menu, and the permission controls who may use the tag.

```yaml
veteran:
  display: "<gold>[Veteran]"
  permission: "reinchat.tag.veteran"
  description: "<gray>Been here since the early days"
```

Use `/tags` or `/tag` to open the menu. You can equip a tag by ID or clear it with `/tag none`, `/tag off`, or `/tag reset`. The selected tag is applied by the chat format where `<tag>` is included.

## Help menu

`help.yml` supports two modes:

```yaml
mode: MENU # MENU or MESSAGE
```

- `MESSAGE`: sends each configured line under `message.lines`.
- `MENU`: opens an inventory GUI for players. Configure the title, row count (1–6), filler item, opening sound, and entries under `menu.items`.

Example menu item:

```yaml
menu:
  rows: 3
  title: "<dark_gray>Server Help"
  items:
    website:
      slot: 13
      material: COMPASS
      name: "<aqua>Website"
      lore:
        - "<gray>Click to view server information"
      permission: ""
      action: MESSAGE
      value: "<aqua>Visit our website!"
```

Menu item settings include `slot`, `material`, `name`, `lore`, optional `permission`, `action`, and `value`. The item permission controls whether the entry is shown to the player. Supported actions in the implementation are `COMMAND`, `CONSOLE_COMMAND`, `MESSAGE`, `CLOSE`, and `NONE`. Use console commands cautiously: clicking such an item dispatches the configured command as the console. The `/help` command itself can be restricted with `menu.permission` when `MENU` mode is active.

## Private messages and SocialSpy

- `/msg <player> <message>` sends a private message.
- `/r <message>` or `/reply <message>` replies to the last conversation.
- `/ignore <player>` toggles whether private messages from that player are accepted.
- `/socialspy` or `/spy` toggles SocialSpy for a player with `reinchat.socialspy`.
- SocialSpy formatting is configured under `private-messages` in `config.yml`.
- Channel-specific SocialSpy detection is controlled by `socialspy-detect` in each channel file.

SocialSpy is a staff feature by default because `reinchat.socialspy` defaults to operators in `plugin.yml`. It is a player-only command; it cannot be toggled from console.

## Filters and moderation

`filters.yml` configures moderation behavior:

- `enabled`: master filter toggle.
- `bypass-permission`: permission that bypasses filters.
- `alerts`: whether alerts are sent in-game and which permission receives them.
- `anti-spam`: minimum delay and repeated-message detection.
- `links`: action (`BLOCK` or `CENSOR`), allow permission, whitelist, and reason.
- `caps`: minimum letter count and maximum uppercase percentage.
- `rules`: additional configured matching rules and actions.
- `discord`: optional webhook settings.

The default Discord integration is disabled. If enabling it, replace the example webhook URL with your own private webhook and never share it publicly. A webhook URL is a secret.

## Broadcasts

`broadcasts.yml` controls automatic and manual broadcasts.

- `enabled`: master toggle for scheduled broadcasts.
- `interval-seconds`: interval between scheduled broadcasts.
- `cycle`: whether the message list cycles.
- `format`: prefix, whether to use it, and separator formatting.
- `messages`: configured scheduled broadcast messages.
- `manual.enabled`: enable or disable `/broadcast`.
- `manual.permission`: permission required for manual broadcasts (defaults to `reinchat.broadcast`).

Use `/broadcast <message>` to send a manual broadcast when enabled and permitted.

## Item and inventory previews

The `show` section in `config.yml` controls preview behavior, including cooldown, snapshot lifetime, maximum snapshots, and item/inventory/ender-chest display settings.

The permissions declared in `plugin.yml` include:

- `reinchat.show.item`
- `reinchat.show.inventory`
- `reinchat.show.enderchest`
- `reinchat.show.bypasscooldown`
- `reinchat.view` for viewing stored snapshots

The `/reinchat view <id>` subcommand is used by the clickable preview links. Snapshot IDs are temporary and may expire according to configuration.

## PlaceholderAPI

PlaceholderAPI is optional. When present, ReinChat registers its expansion.

The source in this build defines these placeholders:

| Placeholder | Meaning |
|---|---|
| `%reinchat_tag%` | Equipped tag rendered as formatted text. |
| `%reinchat_tag_name%` | Equipped tag name as plain text. |
| `%reinchat_has_tag%` | Whether the player has a selected tag (`true`/`false`). |

If a placeholder returns nothing, confirm PlaceholderAPI is installed and enabled and that the placeholder is supported by the installed version.

## Reloading

Use:

```text
/reinchat reload
```

This requires operator status or `reinchat.admin`. Reloading refreshes the plugin configuration and the associated managers. If you change command declarations or add a new dynamically loaded channel, restart the server if the change does not register cleanly. Always inspect the console for YAML parsing errors after editing configuration.

## Building from source

This project uses Maven.

1. Install JDK 21 and Maven.
2. Ensure the ReinCore dependency declared in `pom.xml` is available in your Maven repository.
3. From the project directory, run:

```bash
mvn clean package
```

4. The built JAR will be in `target/`. Install it together with the required ReinCore dependency.

Paper API and PlaceholderAPI are declared as provided dependencies and are not intended to be bundled into the plugin JAR.

## Troubleshooting

### A command does not work
- Confirm the command exists in `plugin.yml` or is dynamically defined by a channel YAML file.
- Check the sender's permissions.
- Look for command aliases that collide with another plugin or channel.

### A channel command is missing
- Check that the channel YAML file is in `plugins/ReinChat/channels/`.
- Confirm `enabled: true`, and check `command`, `aliases`, and permissions.
- Check the console for registration errors. Restart if a new command/alias does not register after reload.

### A menu entry is missing
- Check its `slot` is within the inventory size.
- Check the item's `permission`; entries with a permission are hidden from players who lack it.
- Check material names and YAML indentation.

### A chat color or tag is unavailable
- Confirm the preset/tag exists in `config.yml` or `tags.yml`.
- Check its permission and any permission bundle settings.
- Use the configured ID, not its display name.

### PlaceholderAPI placeholders do not resolve
- Confirm PlaceholderAPI is installed and loaded.
- Confirm the placeholder name is correct and the expansion registered successfully.

### Configuration reload fails
- Validate YAML indentation and quote strings where necessary.
- Review console output for the file and line that caused the parsing error.
- Restore the last known-good backup if needed.

---

## Release checklist

Before publishing a release, verify the README against the exact JAR and source commit:

- [ ] Test all commands and aliases on the target Paper version.
- [ ] Verify `/bc` opens manual broadcasts and `/bch` opens builder chat.
- [ ] Verify every permission default in `plugin.yml` and configurable permission in YAML.
- [ ] Test both `MESSAGE` and `MENU` help modes.
- [ ] Test channel send/read permissions separately.
- [ ] Test PlaceholderAPI placeholders with and without PlaceholderAPI installed.
- [ ] Confirm all example YAML snippets parse correctly.
- [ ] Remove secrets and live server data from release archives.
