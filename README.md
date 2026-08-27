# MNQ Command Desk

A single-page, desktop-only risk calculator and trade journal for **Micro E-mini Nasdaq-100 (MNQ)** futures, built for prop-firm trading across **The 5%ers, Tradiefy, Apex, and Lucid Trading** (selectable via the firm switcher in the header).

**Tech choice: vanilla JS, zero dependencies.** No React/CDN calls at all — this guarantees the app works fully offline from the very first load (no reliance on a CDN script caching correctly), which matters most for a tool you may use on a trading desk with unreliable connectivity.

## Why $/point/contract varies by instrument and (sometimes) broker

For **exchange-traded futures like MNQ**, the dollar value per point is fixed by the exchange (CME), not the broker — one MNQ point = **$2 per contract**. So unlike CFD/forex brokers (where $/pip can vary by broker's pricing), your $/point for MNQ, MES ($1.25), MYM ($0.50), M2K ($0.50) is standardized. It only changes if you're trading a *different* futures product, or a broker quotes fractional/mini contracts differently. The calculator ships MNQ/MES/MYM/M2K as presets but lets you override for any other instrument.

What **does** vary by prop firm is:
- **Max daily loss rules** — usually a fixed $ amount or % of starting balance (5ers, Tradiefy, Lucid typically use a daily loss cap; Apex primarily uses a *trailing* max drawdown instead of a fixed daily cap).
- **Max contracts/lot size allowed** at each account size.
- **Consistency/scaling rules.**

The firm selector only pre-fills the Daily Risk Guard defaults (max daily loss) — always check your specific firm's current rulebook, since these limits change between plans and over time.

## Lot-sizing formula

```
Effective Stop Distance = |Entry − Stop| + Slippage Buffer (pts)
Position Size (contracts) = floor(
   (Capital × Risk%) / (Effective Stop Distance × $/point/contract + Commission per contract)
)
```

- Slippage is **not cosmetic** — it's added directly to the stop distance used both for sizing and for the Potential Loss figure, so your worst-case loss estimate already assumes some adverse fill.
- Commission is a round-turn $ per contract, subtracted from both the loss and profit calculations.
- Margin Required = Position Size × Margin-per-contract (your firm/broker's margin requirement, entered manually — this varies by prop firm and account size).

## Features included

- Position sizing panel with presets for risk % and $/point per contract, long/short toggle
- Advanced margin & costs section (leverage/margin, spread, commission, swap, slippage)
- Partial close / scale-out planner (multiple legs by R-multiple)
- Live results: stop distance, target, potential loss/profit (color coded), margin required, free margin, color-banded risk score, 1:1–1:4 R:R table
- Trade journal: log entries, filter by result/date, export to JSON, stats dashboard (win rate, avg R, total P/L), animated SVG equity curve
- Daily Risk Guard: max daily loss + max trades/day, with a live STOP TRADING banner once either limit is hit today
- Dark theme with teal accent, animated ticker tape, glowing hero lot-size number, fade-in panels
- Installable PWA, fully offline after first load

All data (settings + journal) is stored in your browser's `localStorage` — nothing leaves your device, there's no backend and no account.

## Deploying to GitHub Pages

1. Create a new GitHub repository (e.g. `mnq-command-desk`).
2. Upload all four files to the repo root: `index.html`, `manifest.json`, `service-worker.js`, `icon.svg`.
3. Go to **Settings → Pages**, set **Source** to your main branch (root folder), save.
4. Wait a minute for GitHub to publish, then open the given `https://<username>.github.io/<repo>/` URL **once while online** — this lets the service worker install and cache all the app files.
5. After that first successful online visit, the app works fully offline (add it to your home screen / install as a PWA for the best experience) and will keep working even if you lose connectivity mid-session.
6. To push an update later, bump `CACHE_NAME` in `service-worker.js` (e.g. `mnq-command-desk-v2`) so old caches are cleared and the new version loads.

## Notes / things to double check before relying on this for live risk decisions

- Confirm current daily loss / drawdown rules directly with 5ers, Tradiefy, Apex, and Lucid — prop firm rules change and this tool's presets are a starting point, not a guarantee of compliance.
- Margin-per-contract is entered manually since it depends on your specific firm/account size tier.
- This tool does not place trades or connect to any broker — it's purely a sizing/journal aid.
