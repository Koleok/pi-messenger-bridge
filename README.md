# pi-messenger-bridge

Bridge Matrix chat into pi.

Remote users can interact with your pi coding agent via their messenger app.

https://github.com/user-attachments/assets/cd64360e-e8cd-4820-a67f-bd127c5d6035

## Features

- 📱 Multi-messenger support (Matrix)
- 🔐 Challenge-based authentication (6-digit codes)
- 🎛️ Interactive menu (`/msg-bridge`) for setup and management
- 🔒 Single-instance guard — prevents duplicate bot polling with sub-agents
- 📊 Live status widget (toggleable)
- 💾 Persistent config (auth state, auto-connect, widget preference)
- 🔧 Tool call visibility for remote users
- 📝 Multi-turn conversation support
- 🔑 Secure permissions (chmod 600 for config files, 700 for directories)

## Setup

### 1. Install

```bash
pi install npm:pi-messenger-bridge
```

### 2. Configure Transports

#### Matrix

Works with any Matrix homeserver — Element X, Element Web, FluffyChat, etc. The bot auto-joins rooms it's invited to.

1. Register a bot account on your homeserver (or reuse an existing user)
2. Get an access token: log in once via Element and copy from **Settings → Help & About → Advanced**, or POST to `/_matrix/client/v3/login`
3. Note your homeserver URL (e.g. `https://matrix.org`)

```bash
/msg-bridge configure matrix <homeserver-url> <access-token>
```

Or set via environment variables:

```bash
export PI_MATRIX_HOMESERVER="https://matrix.org"
export PI_MATRIX_ACCESS_TOKEN="syt_..."
```

E2EE is **on by default**. Verify the bot's device once from another Matrix client (Element, etc.) — until verified, encrypted rooms can't be decrypted in either direction.

Set `"encryption": false` in the `matrix` config to disable — useful for non-encrypted rooms only, or to bypass crypto-store/server desync (e.g. `M_UNKNOWN: One time key … already exists`). **Caveat:** with E2EE off, the homeserver sees plaintext, and the bot can't participate in encrypted rooms at all.

### 3. Connect

```bash
/msg-bridge connect
```

### 4. Authenticate Users

When a user messages your bot for the first time, they'll receive a 6-digit challenge code.
The code is displayed in your pi terminal. Share it with the user (e.g., via DM).

The user enters the code in the bot chat to become a trusted user.

## Commands

| Command                                    | Description                                              |
| ------------------------------------------ | -------------------------------------------------------- |
| `/msg-bridge`                              | Open interactive menu (configure, connect, widget, help) |
| `/msg-bridge status`                       | Show connection and user status                          |
| `/msg-bridge connect`                      | Connect to all configured transports                     |
| `/msg-bridge disconnect`                   | Disconnect all transports                                |
| `/msg-bridge configure <platform> [token]` | Set transport credentials via CLI                        |
| `/msg-bridge widget`                       | Toggle status widget on/off                              |
| `/msg-bridge toggletools`                  | Toggle tool call visibility in remote messages           |
| `/msg-bridge help`                         | Show command reference                                   |

### Admin commands (in DM with the bot)

Trusted users can DM the bot directly to manage state. Reply with `/help` for the full list:

| Command                                          | Description                            |
| ------------------------------------------------ | -------------------------------------- |
| `/help`                                          | Show admin command reference           |
| `/trusted`                                       | List trusted users                     |
| `/revoke <userId>`                               | Revoke trust for a user                |
| `/channels`                                      | List enabled channels                  |
| `/enable <chatId> <all\|mentions\|trusted-only>` | Enable a channel                       |
| `/disable <chatId>`                              | Disable a channel                      |
| `/toggletools`                                   | Toggle tool call visibility in replies |

## Configuration

Config is stored at `~/.pi/msg-bridge.json` with secure permissions (chmod 600).

Example config:

```json
{
  "matrix": {
    "homeserverUrl": "https://matrix.org",
    "accessToken": "syt_...",
    "encryption": true
  },
  "auth": {
    "trustedUsers": ["telegram:123", "whatsapp:456"],
    "adminUserId": "telegram:789"
  },
  "autoConnect": true,
  "showWidget": true,
  "debug": false
}
```

## Environment Variables

Environment variables override file config:

- `PI_MATRIX_HOMESERVER` — Matrix homeserver URL (e.g. `https://matrix.org`)
- `PI_MATRIX_ACCESS_TOKEN` — Matrix access token
- `MSG_BRIDGE_DEBUG` — Enable debug logging (true/false)

## Security

- Config file: `~/.pi/msg-bridge.json` (chmod 600 - owner read/write only)
- Config directory: `~/.pi/` (chmod 700 - owner only)
- WhatsApp auth: `~/.pi/msg-bridge-whatsapp-auth/` (chmod 700 - owner only)
- Environment variables take precedence over config file
- Challenge-based authentication for all new users
- Transport-namespaced user IDs prevent impersonation

## Troubleshooting

Enable debug mode to see detailed logs:

```json
{
  "debug": true
}
```

Or:

```bash
export MSG_BRIDGE_DEBUG=true
```

## Architecture

Uses pi's native `sendUserMessage()` and `turn_end` events for two-way communication.
No tool-loop hacks needed — this is the pi-native way.

Single-instance connection guard prevents duplicate polling when sub-agents spawn
(global flag + PID lock file at `~/.pi/msg-bridge.lock`).

## Development

```bash
npm install
npm run build        # compile TypeScript
npm run typecheck    # type-check without emitting
npm run test         # run tests
npm run lint         # biome lint
npm run lint:fix     # biome lint with auto-fix
```

## License

MIT
