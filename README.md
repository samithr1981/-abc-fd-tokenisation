# ABC Bank FD Tokenisation — Local Demo

## What This Runs

A complete Fixed Deposit lifecycle on Chia's in-process blockchain simulator:

```
INR 30,00,000 FD  →  3,000,000 ABCFD-001 tokens minted
                  →  Timelock blocks early redemption
                  →  Clawback tested (bank recalls token)
                  →  365 days simulated (seconds, not real days)
                  →  Tokens burned, INR 32,32,808 paid out
```

No internet. No wallet GUI. No XCH mainnet. Runs entirely in your terminal.

---

## Prerequisites

- **OS**: macOS, Linux, or WSL2 on Windows
- **Python**: 3.10, 3.11, or 3.12
- **RAM**: 4 GB minimum
- **Disk**: 2 GB free

Check your Python version:
```bash
python3 --version   # must be 3.10+
```

---

## Step 1 — Clone the Chia Repo

```bash
git clone https://github.com/Chia-Network/chia-blockchain.git
cd chia-blockchain
```

Or if you already have the zip:
```bash
unzip chia-blockchain-main.zip
cd chia-blockchain-main
```

---

## Step 2 — Install Chia

```bash
sh install.sh
```

This installs Chia and all dependencies into a local Python virtual environment.
Takes 3–5 minutes first time.

---

## Step 3 — Activate the Environment

```bash
. ./activate
```

Your terminal prompt will change to show `(chia-blockchain)`.
You must do this every time you open a new terminal.

---

## Step 4 — Copy the Demo Script

Copy `abcbank_fd_demo.py` into the repo root:

```bash
cp /path/to/abcbank_fd_demo.py .
```

---

## Step 5 — Run the Demo

```bash
python abcbank_fd_demo.py
```

Expected runtime: **30–60 seconds**

---

## Expected Output

```
══════════════════════════════════════════════════════════════════════
  ABC BANK — FIXED DEPOSIT TOKENISATION DEMO
  Instrument: ABCFD-001  |  Principal: INR 30,00,000
  Platform: Chia Blockchain (SpendSim — in-process)
══════════════════════════════════════════════════════════════════════

─────────────────────────────────────────────────────────────────────
  PHASE 0 — SETUP
─────────────────────────────────────────────────────────────────────
  Bank pubkey     : 97f1d3a73197d7942695...
  Depositor pubkey: a572cbea904d67468870...
  ✅  [SETUP]  Simulator started + bank wallet funded  →  SUCCESS

─────────────────────────────────────────────────────────────────────
  PHASE 1 — KYC & DID CREATION
─────────────────────────────────────────────────────────────────────
  ✅  [KYC]  Depositor identity verified and DID issued  →  SUCCESS
  ✅  [KYC]  Nominee DID issued  →  SUCCESS

─────────────────────────────────────────────────────────────────────
  PHASE 2 — DEPOSIT RECEIPT & TOKEN MINTING
─────────────────────────────────────────────────────────────────────
  ABCFD-001 TAIL hash  : 3e7a91b2c4d5f6...
  Deposit block        : 3
  Maturity block       : 1463
  Principal (mojos)    : 3,000,000  (= INR 30,00,000)
  ✅  [MINT]  3,000,000 ABCFD-001 tokens minted to depositor wallet  →  SUCCESS

─────────────────────────────────────────────────────────────────────
  PHASE 3 — TIMELOCK TEST (Early Redemption Attempt)
─────────────────────────────────────────────────────────────────────
  ❌  Redemption BLOCKED — timelock active until block 1463

─────────────────────────────────────────────────────────────────────
  PHASE 4 — CLAWBACK (Regulatory Recall Scenario)
─────────────────────────────────────────────────────────────────────
  ✅  Clawback authorised — bank BLS signature valid
  ✅  Token ABCFD-001 recalled to ABC Bank wallet

─────────────────────────────────────────────────────────────────────
  PHASE 5 — ADVANCING TO MATURITY
─────────────────────────────────────────────────────────────────────
  ✅  Block height now: 1465 — past maturity block 1463

─────────────────────────────────────────────────────────────────────
  PHASE 6 — MATURITY REDEMPTION
─────────────────────────────────────────────────────────────────────
  ✅  Depositor signature VALID
  ✅  Timelock condition satisfied
  ✅  3,000,000 ABCFD-001 tokens BURNED
  ✅  INR 32,32,808 credited to depositor savings account

══════════════════════════════════════════════════════════════════════
  PILOT SUMMARY
══════════════════════════════════════════════════════════════════════
  Instrument       : ABCFD-001
  Principal        : INR 30,00,000
  Maturity payout  : INR 32,32,808
  Final FD status  : REDEEMED
  All stages completed successfully. ✅
══════════════════════════════════════════════════════════════════════
```

---

## Troubleshooting

**`ModuleNotFoundError: No module named 'chia'`**
→ You forgot to activate: `. ./activate`

**`python3 --version` shows 3.9 or lower**
→ Install Python 3.10+: `brew install python@3.11` (Mac) or `sudo apt install python3.11` (Ubuntu)

**`sh install.sh` fails with cmake error**
→ Install build tools: `sudo apt install build-essential cmake` (Ubuntu) or Xcode CLI tools (Mac)

**Script hangs at "Farming blocks..."**
→ Normal — it's farming 1,460 blocks. Wait 30–60 seconds.

---

## What Each Phase Maps to in the Chia Repo

| Phase | What Runs | Chia Module |
|-------|-----------|-------------|
| Setup | SpendSim in-process node | `chia/simulator/` |
| KYC/DID | Key generation + DataLayer write | `chia/wallet/did_wallet/` |
| Minting | XCH coin wrapped into CAT | `chia/wallet/cat_wallet/cat_utils.py` |
| Timelock | ASSERT_HEIGHT_ABSOLUTE guard | `chia/wallet/conditions.py` |
| Clawback | Bank BLS signature recall | `chia/wallet/puzzles/clawback/` |
| Maturity | Depositor sig + height check | `chia/consensus/condition_tools.py` |
| DataLayer | Key-value FD record store | `chia/data_layer/data_store.py` |
