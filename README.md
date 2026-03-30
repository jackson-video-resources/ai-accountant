# AI Accountant — Your Personal Accountancy Firm, Built by Claude Code

> One prompt. Zero accountancy fees. Full tax automation.

This repo contains the one-shot Claude Code prompt that builds a complete, locally-running AI accountancy system on your machine. Drop receipts into a folder. Forward invoices by email. Import trading CSVs from any exchange. Then ask "what do I owe?" and get a real answer.

No cloud. No subscription. No £150/hr accountant. Just Claude Code and a folder.

---

## What It Does

- **Watches your inbox folder** — drop any receipt, invoice, or bank statement in and it gets processed automatically
- **Reads your emails** — forward receipts to a local address and they're filed instantly
- **Imports exchange CSVs** — drop a Binance, Coinbase or Kraken export and it calculates your CGT using the correct pooling method
- **Calculates your taxes** — income tax, NI (Class 2 + 4), capital gains, deductions, payments on account
- **Finds tax savings** — suggests pension contributions, home office claims, CGT allowance usage, and more
- **Alerts you on Telegram** — weekly summaries, deadline warnings at 90/30/7/1 days, and a "confirm to run" button at filing time
- **Remembers everything** — all data lives in a local SQLite database that Claude can query in plain English

Supports **every country in the world**. Pre-built tax adapters for 30+ countries. For anywhere else, Claude generates your country's tax adapter during setup using its knowledge of local tax law — verified and saved to your machine.

---

## Quick Start

**Step 1** — Make sure you have [Claude Code](https://claude.ai/code) and Node.js 18+ installed

**Step 2** — Open Claude Code and paste the contents of [docs/SETUP-PROMPT.md](docs/SETUP-PROMPT.md)

**Step 3** — Answer 8 questions. Watch it build your accountancy firm.

That's it. The entire system is built, configured and running in under 15 minutes.

---

## What Gets Built

```
~/ai-accountant/
├── inbox/
│   ├── receipts/          ← drop any receipt or invoice PDF/image here
│   ├── invoices-sent/     ← your outgoing invoices to clients
│   ├── bank-statements/   ← CSV or PDF bank exports
│   └── exchanges/
│       ├── binance/       ← Binance trade history CSV
│       ├── coinbase/
│       └── kraken/
├── processed/             ← auto-archived after processing
├── reports/               ← generated tax summaries and SA302s
└── exports/               ← accountant-ready packs
```

Plus a full TypeScript application with file watcher, OCR engine, Claude classifier, CGT calculator, UK/US tax engine, Telegram bot, and cron scheduler — all running locally via PM2.

---

## Costs

| Item | Cost |
|------|------|
| Claude Code | $20/month (you likely have this already) |
| Anthropic API usage | ~$2–5/month for a typical freelancer |
| Telegram | Free |
| Everything else | Free — runs on your machine |

You will never pay an accountant £150/hr to sort receipts again.

---

## Prerequisites

- Node.js 18+ ([nodejs.org](https://nodejs.org))
- Claude Code installed and authenticated
- Anthropic API key ([console.anthropic.com](https://console.anthropic.com))
- Telegram account (for alerts — optional but recommended)
- Mac, Windows (WSL2), or Linux

---

## Tax Support

Every country is supported via a `TaxAdapter` — a structured rules file that tells the system your country's tax brackets, filing deadlines, capital gains method, self-employment rules, and expense categories.

**Pre-built adapters (reviewed, maintained):**

| Region | Countries |
|--------|-----------|
| 🇬🇧 British Isles | United Kingdom, Ireland |
| 🇺🇸 Americas | United States, Canada, Australia, New Zealand, Brazil, Mexico |
| 🇩🇪 Europe | Germany, France, Netherlands, Spain, Italy, Portugal, Switzerland, Belgium, Austria, Sweden, Norway, Denmark, Finland, Poland, Czech Republic, Greece |
| 🌏 Asia-Pacific | Singapore, Japan, India, South Africa |
| 🇦🇪 Special | UAE, Cayman Islands (zero-income-tax configs) |

**Every other country:**

During setup, if your country doesn't have a pre-built adapter, Claude generates one on the fly using its knowledge of your country's tax law. The generated adapter is saved to your machine at `~/.ai-accountant/tax-adapters/[country].json`, clearly marked as AI-generated, and includes a prompt to verify the rates against your tax authority's current guidance.

You can correct any values in the adapter file and the system will use your updated version immediately. Community-submitted adapters are welcomed via PR.

---

## Privacy

Everything runs locally. Your financial documents never leave your machine. Only document *text* (not files) is sent to the Anthropic API for classification — and you'll be asked to confirm this during setup.

---

## Contributing

PRs welcome. The highest-value contributions are:

- **Tax adapters** — submit a reviewed `tax-adapters/[country-code].json` for your country. Template at `src/tax-adapters/_template.json`.
- **Bank adapters** — CSV column mappings for your bank. Currently supported: Monzo, Starling, Barclays, HSBC, Revolut, Chase US, CommBank AU. Add yours in `src/bank-parsers/`.
- **Exchange parsers** — trade history CSV formats for new exchanges.
- **VAT/GST support** — the adapter spec includes a `vatThreshold` field; the VAT calculation engine is the next major feature.

---

## Disclaimer

This tool helps you organise your records and estimates your tax liability. It is not a substitute for professional advice on complex situations. Always verify figures with HMRC / your tax authority before filing.

---

*Built for the [AI Accountant Masterclass](https://youtube.com/@lewisjackson) — free, open source, forever.*
