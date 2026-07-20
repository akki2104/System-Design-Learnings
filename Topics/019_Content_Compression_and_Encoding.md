# Topic 019 — Content Compression & Encoding

**Module:** 1 — Networking & Communication Foundations
**Status:** Completed
**Date:** 2026-07-14
**Confidence:** 4/5
**Difficulty:** Easy (Survey)

---

## 1. Why This Topic Exists

Fills in what actually gets sent over the wire — compression trades CPU for bandwidth/latency, the same tradeoff shape seen throughout this module. Completes Module 1.

---

## 2. Core Concepts

### Text compression — negotiation via headers

```
Client: "Accept-Encoding: gzip, br"   ← formats the client can handle
Server: "Content-Encoding: br"        ← format the server actually used
```

```
gzip   → older, universally supported, decent compression, fast to compress
brotli → newer, better compression ratio at same quality, more CPU at max settings
```

**Key insight:** for static assets (JS/CSS bundles), compression happens **once at build/deploy time** — so brotli's higher one-time cost doesn't matter, since it's amortized across millions of served requests. This typically happens at the **reverse proxy/CDN layer** (Topics 016, 018), not application code — transparent and centrally managed.

### Image encoding — lossy vs lossless

```
JPEG        → lossy (discards some detail), great for photos, small size
PNG         → lossless (pixel-perfect), needed for transparency/sharp graphics, larger
WebP / AVIF → modern, better compression than JPEG/PNG at similar quality,
              needs client/browser support (fallback required)
```

**Lossy vs lossless is a real, non-negotiable tradeoff in some cases** — e.g., legal document scans/data extraction, where lossless is required because every pixel/character must be preserved for the extraction to be trustworthy. Lossy is fine where a human eye won't notice the difference (most photos).

### Video encoding — codecs

```
H.264 (older, universal)     → good compression, hardware-decodable everywhere
H.265/HEVC, VP9, AV1 (newer) → better compression, but MORE CPU to encode/decode,
                                 and not universally supported
```

**The "just use the newest codec" trap — three costs, not one:**
```
1. Encode cost   → more CPU/GPU time on the SERVER to compress
2. Decode cost   → more CPU/battery drain on the USER'S device (not the server's)
3. Compatibility → older devices/browsers may not decode it at all — needs a fallback
```

Newer codecs need a **fallback strategy** (e.g., H.264 baseline + AV1 for clients that support it), not a single universal choice — because decode cost and compatibility live on devices you don't control.

---

## Tech Decision Box

```
Compress text (HTML/CSS/JS/API responses) → gzip (safe default) or brotli
                                              (better ratio, modern browsers)
Serve images                              → WebP/AVIF with JPEG/PNG fallback
Serve video                               → H.264 baseline + newer codec for
                                              clients that support it

WHERE IT HAPPENS: reverse proxy / CDN layer (Topics 016, 018) — transparent,
                   centrally managed, not application code
```

**Interview sentence:** "I'll compress API responses with gzip/brotli at the reverse proxy layer since it's a solved, centralized concern. For images I'll serve WebP with a JPEG fallback — better compression at similar quality, with a fallback since not every client supports WebP."

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Thinking "newest codec = always best" only about compression ratio | Also weigh encode cost, decode cost on the user's device, and compatibility/fallback needs |
| Assuming compression happens in application code | It typically happens at the reverse proxy/CDN layer, transparently |

---

## Revision Questions

1. What headers negotiate compression format between client and server, and what does each mean?
2. Why doesn't brotli's higher compression cost matter for static JS/CSS bundles?
3. What's the real distinction between lossy and lossless image formats? Give a scenario where lossless is required.
4. What THREE costs must be weighed before adopting a newer, more efficient video codec?

---

## Cheat Sheet

```
NEGOTIATION
─────────────────────────────────────────────────
Accept-Encoding (client's supported formats) → Content-Encoding (server's choice)

TEXT: gzip (safe default) vs brotli (better ratio, more CPU) — compress ONCE
      at build/deploy for static assets, amortized across millions of requests

IMAGES: lossy (JPEG, small, fine for photos) vs lossless (PNG, pixel-perfect,
        required for legal docs/transparency) vs modern (WebP/AVIF + fallback)

VIDEO CODEC TRADEOFF (not just compression ratio)
─────────────────────────────────────────────────
1. Encode cost   → server CPU/GPU
2. Decode cost   → USER'S device CPU/battery
3. Compatibility → may need fallback (H.264 baseline + newer codec)

WHERE: reverse proxy / CDN layer (Topics 016, 018) — transparent, centralized
```

---

## Summary

- **Compression trades CPU for bandwidth/latency** — the same tradeoff shape as everything else in this module.
- **Accept-Encoding / Content-Encoding** headers negotiate the format between client and server.
- **Static assets compress once, serve millions of times** — brotli's higher one-time cost is fully amortized.
- **Lossy vs lossless is a real, sometimes non-negotiable choice** — legal/precision use cases require lossless.
- **Newer video codecs aren't a free upgrade** — weigh encode cost, decode cost on the USER'S device, and compatibility/fallback needs, not just compression ratio.

> **You now can:** reason about compression tradeoffs for text, images, and video, and correctly evaluate "just use the newest codec" against its full cost — not just its compression ratio.

---

**Module 1 — Networking & Communication Foundations: COMPLETE.** Topics 006–019 done. Next: Module 2 — Data Storage Foundations (Topic 020).
