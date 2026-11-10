# Ask my portfolio — agent

A grounded Q&A agent for [priyeshdave.github.io](https://priyeshdave.github.io). Visitors ask about Priyesh's work; the agent answers only from the site, case studies, résumé and latest dev.to posts, and cites its sources.

```
browser widget ──POST /ask──▶ Cloudflare Worker ──▶ Gemini (gemini-3.1-flash-lite)
                                 │  knowledge.js (built from the site)
                                 └─ latest dev.to posts (cached 1h)
```

- **Grounding:** the whole knowledge base (~16 source-tagged chunks, ~7k tokens) goes into the system instruction; the model returns JSON `{answer, source_ids}` and the Worker maps ids to page links.
- **Guardrails:** answers only about Priyesh, refuses off-topic requests, ignores prompt-injection attempts, never shares a phone number, stays at outcome level on employer work.
- **Abuse limits:** origin allow-list (CORS), 500-character questions, last 6 turns of history, per-IP rate limit, 700 output tokens.
- **Secrets:** `GEMINI_API_KEY` lives only in the Worker — never in the page or the repo.

## Files

| File | Purpose |
|---|---|
| `build_knowledge.py` | Extracts text from `index.html`, `work/*.html` and the résumé PDF into `src/knowledge.js` (phone numbers redacted) |
| `src/index.js` | The Worker: `/ask` and `/health` |
| `wrangler.toml` | Worker name, model and allowed origins |

## Deploy

Pushing to `main` runs `.github/workflows/deploy-agent.yml`, which rebuilds the knowledge base and deploys the Worker. It needs these repository secrets (Settings → Secrets and variables → Actions):

- `CLOUDFLARE_API_TOKEN` — Cloudflare API token from the "Edit Cloudflare Workers" template
- `CLOUDFLARE_ACCOUNT_ID` — from the Cloudflare dashboard
- `GEMINI_API_KEY` — from Google AI Studio

After the first deploy, set `AGENT_URL` in `index.html` to the Worker's `workers.dev` URL. To change the model, edit `GEMINI_MODEL` in `wrangler.toml`.
