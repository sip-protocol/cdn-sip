# cdn-sip

Static asset CDN for the SIP Protocol ecosystem — served at **cdn.sip-protocol.org** via Vercel.

## Contents

- `tokens/` — token logos (consumed by **sip-mobile** `src/data/tokens.ts`)
- `videos/` — demo + showcase videos (consumed by **sip-app** and **sip-website**)

## Notes

- Showcase videos were re-encoded H.264 **CRF 24**, constant 30 fps, `+faststart` — visually
  identical to the source recordings at ~85% smaller (539 MB → ~83 MB). Originals were exported
  at a wasteful 5–26 Mbps; re-encoding reclaimed the waste with no perceptible quality loss.
- CORS (`Access-Control-Allow-Origin: *`) and cache headers are configured in `vercel.json`.
- Hosted on Vercel (project `sip-cdn`, scope `rectors-projects`). Pushing to `main` auto-deploys.
- Migrated off VPS `reclabs3` (nginx `/home/sip/cdn`) on 2026-06-01.
