# openclaw-agentmail

AgentMail email channel plugin for [OpenClaw](https://openclaw.ai) — secure, reply-only email integration via [AgentMail](https://agentmail.to).

> ⚠️ **Status: Work in Progress**
>
> This plugin works when copied into the `extensions/` folder but **external installation via npm/CLI is not yet fully supported** due to OpenClaw's strict config schema validation. See [Known Limitations](#known-limitations).

## Features

- **Real-time email via WebSocket** — No polling, instant message delivery
- **Reply-only mode** — Cannot send emails to arbitrary addresses (security hardening)
- **Sender filtering** — `allowFrom` whitelist for allowed senders
- **Thread context** — Full email thread loaded for AI context
- **Attachment support** — Downloads attachments and makes them available to the agent
- **Single email replies** — Batch mode ensures one coherent response per email
- **No agent tools** — Direct email API access disabled for safety

## Security

This plugin has been hardened for security:

1. **Outbound restricted to replies only** — The plugin throws an error if attempting to send a new email without a `replyToId`. Only replies to existing threads are allowed.

2. **Agent tools disabled** — The AgentMail toolkit is not exposed to the agent, preventing programmatic email sending.

3. **Sender filtering** — Only emails from addresses/domains in `allowFrom` trigger the agent.

## Installation

### Option 1: Copy to extensions/ (Recommended)

The most reliable method — copy the plugin source directly:

```bash
# Clone and copy to OpenClaw extensions folder
git clone https://github.com/wko/openclaw-agentmail.git
cp -r openclaw-agentmail ~/.openclaw/extensions/agentmail

# Or for a clawdbot fork/container:
cp -r openclaw-agentmail /path/to/clawdbot/extensions/agentmail
```

Then configure in your OpenClaw config (see [Configuration](#configuration)).

### Option 2: Docker / Containerized

For Docker deployments, copy the plugin into your `extensions/` folder in the repo:

```bash
# In your clawdbot fork
cp -r /path/to/openclaw-agentmail extensions/agentmail
rm -rf extensions/agentmail/node_modules extensions/agentmail/.git
git add extensions/agentmail
git commit -m "feat: add agentmail extension"
```

OpenClaw discovers plugins via the `extensions/` folder automatically.

### ❌ Not Yet Working: CLI Install / npm Dependency

The following methods **do not currently work** due to config schema limitations:

```bash
# These will fail with schema validation errors:
openclaw plugins install github:wko/openclaw-agentmail
openclaw plugins install openclaw-agentmail
```

See [Known Limitations](#known-limitations) for details.

## Configuration

Add to your OpenClaw config (`~/.openclaw/openclaw.json` or `config.yaml`):

```json5
{
  channels: {
    agentmail: {
      enabled: true,
      token: "am_your_agentmail_api_token",
      emailAddress: "your-inbox@agentmail.to",
      allowFrom: [
        "trusted@example.com",
        "example.org"  // allows all @example.org
      ]
    }
  }
}
```

### Environment Variables

Alternatively, use environment variables:

- `AGENTMAIL_TOKEN` — API token
- `AGENTMAIL_EMAIL_ADDRESS` — Inbox address

## Configuration Options

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable/disable the channel |
| `token` | string | — | AgentMail API token (required) |
| `emailAddress` | string | — | Inbox email address (required) |
| `allowFrom` | string[] | `[]` | Allowed sender emails/domains (empty = allow all) |
| `blockStreaming` | boolean | `false` | Send intermediate replies as separate emails. When `false` (default), only the final reply is sent as one email. |

## How It Works

1. Plugin connects to AgentMail via WebSocket on gateway start
2. Incoming emails are filtered by `allowFrom`
3. **Attachments are downloaded** and saved to temp directory
4. Full thread context is fetched for the AI
5. Agent receives email with `MediaPath`/`MediaPaths` pointing to attachments
6. Agent's reply is sent via `replyAll` to maintain threading
7. Only replies are allowed — no new emails to arbitrary addresses

## Attachment Handling

Email attachments are automatically downloaded and made available to the agent:

- **Images** (JPEG, PNG, etc.) — Agent can view via vision capabilities
- **Documents** (PDF, TXT, etc.) — Saved locally, path provided to agent
- **All files** — Downloaded to temp directory, paths in `MediaPaths` context field

The agent sees attachments the same way as other channels (Telegram, WhatsApp):

```
MediaPath: /tmp/openclaw-agentmail/<uuid>/invoice.pdf
MediaPaths: ["/tmp/.../invoice.pdf", "/tmp/.../photo.jpg"]
MediaTypes: ["application/pdf", "image/jpeg"]
```

**Note:** Attachments are saved to a temp directory and cleaned up by the OS. For persistent storage, the agent can copy files to a permanent location.

## Known Limitations

### External Plugin Loading Not Supported

OpenClaw's current plugin system has limitations that prevent external plugins from being loaded via npm:

1. **Strict Config Schema** — `plugins.entries` only accepts predefined plugin IDs. Adding `agentmail` to the config fails Zod validation with "additionalProperties not allowed".

2. **No Dynamic Schema Extension** — External plugins cannot register their config schema at runtime, so their configuration options are rejected.

3. **`plugins.load.paths` Insufficient** — While plugins can be loaded from custom paths, configuration still fails schema validation.

**Workaround:** Copy the plugin source to `extensions/` as described above. The plugin is then discovered and loaded as a "known" extension.

**Future:** This will be fixed when OpenClaw supports dynamic plugin schemas or relaxes `plugins.entries` validation for external plugins.

### npm Package

The package is published to npm as `openclaw-agentmail` but cannot be used as an npm dependency until the schema issues are resolved. The npm package exists for future compatibility.

## Development

```bash
git clone https://github.com/wko/openclaw-agentmail.git
cd openclaw-agentmail
pnpm install
pnpm test
```

To test locally with OpenClaw:

```bash
# Copy to your local OpenClaw extensions
cp -r . ~/.openclaw/extensions/agentmail
# Restart OpenClaw gateway
```

## Credits

Based on [PR #2499](https://github.com/openclaw/openclaw/pull/2499) by [@Haakam21](https://github.com/Haakam21), with security hardening.

## License

MIT
