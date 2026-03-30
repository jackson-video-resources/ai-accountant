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

**Q2:** "Which country are you in? Just type the country name — I support every country in the world.

  For 30+ countries I have pre-built, reviewed tax adapters. For anywhere else, I'll generate your country's tax rules on the fly using my knowledge of local tax law and save them to your machine for you to verify.

  Examples: United Kingdom, Germany, Australia, Canada, India, Singapore, Brazil, UAE..."
→ Store as `COUNTRY_INPUT` (free text). Normalise to ISO 3166-1 country name. Store ISO 3166-1 alpha-2 code as `COUNTRY_CODE` (e.g. "GB", "US", "DE", "AU").

**Q3:** "What's your tax ID number? (e.g. UK UTR, US SSN/EIN, Australian TFN, German Steueridentifikationsnummer). Press Enter to skip for now."
→ Store as `TAX_ID` (optional, label varies by country)

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
- Country: [COUNTRY_INPUT] ([COUNTRY_CODE])
- Crypto tracking: [yes/no]
- Telegram: [connected/skipped]
- Install location: [INSTALL_DIR]

Building now..."

---

## Phase 2b: Tax Adapter Resolution

Before building anything else, determine which tax adapter to use.

**Check if a pre-built adapter exists** for `COUNTRY_CODE` by looking for it in the list below. Pre-built adapters are embedded directly in this prompt.

### Pre-built Tax Adapters

Each adapter follows this TypeScript interface:

```typescript
interface TaxAdapter {
  countryCode: string;          // ISO 3166-1 alpha-2
  countryName: string;
  currency: string;             // ISO 4217
  currencySymbol: string;
  taxAuthority: string;
  taxAuthorityUrl: string;
  taxYear: {
    type: 'calendar' | 'fiscal';
    fiscalStartMonth?: number;  // 1-12, only for fiscal
    fiscalStartDay?: number;
    labelFormat: 'single' | 'split';  // "2024" vs "2024-25"
  };
  incomeTax: {
    personalAllowance: number;
    personalAllowanceTaperThreshold?: number;
    personalAllowanceTaperRate?: number;
    brackets: Array<{ min: number; max: number | null; rate: number; label?: string }>;
  };
  selfEmploymentLevy?: {
    name: string;
    description: string;
    components: Array<{
      name: string;
      rate: number;
      onIncomeAbove?: number;
      onIncomeUpTo?: number;
      flatAnnualAmount?: number;
      flatAmountIfAboveThreshold?: number;
    }>;
  };
  capitalGains?: {
    annualExemption: number;
    rates: Array<{ condition: string; rate: number }>;
    cryptoMethod: 'pool_section104' | 'fifo' | 'lifo' | 'specific_id' | 'average_cost';
    cryptoNotes?: string;
    holdingPeriodForReducedRate?: number;  // days
  };
  vat?: {
    standardRate: number;
    registrationThreshold: number;
    name: string;  // "VAT", "GST", "TVA", "MwSt" etc.
  };
  filingDeadlines: Array<{
    name: string;
    description: string;
    month: number;
    day: number;
    alertDaysBefore: number[];
  }>;
  mainForms: string[];
  tradingAllowance?: number;
  pensionContributionRelief?: boolean;
  notes?: string;
  generatedByAI: boolean;
  lastVerified: string;  // ISO date
}
```

Write the adapter file to `~/.ai-accountant/tax-adapters/[COUNTRY_CODE].json`.

---

#### 🇬🇧 GB — United Kingdom

```json
{
  "countryCode": "GB",
  "countryName": "United Kingdom",
  "currency": "GBP",
  "currencySymbol": "£",
  "taxAuthority": "HMRC",
  "taxAuthorityUrl": "https://www.gov.uk/self-assessment-tax-returns",
  "taxYear": { "type": "fiscal", "fiscalStartMonth": 4, "fiscalStartDay": 6, "labelFormat": "split" },
  "incomeTax": {
    "personalAllowance": 12570,
    "personalAllowanceTaperThreshold": 100000,
    "personalAllowanceTaperRate": 0.5,
    "brackets": [
      { "min": 0, "max": 12570, "rate": 0, "label": "Personal Allowance" },
      { "min": 12570, "max": 50270, "rate": 0.20, "label": "Basic Rate" },
      { "min": 50270, "max": 125140, "rate": 0.40, "label": "Higher Rate" },
      { "min": 125140, "max": null, "rate": 0.45, "label": "Additional Rate" }
    ]
  },
  "selfEmploymentLevy": {
    "name": "National Insurance",
    "description": "Class 2 and Class 4 NI contributions for self-employed",
    "components": [
      { "name": "Class 2", "flatAnnualAmount": 179, "flatAmountIfAboveThreshold": 12570 },
      { "name": "Class 4 (lower)", "rate": 0.09, "onIncomeAbove": 12570, "onIncomeUpTo": 50270 },
      { "name": "Class 4 (upper)", "rate": 0.02, "onIncomeAbove": 50270 }
    ]
  },
  "capitalGains": {
    "annualExemption": 3000,
    "rates": [
      { "condition": "basic rate taxpayer", "rate": 0.10 },
      { "condition": "higher/additional rate taxpayer", "rate": 0.20 },
      { "condition": "residential property (basic)", "rate": 0.18 },
      { "condition": "residential property (higher)", "rate": 0.24 }
    ],
    "cryptoMethod": "pool_section104",
    "cryptoNotes": "HMRC requires Section 104 pooling. Same-day rule and 30-day bed-and-breakfast rule take precedence. Every crypto-to-crypto swap is a disposal event.",
    "holdingPeriodForReducedRate": null
  },
  "vat": { "standardRate": 0.20, "registrationThreshold": 90000, "name": "VAT" },
  "filingDeadlines": [
    { "name": "Online Self Assessment", "description": "File SA return and pay tax due", "month": 1, "day": 31, "alertDaysBefore": [90, 60, 30, 14, 7, 1] },
    { "name": "Payment on Account 2", "description": "Second payment on account for prior year", "month": 7, "day": 31, "alertDaysBefore": [30, 14, 7] }
  ],
  "mainForms": ["SA100", "SA103F", "SA108"],
  "tradingAllowance": 1000,
  "pensionContributionRelief": true,
  "generatedByAI": false,
  "lastVerified": "2025-03-01"
}
```

---

#### 🇺🇸 US — United States

```json
{
  "countryCode": "US",
  "countryName": "United States",
  "currency": "USD",
  "currencySymbol": "$",
  "taxAuthority": "IRS",
  "taxAuthorityUrl": "https://www.irs.gov/businesses/small-businesses-self-employed",
  "taxYear": { "type": "calendar", "labelFormat": "single" },
  "incomeTax": {
    "personalAllowance": 14600,
    "brackets": [
      { "min": 0, "max": 11600, "rate": 0.10 },
      { "min": 11600, "max": 47150, "rate": 0.12 },
      { "min": 47150, "max": 100525, "rate": 0.22 },
      { "min": 100525, "max": 191950, "rate": 0.24 },
      { "min": 191950, "max": 243725, "rate": 0.32 },
      { "min": 243725, "max": 609350, "rate": 0.35 },
      { "min": 609350, "max": null, "rate": 0.37 }
    ]
  },
  "selfEmploymentLevy": {
    "name": "Self-Employment Tax",
    "description": "SE tax covers Social Security and Medicare. Deduct 50% from income.",
    "components": [
      { "name": "Social Security", "rate": 0.124, "onIncomeUpTo": 168600 },
      { "name": "Medicare", "rate": 0.029, "onIncomeAbove": 0 },
      { "name": "Additional Medicare", "rate": 0.009, "onIncomeAbove": 200000 }
    ]
  },
  "capitalGains": {
    "annualExemption": 0,
    "rates": [
      { "condition": "short-term (held < 1 year)", "rate": "ordinary_income_rate" },
      { "condition": "long-term, income <= $47,025", "rate": 0.00 },
      { "condition": "long-term, income <= $518,900", "rate": 0.15 },
      { "condition": "long-term, income > $518,900", "rate": 0.20 }
    ],
    "cryptoMethod": "fifo",
    "cryptoNotes": "IRS treats crypto as property. FIFO is default but specific identification is permitted. Every disposal (including crypto-to-crypto) is a taxable event. Wash sale rules do NOT currently apply to crypto.",
    "holdingPeriodForReducedRate": 365
  },
  "vat": { "standardRate": 0, "registrationThreshold": 0, "name": "Sales Tax (state-level, not federal)" },
  "filingDeadlines": [
    { "name": "Annual Tax Return", "description": "Form 1040 with Schedule C and D", "month": 4, "day": 15, "alertDaysBefore": [90, 60, 30, 14, 7] },
    { "name": "Q1 Estimated Tax", "description": "First quarterly estimated payment", "month": 4, "day": 15, "alertDaysBefore": [14, 7] },
    { "name": "Q2 Estimated Tax", "description": "Second quarterly estimated payment", "month": 6, "day": 17, "alertDaysBefore": [14, 7] },
    { "name": "Q3 Estimated Tax", "description": "Third quarterly estimated payment", "month": 9, "day": 16, "alertDaysBefore": [14, 7] },
    { "name": "Q4 Estimated Tax", "description": "Fourth quarterly estimated payment", "month": 1, "day": 15, "alertDaysBefore": [14, 7] }
  ],
  "mainForms": ["1040", "Schedule C", "Schedule D", "Form 8949", "SE"],
  "pensionContributionRelief": true,
  "notes": "State income tax varies by state (0–13.3%). Configure your state rate separately.",
  "generatedByAI": false,
  "lastVerified": "2025-03-01"
}
```

---

#### 🇨🇦 CA — Canada

```json
{
  "countryCode": "CA",
  "countryName": "Canada",
  "currency": "CAD",
  "currencySymbol": "$",
  "taxAuthority": "Canada Revenue Agency (CRA)",
  "taxAuthorityUrl": "https://www.canada.ca/en/revenue-agency.html",
  "taxYear": { "type": "calendar", "labelFormat": "single" },
  "incomeTax": {
    "personalAllowance": 15705,
    "brackets": [
      { "min": 0, "max": 55867, "rate": 0.15 },
      { "min": 55867, "max": 111733, "rate": 0.205 },
      { "min": 111733, "max": 154906, "rate": 0.26 },
      { "min": 154906, "max": 220000, "rate": 0.29 },
      { "min": 220000, "max": null, "rate": 0.33 }
    ]
  },
  "selfEmploymentLevy": {
    "name": "Canada Pension Plan (CPP)",
    "description": "Self-employed pay both employee and employer portions of CPP",
    "components": [
      { "name": "CPP (self-employed rate)", "rate": 0.114, "onIncomeAbove": 3500, "onIncomeUpTo": 68500 }
    ]
  },
  "capitalGains": {
    "annualExemption": 0,
    "rates": [{ "condition": "all capital gains (50% inclusion rate)", "rate": "50%_of_income_rate" }],
    "cryptoMethod": "fifo",
    "cryptoNotes": "CRA treats crypto as a commodity. 50% of capital gains are included in income. Day trading crypto may be treated as business income (100% taxable).",
    "holdingPeriodForReducedRate": null
  },
  "vat": { "standardRate": 0.05, "registrationThreshold": 30000, "name": "GST/HST" },
  "filingDeadlines": [
    { "name": "T1 Personal Return", "description": "Self-employed deadline (extended from April 30)", "month": 6, "day": 15, "alertDaysBefore": [90, 30, 14, 7] },
    { "name": "Balance owing", "description": "Any tax owed still due April 30 despite June 15 filing extension", "month": 4, "day": 30, "alertDaysBefore": [30, 14, 7] }
  ],
  "mainForms": ["T1", "T2125"],
  "notes": "Provincial income tax varies by province. Federal rates shown. Ontario adds ~9.15–13.16%, BC adds ~5.06–20.5% etc.",
  "generatedByAI": false,
  "lastVerified": "2025-03-01"
}
```

---

#### 🇦🇺 AU — Australia

```json
{
  "countryCode": "AU",
  "countryName": "Australia",
  "currency": "AUD",
  "currencySymbol": "$",
  "taxAuthority": "Australian Taxation Office (ATO)",
  "taxAuthorityUrl": "https://www.ato.gov.au",
  "taxYear": { "type": "fiscal", "fiscalStartMonth": 7, "fiscalStartDay": 1, "labelFormat": "split" },
  "incomeTax": {
    "personalAllowance": 18200,
    "brackets": [
      { "min": 0, "max": 18200, "rate": 0 },
      { "min": 18200, "max": 45000, "rate": 0.19 },
      { "min": 45000, "max": 120000, "rate": 0.325 },
      { "min": 120000, "max": 180000, "rate": 0.37 },
      { "min": 180000, "max": null, "rate": 0.45 }
    ]
  },
  "selfEmploymentLevy": {
    "name": "Medicare Levy",
    "description": "2% levy on taxable income for Medicare funding",
    "components": [{ "name": "Medicare Levy", "rate": 0.02, "onIncomeAbove": 26000 }]
  },
  "capitalGains": {
    "annualExemption": 0,
    "rates": [
      { "condition": "held < 12 months", "rate": "ordinary_income_rate" },
      { "condition": "held >= 12 months (50% discount)", "rate": "50%_of_income_rate" }
    ],
    "cryptoMethod": "fifo",
    "cryptoNotes": "ATO treats crypto as property. 50% CGT discount applies if held > 12 months. Each crypto-to-crypto swap is a CGT event.",
    "holdingPeriodForReducedRate": 365
  },
  "vat": { "standardRate": 0.10, "registrationThreshold": 75000, "name": "GST" },
  "filingDeadlines": [
    { "name": "Individual Tax Return", "description": "Self-lodgement deadline", "month": 10, "day": 31, "alertDaysBefore": [90, 30, 14, 7] }
  ],
  "mainForms": ["Individual Tax Return (myTax)", "Business Schedule"],
  "generatedByAI": false,
  "lastVerified": "2025-03-01"
}
```

---

#### 🇩🇪 DE — Germany

```json
{
  "countryCode": "DE",
  "countryName": "Germany",
  "currency": "EUR",
  "currencySymbol": "€",
  "taxAuthority": "Finanzamt",
  "taxAuthorityUrl": "https://www.elster.de",
  "taxYear": { "type": "calendar", "labelFormat": "single" },
  "incomeTax": {
    "personalAllowance": 11604,
    "brackets": [
      { "min": 0, "max": 11604, "rate": 0 },
      { "min": 11604, "max": 17005, "rate": 0.14 },
      { "min": 17005, "max": 66760, "rate": "progressive_14_to_42" },
      { "min": 66760, "max": 277825, "rate": 0.42 },
      { "min": 277825, "max": null, "rate": 0.45, "label": "Reichensteuer" }
    ]
  },
  "selfEmploymentLevy": {
    "name": "Kranken- und Rentenversicherung",
    "description": "Self-employed (Selbständige) pay health and pension insurance directly. Rates vary. Künstlersozialkasse (KSK) applies to creative freelancers at ~50% reduced rate.",
    "components": [
      { "name": "Health Insurance (approx)", "rate": 0.149, "onIncomeAbove": 0 },
      { "name": "Pension Insurance (optional)", "rate": 0.186, "onIncomeAbove": 0 }
    ]
  },
  "capitalGains": {
    "annualExemption": 1000,
    "rates": [{ "condition": "Abgeltungsteuer (flat)", "rate": 0.25 }],
    "cryptoMethod": "fifo",
    "cryptoNotes": "Crypto held > 1 year is TAX FREE in Germany (Spekulationsfrist). Held < 1 year: taxed as ordinary income. Staking/lending may remove the 1-year exemption.",
    "holdingPeriodForReducedRate": 365
  },
  "vat": { "standardRate": 0.19, "registrationThreshold": 22000, "name": "Mehrwertsteuer (MwSt)" },
  "filingDeadlines": [
    { "name": "Einkommensteuererklärung", "description": "Annual income tax return via ELSTER", "month": 7, "day": 31, "alertDaysBefore": [90, 30, 14, 7] }
  ],
  "mainForms": ["Anlage S (Selbständige)", "Anlage SO (Sonstige Einkünfte)", "Anlage KAP"],
  "notes": "Gewerbesteuer (trade tax) applies if running a Gewerbe (commercial business), not a Freiberufler. Freiberufler (freelancers in recognised professions) are exempt.",
  "generatedByAI": false,
  "lastVerified": "2025-03-01"
}
```

---

#### 🇫🇷 FR — France

```json
{
  "countryCode": "FR", "countryName": "France", "currency": "EUR", "currencySymbol": "€",
  "taxAuthority": "Direction Générale des Finances Publiques (DGFiP)",
  "taxAuthorityUrl": "https://www.impots.gouv.fr",
  "taxYear": { "type": "calendar", "labelFormat": "single" },
  "incomeTax": {
    "personalAllowance": 10777,
    "brackets": [
      { "min": 0, "max": 10777, "rate": 0 },
      { "min": 10777, "max": 27478, "rate": 0.11 },
      { "min": 27478, "max": 78570, "rate": 0.30 },
      { "min": 78570, "max": 168994, "rate": 0.41 },
      { "min": 168994, "max": null, "rate": 0.45 }
    ]
  },
  "selfEmploymentLevy": {
    "name": "Cotisations sociales (auto-entrepreneur)",
    "description": "Auto-entrepreneur regime: flat rate on turnover covers all social contributions",
    "components": [
      { "name": "Liberal professions (CIPAV)", "rate": 0.221, "onIncomeAbove": 0 },
      { "name": "Commercial/service activities", "rate": 0.124, "onIncomeAbove": 0 }
    ]
  },
  "capitalGains": {
    "annualExemption": 0,
    "rates": [{ "condition": "Flat Tax (Prélèvement Forfaitaire Unique)", "rate": 0.30 }],
    "cryptoMethod": "average_cost",
    "cryptoNotes": "France uses CUMP (coût unitaire moyen pondéré) — weighted average cost per asset. 30% flat tax (PFU) on gains. Annual allowance of €305 below which gains are exempt.",
    "holdingPeriodForReducedRate": null
  },
  "vat": { "standardRate": 0.20, "registrationThreshold": 36800, "name": "TVA" },
  "filingDeadlines": [
    { "name": "Déclaration de revenus", "description": "Online filing deadline (varies by department)", "month": 6, "day": 8, "alertDaysBefore": [60, 30, 14, 7] }
  ],
  "mainForms": ["2042", "2042-C-PRO", "2086 (crypto)"],
  "generatedByAI": false, "lastVerified": "2025-03-01"
}
```

---

#### 🇳🇱 NL — Netherlands

```json
{
  "countryCode": "NL", "countryName": "Netherlands", "currency": "EUR", "currencySymbol": "€",
  "taxAuthority": "Belastingdienst", "taxAuthorityUrl": "https://www.belastingdienst.nl",
  "taxYear": { "type": "calendar", "labelFormat": "single" },
  "incomeTax": {
    "personalAllowance": 0,
    "brackets": [
      { "min": 0, "max": 75518, "rate": 0.3697, "label": "Box 1 lower (includes social premiums)" },
      { "min": 75518, "max": null, "rate": 0.495, "label": "Box 1 upper" }
    ]
  },
  "selfEmploymentLevy": {
    "name": "Zelfstandigenaftrek + MKB-winstvrijstelling",
    "description": "Self-employed deduction of €5,030 + 12.7% SME profit exemption. Social premiums included in Box 1 rate.",
    "components": []
  },
  "capitalGains": {
    "annualExemption": 57000,
    "rates": [{ "condition": "Box 3 deemed return (fictitious rate ~6.04%)", "rate": 0.36 }],
    "cryptoMethod": "average_cost",
    "cryptoNotes": "Netherlands taxes crypto under Box 3 (wealth tax) — not on actual gains but on deemed return of ~6% on the asset value on Jan 1. Report crypto value on Jan 1 each year.",
    "holdingPeriodForReducedRate": null
  },
  "vat": { "standardRate": 0.21, "registrationThreshold": 0, "name": "BTW" },
  "filingDeadlines": [
    { "name": "Inkomstenbelasting aangifte", "description": "Annual income tax return", "month": 5, "day": 1, "alertDaysBefore": [60, 30, 14, 7] }
  ],
  "mainForms": ["M-biljet", "IB-aangifte"],
  "generatedByAI": false, "lastVerified": "2025-03-01"
}
```

---

#### 🇮🇪 IE — Ireland

```json
{
  "countryCode": "IE", "countryName": "Ireland", "currency": "EUR", "currencySymbol": "€",
  "taxAuthority": "Revenue Commissioners", "taxAuthorityUrl": "https://www.revenue.ie",
  "taxYear": { "type": "calendar", "labelFormat": "single" },
  "incomeTax": {
    "personalAllowance": 1875,
    "brackets": [
      { "min": 0, "max": 42000, "rate": 0.20, "label": "Standard Rate" },
      { "min": 42000, "max": null, "rate": 0.40, "label": "Higher Rate" }
    ]
  },
  "selfEmploymentLevy": {
    "name": "PRSI + USC",
    "description": "Pay Related Social Insurance (Class S: 4%) + Universal Social Charge",
    "components": [
      { "name": "PRSI Class S", "rate": 0.04, "onIncomeAbove": 5000 },
      { "name": "USC Band 1", "rate": 0.005, "onIncomeAbove": 0, "onIncomeUpTo": 12012 },
      { "name": "USC Band 2", "rate": 0.02, "onIncomeAbove": 12012, "onIncomeUpTo": 25760 },
      { "name": "USC Band 3", "rate": 0.04, "onIncomeAbove": 25760, "onIncomeUpTo": 70044 },
      { "name": "USC Band 4", "rate": 0.08, "onIncomeAbove": 70044 }
    ]
  },
  "capitalGains": {
    "annualExemption": 1270,
    "rates": [{ "condition": "standard", "rate": 0.33 }],
    "cryptoMethod": "fifo",
    "cryptoNotes": "Revenue treats crypto as a chargeable asset. 33% CGT. Annual exemption €1,270.",
    "holdingPeriodForReducedRate": null
  },
  "vat": { "standardRate": 0.23, "registrationThreshold": 37500, "name": "VAT" },
  "filingDeadlines": [
    { "name": "Form 11 (ROS)", "description": "Self-assessed income tax return", "month": 11, "day": 14, "alertDaysBefore": [60, 30, 14, 7] },
    { "name": "Preliminary Tax", "description": "90% of current year liability due", "month": 11, "day": 14, "alertDaysBefore": [30, 14, 7] }
  ],
  "mainForms": ["Form 11", "Form CG1"],
  "generatedByAI": false, "lastVerified": "2025-03-01"
}
```

---

#### 🇪🇸 ES — Spain

```json
{
  "countryCode": "ES", "countryName": "Spain", "currency": "EUR", "currencySymbol": "€",
  "taxAuthority": "Agencia Estatal de Administración Tributaria (AEAT)", "taxAuthorityUrl": "https://www.agenciatributaria.es",
  "taxYear": { "type": "calendar", "labelFormat": "single" },
  "incomeTax": {
    "personalAllowance": 5550,
    "brackets": [
      { "min": 0, "max": 12450, "rate": 0.19 },
      { "min": 12450, "max": 20200, "rate": 0.24 },
      { "min": 20200, "max": 35200, "rate": 0.30 },
      { "min": 35200, "max": 60000, "rate": 0.37 },
      { "min": 60000, "max": 300000, "rate": 0.45 },
      { "min": 300000, "max": null, "rate": 0.47 }
    ]
  },
  "selfEmploymentLevy": {
    "name": "Cuota de autónomos (RETA)",
    "description": "Monthly flat contribution based on estimated net income bracket",
    "components": [{ "name": "Cuota mínima (net income < €670/mo)", "flatAnnualAmount": 3456 }]
  },
  "capitalGains": {
    "annualExemption": 0,
    "rates": [
      { "condition": "up to €6,000", "rate": 0.19 },
      { "condition": "€6,001–€50,000", "rate": 0.21 },
      { "condition": "€50,001–€200,000", "rate": 0.23 },
      { "condition": "above €200,000", "rate": 0.26 }
    ],
    "cryptoMethod": "fifo",
    "cryptoNotes": "Spain taxes crypto gains as savings income (renta del ahorro). FIFO method. Modelo 721 required for crypto held abroad > €50,000.",
    "holdingPeriodForReducedRate": null
  },
  "vat": { "standardRate": 0.21, "registrationThreshold": 0, "name": "IVA" },
  "filingDeadlines": [
    { "name": "Declaración de la Renta (IRPF)", "description": "Annual income tax return", "month": 6, "day": 30, "alertDaysBefore": [60, 30, 14, 7] }
  ],
  "mainForms": ["Modelo 100", "Modelo 130 (quarterly)", "Modelo 721 (crypto abroad)"],
  "generatedByAI": false, "lastVerified": "2025-03-01"
}
```

---

#### 🇮🇹 IT — Italy

```json
{
  "countryCode": "IT", "countryName": "Italy", "currency": "EUR", "currencySymbol": "€",
  "taxAuthority": "Agenzia delle Entrate", "taxAuthorityUrl": "https://www.agenziaentrate.gov.it",
  "taxYear": { "type": "calendar", "labelFormat": "single" },
  "incomeTax": {
    "personalAllowance": 8174,
    "brackets": [
      { "min": 0, "max": 28000, "rate": 0.23 },
      { "min": 28000, "max": 50000, "rate": 0.35 },
      { "min": 50000, "max": null, "rate": 0.43 }
    ]
  },
  "selfEmploymentLevy": {
    "name": "Contributi previdenziali (Gestione Separata INPS)",
    "description": "Separate management fund for freelancers without a specific professional fund",
    "components": [{ "name": "Gestione Separata", "rate": 0.2607, "onIncomeAbove": 0 }]
  },
  "capitalGains": {
    "annualExemption": 0,
    "rates": [{ "condition": "imposta sostitutiva", "rate": 0.26 }],
    "cryptoMethod": "lifo",
    "cryptoNotes": "Italy taxes crypto gains at 26% flat. Exempt if total gains < €2,000/year. RW form required for foreign crypto holdings.",
    "holdingPeriodForReducedRate": null
  },
  "vat": { "standardRate": 0.22, "registrationThreshold": 0, "name": "IVA" },
  "filingDeadlines": [
    { "name": "Modello Redditi", "description": "Annual tax return", "month": 11, "day": 30, "alertDaysBefore": [60, 30, 14, 7] }
  ],
  "mainForms": ["Modello Redditi PF", "Quadro LM (forfettario)", "Quadro RW"],
  "notes": "Regime forfettario (flat-rate scheme) available for turnover < €85,000. Tax rate 5% (first 5 years) or 15% on 78% of turnover. Very favourable for new freelancers.",
  "generatedByAI": false, "lastVerified": "2025-03-01"
}
```

---

#### 🇵🇹 PT — Portugal

```json
{
  "countryCode": "PT", "countryName": "Portugal", "currency": "EUR", "currencySymbol": "€",
  "taxAuthority": "Autoridade Tributária e Aduaneira (AT)", "taxAuthorityUrl": "https://www.portaldasfinancas.gov.pt",
  "taxYear": { "type": "calendar", "labelFormat": "single" },
  "incomeTax": {
    "personalAllowance": 8500,
    "brackets": [
      { "min": 0, "max": 7703, "rate": 0.135 },
      { "min": 7703, "max": 11623, "rate": 0.18 },
      { "min": 11623, "max": 16472, "rate": 0.23 },
      { "min": 16472, "max": 21321, "rate": 0.26 },
      { "min": 21321, "max": 27146, "rate": 0.3275 },
      { "min": 27146, "max": 39791, "rate": 0.37 },
      { "min": 39791, "max": 51997, "rate": 0.435 },
      { "min": 51997, "max": 81199, "rate": 0.45 },
      { "min": 81199, "max": null, "rate": 0.48 }
    ]
  },
  "selfEmploymentLevy": {
    "name": "Segurança Social (Recibos Verdes)",
    "description": "Social security for self-employed (green receipts). Based on relevant income.",
    "components": [{ "name": "Segurança Social", "rate": 0.214, "onIncomeAbove": 0 }]
  },
  "capitalGains": {
    "annualExemption": 0,
    "rates": [{ "condition": "standard", "rate": 0.28 }],
    "cryptoMethod": "fifo",
    "cryptoNotes": "Portugal introduced crypto tax in 2023. Gains from crypto held < 1 year taxed at 28%. Held > 1 year is TAX FREE. NHR (Non-Habitual Resident) regime may offer 0% on foreign source crypto.",
    "holdingPeriodForReducedRate": 365
  },
  "vat": { "standardRate": 0.23, "registrationThreshold": 13500, "name": "IVA" },
  "filingDeadlines": [
    { "name": "IRS Declaration", "description": "Annual income tax return", "month": 6, "day": 30, "alertDaysBefore": [60, 30, 14, 7] }
  ],
  "mainForms": ["Modelo 3", "Anexo B (self-employment)", "Anexo G (capital gains)"],
  "notes": "NHR regime (or its replacement IFICI from 2024) can provide significant tax advantages for new residents. Worth investigating.",
  "generatedByAI": false, "lastVerified": "2025-03-01"
}
```

---

#### 🇸🇬 SG — Singapore

```json
{
  "countryCode": "SG", "countryName": "Singapore", "currency": "SGD", "currencySymbol": "S$",
  "taxAuthority": "Inland Revenue Authority of Singapore (IRAS)", "taxAuthorityUrl": "https://www.iras.gov.sg",
  "taxYear": { "type": "calendar", "labelFormat": "single" },
  "incomeTax": {
    "personalAllowance": 20000,
    "brackets": [
      { "min": 0, "max": 20000, "rate": 0 },
      { "min": 20000, "max": 30000, "rate": 0.02 },
      { "min": 30000, "max": 40000, "rate": 0.035 },
      { "min": 40000, "max": 80000, "rate": 0.07 },
      { "min": 80000, "max": 120000, "rate": 0.115 },
      { "min": 120000, "max": 160000, "rate": 0.15 },
      { "min": 160000, "max": 200000, "rate": 0.18 },
      { "min": 200000, "max": 240000, "rate": 0.19 },
      { "min": 240000, "max": 280000, "rate": 0.195 },
      { "min": 280000, "max": 320000, "rate": 0.20 },
      { "min": 320000, "max": 500000, "rate": 0.22 },
      { "min": 500000, "max": 1000000, "rate": 0.23 },
      { "min": 1000000, "max": null, "rate": 0.24 }
    ]
  },
  "selfEmploymentLevy": {
    "name": "Medisave (CPF)",
    "description": "Self-employed contribute to Medisave only (not full CPF). Rate depends on age and income.",
    "components": [{ "name": "Medisave contribution", "rate": 0.085, "onIncomeAbove": 6000 }]
  },
  "capitalGains": {
    "annualExemption": 0,
    "rates": [{ "condition": "NO capital gains tax in Singapore", "rate": 0 }],
    "cryptoMethod": "fifo",
    "cryptoNotes": "Singapore has NO capital gains tax. However, frequent crypto trading may be treated as income (trading income). IRAS applies 'badges of trade' test.",
    "holdingPeriodForReducedRate": null
  },
  "vat": { "standardRate": 0.09, "registrationThreshold": 1000000, "name": "GST" },
  "filingDeadlines": [
    { "name": "Income Tax Return (Form B/B1)", "description": "Self-employed individuals", "month": 4, "day": 15, "alertDaysBefore": [60, 30, 14, 7] }
  ],
  "mainForms": ["Form B", "Form B1"],
  "generatedByAI": false, "lastVerified": "2025-03-01"
}
```

---

#### 🇦🇪 AE — United Arab Emirates

```json
{
  "countryCode": "AE", "countryName": "United Arab Emirates", "currency": "AED", "currencySymbol": "د.إ",
  "taxAuthority": "Federal Tax Authority (FTA)", "taxAuthorityUrl": "https://tax.gov.ae",
  "taxYear": { "type": "calendar", "labelFormat": "single" },
  "incomeTax": {
    "personalAllowance": 999999999,
    "brackets": [{ "min": 0, "max": null, "rate": 0, "label": "No personal income tax in UAE" }]
  },
  "selfEmploymentLevy": { "name": "None", "description": "No self-employment tax for individuals", "components": [] },
  "capitalGains": {
    "annualExemption": 999999999,
    "rates": [{ "condition": "No capital gains tax on individuals", "rate": 0 }],
    "cryptoMethod": "fifo",
    "cryptoNotes": "No personal capital gains tax in UAE. However, Corporate Tax at 9% applies to businesses with net income > AED 375,000 from June 2023. Freelancers with a Freezone license may qualify for 0% CT.",
    "holdingPeriodForReducedRate": null
  },
  "vat": { "standardRate": 0.05, "registrationThreshold": 375000, "name": "VAT" },
  "filingDeadlines": [
    { "name": "VAT Return (if registered)", "description": "Quarterly VAT return", "month": 0, "day": 28, "alertDaysBefore": [14, 7] }
  ],
  "mainForms": ["VAT Return (if applicable)", "Corporate Tax Return (if business)"],
  "notes": "UAE has no personal income tax. Track income for record-keeping and VAT compliance only. Corporate Tax implications apply if operating as a company.",
  "generatedByAI": false, "lastVerified": "2025-03-01"
}
```

---

#### 🇮🇳 IN — India

```json
{
  "countryCode": "IN", "countryName": "India", "currency": "INR", "currencySymbol": "₹",
  "taxAuthority": "Income Tax Department", "taxAuthorityUrl": "https://www.incometax.gov.in",
  "taxYear": { "type": "fiscal", "fiscalStartMonth": 4, "fiscalStartDay": 1, "labelFormat": "split" },
  "incomeTax": {
    "personalAllowance": 300000,
    "brackets": [
      { "min": 0, "max": 300000, "rate": 0, "label": "New Regime" },
      { "min": 300000, "max": 600000, "rate": 0.05 },
      { "min": 600000, "max": 900000, "rate": 0.10 },
      { "min": 900000, "max": 1200000, "rate": 0.15 },
      { "min": 1200000, "max": 1500000, "rate": 0.20 },
      { "min": 1500000, "max": null, "rate": 0.30 }
    ]
  },
  "selfEmploymentLevy": { "name": "None (separate)", "description": "No separate SE levy; income included in IT slab", "components": [] },
  "capitalGains": {
    "annualExemption": 100000,
    "rates": [
      { "condition": "short-term (equity < 1 year)", "rate": 0.20 },
      { "condition": "long-term (equity > 1 year, above ₹1L exemption)", "rate": 0.125 },
      { "condition": "other assets short-term", "rate": "slab_rate" },
      { "condition": "other assets long-term (> 2 years)", "rate": 0.125 }
    ],
    "cryptoMethod": "fifo",
    "cryptoNotes": "India taxes crypto/VDA (Virtual Digital Assets) at a flat 30% regardless of holding period. No deductions allowed except cost of acquisition. 1% TDS deducted at source on transactions > ₹10,000.",
    "holdingPeriodForReducedRate": null
  },
  "vat": { "standardRate": 0.18, "registrationThreshold": 2000000, "name": "GST" },
  "filingDeadlines": [
    { "name": "ITR Filing", "description": "Income Tax Return (non-audit cases)", "month": 7, "day": 31, "alertDaysBefore": [60, 30, 14, 7] },
    { "name": "Advance Tax Q1", "description": "First instalment of advance tax", "month": 6, "day": 15, "alertDaysBefore": [14, 7] }
  ],
  "mainForms": ["ITR-3 (business/profession)", "ITR-4 (Sugam presumptive)"],
  "generatedByAI": false, "lastVerified": "2025-03-01"
}
```

---

#### 🇿🇦 ZA — South Africa

```json
{
  "countryCode": "ZA", "countryName": "South Africa", "currency": "ZAR", "currencySymbol": "R",
  "taxAuthority": "South African Revenue Service (SARS)", "taxAuthorityUrl": "https://www.sars.gov.za",
  "taxYear": { "type": "fiscal", "fiscalStartMonth": 3, "fiscalStartDay": 1, "labelFormat": "split" },
  "incomeTax": {
    "personalAllowance": 95750,
    "brackets": [
      { "min": 0, "max": 237100, "rate": 0.18 },
      { "min": 237100, "max": 370500, "rate": 0.26 },
      { "min": 370500, "max": 512800, "rate": 0.31 },
      { "min": 512800, "max": 673000, "rate": 0.36 },
      { "min": 673000, "max": 857900, "rate": 0.39 },
      { "min": 857900, "max": 1817000, "rate": 0.41 },
      { "min": 1817000, "max": null, "rate": 0.45 }
    ]
  },
  "selfEmploymentLevy": { "name": "None separate", "description": "Provisional taxpayer system — two advance payments during the year", "components": [] },
  "capitalGains": {
    "annualExemption": 40000,
    "rates": [{ "condition": "40% inclusion rate, then taxed at income rate", "rate": "40%_of_income_rate" }],
    "cryptoMethod": "fifo",
    "cryptoNotes": "SARS treats crypto as an asset. Gains included at 40% in taxable income for individuals. Active traders may be taxed as revenue (100% inclusion).",
    "holdingPeriodForReducedRate": null
  },
  "vat": { "standardRate": 0.15, "registrationThreshold": 1000000, "name": "VAT" },
  "filingDeadlines": [
    { "name": "ITR12 Return", "description": "Individual income tax return", "month": 10, "day": 23, "alertDaysBefore": [60, 30, 14, 7] }
  ],
  "mainForms": ["ITR12"],
  "generatedByAI": false, "lastVerified": "2025-03-01"
}
```

---

For **🇨🇭 CH, 🇸🇪 SE, 🇳🇴 NO, 🇩🇰 DK, 🇫🇮 FI, 🇧🇪 BE, 🇦🇹 AT, 🇵🇱 PL, 🇨🇿 CZ, 🇬🇷 GR, 🇳🇿 NZ, 🇧🇷 BR, 🇲🇽 MX, 🇯🇵 JP** — write the adapter file using the same schema with accurate rates from your training knowledge for that country, clearly set `"generatedByAI": false` only if you are confident in the values. Mark as `"generatedByAI": true` with today's date if there is any uncertainty.

---

### Dynamic Adapter Generation (all other countries)

If `COUNTRY_CODE` is not in the pre-built list above, generate a tax adapter dynamically:

1. Tell the user: "I don't have a pre-built adapter for [country]. I'm going to generate one now using my knowledge of [country]'s tax system. You should verify the rates against [country's tax authority website] before relying on them for filing."

2. Call Claude (yourself) with this prompt to generate the adapter JSON:

```
Generate a complete TaxAdapter JSON for [COUNTRY_NAME] ([COUNTRY_CODE]).
Use the schema exactly as defined. Include:
- Correct income tax brackets for the current tax year
- Self-employment tax / social security equivalent
- Capital gains rules including crypto treatment
- VAT/GST threshold and rate
- Filing deadlines with correct month/day
- Main tax forms used
- Any important notes (special regimes, state/provincial variations, etc.)

Set generatedByAI: true and lastVerified to today's date.
Include a disclaimer note explaining what should be verified.
```

3. Save the generated adapter to `~/.ai-accountant/tax-adapters/[COUNTRY_CODE].json`

4. Tell the user: "Tax adapter generated for [country]. Please review it at `~/.ai-accountant/tax-adapters/[COUNTRY_CODE].json` and correct any values before your first filing. You can edit the JSON directly — changes take effect immediately."

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
