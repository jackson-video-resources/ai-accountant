# AI Accountant — Your Personal Accountancy Firm, Built by Claude Code

> One prompt. Zero accountancy fees. Full tax automation. Every country.

**You don't need to know how to code.** You paste one thing into a terminal. It builds everything. You never open a code file again.

---

## What Is This?

This is a free, open-source system that runs entirely on your own computer. It:

- **Watches a folder** — drop any receipt, invoice, or bank statement in and it reads, categorises, and files it automatically within about 20 seconds
- **JaxTax web dashboard** *(optional)* — upload receipts and invoices from your browser, review AI extraction before it's filed, search all your transactions visually at `http://localhost:7331`
- **Reads your email** — forward receipts from Amazon, Adobe, AWS, etc. and they're filed without you touching anything
- **Imports exchange data** — drop a CSV from any exchange in the world and it calculates your crypto tax using the correct legal method for your country. Known exchanges are parsed instantly; unknown ones are auto-detected by AI in seconds
- **Calculates your taxes** — income tax, corporation tax, capital gains, NI contributions, salary/dividend splits, payments on account — whatever applies to your situation
- **Finds savings you're missing** — pension contributions, home office, mileage, salary/dividend optimisation, CGT allowance usage, R&D credits — specific to your structure and country
- **Texts you on Telegram** — weekly summaries, deadline alerts at 90/30/7/1 days, and a confirm button when it's time to file
- **Answers your questions** — open a terminal and ask "what do I owe?" or "what can I claim for my laptop?" and get a real answer based on your actual data

Everything runs locally. Your financial data never leaves your computer.

---

## Before You Start — What You Need

You need **4 things**. Each one takes about 2 minutes to set up.

### 1. Node.js (the engine that runs the system)

Go to **[nodejs.org](https://nodejs.org)** and download the LTS version.

- **Mac:** You can also run `brew install node` in Terminal if you have Homebrew
- **Windows:** Download the installer, run it, click Next through everything
- **Linux:** Run `nvm install 20`

After installing, open Terminal (Mac/Linux) or Command Prompt (Windows) and type `node --version`. You should see `v18` or higher.

### 2. Claude Code (the AI that builds and runs everything)

Go to **[claude.ai/code](https://claude.ai/code)** and follow the install instructions.

You'll need a Claude account ($20/month plan). If you already pay for Claude, you likely have access.

### 3. Anthropic API Key (the key the system uses to read your receipts)

Go to **[console.anthropic.com](https://console.anthropic.com)**, sign in with your Claude account, go to **API Keys**, and create a new key. It starts with `sk-ant-`. Copy it — you'll paste it during setup.

**Cost:** About $2–5/month for a typical freelancer. You're charged per document processed, not a flat fee.

### 4. Telegram (how the system texts you)

Download Telegram if you don't have it. Then:

1. Search for **@BotFather** in Telegram and start a chat
2. Send `/newbot` — it'll ask for a name, call it whatever you like (e.g. "My Tax Bot")
3. It gives you a **token** — a long string like `123456789:ABCdefGHI...` — copy it
4. Search for **@userinfobot** in Telegram and send it any message — it replies with your **chat ID** (a number like `987654321`) — copy that too

You'll need both during setup.

> **Telegram is optional.** If you skip it, the system still works — you just won't get automatic alerts.

### 5. Docker *(optional — only needed for the JaxTax web dashboard)*

If you want the browser-based JaxTax interface, you'll need Docker. It's free.

Go to **[docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)** and download Docker Desktop for your platform. Install it and make sure it's running (you'll see the Docker whale icon in your menu bar / taskbar).

> **Docker is optional.** If you skip it, you won't get the JaxTax web dashboard — but the folder-drop system, email listener, and everything else still works.

---

## Setup — 3 Steps

### Step 1 — Open Claude Code

On **Mac/Linux:** Open Terminal, type `claude`, press Enter.

On **Windows:** Open Command Prompt or PowerShell, type `claude`, press Enter.

### Step 2 — Copy the setup prompt

Go to **[docs/SETUP-PROMPT.md](docs/SETUP-PROMPT.md)** on GitHub. Click the **copy** button (top-right of the file). The entire contents are now in your clipboard.

### Step 3 — Paste and answer 8 questions

Paste into Claude Code and press Enter. It will:
1. Check your machine is ready
2. Ask you 8 questions (name, country, structure, crypto, API key, Telegram)
3. Build your entire accountancy system automatically
4. Run a test to confirm everything works
5. Show you a live summary

**Total time: about 12 minutes.** Most of that is package installation running in the background.

---

## What Gets Built on Your Computer

```
~/ai-accountant/
├── inbox/
│   ├── receipts/          ← drop any receipt or invoice PDF/image here
│   ├── invoices-sent/     ← your outgoing invoices to clients
│   ├── bank-statements/   ← CSV or PDF bank exports
│   └── exchanges/
│       ├── binance/       ← Binance trade history CSV
│       ├── coinbase/
│       ├── kraken/
│       └── ...
├── processed/             ← files move here after processing
├── reports/               ← generated tax summaries and PDFs
└── exports/               ← accountant-ready document packs
```

Behind the scenes: a full TypeScript application with a file watcher, OCR engine, AI classifier, CGT calculator, tax engine, Telegram bot, and deadline scheduler — all running silently in the background via PM2.

---

## JaxTax — The Optional Web Dashboard

During setup, you can choose to install **JaxTax** — a browser-based interface for uploading and reviewing documents.

![JaxTax runs at http://localhost:7331]

**What JaxTax adds:**
- Upload receipts, invoices, and photos from your browser — no folder drag-and-drop needed
- See the AI extraction result before it's filed: vendor, date, amount, category
- Review and correct documents visually
- Search, filter, and browse all your transactions
- Works on any device on your local network (phone, tablet, other laptop)

**How it integrates:** Files uploaded through JaxTax land directly in your `inbox/receipts/` folder. Your ai-accountant system picks them up automatically — the two work as one.

**It runs at:** `http://localhost:7331`

**Built on:** [TaxHacker](https://github.com/vas3k/TaxHacker) (MIT licence) — an excellent open-source accounting UI by [@vas3k](https://github.com/vas3k), rebranded and extended with Anthropic/Claude support.

---

## What It Costs

| Thing | Cost | Notes |
|-------|------|-------|
| Claude Code | $20/month | You likely already pay this |
| Anthropic API | ~$2–5/month | Per document processed — typical freelancer |
| Anthropic API | ~$8–15/month | Active trader with lots of receipts |
| Telegram | Free | Free forever |
| Docker Desktop | Free | Only needed for JaxTax web dashboard |
| JaxTax web dashboard | Free | Open source, runs locally |
| Node.js, PM2, everything else | Free | All open source |
| This system | Free | Free forever |

**Compared to:** UK accountant £300–750/year. US CPA $500–2,000/year. Dedicated crypto tax tool (Koinly etc.) £100–200/year.

---

## Works On

| System | Support |
|--------|---------|
| Mac (Intel or Apple Silicon) | ✅ Full support |
| Windows 10/11 | ✅ Full support |
| Linux (Ubuntu, Debian, etc.) | ✅ Full support |
| Linux VPS ($4–6/month, always on) | ✅ Recommended for always-on |

---

## What Structure Are You?

During setup, you choose your business structure. Not sure? It asks you a few questions and recommends the best option.

| Structure | Who it's for | What it tracks |
|-----------|-------------|----------------|
| **Sole Trader / Freelancer** | Self-employed individuals | Personal income tax, NI/SE levy, expenses, CGT |
| **Limited Company / Corporation** | Directors, company owners | Corporation tax + personal salary/dividend tax + Director Loan Account |
| **Partnership / LLP** | Two or more people sharing profits | Per-partner tax split, partnership return |

The system optimises differently for each. A limited company director gets salary/dividend split recommendations. A sole trader gets pension contribution and home office suggestions.

---

## Tax Support — Every Country

Pre-built tax adapters for 30+ countries. For any other country, the system generates your adapter during setup using Claude's knowledge of your local tax law, then waits for you to verify the rates before using them.

### Pre-built (reviewed and maintained)

| 🇬🇧 United Kingdom | 🇺🇸 United States | 🇨🇦 Canada | 🇦🇺 Australia |
|---|---|---|---|
| SA100 + SA103F | 1040 + Schedule C | T1 + T2125 | myTax Individual |
| CGT Section 104 | FIFO / Specific ID | 50% inclusion | 50% discount (12+ months) |
| Class 2/4 NI | SE Tax 15.3% | CPP contributions | Medicare levy |
| Jan 31 deadline | Apr 15 deadline | Jun 15 deadline | Oct 31 deadline |

| 🇩🇪 Germany | 🇫🇷 France | 🇳🇱 Netherlands | 🇮🇪 Ireland |
|---|---|---|---|
| Einkommensteuer | Impôt sur le revenu | Inkomstenbelasting | Form 11 (ROS) |
| Crypto held >1yr = 0% tax | PFU 30% flat | Box 3 wealth tax | CGT 33% |
| ELSTER Jul 31 | DGFiP Jun 30 | Belastingdienst May 1 | Revenue Nov 14 |

| 🇪🇸 Spain | 🇮🇹 Italy | 🇵🇹 Portugal | 🇨🇭 Switzerland |
|---|---|---|---|
| IRPF Modelo 100 | Modello Redditi PF | IRS Modelo 3 | Kantonssteuer |
| AEAT Jun 30 | Nov 30 | AT Jun 30 | Mar 31 |

| 🇸🇪 Sweden | 🇳🇴 Norway | 🇩🇰 Denmark | 🇫🇮 Finland |
|---|---|---|---|
| Skatteverket | Skatteetaten | SKAT | Vero |

| 🇧🇪 Belgium | 🇦🇹 Austria | 🇵🇱 Poland | 🇨🇿 Czech Republic |
|---|---|---|---|
| SPF Finances | Finanzamt AUT | Urząd Skarbowy | Finanční úřad |

| 🇬🇷 Greece | 🇸🇬 Singapore | 🇯🇵 Japan | 🇮🇳 India |
|---|---|---|---|
| AADE | IRAS Form B | NTA | ITR-3 |
| No CGT | No CGT | Crypto income tax | 30% flat crypto |

| 🇿🇦 South Africa | 🇳🇿 New Zealand | 🇧🇷 Brazil | 🇲🇽 Mexico |
|---|---|---|---|
| SARS ITR12 | IR3 | Receita Federal | SAT |

| 🇦🇪 UAE | 🇰🇾 Cayman Islands |
|---|---|
| No income tax | No income tax |

**Any other country:** Claude generates your tax adapter during setup. You verify the rates against your tax authority before they're used. The adapter file is editable — change any value and it takes effect immediately.

---

## After Setup — How to Use It Day-to-Day

**Logging expenses:** Drop the PDF or photo into `~/ai-accountant/inbox/receipts/`. Done. Within 20 seconds you'll get a Telegram message confirming what was logged.

**Logging email receipts:** Set up a Gmail filter to forward receipts to your local email address. The system reads it automatically.

**Crypto trading:** Export your trade history from your exchange as a CSV. Drop it in `~/ai-accountant/inbox/exchanges/[exchange-name]/`. CGT is calculated automatically using the correct method for your country. If your exchange isn't one of the built-in ones, the column layout is detected automatically — you'll get a Telegram message showing the first few parsed transactions so you can confirm it looks right.

**Bank statements:** Export a CSV from your bank and drop it in `~/ai-accountant/inbox/bank-statements/`. 10 UK/US/AU banks are supported out of the box. Any other bank is auto-detected the same way. It matches transactions against your receipts and flags any gaps.

**Asking questions:** Open Terminal, navigate to your ai-accountant folder, type `claude` and ask anything:
- *"what do I owe this year?"*
- *"what can I claim for my home office?"*
- *"show me my CGT summary"*
- *"what are my biggest expenses this month?"*
- *"prepare me for filing"*

**Using your voice:** Install [Wispr Flow](https://wispr.flow) and dictate directly to Claude Code instead of typing.

---

## Frequently Asked Questions

**Do I need to know how to code?**
No. You paste one thing into a terminal window, answer 8 questions, and it builds itself. After that, you interact with it using normal language.

**Is my financial data safe?**
Yes. Everything runs on your computer. The only data that leaves your machine is the *text content* of documents (not the files themselves) sent to Anthropic's API for classification, and you consent to this during setup. Your bank details, account numbers, and tax IDs stay on your computer.

**What if something goes wrong during setup?**
The setup prompt handles errors automatically and tells you exactly what to do if something needs manual attention. Common issues (Node version too old, package install failures, port conflicts) are fixed automatically.

**Can I use this alongside an accountant?**
Yes — many people use it to keep their records organised and then hand a clean export to their accountant. It dramatically reduces accountant time (and therefore cost).

**What if my country isn't in the pre-built list?**
Type your country name during setup. Claude will generate a tax adapter using its knowledge of your country's tax law, tell you exactly which values to verify, and wait for your confirmation before using it.

**Does it work if I'm both employed AND self-employed?**
Yes. You can configure multiple income sources. PAYE employment income is factored into personal allowance calculations.

**What about VAT?**
VAT tracking is the next major feature. The adapter spec includes VAT fields — the calculation engine is in development. For now, VAT amounts are captured on each transaction.

**I already filed last year — can I import old data?**
Yes. During or after setup, drop prior-year bank statements and exchange CSVs into the inbox. The system will assign transactions to the correct tax year based on date.

**Does it actually file my taxes for me?**
It calculates everything and prepares all the data. You still press submit on your tax authority's website (HMRC, IRS, etc.). This takes 20 minutes when all your numbers are already organised. Automated filing is a future feature pending API access from tax authorities.

**What is JaxTax and do I need it?**
JaxTax is an optional web dashboard that lets you upload documents from a browser instead of (or as well as) dropping files in a folder. It's useful if you prefer a visual interface or want to upload from your phone. You don't need it — the folder-drop system works perfectly without it. During setup, just answer "no" when asked and you can always add it later.

**Does JaxTax replace the folder system?**
No — it adds to it. Files uploaded through JaxTax land in the same `inbox/receipts/` folder, and the same tax engine processes them. The two systems work as one.

---

## Supported Banks (automatic CSV parsing)

| Bank | Country | Detection |
|------|---------|-----------|
| Monzo | UK | Instant (hardcoded) |
| Starling | UK | Instant (hardcoded) |
| Barclays | UK | Instant (hardcoded) |
| HSBC | UK | Instant (hardcoded) |
| Revolut | UK / EU | Instant (hardcoded) |
| Lloyds | UK | Instant (hardcoded) |
| NatWest / RBS | UK | Instant (hardcoded) |
| Chase | US | Instant (hardcoded) |
| Bank of America | US | Instant (hardcoded) |
| CommBank | AU | Instant (hardcoded) |
| **Any other bank** | **Any country** | **AI auto-detect** |

**Any bank in the world works.** For banks not in the list above, the system passes your CSV headers and a sample row to Claude, which maps the columns automatically. The detected mapping is cached so repeat imports are instant.

---

## Supported Exchanges (automatic CGT calculation)

| Exchange | Detection |
|----------|-----------|
| Binance, Coinbase, Coinbase Pro, Kraken | Instant (hardcoded) |
| Revolut Crypto, Gemini, ByBit, OKX, KuCoin | Instant (hardcoded) |
| **Any other exchange** | **AI auto-detect** |

**Any exchange in the world works.** Drop any trade history CSV and the system detects which columns are the date, asset, side, quantity, price, and fee — automatically. Works for Gate.io, Huobi, Bitfinex, and any exchange that exports CSVs. The detected mapping is cached so you only pay for one API call per exchange format.

---

## Privacy & Security

- All financial data stored locally in `~/.ai-accountant/data.db` (SQLite, permissions 600 — only your user can read it)
- API keys stored in `.env` with permissions 600
- Only document text (not files) sent to Anthropic API for classification — never amounts, names, or tax IDs
- FX rates fetched from `api.frankfurter.app` — only currency codes and dates sent, nothing financial
- Telegram messages sent only to your configured chat ID — the bot ignores all other senders
- No telemetry. No analytics. No phone-home. This is your data on your machine.

---

## Contributing

PRs welcome. Highest-value contributions:

- **Tax adapters** — reviewed JSON for your country. Template: `docs/tax-adapter-template.json`
- **Bank parser fingerprints** — if your bank's CSV is consistently auto-detected, submit the header fingerprint to `src/bank-parsers/index.ts` so it becomes instant (no API call) for everyone
- **Exchange parser fingerprints** — same deal for exchanges not yet in the hardcoded list
- **Translations** — setup prompt and output messages in other languages

---

## Disclaimer

This tool helps you organise your financial records and calculate your estimated tax liability. It is not a substitute for professional advice on complex tax situations (IR35 status, international income, inheritance tax, etc.). Always verify your final figures with your tax authority before submitting.

---

*Built for the [AI Accountant Masterclass](https://youtube.com/@lewisjackson) — free, open source, forever.*
