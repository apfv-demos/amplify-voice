# amplify-voice

A Claude Code plugin with Voice AI tools for the community. Deploy webhook infrastructure, create demo agents, and manage Retell AI integrations.

## Available Skills

| Skill                        | Command                 | Description                                                                                                           |
| ---------------------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Retell Webhook Forwarder** | `/amplify-voice:deploy` | Deploy a Cloudflare Worker that filters Retell AI webhooks before forwarding to n8n. Saves 50-80% on execution costs. |

*More skills coming soon.*

## Install

### Option A: Claude Desktop — Code Tab (Recommended)

Best for a fully hands-on experience. Claude runs commands directly on your machine.

1. **[Download amplify-voice.zip](https://github.com/apfv-demos/amplify-voice/raw/main/amplify-voice.zip)**
2. Open Claude Desktop > **Settings** > **Plugins**
3. Click **+** > **Upload plugin**
4. Select the downloaded ZIP

Use `/amplify-voice:deploy` to start deploying the webhook forwarder.

### Option B: Claude Code CLI / VS Code

If you only need the deploy skill without the full plugin:

```bash
git clone https://github.com/apfv-demos/retell-webhook-forwarder.git
cp -r retell-webhook-forwarder/skill ~/.claude/skills/retell-webhook-forwarder
```

Then type `/retell-webhook-forwarder` to start the guided setup.

### Option C: Claude Desktop — Chat Tab (Skill Upload)

Download the skill ZIP for the simplest guided experience (Claude explains steps, you run commands):

1. **[Download retell-webhook-forwarder.zip](https://github.com/apfv-demos/retell-webhook-forwarder/raw/main/retell-webhook-forwarder.zip)**
2. Open Claude Desktop > **Settings** > **Capabilities** > **Skills**
3. Click **+ Add** > **Upload a skill**
4. Select the downloaded ZIP

## Related Repos

- [retell-webhook-forwarder](https://github.com/apfv-demos/retell-webhook-forwarder) — The Cloudflare Worker source code
- [retell-voice-demo](https://github.com/apfv-demos/retell-voice-demo) — Voice agent demo template

## License

MIT
