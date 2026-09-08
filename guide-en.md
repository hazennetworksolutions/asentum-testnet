<div align="center">

# 🔐 Asentum Testnet Full Node & Validator Setup Guide

**A complete guide to running an Asentum testnet node and registering as a validator**

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04+-E95420?style=flat-square&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Asentum](https://img.shields.io/badge/Asentum-Testnet-6C4DF6?style=flat-square)](https://www.asentum.com)
[![Chain ID](https://img.shields.io/badge/Chain%20ID-1418-blue?style=flat-square)](https://www.asentum.com/docs/reference/network-parameters)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[hazennetworksolutions.com](https://hazennetworksolutions.com)

</div>

---

> **Author:** HazenNetworkSolutions
> **Network:** Asentum Testnet (Chain ID: `1418` / `0x58a`)
> **Node software:** `asentum-validator` (VPS installer) / `asentum` CLI
> **Last Updated:** September 2026

---

## Table of Contents

- [Hardware Requirements](#hardware-requirements)
- [Network Endpoints](#network-endpoints)
- [Step 1 — System Verification](#step-1--system-verification)
- [Step 2 — Update & Dependencies](#step-2--update--dependencies)
- [Step 3 — Source Trust Warning](#step-3--source-trust-warning)
- [Step 4 — VPS One-Liner Install](#step-4--vps-one-liner-install)
- [Step 5 — Standalone CLI (alternative)](#step-5--standalone-cli-alternative)
- [Step 6 — Verify Sync](#step-6--verify-sync)
- [Step 7 — Wallet & Faucet](#step-7--wallet--faucet)
- [Step 8 — Bond as a Validator](#step-8--bond-as-a-validator)
- [Step 9 — Confirm Active Committee](#step-9--confirm-active-committee)
- [Monitoring](#monitoring)
- [Useful Commands](#useful-commands)
- [Slashing Risks](#slashing-risks)
- [Troubleshooting](#troubleshooting)
- [Firewall](#firewall)
- [Legal Note](#legal-note)
- [About the Author](#about-the-author)

---

## Hardware Requirements

| Component | Minimum | Recommended |
|---|---|---|
| OS | Ubuntu 22.04+ | Ubuntu 24.04 |
| CPU | 2 cores | 4 cores |
| RAM | 4 GB | 8 GB |
| Disk | 10–40 GB SSD | 40 GB+ |
| Network | Broadband | 50 Mbps+ |

---

## Network Endpoints

| Type | Endpoint |
|---|---|
| Chain ID | `1418` (`0x58a`) |
| RPC / Explorer | https://testnet.asentum.com |
| Docs | https://www.asentum.com/docs |
| Validators | https://testnet.asentum.com/validators |
| Telegram | https://t.me/asentum |

---

## Step 1 — System Verification

```bash
lsb_release -a
uname -r
free -h
df -h
```

---

## Step 2 — Update & Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git wget htop tmux jq unzip
```

Asentum's node is a Node.js app (Node 22) — the installer fetches Node.js itself if missing.

---

## Step 3 — Source Trust Warning

⚠️ Asentum's own docs disagree on min bond (500 vs 50,000 ASE), block time, and consensus type. Two of its install code blocks (VPS + CLI installers) were flagged as suspicious/embedded content during research and could not be fully verified. Always download and read a script before piping it to `bash`:

```bash
curl -fsSL testnet.asentum.com/install/validator -o /tmp/asentum-install.sh
less /tmp/asentum-install.sh
sudo bash /tmp/asentum-install.sh
```

This is a testnet — don't point real funds or a shared production box at it without isolating it (separate user/VM).

---

## Step 4 — VPS One-Liner Install

What it does: installs Node.js, downloads the validator bundle + a chain snapshot, generates a Dilithium3 validator key, sets up a `asentum-validator` systemd service, syncs, funds via faucet, and auto-bonds.

```bash
curl -fsSL testnet.asentum.com/install/validator -o /tmp/asentum-install.sh
less /tmp/asentum-install.sh
sudo bash /tmp/asentum-install.sh
```

---

## Step 5 — Standalone CLI (alternative)

Single binary (~55 MB), no Node.js needed. Config: `~/.asentum/config.json`. Wallets: `~/.asentum/keystore/`. Exact installer text: [Install via CLI](https://www.asentum.com/docs/getting-started/install-cli) — same trust warning applies.

```bash
asentum --version
asentum status
```

---

## Step 6 — Verify Sync

```bash
asentum-validator status
```

Cross-check at [testnet.asentum.com](https://testnet.asentum.com).

---

## Step 7 — Wallet & Faucet

```bash
asentum-validator balance
```

Manual faucet: paste your address at [testnet.asentum.com](https://testnet.asentum.com) (100 test ASE, 1/min/IP).

---

## Step 8 — Bond as a Validator

The installer auto-bonds via `bond(pubKey)` on the staking contract.

- Min self-bond: `500 ASE` (new ref) or `50,000 ASE` (old guide) — check [validators](https://testnet.asentum.com/validators) for the live minimum.
- No max cap (10% per-block reward cap instead).
- Activation: ~10 blocks after bonding.

---

## Step 9 — Confirm Active Committee

New validators are dormant ~2 hours after bonding (committee rotates every ~1,440 blocks).

```bash
asentum-validator status
```

Still not in set after 4h? Check port `8545` is open:

```bash
curl http://YOUR-IP:8545 -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'
```

---

## Monitoring

```bash
asentum-validator status
asentum-validator balance
asentum-validator earnings
asentum-validator logs
journalctl -u asentum-validator -f --no-pager
```

---

## Useful Commands

```bash
asentum-validator restart
asentum-validator update       # upgrade bundle + snapshot, keeps key & bond
sudo systemctl status asentum-validator
```

---

## Slashing Risks

| Offense | Penalty | Tombstone |
|---|---|---|
| Double-signing | ~5% stake burned | Yes (permanent) |
| Prolonged downtime | ~0.01%/window | No |

- One signing key, one machine — never run it twice (no HA failover).
- Back up the validator key (`/opt/asentum/data/validator.key`) separately from the wallet phrase.
- Alert on missed blocks / peer count / sync lag.

---

## Troubleshooting

**Dormant after bonding** — normal for ~2h, see Step 9.

**"not in set" after update** — status lags during re-sync (1–5 min), re-check or verify on-chain.

**"Downloading validator bundle" fails** — truncated download, wait ~10 min and retry.

**Node metadata read error** — check `journalctl -u asentum-validator`; often a corrupted `/opt/asentum/data` from a failed install — remove and re-run installer.

**Chain reset / hard fork** — run `asentum-validator update`.

---

## Firewall

```bash
sudo ufw allow 8545/tcp comment "asentum RPC/broadcast"
```

---

## Legal Note

Testnet only, no monetary value. Technical documentation, not financial/legal advice.

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
