# agent-media

**One app. Every genre.**

Short drama series from Claude, ChatGPT or Codex.

[![License](https://img.shields.io/badge/license-Apache--2.0-green)](LICENSE)

agent-media makes short drama series (micro-dramas, sitcoms, K-drama style) and other short vertical videos, entirely from Claude, ChatGPT or Codex through one MCP connection. OAuth sign-in, no API keys, nothing to install.

Your AI writes the script, casts two leads and builds three locations. It shows you the cast, the Ep1 script and the price for approval, films a pilot, then the rest of the season with the same cast every episode.

- Micro-dramas, sitcoms, K-drama style and other short vertical videos
- Works from Claude, ChatGPT or Codex through one MCP connection, OAuth sign-in, no API keys
- You approve the cast, the Ep1 script and the price before the pilot is filmed
- Same cast every episode, from the pilot to the end of the season
- Vertical 9:16 episodes up to 60 s, 480p or 720p, captions, no watermark

Watch examples: [agent-media.ai/series](https://agent-media.ai/series) (the first episodes of every show are free to watch).

## Connect

Server URL:

```
https://api.agent-media.ai/mcp
```

**Claude:** Customize, Connectors, +, Add custom connector. Paste the URL, choose Connect and sign in through the browser. Then turn on Agent Media in your chat.

**Claude Code:** `claude mcp add --transport http --scope user agent-media https://api.agent-media.ai/mcp`, then run `/mcp`, select agent-media and sign in.

**ChatGPT:** Plugins, +, Add custom MCP server. Paste the URL, create it as a plugin and sign in through the browser. Custom connectors depend on your ChatGPT plan.

**Codex:** `codex mcp add agent-media --url https://api.agent-media.ai/mcp`, then `codex mcp login agent-media` and sign in through the browser.

## Try asking

- "Make a K-drama style micro-drama in Korean about two rivals who run cafes on the same street."
- "Start a sitcom about two roommates who share one tiny apartment. Show me the cast and the Ep1 script before you film anything."
- "Make a short drama series in Spanish at 720p. Film the pilot, then the rest of the season with the same cast."

## Help and community

Questions, ideas or an episode to share: join the [agent-media Discord](https://discord.gg/quSTCdmnF). Account and billing: support@agent-media.ai.

## Plans

| Plan | Seconds at 480p | Seconds at 720p |
| --- | --- | --- |
| $49/mo | 300 | 150 |
| $149/mo | 900 | 450 |
| $499/mo, for brands | 3,600 | 1,800 |
| $2,000/mo, production | 18,000 | 9,000 |

Monthly plans, paid at signup, no free trial. Full refund of your first payment within 7 days if you haven't filmed an episode yet.

Enterprise (more than 18,000 seconds a month): yuval@agent-media.ai.

Website: [agent-media.ai](https://agent-media.ai). Support: support@agent-media.ai.
