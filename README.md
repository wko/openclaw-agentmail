# openclaw-agentmail

AgentMail email channel plugin for [OpenClaw](https://openclaw.ai) — secure, reply-only email integration via [AgentMail](https://agentmail.to).

## Features

- **Real-time email via WebSocket** — No polling, instant message delivery
- **Reply-only mode** — Cannot send emails to arbitrary addresses (security hardening)
- **Sender filtering** — `allowFrom` whitelist for allowed senders
- **Thread context** — Full email thread loaded for AI context
- **No agent tools** — Direct email API access disabled for safety

## Security

This plugin has been hardened for security:

1. **Outbound restricted to replies only** — The plugin throws an error if attempting to send a new email without a `replyToId`. Only replies to existing threads are allowed.

2. **Agent tools disabled** — The AgentMail toolkit is not exposed to the agent, preventing programmatic email sending.

3. **Sender filtering** — Only emails from addresses/domains in `allowFrom` trigger the agent.

## Installation

```bash
openclaw plugins install openclaw-agentmail
```

Or add to your OpenClaw config:

```json5
{
  plugins: {
    load: {
      paths: ["node_modules/openclaw-agentmail"]
    }
  }
}
```

## Configuration

Add to your OpenClaw config:

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

| Key | Type | Description |
|-----|------|-------------|
| `enabled` | boolean | Enable/disable the channel |
| `token` | string | AgentMail API token (required) |
| `emailAddress` | string | Inbox email address (required) |
| `allowFrom` | string[] | Allowed sender emails/domains (empty = allow all) |

## How It Works

1. Plugin connects to AgentMail via WebSocket on gateway start
2. Incoming emails are filtered by `allowFrom`
3. Full thread context is fetched for the AI
4. Agent's reply is sent via `replyAll` to maintain threading
5. Only replies are allowed — no new emails to arbitrary addresses

## Credits

Based on [PR #2499](https://github.com/openclaw/openclaw/pull/2499) by [@Haakam21](https://github.com/Haakam21), with security hardening.

## License

MIT
