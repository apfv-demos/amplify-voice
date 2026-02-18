# amplify-voice

A Claude Code plugin with Voice AI tools for the community. Deploy webhook infrastructure, create demo agents, and manage Retell AI integrations.

## Available Skills

| Skill                        | Command                                    | Description                                                                                                           |
| ---------------------------- | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| **Retell Webhook Forwarder** | `/amplify-voice:deploy-webhook-forwarder`  | Deploy a Cloudflare Worker that filters Retell AI webhooks before forwarding to n8n. Saves 50-80% on execution costs. |
| **Retell Web Call Demo**     | `/amplify-voice:deploy-webcall-demo`       | Deploy a secure voice agent demo website with Google auth, reCAPTCHA, daily quotas, and a polished UI.                |

## Install

### Option A: Claude Desktop — Code Tab (Recommended)

Best for a fully hands-on experience. Claude runs commands directly on your machine.

1. **[Download amplify-voice.zip](https://github.com/apfv-demos/amplify-voice/raw/main/amplify-voice.zip)**
2. Open Claude Desktop and switch to the **Code** tab
3. Click the **Plugins icon** (bottom-left corner, above your profile icon)
4. Click **+** (Add plugin) > **Upload plugin**
5. Browse for or drag in the downloaded ZIP, then click **Upload**
6. Select **Amplify voice** from the plugin list to enable it

Type `/` in the chat to see the available skills, then select `deploy-webcall-demo` or `deploy-webhook-forwarder` to start.

### Option B: Claude Code CLI / VS Code

If you only need a single skill without the full plugin:

```bash
# Webhook Forwarder skill
git clone https://github.com/apfv-demos/retell-webhook-forwarder.git
cp -r retell-webhook-forwarder/skill ~/.claude/skills/retell-webhook-forwarder

# Web Call Demo skill
git clone https://github.com/apfv-demos/amplify-voice.git
cp -r amplify-voice/skills/deploy-webcall-demo ~/.claude/skills/deploy-webcall-demo
```

Then type `/retell-webhook-forwarder` or `/deploy-webcall-demo` to start the guided setup.

### Option C: Claude Desktop — Chat Tab (Skill Upload)

Download individual skill ZIPs for the simplest guided experience (Claude explains steps, you run commands):

**Webhook Forwarder:**
1. **[Download retell-webhook-forwarder.zip](https://github.com/apfv-demos/retell-webhook-forwarder/raw/main/retell-webhook-forwarder.zip)**
2. Open Claude Desktop > **Settings** > **Capabilities** > **Skills**
3. Click **+ Add** > **Upload a skill**
4. Select the downloaded ZIP

## Related Repos

- [retell-webhook-forwarder](https://github.com/apfv-demos/retell-webhook-forwarder) — The Cloudflare Worker source code
- [retell-voice-demo](https://github.com/apfv-demos/retell-voice-demo) — Voice agent demo template

## License

MIT
