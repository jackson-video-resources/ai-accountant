# AI Accountant Masterclass — Video Production Outline

**Title:** I Built an AI Accountancy Firm That Does My Taxes Automatically (Full Tutorial)
**Format:** Screen share + talking head
**Target runtime:** 65–75 minutes
**Platform:** YouTube (long-form masterclass)
**CTA:** GitHub link in description → copy SETUP-PROMPT.md → paste into Claude Code

---

## Thumbnail Concept

Split screen: LEFT — crumpled pile of receipts, a £150/hr accountant invoice, a stress-face emoji. RIGHT — clean Telegram notification on a phone: "Tax bill calculated: £4,204. Confirm to proceed?" with a green checkmark button below.

Text overlay: **"I FIRED MY ACCOUNTANT"** (large, top) + **"AI did my taxes in 11 mins"** (smaller, bottom)

---

## B-Roll Requirements

Before filming, capture:
- Telegram phone notifications arriving (multiple: receipt logged, weekly summary, deadline alert)
- Terminal scrolling while building the system (speed up 4x in edit)
- A real Binance CSV open in Excel — let the camera zoom in on the 500+ rows
- Finder/File Explorer showing a receipt being dragged into the inbox folder
- The tax calculation output in terminal (full clean breakdown)
- Clock ticking / calendar visual for Jan 31 deadline
- Split screen: accountant invoice vs £0 AI cost

---

## Production Notes

- Film demo sections on actual machine (not VM) — real latency makes it believable
- Record terminal at 1920×1080 with a large, legible font (Fira Code or JetBrains Mono, 16pt minimum)
- Use a second phone camera for Telegram notification shots — don't screencapture them
- Chapter markers must match timestamps for YouTube chapters
- Pin GitHub link as first comment
- End screen: 20s with subscribe prompt + link to next video in series

---

---

# SECTION 1: HOOK
**[00:00 — 02:30] | Talking head + B-roll cutaways**

**What to say:**
"Last January 31st, I was sat at my desk at 11pm, sweating, trying to remember what that £340 payment in October was for, while my accountant's invoice sat open in another tab — £750 for six hours of work that I could have done myself if I'd just been organised.

That's not happening again.

[CUT TO: Telegram notification arriving on phone]

What you're looking at is my AI accountant sending me a message. I just dropped a receipt in a folder on my computer. That's it. Within about 20 seconds it logged the expense, categorised it, calculated the tax saving, and updated my running total for the year.

[CUT TO: terminal showing full tax calculation]

This is my entire tax calculation for 2024. Income. Expenses. Capital gains. National Insurance. Payment on account. Every number. And I didn't calculate any of it.

By the end of this video, you'll have exactly this running on your computer. It's free. It's open source. And it takes about 12 minutes to set up.

Let's go."

**B-roll:** Telegram notification → tax calculation terminal → quick cut montage of all features

---

# SECTION 2: THE PROBLEM
**[02:30 — 07:00] | Talking head**

**What to say:**
"Here's the thing about tax. It's not actually complicated. For most freelancers and self-employed people, it's three things: what did you earn, what did you spend, and what's the difference.

The problem isn't the maths. The problem is the admin. It's keeping track of 200 receipts across a year. It's remembering that the domain renewal in March was a business expense. It's knowing that HMRC lets you claim 45p a mile for the first 10,000 miles. It's knowing that your phone bill is 60% claimable if you use it 60% for work.

And then there's crypto. If you've bought and sold any crypto this year — even swapping one coin for another — that's a capital gains event. Every single one. HMRC wants to know about every one. The correct way to calculate this is called the Section 104 pool method and it involves tracking the average cost basis across your entire history of acquisitions. This is genuinely complex.

The average accountant charges £150 an hour. A basic self-assessment costs £300–600. If you have crypto it's often more.

What we're building today costs about £2 a month in API usage.

The reason this is possible now — and wasn't two years ago — is Claude Code. This isn't just Claude the chatbot. This is Claude running directly in your terminal, with the ability to write code, read your files, run commands, and build entire systems. It can look at a receipt image and extract the amount and vendor. It can read a 500-row Binance CSV and calculate your capital gains correctly. And it can do all of this automatically, in the background, while you get on with your life."

---

# SECTION 3: WHAT WE'RE BUILDING
**[07:00 — 13:00] | Screen share — architecture diagram + demo preview**

**What to say:**
"Let me show you exactly what this system does before we build it — so when you're following along you understand why each piece matters.

[SHOW: folder structure in Finder/Explorer]

There's a folder on your computer called ai-accountant. Inside it, there's an inbox folder. That inbox is the front door to your accountancy firm. Everything financial goes in here.

[SHOW: drag receipt into folder → Telegram notification]

Drop a receipt in. Boom. Within 20 seconds it's been read, classified, the vendor and amount extracted, the category determined, the deductibility percentage calculated, and you get a Telegram message telling you exactly what happened and what it saved you in tax.

[SHOW: email forwarding explanation]

You can also forward emails. Got a receipt from Adobe, AWS, Notion, Spotify for Business? You set up a filter in Gmail or Outlook to forward those straight to your local accountant. They're filed automatically.

[SHOW: Binance CSV drop]

And then there's this. This is a Binance trade history export. 500 rows. Dates, pairs, amounts, fees. I'm going to drag this into the exchanges folder.

[DROP FILE — wait — show Telegram message]

47 capital gains events. Net gain for the year: £3,241. Annual exempt amount: £3,000. Taxable gain: £241. CGT due: £24.

That calculation used the Section 104 pool method — the actual HMRC-required calculation. Not a rough estimate. Not a flat rate. The real number.

[SHOW: full tax calculation]

And when you ask it 'what do I owe this year?' this is what you get. Every line. Every number. The calculation breakdown. The payment on account that catches everyone by surprise in their first year of self-employment. All of it.

And this all runs locally. On your machine. Nothing goes to the cloud except the text of your documents going to Anthropic's API for classification — and you'll be asked to confirm that consent during setup. Your actual files never leave your computer."

**Key point to emphasise:** "This isn't a SaaS. You don't have a subscription. You own this."

---

# SECTION 4: PREREQUISITES
**[13:00 — 20:00] | Screen share — install walkthroughs**

**What to say:**
"You need four things. Let me walk you through each one."

### 4a: Node.js [13:00 — 14:30]

"Node.js is the runtime that powers this system. Think of it as the engine.

[SHOW: nodejs.org]

Go to nodejs.org, download the LTS version. On Mac you can also use Homebrew: `brew install node`. On Windows, download the installer, run it, defaults are fine.

Once installed, open your terminal and run `node --version`. You should see v18 or higher.

[SHOW: terminal output]

Windows users — if you're on Windows, you'll want to use WSL2. That's the Windows Subsystem for Linux. It sounds scarier than it is — it just gives you a Linux terminal inside Windows, and it makes everything work properly. Go to the Microsoft Store, search for Ubuntu, install it, and use that terminal for everything that follows."

### 4b: Claude Code [14:30 — 16:30]

"Claude Code is Anthropic's CLI — their command line tool. This is what you're actually going to be using to build everything.

[SHOW: claude.ai/code]

You'll need a Claude account. If you don't have one, sign up at claude.ai. There's a $20/month plan that includes Claude Code access.

Install it by running: `npm install -g @anthropic/claude-code`

Once installed, run `claude` in your terminal and authenticate with your account.

On Windows WSL2: same command, run it from your Ubuntu terminal."

### 4c: Anthropic API Key [16:30 — 18:00]

"Separately from your Claude account, you need an Anthropic API key. This is what the system uses to classify receipts and answer your questions in the background — Claude Code itself uses your account, but the automated processing uses the API directly.

[SHOW: console.anthropic.com]

Go to console.anthropic.com, sign in, go to API Keys, create a new key. Copy it. You'll need it in a moment.

The cost for a typical freelancer is about £2–5 a month. The system uses the cheapest Claude model — Haiku — for routine classification. It only uses the more expensive model when you're having a conversation."

### 4d: Telegram Bot [18:00 — 20:00]

"Telegram is how the system talks to you. You'll get a message every time a receipt is processed, a weekly summary, and alerts as filing deadlines approach.

[SHOW: Telegram on phone]

Open Telegram and search for @BotFather. Send it `/newbot`. It'll ask for a name — call it whatever you want, maybe '[YourName] Accountant'. Then it'll give you a token. Copy that.

Now search for @userinfobot on Telegram. Send it any message. It replies with your chat ID. Copy that too.

You'll need both in a moment."

---

# SECTION 5: THE ONE-SHOT SETUP
**[20:00 — 38:00] | Screen share — this is the centrepiece of the video**

**What to say (intro):**
"Alright. This is the part of the video that made me go 'oh, this is actually insane.'

I'm going to open Claude Code. I'm going to paste in one prompt. And then I'm going to watch it build an entire accountancy firm.

The prompt is in the GitHub repo — link in the description. There's a copy button on GitHub, so it's literally: click link, click copy, paste. That's it.

Let me show you."

### 5a: Open Claude Code and paste [20:00 — 21:00]

[SHOW: terminal, `claude` running, paste the prompt]

"Claude Code is now reading these instructions. Watch what it does."

### 5b: Phase 0 — Introduction [21:00 — 21:30]

[SHOW: Claude Code printing the intro message about what's being built]

"It starts by telling you what's about to happen. It's going to check your environment, ask you eight questions, then build everything."

### 5c: Phase 1 — Environment check [21:30 — 22:00]

[SHOW: Node, npm, PM2 check output]

"Checks Node.js is installed, the right version, installs PM2 if it's missing. Done."

### 5d: Phase 2 — The interview [22:00 — 26:00]

[SHOW: each question being asked, answer each one live]

"It's going to ask me eight questions. I'll answer them and you can see what to put for yours.

Name — easy.
Country — I'm UK so I'm picking option 1.
UTR — I'll put mine in. Yours is on any letter from HMRC. Skip if you don't have it.
Crypto — Yes.
Exchanges — Binance and Coinbase for me.
Anthropic API key — I've got mine ready, I'll paste it in.
Telegram — Yes. I'm going to paste my bot token and chat ID.
Install location — I'll leave it as the default: ~/ai-accountant."

[SHOW: confirmation summary before building]

"There it is. It's confirming everything before it starts. I say yes and..."

### 5e: The build [26:00 — 36:00]

[SHOW: full build unfolding — speed up in edit to 2x during package install, return to normal speed when it's writing files]

Narrate what's happening at each stage:
- "Creating the folder structure. Receipts. Bank statements. Exchanges. Done."
- "Writing the package.json and installing dependencies. You can see 50-odd packages going in. Better-sqlite3 for the database, Tesseract.js for OCR, chokidar for file watching..."
- "Now it's writing the actual source code. Watch this — it's writing the file watcher, the OCR engine, the Claude classifier..."
- "This is the tax engine. It's writing the UK Self Assessment calculation. Personal allowance, basic rate, higher rate, Class 4 National Insurance..."
- "CGT calculator. Section 104 pool method. Same-day rule, 30-day bed and breakfast rule, then the pool..."
- "Telegram bot. Cron scheduler. Main entry point. PM2 configuration."

**Key narration moment** (when CLAUDE.md gets written): "This is an important one. It's writing the CLAUDE.md — the file that tells every future Claude Code session in this folder to behave as a professional accountant. This is what makes 'what do I owe?' give you a real answer instead of 'I don't have access to your finances.' Once this is in place, every conversation in this folder is a conversation with your AI accountant."

### 5f: Startup and verification [36:00 — 38:00]

[SHOW: PM2 starting, health check, test receipt being processed]

"PM2 is starting the system. It's running the health check — database connected, file watcher running, Telegram connected. Now it's dropping a test receipt in the inbox to make sure the whole pipeline works.

[SHOW: Telegram notification arriving on phone]

And there it is on my phone. £12.99. Amazon. USB-C Cable. Office supplies. 100% deductible. Tax saving: £2.60.

It's working."

[SHOW: the startup summary box]

"Your AI accountant is live."

---

# SECTION 6: FIRST REAL RECEIPT
**[38:00 — 46:00] | Screen share + phone cam**

**What to say:**
"Now let me show you this working with a real receipt. I've got a PDF here from my last Adobe Creative Cloud invoice."

[DRAG FILE into inbox/receipts/]

"Dropped. Now watch."

[SHOW: terminal logs processing it → Telegram arrives]

"Here's the Telegram. Adobe Creative Cloud. £54.99. Software. 100% deductible. Tax saving: £11.00. It even pulled the date off the invoice.

Now let me open Claude Code and ask it something.

'What are my top expense categories this year?'

[SHOW: Claude Code querying the database and responding]

Software is number one with £342. Office supplies second. Travel third.

'Can I claim for my home office?'

[SHOW: response]

This is where it gets good. It's not just saying yes or no. It's explaining both methods — the £6 per week flat rate, and the actual room percentage method. It's telling me which one is better for me based on what it already knows about my setup. It's telling me that if my home office is 10% of my floor area, I could claim more than the flat rate and it would be worth calculating.

And here's something I didn't know before I built this: it's flagging that I haven't claimed Working From Home expenses at all this year. That's a missed £312.

This is what a good accountant does. They don't just answer the question you asked. They notice what you didn't ask."

---

# SECTION 7: CRYPTO — THE SECTION 104 DEMO
**[46:00 — 54:00] | Screen share**

**What to say:**
"If you don't have any crypto, feel free to skip to [timestamp]. But if you do — stay with me, because this section could save you from a significant HMRC problem.

Here's the deal with crypto and UK tax. Every time you dispose of a crypto asset — sell it, swap it for another coin, use it to buy something — that's a capital gains event. HMRC doesn't care that you swapped ETH for a memecoin at 2am. That's a disposal. They want to know your gain or loss on that disposal.

[SHOW: HMRC crypto guidance page briefly]

The way HMRC requires you to calculate this is called the Section 104 pool method. Here's how it works.

[SHOW: whiteboard-style simple diagram]

Imagine a pool. Every time you buy bitcoin, you drop your coins in the pool. You also drop in how much you paid for them. The pool now has a quantity and a total cost. Your average cost per coin is total cost divided by quantity.

When you sell, you scoop out your coins from the pool. Your cost basis is the average cost per coin times the number you sold. Your gain is what you got for them minus that cost basis.

But there are two rules that override this. If you buy and sell on the same day, those trades are matched directly. And if you sell and then buy back within 30 days — that's the bed and breakfast rule — HMRC matches those instead of using the pool.

This prevents people selling at a loss on December 31st, claiming the loss, and then buying back on January 1st.

[SHOW: Binance CSV in Excel]

Here's my Binance trade history. 500 rows. Every trade I did this year. I'm going to drop this into my exchanges/binance folder."

[DROP FILE]

"And watch."

[SHOW: terminal processing → Telegram notification]

"47 CGT events processed. The system applied every same-day match, checked for bed and breakfast situations, then ran the pool calculation on the rest.

[ASK in Claude Code]: 'Show me my CGT summary for 2024-25'

[SHOW: response with per-asset breakdown]

Bitcoin: £2,841 gain. ETH: £233 loss. SOL: £167 gain. Net: £3,241. Annual exempt amount: £3,000. Taxable gain: £241. CGT due at 10%: £24.

Without this system, you'd either be paying an accountant to do this, or using a dedicated crypto tax tool like Koinly that costs £100-200 a year. This does it for free, automatically, as part of your overall tax picture."

---

# SECTION 8: BANK STATEMENT IMPORT
**[54:00 — 58:00] | Screen share**

**What to say:**
"Most modern banks — Monzo, Starling, Revolut, and most traditional banks — let you export your transactions as a CSV. This is really useful for catching any business expenses you forgot to keep receipts for.

[SHOW: Monzo → export transactions]

In Monzo, you go to your account, tap the three dots, Export Statement, choose CSV.

[DRAG into inbox/bank-statements/]

The system reads your bank CSV and does two things. First, it matches transactions against receipts you've already filed — so if you've got a receipt for the Adobe charge and the bank statement also shows it, it links them. Second, it flags everything over £25 that doesn't have a matching receipt. Those are your gaps.

[ASK: 'missing receipts']

'You have 8 transactions over £25 with no receipt attached. The largest is £89 at AWS on September 14th.' That's £89 I need to find a receipt for — or if I can't find one, I can still claim it if I can confirm what it was.

This is the kind of thing your accountant would ask you about in a review meeting. Now you get it surfaced automatically."

---

# SECTION 9: ASK YOUR ACCOUNTANT ANYTHING
**[58:00 — 64:00] | Screen share — the "Wispr Flow" moment**

**What to say:**
"Now I want to show you something that completely changes how you interact with this.

The system has been running, processing all your documents, storing everything in a database. And because we have a CLAUDE.md in this folder, Claude Code knows it's operating as your personal accountant — not a general assistant. It has access to your actual numbers.

Let me just ask it things.

'What do I owe this year?'

[SHOW: full tax calculation response]

There it is. £6,404 due January 31st. Broken down: £4,260 income tax, £1,917 Class 4 NI, £179 Class 2 NI, £24 CGT, £24 balancing payment from the POA calculation. And it's telling me that £3,202 of that January payment is actually the first payment on account for next year — so my true 2024-25 bill is £3,202.

'How can I reduce it?'

[SHOW: optimizations response]

Pension contributions. It's telling me that if I put £3,000 into a pension before April 5th, I reduce my taxable income below the higher rate band and save £1,200 in tax. The pension itself is worth £4,000 after basic rate relief is added. I get £1,200 in tax savings plus £1,000 in government top-up. On £3,000 invested.

This is genuinely good accountancy advice, delivered instantly.

'Can I claim for my laptop?'

It's asking me: is this exclusively for business use? If yes — 100% capital allowance, full deduction in year of purchase. If mixed use — you need to estimate the business percentage and only claim that.

[SHOW: Wispr Flow usage]

Here's one more thing. There's an app called Wispr Flow — it lets you dictate anything, anywhere, using your voice. Install it, and you can just speak to your accountant. 'What were my biggest expenses in February?' and you get the answer. No typing required.

This is the experience. You drop files in a folder, forward your emails, export your bank statement once a month. And at any point you can ask your accountant anything and get a real answer based on your real data."

---

# SECTION 10: DEADLINE AUTOMATION + FILING
**[64:00 — 69:00] | Screen share + phone cam**

**What to say:**
"Let's talk about the automation side.

The system has a cron scheduler — a set of timed jobs that run automatically. Let me show you the most important ones.

[SHOW: scheduler.ts with job list]

Every Monday morning at 9am, you get a Telegram message with your weekly income, expenses, and YTD totals. Small thing, but it keeps you aware of your numbers all year. No January panic.

[MANUALLY TRIGGER the 30-day warning message]

I'm going to manually trigger the 30-day warning message so you can see what it looks like.

[SHOW: Telegram message]

'30 days until your Self Assessment filing deadline. Here's where you are: estimated tax due: £6,404. Gaps: 3 transactions over £25 need receipts. Outstanding: home office calculation not set. Action: open Claude Code and run /prepare. Time: you've got 30 days. Don't leave it to the 28th.'

That's what lands on your phone on January 1st. You have 30 days, it tells you exactly what's left, and you've been doing this all year in 10-second chunks instead of six hours in January.

[SHOW: 'prepare for filing' query]

When you're ready to file, you go into Claude Code and say 'prepare me for filing.' It runs a full reconciliation — every transaction reviewed, every gap flagged, a gaps report generated, the full SA103 data exported. You take that to HMRC's online portal and it's all there.

The actual filing — pressing submit on HMRC's website — that's still something you do yourself. It takes about 20 minutes when you have all the numbers. In the future there may be ways to automate even that, but for now: you're giving the system 10 seconds a month, and filing takes 20 minutes instead of a day."

---

# SECTION 11: CUSTOMISATION + CONTRIBUTING
**[69:00 — 73:00] | Screen share**

**What to say:**
"The system is open source. Everything is on GitHub. Link in the description.

A few things you might want to customise.

**Adding your bank.** The system comes with adapters for Monzo, Starling, Barclays, HSBC and a generic CSV fallback. If yours isn't there, the CLAUDE.md makes it easy to add one — open Claude Code and say 'add a bank adapter for [your bank], here's a sample of the CSV' and paste in the headers. It'll write the adapter for you.

**Adding an exchange.** Same thing. If you're on ByBit or OKX, open Claude Code and say 'add an exchange parser for ByBit, here are the CSV headers.' Done.

**US users.** The system detects your country during setup and switches to a US tax engine — Schedule C self-employment income, federal brackets, short-term vs long-term capital gains. It's not as battle-tested as the UK version yet but it's solid for most freelancers. If you find something that needs fixing, pull requests are very welcome.

**If you want to contribute,** the repo is at the link below. Priority areas: the US tax engine, more bank adapters, VAT support for registered businesses. Come build with us.

Every person who uses this is a person not paying £300 to an accountant for something Claude Code can do in 12 minutes. The more people who use it, the better it gets."

---

# SECTION 12: CLOSE
**[73:00 — 75:00] | Talking head**

**What to say:**
"Here's the thing. Accountancy is not magic. It's data management plus some rules. And we are now living in a time where you can give an AI both the data and the rules and it does the work.

The 12 minutes you just spent setting this up will save you hours every year, and potentially hundreds of pounds in accountant fees. And more than that — you actually know your numbers now. You know what you owe. You know what you can claim. You know what pension contribution makes sense. You're not waiting for someone else to tell you.

That's what I wanted to build with this. Not just a tax tool — a way for anyone who's self-employed or freelancing to have the same quality of financial awareness that big companies have.

The GitHub is in the description. It's free. It's open source. The setup prompt is literally one copy-paste.

If you found this useful, subscribe — I'm building more of these. And if something doesn't work for your setup, leave a comment and I'll help you fix it.

See you in the next one."

**End screen:** 20s — Subscribe button + "Watch next: [next video]"

---

## Description Template

```
I built a complete AI accountancy firm using Claude Code — it runs locally on your machine, automatically processes receipts and invoices, calculates your UK taxes (or US), handles crypto CGT with the Section 104 method, and sends Telegram alerts before every filing deadline.

It costs about £2/month in API usage. One copy-paste to set up.

→ GitHub (free, open source): https://github.com/jackson-video-resources/ai-accountant

CHAPTERS:
00:00 - Hook: what this does
02:30 - The problem with tax admin
07:00 - System overview
13:00 - Prerequisites (Node.js, Claude Code, API key, Telegram)
20:00 - The one-shot setup
38:00 - Processing a real receipt
46:00 - Crypto & CGT (Section 104)
54:00 - Bank statement import
58:00 - Talking to your accountant (+ Wispr Flow)
64:00 - Deadline automation
69:00 - Customisation & contributing
73:00 - Close

LINKS:
→ Claude Code: https://claude.ai/code
→ Anthropic API: https://console.anthropic.com
→ Wispr Flow: https://wispr.flow
→ PM2: https://pm2.keymetrics.io

DISCLAIMER: This tool helps you organise your financial records and estimate your tax liability. It is not a substitute for professional advice on complex tax situations. Always verify your final numbers with HMRC before filing.
```

---

## Pinned Comment Template

```
GitHub → https://github.com/jackson-video-resources/ai-accountant

Setup:
1. Install Node.js 18+ and Claude Code
2. Get an Anthropic API key (console.anthropic.com)
3. Create a Telegram bot (@BotFather)
4. Open the SETUP-PROMPT.md file on GitHub
5. Copy → paste into Claude Code
6. Answer 8 questions

That's it. Questions? Drop them below ⬇️
```
