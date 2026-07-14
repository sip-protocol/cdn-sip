<!-- Satellite context file — extends the global hub (~/.claude/CLAUDE.md | ~/.pi/agent/AGENTS.md). Host-neutral; project-specific only. Do not duplicate hub standards here. -->

# cdn-sip

> Static asset CDN for the SIP Protocol ecosystem — token logos + demo/showcase videos, served at **cdn.sip-protocol.org** via Vercel.

**Ecosystem hub:** See [sip-protocol/sip-protocol/AGENTS.md](https://github.com/sip-protocol/sip-protocol/blob/main/AGENTS.md) for full ecosystem context.

## Quick Reference

**Type:** Static assets — no build step, no framework. Vercel serves the repo **root** directly (`Output Directory: .`).
**Deployment:** `cdn.sip-protocol.org` → Vercel project `sip-cdn` (scope `rectors-projects`). Push to `main` auto-deploys. Migrated off VPS reclabs3 (nginx `/home/sip/cdn`) on 2026-06-01.

```bash
# Add/replace an asset → commit → push (auto-deploys)
git add tokens/... videos/... && git commit -m "..." && git push

# ALWAYS re-encode screen-recording videos before adding (they ship huge):
ffmpeg -i in.mp4 -c:v libx264 -crf 24 -preset medium -pix_fmt yuv420p \
  -movflags +faststart -fps_mode cfr -r 30 out.mp4

# Verify live headers / CORS
curl -sI https://cdn.sip-protocol.org/tokens/SOL.png
```

## Layout

| Path | Description | Consumed by |
|------|-------------|-------------|
| `tokens/<SYM>.{png,jpg,webp}` | Token logos (11) | **sip-mobile** `src/data/tokens.ts` |
| `videos/showcase/solana-privacy-2026/*` | 8 showcase videos | sip-app + sip-website |
| `videos/showcase/monolith-2026/*` | Monolith demo videos | sip-website |
| `videos/showcase/sip-app-demo.mp4` | App demo | sip-app graveyard page |
| `videos/sip-demo.mp4` + `sip-demo-poster.jpg` | Landing demo | sip-website `video-demo.tsx` |
| `vercel.json` | CORS `*` + cache headers (images 1y-immutable, videos 1-day) | — |
| `index.html` | Minimal landing page | — |

## Repo-Specific Guidelines

**DO:**
- **Compress videos before committing.** Screen recordings often export at 5–26 Mbps; the recipe above re-encodes to ~0.3–0.5 Mbps with no visible loss (CDN cut **539 MB → 83 MB**).
- Keep `vercel.json` CORS `*` — consumers are cross-origin (mobile app + other sip sites).
- Bust caches with `?v=N` when replacing a video in place (clients cache `max-age=86400`).
- Verify a re-encode by **decoded frame-count** (`ffprobe -count_frames` ≈ `30 × duration`) **+ a visual still** — NOT PSNR (VFR screen-recording sources misalign frames → false-low PSNR).

**DON'T:**
- Commit a single file > 100 MB (GitHub hard limit) — compress first.
- Rename a path a consumer references without updating the consumer (token symbols, showcase filenames).
- Trust a Chrome-MCP tab to confirm playback — a non-focused MCP tab is `visibilityState:hidden`, so Chrome suspends media. Use ffprobe + Range serving.