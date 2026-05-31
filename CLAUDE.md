# CLAUDE.md - cdn-sip

> **Ecosystem Hub:** See [sip-protocol/CLAUDE.md](https://github.com/sip-protocol/sip-protocol/blob/main/CLAUDE.md) for full ecosystem context.

**Repository:** https://github.com/sip-protocol/cdn-sip
**Purpose:** Static asset CDN for the SIP Protocol ecosystem — token logos + demo/showcase videos, served at **cdn.sip-protocol.org** via Vercel.

---

## Quick Reference

**Type:** Static assets — no build step, no framework. Vercel serves the repo **root** directly (`Output Directory: .`).
**Deployment:** `cdn.sip-protocol.org` → Vercel project `sip-cdn` (scope `rectors-projects`). Push to `main` auto-deploys. Migrated off VPS reclabs3 (nginx `/home/sip/cdn`) on 2026-06-01.

```bash
# Add/replace an asset → commit → push (auto-deploys to cdn.sip-protocol.org)
git add tokens/... videos/... && git commit -m "..." && git push

# ALWAYS re-encode screen-recording videos before adding (they ship huge):
ffmpeg -i in.mp4 -c:v libx264 -crf 24 -preset medium -pix_fmt yuv420p \
  -movflags +faststart -fps_mode cfr -r 30 out.mp4

# Verify live headers / CORS
curl -sI https://cdn.sip-protocol.org/tokens/SOL.png
```

---

## Layout

| Path | Description | Consumed by |
|------|-------------|-------------|
| `tokens/<SYM>.{png,jpg,webp}` | Token logos (11) | **sip-mobile** `src/data/tokens.ts` |
| `videos/showcase/solana-privacy-2026/*` | 8 showcase videos | sip-app + sip-website |
| `videos/showcase/monolith-2026/*` | Monolith demo videos | sip-website |
| `videos/showcase/sip-app-demo.mp4` | App demo | sip-app graveyard page |
| `videos/sip-demo.mp4` + `sip-demo-poster.jpg` | Landing demo | sip-website `video-demo.tsx` |
| `vercel.json` | CORS `*` + cache headers (images 1y-immutable, videos 1-day) | — |
| `index.html` | Minimal landing page (not a file index) | — |

---

## Repo-Specific Guidelines

**DO:**
- **Compress videos before committing.** Screen recordings are often exported at 5–26 Mbps; the recipe above re-encodes to ~0.3–0.5 Mbps with no visible loss (the CDN was cut **539 MB → 83 MB** this way — only the over-encoded files; already-lean ones were left untouched).
- Keep `vercel.json` CORS `*` — consumers are cross-origin (mobile app + other sip sites).
- Bust caches with a `?v=N` query when replacing a video **in place** (clients cache videos `max-age=86400`).
- Verify a re-encode by **decoded frame-count** (`ffprobe -count_frames` ≈ `30 × duration`) **+ a visual still** — NOT PSNR (VFR screen-recording sources misalign frames → false-low PSNR even when identical).

**DON'T:**
- Don't commit a single file > 100 MB (GitHub hard limit) — compress first.
- Don't rename a path a consumer references without updating the consumer (token symbols, showcase filenames).
- Don't trust a Chrome-MCP tab to confirm playback — a non-focused MCP tab is `visibilityState:hidden`, so Chrome suspends media (`readyState` stuck at 0). Use ffprobe + Range serving instead.

---

## Rollback / VPS

VPS nginx (`reclabs3:/home/sip/cdn`) stays as rollback until ~2026-06-08, then decommissioned (its `/home/sip/cdn` files kept 30+ days as backup). DNS rollback: Cloudflare record `cdn.sip-protocol.org` → `A 151.245.137.75, proxied:true`.

---

**Last Updated:** 2026-06-01
