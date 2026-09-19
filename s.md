# 5-Minute YouTube Walkthrough Video Script
### *Simplify Money — Software Engineer / Intern (Backend, Java) Assignment*
**Target Duration:** Exactly 4 minutes 45 seconds (Under the 5-minute ceiling)  
**Tone:** Confident, concise, technical, professional  
**Recording Mode:** Screen share + Camera bubble (or clear voiceover)

---

## Pre-Recording Checklist (1 Minute Setup)
1. **Terminal:** Clear the terminal screen (`clear` or `cls`). Increase font size to 16px–18px for clear readability.
2. **VS Code / IDE:** Have the following files open in tabs:
   - `src/main/java/in/simplifymoney/ledgersync/parse/Amounts.java`
   - `fixtures/corpus-a-totals.json`
   - `submission/summary.json`
   - `submission/reconciliation.json`
   - `src/main/java/in/simplifymoney/ledgersync/store/IndexedDocumentStore.java`
3. **Audio:** Test microphone; ensure no background noise.

---

## Master Video Timeline Overview

| Timestamp | Phase / Topic | Screen Focus | Key Action / Command |
|---|---|---|---|
| **0:00 – 0:40** | **1. Introduction & Boot** | Terminal | `docker compose up -d` & `.\gradlew.bat selfCheck` |
| **0:40 – 1:30** | **2. Ingestion & Reports** | Terminal & File Explorer | Ingest `corpus-a.jsonl`, generate `submission/*.json` |
| **1:30 – 2:25** | **3. Reconciliation Insight** | `summary.json` & `reconciliation.json` | Explain the unevidenced ₹7,500 bank drop on `4821` |
| **2:25 – 3:15** | **4. Incident INC-2026-09-11** | `Amounts.java` & `AmountsTest.java` | Show the integer regex fix & blast radius |
| **3:15 – 4:15** | **5. Document Store & 100k** | `IndexedDocumentStore.java` & `README.md` | Show 1:1 ratio (6 metrics), Backfill & ConsistencyChecker |
| **4:15 – 4:45** | **6. Test Suite & Wrap-Up** | Terminal | Run `.\gradlew.bat test` (all 20 green) & closing statement |

---

# Word-for-Word Recording Script

---

### [0:00 – 0:40] Phase 1: Introduction & System Boot

**[Visual:]**  
Full-screen terminal.

**[Voiceover / Spoken Words:]**  
> *"Hi everyone, my name is Shivanagu Reddy, and this is my walkthrough for the Simplify Money Backend Engineering assignment. Today, I'll demonstrate the complete lifecycle of our ledger synchronization service: from zero-dependency boot, to real-world corpus ingestion, incident resolution, honest reconciliation, and our 100,000-scale document store migration.*
>
> *Let's start by bringing up our containerized document store with one command:"*

**[Action in Terminal:]**
```bash
docker compose up -d
```
*(Wait 2 seconds for MongoDB container to show started)*

**[Voiceover / Spoken Words:]**  
> *"MongoDB 7.0 is now live on port 27017. Now, let's run the zero-dependency verification script. This compiles and validates the engine using pure JDK 21 standard libraries, with zero external jars or network calls."*

**[Action in Terminal:]**
```bash
.\gradlew.bat selfCheck
```
*(Or `./verify.sh` if in Git Bash)*

**[Voiceover / Spoken Words:]**  
> *"As you can see, it reads all 522 messages, compiles cleanly, and outputs our initial reconciliation status."*

---

### [0:40 – 1:30] Phase 2: Ingestion & Generating the 3 Output Files

**[Visual:]**  
Terminal.

**[Voiceover / Spoken Words:]**  
> *"Now let's run the production pipeline across our database migration, corpus ingestion, and report generation."*

**[Action in Terminal:]**
```bash
# 1. Apply database migrations
.\gradlew.bat run --args="migrate"

# 2. Ingest corpus-a
.\gradlew.bat run --args="ingest fixtures/corpus-a.jsonl"

# 3. Generate the 3 submission reports
.\gradlew.bat run --args="report submission/"
```

**[Voiceover / Spoken Words:]**  
> *"Notice that the pipeline read 522 messages, safely filtered out 41 non-transactional messages—including phishing SMS from VK-ICICIB, delivery updates, card OTPs, and loan promos—and wrote 256 real transactions. It is also completely idempotent: re-running ingestion produces zero duplicate rows.*
>
> *Three deterministic JSON files have been generated inside the `submission/` directory: `ledger.json`, `summary.json`, and `reconciliation.json`."*

---

### [1:30 – 2:25] Phase 3: Reconciliation & The ₹7,500.00 Insight

**[Visual:]**  
Switch to IDE: Open `submission/summary.json` on the left and `submission/reconciliation.json` on the right.

**[Voiceover / Spoken Words:]**  
> *"Now let's examine the numbers. Looking at `summary.json`:*
> - *Account `9075` (ICICI) reconciles to the paisa: 91 transactions, and the ledger balance matches the bank's closing balance of ₹51,210.63 with exactly 0.00 difference.*
> - *Credit Card `3310` accounts for all 20 transactions totaling ₹23,941.15 in spend.*
> - *Self-transfers between 4821 and 9075 are symmetrically paired within a 10-minute window: ₹25,000 out of 4821 and into 9075; ₹6,000 out of 9075 and into 4821.*
>
> *Now, notice Account `4821`: The totals file expects 146 transactions, but our ledger produces 145. Why?*
>
> *An audit of the raw corpus reveals that on July 29th, between 11:53 AM and 5:06 PM, the bank's stated balance dropped by ₹7,575.00, but only one ₹75.00 tea payment SMS was delivered to the device. Exactly ₹7,500.00 left the user's bank account with no evidencing SMS or email in the corpus.*
>
> *Because `NormalizedTxn` strictly mandates message evidence, fabricating a phantom transaction is prohibited. Rather than faking data to force the numbers to match, our `reconciliation.json` explicitly detects, flags, and explains this exact ₹7,500.00 divergence."*

---

### [2:25 – 3:15] Phase 4: Incident INC-2026-09-11 (The ₹92,213.10 Water Can Bug)

**[Visual:]**  
Switch to `src/main/java/in/simplifymoney/ledgersync/parse/Amounts.java`. Highlight lines 17–23.

**[Voiceover / Spoken Words:]**  
> *"Next is the live production incident, INC-2026-09-11, where a customer spent ₹5 on a water can and was shown a debit of ₹92,213.10.*
>
> *Here was the root cause: The original regex in `Amounts.java` strictly required two decimal places (`\\.[0-9]{2}`). When the SMS arrived stating `Rs.5 debited... Avl Bal: Rs.92,213.10`, the parser skipped integer `Rs.5` and captured the customer's Available Balance as the spend amount!*
>
> *The blast radius across `corpus-a` affected 44 integer messages: 38 transactions were corrupted with balance figures, and 2 valid transactions without balances were completely dropped.*
>
> *We fixed this by updating the pattern to make decimals optional with scale 2 (`(?:Rs\.?|INR)\s*([0-9,]+(?:\.[0-9]{2})?)`), and stripping trailing balance contexts before extraction. We added regression unit tests in `AmountsTest.java`, and published our 5-line incident note in `incident/INC-2026-09-11.md`."*

---

### [3:15 – 4:15] Phase 5: Document Store Migration & 100,000 Scale Benchmarks

**[Visual:]**  
Switch to `README.md` (Table of 6 Metrics) and `src/main/java/in/simplifymoney/ledgersync/store/IndexedDocumentStore.java`.

**[Voiceover / Spoken Words:]**  
> *"For Task 4, we migrated the ledger from SQL to MongoDB 7.0. We designed our document model to eliminate full-collection scans across all 3 query contracts.*
>
> *Here are our 6 benchmark numbers at 100,000 transactions:*
> 1. *For **Q1 (`forAccountMonth`)**: We use a compound index on `account_last4`, `month`, and `occurred_at DESC`. Total documents examined: 75; returned: 75. A perfect 1:1 ratio with zero in-memory sorting.*
> 2. *For **Q2 (`categoryTotals`)**: We maintain a materialized summary document in the `account_totals` collection. Total documents examined: 1; returned: 1. Constant time O(1) lookups.*
> 3. *For **Q3 (`byMessageId`)**: We index the `source_message_ids` array using a multikey index. Total documents examined: 1; returned: 1.*
>
> *We also implemented `Backfill.java`, which uses deterministic SHA-256 keys to deduplicate dirty legacy SQL seeds from `V2__seed.sql` and merge message IDs idempotently, and `ConsistencyChecker.java`, which performs deep field-level comparisons to catch tampered values."*

---

### [4:15 – 4:45] Phase 6: Test Suite & Closing

**[Visual:]**  
Switch back to terminal.

**[Voiceover / Spoken Words:]**  
> *"Finally, let's run the complete test suite:"*

**[Action in Terminal:]**
```bash
.\gradlew.bat test
```
*(Wait 15–20 seconds as tests run and output `BUILD SUCCESSFUL`)*

**[Voiceover / Spoken Words:]**  
> *"All 20 tests pass with 0 failures: including our amount regression tests, backfill idempotency, tampering detection, frozen contract tests, and the 100k benchmark.*
>
> *Our git history has 6 clean, progressive commits, our README includes a 10-point decision log and AI disclosure, and Tasks 0 and 1 are fully documented in our teardown and feedback reports.*
>
> *Thank you for watching, and I look forward to the next round!"*

---

## Video Recording Tips & Pro-Advice

1. **Practice Once Before Recording:** Run through the commands once so Gradle caches the daemon and commands execute fast.
2. **Resolution:** Record at **1080p (1920x1080)** at 30 or 60 fps.
3. **Cursor Visibility:** Enable mouse click highlights in Loom / OBS so evaluators can easily follow what you click.
4. **Publishing on YouTube:** Set privacy to **Unlisted** (not Private, not Public) and test the link in an incognito window before sending.
