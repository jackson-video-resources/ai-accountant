# AI Accountant — One-Shot Setup Prompt

> Copy everything below the line and paste it into Claude Code. That's it.

---

You are going to build a complete, locally-running AI accountancy firm on this machine. By the end of this setup, the user will have a fully automated system that watches folders for receipts and invoices, processes them with OCR and AI classification, calculates their taxes in real time, and sends Telegram alerts at every filing deadline.

Work through each phase in order. Do not skip phases. At the end of each phase, confirm it is complete before moving on. If anything fails, diagnose and fix it before continuing — do not move on with a broken phase.

---

## Phase 0: What We Are Building

Before asking any questions, explain to the user what is about to happen:

Tell them: "I'm going to build your personal AI accountancy firm right here on your machine. Here's what you'll have when we're done:

- A watched folder on your computer — drop any receipt, invoice, or bank statement in and it gets processed automatically
- An email inbox — forward receipt emails here and they're filed instantly
- Exchange CSV support — drop a Binance, Coinbase or Kraken export and your CGT is calculated using the correct pooling method
- A tax engine — income tax, NI, capital gains, deductions, payments on account
- A Telegram bot — weekly summaries, deadline alerts, and a confirm button at filing time
- Natural language queries — just ask 'what do I owe?' and get the answer

This will take about 10–15 minutes. I'll ask you 8 questions, then build everything. Ready? Let's go."

---

## Phase 1: Environment Check

Before asking any questions, silently check:

1. Is Node.js installed? Run `node --version`. Required: v18 or higher.
   - If missing or too old: output clear install instructions for their platform (Mac: `brew install node`, Windows: nodejs.org download, Linux: `nvm install 20`) and wait for them to install it before continuing.

2. Is npm available? Run `npm --version`.

3. Is PM2 installed globally? Run `pm2 --version`.
   - If missing: run `npm install -g pm2` automatically.

4. What platform is this? Run `uname -s` (Mac/Linux) or check `process.platform`. Store as `PLATFORM` (darwin / win32 / linux).

5. Check available disk space. On Mac/Linux: `df -h ~`. Warn (but do not stop) if less than 2GB free.

Report results concisely: "Node v20.11.0 ✓ | npm 10.2.4 ✓ | PM2 installed ✓ | Platform: Mac | Disk: 47GB free ✓"

---

## Phase 2: The Setup Interview

Ask these questions one at a time. Wait for each answer before asking the next. Store all answers — you will use them throughout the build.

**Q1:** "What's your first name? (I'll use this in reports and Telegram messages)"
→ Store as `USER_NAME`

**Q2:** "Which country are you in? This determines which tax system I'll set up.
  1. United Kingdom (Self Assessment, CGT Section 104, HMRC rules)
  2. United States (Schedule C, 1040, IRS rules)
  3. Other (I'll set up a universal tracker — you'll need to map your own tax rules)"
→ Store as `COUNTRY` (uk / us / other)

**Q3 (UK only):** "What's your UTR — Unique Taxpayer Reference? It's 10 digits on any letter from HMRC. Press Enter to skip for now."
→ Store as `UTR` (optional)

**Q3 (US only):** "What's your SSN or EIN for tax filing? Press Enter to skip for now."
→ Store as `TAX_ID` (optional)

**Q4:** "Do you have crypto or investment trading activity? (yes / no)"
→ Store as `HAS_CRYPTO`

**Q5 (if HAS_CRYPTO = yes):** "Which exchanges do you use? Type all that apply separated by commas.
  Options: binance, coinbase, kraken, revolut, gemini, bybit, okx, other"
→ Store as `EXCHANGES` (array)

**Q6:** "What's your Anthropic API key? I'll use this for receipt classification and tax queries.
  Get one at: console.anthropic.com → API Keys
  (It will be stored securely in .env — never committed to git)"
→ Store as `ANTHROPIC_API_KEY`. Validate it starts with `sk-ant-`.

**Q7:** "Do you want Telegram alerts for filing deadlines, weekly summaries, and tax calculations? (yes / no — you can add this later)"
→ Store as `WANTS_TELEGRAM`

**Q7a (if WANTS_TELEGRAM = yes):** "Paste your Telegram bot token. Create one by messaging @BotFather on Telegram → /newbot → follow the steps. It looks like: 123456789:ABCdef..."
→ Store as `TELEGRAM_TOKEN`

**Q7b (if WANTS_TELEGRAM = yes):** "Now paste your Telegram chat ID. Get it by messaging @userinfobot on Telegram — it replies with your ID. It's a number like: 987654321"
→ Store as `TELEGRAM_CHAT_ID`

**Q8:** "Where should I create your accountancy system? Press Enter for the default, or type a custom path.
  Default: ~/ai-accountant"
→ Store as `INSTALL_DIR`. Expand `~` to absolute path.

Confirm before building: "Perfect. Here's what I'm about to set up:
- Name: [USER_NAME]
- Tax system: [COUNTRY]
- Crypto tracking: [yes/no]
- Telegram: [connected/skipped]
- Install location: [INSTALL_DIR]

Building now..."

---

## Phase 3: Create Folder Structure

Create the following directory structure. Use `mkdir -p` for each path.

```
[INSTALL_DIR]/
├── inbox/
│   ├── receipts/
│   ├── invoices-sent/
│   ├── bank-statements/
│   └── exchanges/
│       ├── binance/
│       ├── coinbase/
│       ├── kraken/
│       ├── revolut/
│       ├── gemini/
│       └── other/
├── processed/
├── reports/
└── exports/
```

Also create the hidden data directory:
```
~/.ai-accountant/
├── documents/
└── logs/
```

After creating: "Folder structure created ✓"

---

## Phase 4: Generate package.json and Install Dependencies

Create `[INSTALL_DIR]/package.json`:

```json
{
  "name": "ai-accountant",
  "version": "1.0.0",
  "description": "Your personal AI accountancy firm",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "reconcile": "tsx src/scripts/reconcile.ts",
    "report": "tsx src/scripts/generate-report.ts",
    "health": "tsx src/scripts/health-check.ts",
    "migrate": "tsx src/scripts/migrate.ts"
  },
  "dependencies": {
    "@anthropic-ai/sdk": "^0.36.3",
    "better-sqlite3": "^9.4.3",
    "chokidar": "^3.6.0",
    "node-cron": "^3.0.3",
    "node-telegram-bot-api": "^0.66.0",
    "papaparse": "^5.4.1",
    "pdf-parse": "^1.1.1",
    "pdfkit": "^0.15.0",
    "pino": "^9.1.0",
    "smtp-server": "^3.13.4",
    "tesseract.js": "^5.0.5",
    "uuid": "^10.0.0"
  },
  "devDependencies": {
    "@types/better-sqlite3": "^7.6.8",
    "@types/node": "^20.11.5",
    "@types/node-telegram-bot-api": "^0.64.3",
    "@types/papaparse": "^5.3.14",
    "@types/pdfkit": "^0.13.4",
    "@types/smtp-server": "^3.5.10",
    "@types/uuid": "^10.0.0",
    "tsx": "^4.19.1",
    "typescript": "^5.3.3"
  }
}
```

Create `[INSTALL_DIR]/tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "moduleResolution": "node"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Run: `cd [INSTALL_DIR] && npm install`

If any package fails, try installing it separately. If `tesseract.js` fails, note it and continue — the system will use PDF text extraction only as a fallback. Report: "Dependencies installed ✓ (X packages)"

---

## Phase 5: Create .env and config

Create `[INSTALL_DIR]/.env`:

```
ANTHROPIC_API_KEY=[ANTHROPIC_API_KEY]
TELEGRAM_BOT_TOKEN=[TELEGRAM_TOKEN or empty]
TELEGRAM_CHAT_ID=[TELEGRAM_CHAT_ID or empty]
INSTALL_DIR=[INSTALL_DIR]
DATA_DIR=[HOME]/.ai-accountant
NODE_ENV=production
```

Create `~/.ai-accountant/config.json` (this is outside the install dir and never committed to git):

```json
{
  "userName": "[USER_NAME]",
  "country": "[COUNTRY]",
  "taxId": "[UTR or TAX_ID or null]",
  "hasCrypto": [true/false],
  "exchanges": [EXCHANGES array],
  "installDir": "[INSTALL_DIR]",
  "dataDir": "[HOME]/.ai-accountant",
  "createdAt": "[ISO timestamp]",
  "version": "1.0.0"
}
```

Create `[INSTALL_DIR]/.gitignore`:

```
.env
node_modules/
dist/
*.db
inbox/
processed/
reports/
exports/
~/.ai-accountant/
```

Set permissions on the data directory (Mac/Linux only): `chmod 700 ~/.ai-accountant`

---

## Phase 6: Build the Core System

Now generate the full TypeScript source. Create each file below completely. Do not abbreviate or add placeholder comments — write the full working implementation.

### src/config.ts

Load and validate all configuration. Export a typed `Config` object. Read from `.env` and `~/.ai-accountant/config.json`. Include the full UK and US tax constants for 2023-24, 2024-25, and 2025-26 tax years.

UK tax constants must include: personalAllowance (12570), personalAllowanceTaperThreshold (100000), basicRateThreshold (50270), higherRateThreshold (125140), basicRate (0.20), higherRate (0.40), additionalRate (0.45), cgtAnnualExemptAmount (3000), cgtBasicRate (0.10), cgtHigherRate (0.20), class2NIWeeklyRate (3.45), class2NISmallProfitsThreshold (12570), class4NILowerProfitsLimit (12570), class4NIUpperProfitsLimit (50270), class4NILowerRate (0.09), class4NIUpperRate (0.02), tradingAllowance (1000), saFilingDeadline ("YYYY-01-31"), poaDeadline1 ("YYYY-01-31"), poaDeadline2 ("YYYY-07-31").

US constants: standardDeduction (14600 single / 29200 married 2024), selfEmploymentTaxRate (0.153), selfEmploymentDeductionRate (0.5), brackets for single and married filing (10/12/22/24/32/35/37%), ltcgBrackets (0/15/20%).

Include a helper `getCurrentTaxYear(country: string): string` that returns the current open tax year label based on today's date.

### src/database.ts

Set up better-sqlite3. Database path: `~/.ai-accountant/data.db`. Run all migrations on startup. Export a `db` singleton.

Create these tables:

```sql
CREATE TABLE IF NOT EXISTS tax_years (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  year_label TEXT NOT NULL UNIQUE,
  country TEXT NOT NULL,
  start_date TEXT NOT NULL,
  end_date TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'open',
  total_income_pence INTEGER DEFAULT 0,
  total_expenses_pence INTEGER DEFAULT 0,
  taxable_profit_pence INTEGER DEFAULT 0,
  income_tax_due_pence INTEGER DEFAULT 0,
  ni_due_pence INTEGER DEFAULT 0,
  cgt_due_pence INTEGER DEFAULT 0,
  total_due_pence INTEGER DEFAULT 0,
  last_calculated_at TEXT,
  filed_at TEXT,
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS transactions (
  id TEXT PRIMARY KEY,
  tax_year_id INTEGER REFERENCES tax_years(id),
  type TEXT NOT NULL CHECK(type IN ('income','expense','transfer','unknown')),
  amount_pence INTEGER NOT NULL,
  original_amount_pence INTEGER,
  original_currency TEXT,
  exchange_rate REAL,
  date TEXT NOT NULL,
  vendor TEXT,
  description TEXT,
  category TEXT,
  subcategory TEXT,
  deductibility_percentage REAL DEFAULT 100,
  deductible_amount_pence INTEGER,
  vat_amount_pence INTEGER DEFAULT 0,
  source TEXT NOT NULL,
  document_id TEXT,
  bank_reference TEXT,
  notes TEXT,
  confidence_score REAL,
  manually_reviewed INTEGER DEFAULT 0,
  is_void INTEGER DEFAULT 0,
  created_at TEXT NOT NULL DEFAULT (datetime('now')),
  updated_at TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS documents (
  id TEXT PRIMARY KEY,
  original_filename TEXT NOT NULL,
  stored_path TEXT NOT NULL,
  file_hash TEXT NOT NULL UNIQUE,
  mime_type TEXT,
  source_type TEXT NOT NULL,
  ocr_text TEXT,
  claude_classification TEXT,
  processing_status TEXT NOT NULL DEFAULT 'pending'
    CHECK(processing_status IN ('pending','processing','done','error','needs_review')),
  error_message TEXT,
  processed_at TEXT,
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS cgt_acquisitions (
  id TEXT PRIMARY KEY,
  asset TEXT NOT NULL,
  exchange TEXT,
  date TEXT NOT NULL,
  quantity_satoshi INTEGER NOT NULL,
  price_per_unit_pence INTEGER NOT NULL,
  total_cost_pence INTEGER NOT NULL,
  fee_pence INTEGER DEFAULT 0,
  pool_quantity_satoshi INTEGER NOT NULL,
  pool_cost_pence INTEGER NOT NULL,
  tax_year_id INTEGER REFERENCES tax_years(id),
  document_id TEXT REFERENCES documents(id),
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS cgt_disposals (
  id TEXT PRIMARY KEY,
  asset TEXT NOT NULL,
  exchange TEXT,
  date TEXT NOT NULL,
  quantity_satoshi INTEGER NOT NULL,
  proceeds_pence INTEGER NOT NULL,
  fee_pence INTEGER DEFAULT 0,
  cost_basis_pence INTEGER NOT NULL,
  gain_loss_pence INTEGER NOT NULL,
  matching_rule TEXT NOT NULL CHECK(matching_rule IN ('same_day','bed_and_breakfast','section104','fifo')),
  tax_year_id INTEGER REFERENCES tax_years(id),
  document_id TEXT REFERENCES documents(id),
  notes TEXT,
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS cgt_pools (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  asset TEXT NOT NULL,
  snapshot_date TEXT NOT NULL,
  pool_quantity_satoshi INTEGER NOT NULL,
  pool_cost_pence INTEGER NOT NULL,
  event_type TEXT NOT NULL CHECK(event_type IN ('acquisition','disposal')),
  event_id TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS fx_rates (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  date TEXT NOT NULL,
  from_currency TEXT NOT NULL,
  to_currency TEXT NOT NULL DEFAULT 'GBP',
  rate REAL NOT NULL,
  source TEXT,
  UNIQUE(date, from_currency, to_currency)
);

CREATE TABLE IF NOT EXISTS audit_log (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  timestamp TEXT NOT NULL DEFAULT (datetime('now')),
  action TEXT NOT NULL,
  entity_type TEXT,
  entity_id TEXT,
  details TEXT,
  triggered_by TEXT NOT NULL DEFAULT 'system'
);
```

After creating tables, seed the current and previous tax year records based on the user's country. For UK: if today is before April 6, current year is (current calendar year - 1 to current calendar year), e.g. "2024-25". For US: current calendar year.

### src/classifier.ts

Build the Claude classifier. This is the brain of the system.

Use `@anthropic-ai/sdk`. Model: `claude-haiku-4-5-20251001` (use Haiku — it's fast and cheap for classification).

Function `classifyDocument(text: string, filename: string): Promise<ClassificationResult>`:

The prompt to Claude should be:

```
You are a UK/US tax accountancy classification engine. Classify this financial document.

Document filename: [filename]
Document text: [first 2000 chars of text]

Respond with ONLY valid JSON matching this exact schema:
{
  "type": "income" | "expense" | "transfer" | "unknown",
  "amount": number (in major currency unit, e.g. 12.99),
  "currency": "GBP" | "USD" | "EUR" | string,
  "date": "YYYY-MM-DD" | null,
  "vendor": string | null,
  "description": string,
  "category": "office_supplies" | "software" | "travel" | "equipment" | "professional_services" | "marketing" | "phone_internet" | "training" | "bank_charges" | "home_office" | "meals_entertainment" | "utilities" | "rent" | "insurance" | "wages" | "crypto_fee" | "other_expense" | "client_payment" | "consulting_income" | "product_sale" | "investment_income" | "other_income",
  "deductibility_percentage": 0 | 25 | 50 | 75 | 100,
  "vat_amount": number | null,
  "confidence": 0.0-1.0,
  "notes": string | null
}
```

If Claude returns malformed JSON, retry once with a simpler prompt. If it fails again, return a result with type "unknown" and confidence 0.

### src/watcher.ts

File system watcher using chokidar. Watch `[INSTALL_DIR]/inbox/**/*`.

On new file:
1. Check file extension — ignore .DS_Store, Thumbs.db, .tmp files
2. Compute SHA-256 hash of file content
3. Check if hash already exists in `documents` table — if yes, skip (already processed)
4. Copy file to `~/.ai-accountant/documents/[YYYY]/[MM]/[uuid].[ext]`
5. Insert record in `documents` table with status 'pending'
6. Emit 'new_document' event to the document processor
7. Move original from inbox to `processed/[YYYY]/[MM]/[filename]`

Cross-platform path handling: always use `path.join()`, never string concatenation with `/`.

### src/processor.ts

Document processor. Listens for 'new_document' events from the watcher.

For each document:
1. Update status to 'processing'
2. Determine document type from path and extension:
   - `inbox/exchanges/[exchange]/` → call exchange CSV parser
   - `inbox/bank-statements/` → call bank statement parser
   - `inbox/receipts/` or `inbox/invoices-sent/` → call OCR + classifier pipeline
3. **OCR pipeline** (for receipts/invoices):
   - If PDF: use `pdf-parse` to extract text layer first
   - If text layer is empty or < 50 chars: fall back to `tesseract.js` OCR
   - If image file: use `tesseract.js` directly
4. Send extracted text to `classifier.ts`
5. Convert classified amount to pence (multiply by 100, round to integer)
6. Look up FX rate if currency is not GBP/USD (based on user's country)
7. Determine correct tax year for the transaction date
8. Insert transaction record into database
9. Update document status to 'done'
10. Send Telegram notification (if configured)

Telegram message format for a processed receipt:
```
💰 Expense logged
[vendor] — £[amount]
[category] | [deductibility]% deductible
Tax saving: £[amount × deductibility% × basic_rate]
[tax_year] YTD expenses: £[total]
```

For income:
```
📈 Income logged
£[amount] from [vendor]
[description]
[tax_year] YTD income: £[total]
```

On error: update status to 'error', store error message, send Telegram alert if critical.

### src/exchange-parsers/index.ts

Auto-detect exchange type from CSV headers. Load parsers for each exchange.

Header fingerprints:
```typescript
const EXCHANGE_HEADERS: Record<string, string[]> = {
  binance: ['Date(UTC)', 'Pair', 'Side', 'Price', 'Executed', 'Amount', 'Fee'],
  coinbase: ['Timestamp', 'Transaction Type', 'Asset', 'Quantity Transacted', 'Price Currency'],
  kraken: ['txid', 'ordertxid', 'pair', 'time', 'type', 'ordertype', 'price', 'cost', 'fee', 'vol'],
  revolut: ['Type', 'Product', 'Started Date', 'Completed Date', 'Description', 'Amount', 'Currency'],
  gemini: ['Date', 'Time (UTC)', 'Type', 'Symbol', 'Specification', 'Liquidity Indicator', 'Trading Fee Currency', 'Trading Fee Amount', 'USD Amount'],
};
```

Each parser normalises to a common `Trade` interface:
```typescript
interface Trade {
  date: string;           // ISO 8601
  asset: string;          // e.g. "BTC"
  quoteCurrency: string;  // e.g. "GBP" or "USDT"
  side: 'buy' | 'sell';
  quantity: number;       // in base asset units
  price: number;          // in quote currency
  fee: number;            // in quote currency
  feeAsset: string;
  exchangeOrderId: string;
}
```

Use `papaparse` with `step` callback (streaming) for large files.

### src/cgt-calculator.ts

CGT calculation using UK HMRC Section 104 pooling method.

Process trades in chronological order. For each disposal:

**Step 1 — Same-day rule:** Check if there are any acquisitions of the same asset on the same calendar date. Match against those first. Cost basis = same-day acquisition cost.

**Step 2 — 30-day rule (bed and breakfast):** Check if there are any acquisitions of the same asset in the 30 days FOLLOWING the disposal date. Match against the earliest such acquisition. Cost basis = that future acquisition cost.

**Step 3 — Section 104 pool:** All remaining acquisitions form the pool. Cost basis = (disposal_quantity / pool_quantity) × pool_cost. Reduce pool by disposed quantity and proportional cost.

**Gain/Loss calculation:**
```
gain_loss = proceeds - cost_basis - allowable_fees
```

Store each disposal in `cgt_disposals`. Update `cgt_pools` after each event.

After processing all trades for a tax year:
- Sum all gains and losses
- Apply annual exempt amount
- Calculate CGT due (determine if basic or higher rate taxpayer by comparing to income tax band usage)

### src/tax-engine.ts

UK Self Assessment calculation engine.

```typescript
interface TaxCalculation {
  taxYear: string;
  grossIncome: number;        // pence
  allowableExpenses: number;  // pence
  tradingProfit: number;      // pence
  personalAllowance: number;  // pence (may be tapered)
  taxableIncome: number;      // pence
  basicRateTax: number;       // pence
  higherRateTax: number;      // pence
  additionalRateTax: number;  // pence
  totalIncomeTax: number;     // pence
  class2NI: number;           // pence
  class4NI: number;           // pence
  totalNI: number;            // pence
  cgtGain: number;            // pence
  cgtExemptUsed: number;      // pence
  cgtTaxable: number;         // pence
  cgtDue: number;             // pence
  totalTaxDue: number;        // pence
  paymentOnAccount1: number;  // pence
  paymentOnAccount2: number;  // pence
  balancingPayment: number;   // pence
  effectiveTaxRate: number;   // percentage
  marginRate: number;         // percentage (marginal rate at current income)
}
```

The calculation must:
1. Taper personal allowance correctly if adjusted net income > £100,000
2. Calculate POA only if total tax > £1,000 AND less than 80% was collected at source
3. Store results back to `tax_years` table
4. Log to audit_log with full breakdown as JSON

Also build a US equivalent: `calculateUSTax()` using Schedule C (self-employment income minus expenses = net profit → SE tax at 15.3% → half SE tax deducted → apply standard deduction → apply federal brackets → state tax placeholder).

### src/optimizations.ts

Analyze current year data and return actionable suggestions.

Check for:
1. **Unused CGT allowance** — if remaining CGT annual exempt > 0, suggest realising gains before tax year end
2. **Personal allowance taper** — if profit is between £100k and £125,140, suggest pension contributions to restore allowance (calculate exact saving)
3. **Pension contributions** — if no pension contributions recorded, calculate how much basic rate relief they're missing
4. **Home office** — if no home_office expenses and there are software/equipment expenses (suggesting WFH activity), prompt them to claim
5. **Mileage vs actual** — if travel expenses recorded, check if mileage rate would be more beneficial
6. **Missing receipts** — count transactions over £25 without documents
7. **ISA headroom** — if in UK and near tax year end, remind about £20,000 ISA allowance (for investment income reduction)
8. **Payment on account warning** — if this will be their first POA year, warn them prominently

Return array of `Optimization` objects sorted by estimated saving descending.

### src/telegram-bot.ts

If `TELEGRAM_BOT_TOKEN` is set, start the bot. Otherwise, export no-op functions.

Bot commands:
- `/status` — current tax year overview (income, expenses, estimated tax)
- `/deadlines` — next 3 deadlines with days remaining
- `/cgt` — CGT summary for current year
- `/gaps` — transactions needing receipts (>£25 / >$50 no document)
- `/report` — trigger PDF report generation and send as document
- `/optimise` — run optimizations engine, send top suggestions
- `/help` — show all commands

Also support incoming forwarded emails: if a message contains a document attachment (via the Telegram `document` message type), save it to `inbox/receipts/` and it will be picked up by the file watcher.

All incoming messages must validate that `msg.chat.id === parseInt(TELEGRAM_CHAT_ID)`. Ignore all other senders silently.

### src/scheduler.ts

Set up cron jobs using `node-cron`.

```typescript
const jobs = [
  { name: 'nightly_reconcile', cron: '0 2 * * *' },           // 2am every night
  { name: 'weekly_summary', cron: '0 9 * * 1' },              // 9am Monday
  // UK-specific deadline alerts
  { name: 'uk_90day_warning', cron: '0 9 6 11 *' },           // Nov 6 (90d before Jan 31)
  { name: 'uk_30day_warning', cron: '0 9 1 1 *' },            // Jan 1
  { name: 'uk_7day_warning', cron: '0 9 24 1 *' },            // Jan 24
  { name: 'uk_final_warning', cron: '0 9 30 1 *' },           // Jan 30
  { name: 'uk_poa_july', cron: '0 9 24 7 *' },                // Jul 24 (7d before Jul 31)
  // US-specific
  { name: 'us_q1_estimate', cron: '0 9 8 4 *' },              // Apr 8 (7d before Apr 15)
  { name: 'us_annual_filing', cron: '0 9 8 4 *' },            // Apr 8
];
```

Nightly reconciliation:
1. Find all files in `inbox/` that haven't been processed (failsafe for files the watcher missed)
2. Find transactions with `processing_status = 'pending'`
3. Run them through the processor
4. Log results to audit_log

Weekly summary Telegram message:
```
📊 Weekly Tax Summary — [week ending date]

Income this week: £[amount]
Expenses this week: £[amount]

YTD [tax year]:
Income: £[total]
Expenses: £[total]
Estimated tax: £[estimate]

Next deadline: [deadline] ([X] days)
```

### src/email-listener.ts

Start a local SMTP server on `127.0.0.1` port `2525` using `smtp-server`.

On receiving an email:
1. Extract the sender, subject, and all attachments
2. Save attachments to `inbox/receipts/[filename]`
3. If no attachment but email body contains financial data (keywords: total, amount, invoice, receipt, £, $), save the email body as a .txt file in inbox/receipts/
4. The file watcher will pick it up from there

Bind to `127.0.0.1` only — never `0.0.0.0`. Log the local SMTP address at startup.

### src/index.ts

Main entry point. Start all services in order:
1. Initialize database (run migrations, seed tax years)
2. Start file watcher
3. Start email listener
4. Start Telegram bot
5. Start scheduler
6. Print startup summary

Startup summary format:
```
╔══════════════════════════════════════════════════════╗
║         AI ACCOUNTANT IS LIVE                        ║
╠══════════════════════════════════════════════════════╣
║  [USER_NAME]'s personal accountancy firm             ║
║  Tax system: [COUNTRY] | [current tax year]          ║
║  Drop receipts → [INSTALL_DIR]/inbox/receipts/       ║
║  Forward emails → receipts@127.0.0.1:2525            ║
║  Next filing deadline: [deadline] ([X] days)         ║
║  Telegram: [Connected ✓ / Not configured]            ║
╚══════════════════════════════════════════════════════╝

Ask me anything: "what do I owe?", "what can I claim for X?", "cgt summary"
```

### src/scripts/health-check.ts

Run a quick health check:
1. Can we connect to the database?
2. Is the file watcher running?
3. Is the Telegram bot connected?
4. Is the email listener running?
5. Any documents stuck in 'processing' status for > 1 hour?
6. Any critical errors in the last 24 hours?

Print results and exit 0 (healthy) or 1 (issues found).

### src/scripts/generate-report.ts

Generate a PDF tax report for the current tax year using `pdfkit`.

Report sections:
1. Cover page: name, tax year, generated date
2. Income summary: total, by source
3. Expense summary: total, by category, deductible total
4. CGT summary: disposals table, gain/loss per asset, total
5. Tax calculation breakdown: income tax by band, NI breakdown, CGT due, total
6. Payments on account: amounts and due dates
7. Optimization suggestions
8. Document audit trail: count of receipts processed, any gaps

Save to `[INSTALL_DIR]/reports/[tax_year]/tax-summary-[date].pdf` and send via Telegram if configured.

---

## Phase 7: Create PM2 Configuration

Create `[INSTALL_DIR]/ecosystem.config.cjs`:

```javascript
module.exports = {
  apps: [{
    name: 'ai-accountant',
    script: 'src/index.ts',
    interpreter: 'npx',
    interpreter_args: 'tsx',
    cwd: '[INSTALL_DIR]',
    env: {
      NODE_ENV: 'production'
    },
    watch: false,
    autorestart: true,
    max_restarts: 10,
    restart_delay: 5000,
    error_file: '[HOME]/.ai-accountant/logs/error.log',
    out_file: '[HOME]/.ai-accountant/logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss'
  }]
}
```

---

## Phase 8: Copy CLAUDE.md

Copy the CLAUDE.md from this repository into `[INSTALL_DIR]/CLAUDE.md`. Personalise it by replacing placeholders:
- `[USER_NAME]` with the actual name
- `[COUNTRY]` with uk / us / other
- `[INSTALL_DIR]` with the actual path
- Comment out the UK tax section if country is US, and vice versa

This is critical — it makes every future Claude Code session in this directory behave as the user's personal AI accountant.

---

## Phase 9: Start the System and Verify

1. Start the system: `cd [INSTALL_DIR] && pm2 start ecosystem.config.cjs`

2. Wait 5 seconds for startup.

3. Check it's running: `pm2 list` — status should be 'online'

4. Run health check: `npm run health`

5. **Drop a test receipt:** Create a simple text file in `inbox/receipts/` called `test-receipt.txt` with content:
   ```
   AMAZON.CO.UK
   Order #202-1234567-8901234
   Date: [today's date]

   1x USB-C Cable £12.99
   Total: £12.99 GBP
   ```
   Wait 10 seconds. Check that:
   - A transaction record exists in the database
   - A Telegram message was sent (if configured)

6. If Telegram is configured: send a test message with the startup summary.

7. Set PM2 to start on system reboot:
   - Mac/Linux: `pm2 startup && pm2 save`
   - Windows: provide instructions for Task Scheduler instead (PM2 startup doesn't work reliably on Windows)

---

## Phase 10: Final Summary

Print the live summary box (from src/index.ts).

Then print:

```
🎉 Setup complete!

Your AI accountancy firm is running. Here's how to use it:

DROP FILES:
  Receipts/invoices → [INSTALL_DIR]/inbox/receipts/
  Bank statements  → [INSTALL_DIR]/inbox/bank-statements/
  Exchange CSVs    → [INSTALL_DIR]/inbox/exchanges/[exchange]/

EMAIL FORWARDING:
  Set up email forwarding or a filter in Gmail/Outlook to forward
  receipts to: receipts@127.0.0.1 on port 2525
  (On Mac you can use a rule in Mail.app)

TALK TO YOUR ACCOUNTANT:
  Open Claude Code in [INSTALL_DIR] and ask anything:
  - "what do I owe this year?"
  - "what can I claim for my home office?"
  - "cgt summary"
  - "prepare me for filing"
  - "what's my most expensive category this month?"

USE YOUR VOICE:
  Install Wispr Flow and dictate directly to Claude Code —
  just speak your questions naturally.

USEFUL COMMANDS:
  pm2 status              — check the system is running
  pm2 logs ai-accountant  — view live logs
  npm run health          — run health check
  npm run report          — generate PDF tax report
  pm2 restart ai-accountant — restart after changes

Next filing deadline: [deadline] — [X] days away.
```

---

## Error Handling & Fallbacks

If any phase fails, do not silently continue. Report clearly:

```
❌ Phase [X] failed: [error message]

Attempting fix: [what you're trying]
```

Try to fix automatically where possible. If you cannot fix it:

```
⚠️  Manual action needed:
[clear instructions for what the user needs to do]

Once done, tell me and I'll continue from Phase [X].
```

Common issues and fixes:
- `better-sqlite3` native binding failure on Windows → try `npm install better-sqlite3 --build-from-source`
- `tesseract.js` WASM load failure → skip OCR, note that image receipts won't be read (PDF text extraction still works)
- PM2 not found → `npm install -g pm2`
- Telegram bot 401 error → token is invalid, prompt user to check @BotFather
- Port 2525 already in use → try port 2526, update .env

---

*This prompt was generated for the [AI Accountant Masterclass](https://youtube.com/@lewisjackson). Free, open source, forever.*
