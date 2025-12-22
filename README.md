# n8n Self-Hosting Journal: Docker → AI Agents

My real-time journal of wrestling n8n into a secure, self-hosted beast on macOS—complete with Docker, Cloudflare tunnels, and the MCP roadmap to Claude Desktop agents.

## What's Inside

- **Day 1:** Ditched the n8n desktop app. Docker Compose took over.
- **The Pivot:** `docker-compose.yml` + persistent volumes. No more workflow funerals on updates.
- **Security Lockdown:** Cloudflare Zero Trust tunnel. Zero router ports opened. `https://n8n.yourdomain.com` just works.
- **Next Up:** MCP server mode. Expose n8n workflows as tools for Claude Desktop on my Mac. AI agents that actually touch real data.


## Tech Stack

```
Docker Desktop
├── n8n (latest)
├── Cloudflared tunnel
├── .env (secrets safe)
└── Custom n8n-network
```

## Why Read This?

Tired of SaaS lock-in? Want n8n running 24/7 without your MacBook exploding? This is the no-BS path. Skip the port-forwarding nightmares. [Follow my Cloudflare tunnel setup](https://github.com/rc-vader-ai/n8n-journal/blob/main/cloudflare-tunnel-guide.md).

**Pro Tip:** Generate your `N8N_ENCRYPTION_KEY` with `openssl rand -base64 32`. Don't skip it.

## Roadmap 🚀

- ✅ Docker + Compose on Mac
- ✅ Cloudflare Zero Trust (no ports!)
- 🔄 MCP Server → Claude Desktop integration
- 🔄 AI Agent nodes w/ Sonnet 3.5
- ⏳ Local file tools for server monitoring


## Connect
- [LinkedIn Post]
- Star/Fork if you're self-hosting too. PRs welcome.


<div align="center">⁂</div>

[^1]: Cloudflare-Zero-Trust-Network-Tunnel-with-Docker-n-2c933a6c2f2680c9a2aadf4f297665b6.md

[^2]: https://www.youtube.com/watch?v=Vyppuv98Cgs

