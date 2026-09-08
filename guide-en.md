<div align="center">

# 🔐 Asentum Testnet Full Node & Validator Setup Guide

**A complete guide to running an Asentum testnet node and registering as a validator**
*VPS one-liner install, standalone CLI, wallet/faucet funding, bonding, monitoring, and slashing-risk avoidance — step by step.*

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04+-E95420?style=flat-square&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Asentum](https://img.shields.io/badge/Asentum-Testnet-6C4DF6?style=flat-square)](https://www.asentum.com)
[![Chain ID](https://img.shields.io/badge/Chain%20ID-1418-blue?style=flat-square)](https://www.asentum.com/docs/reference/network-parameters)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[hazennetworksolutions.com](https://hazennetworksolutions.com)

</div>

---

> **Author:** HazenNetworkSolutions
> **Network:** Asentum Testnet (Chain ID: `1418` / `0x58a`)
> **Node software:** `asentum-validator` (VPS installer) / `asentum` CLI (single Node SEA binary)
> **Last Updated:** September 2026

---

## Table of Contents

- [Hardware Requirements](#hardware-requirements)
- [Network Endpoints](#network-endpoints)
- [Step 1 — System Verification](#step-1--system-verification)
- [Step 2 — System Update and Dependencies](#step-2--system-update-and-dependencies)
- [Step 3 — Choose an Install Method](#step-3--choose-an-install-method)
- [Step 3A — VPS One-Liner](#step-3a--vps-one-liner-recommended-for-servers)
- [Step 3B — Standalone CLI](#step-3b--standalone-cli-manual-control)
- [Step 4 — Verify Sync](#step-4--verify-sync)
- [Step 5 — Create / Fund a Wallet (Faucet)](#step-5--create--fund-a-wallet-faucet)
- [Step 6 — Bond as a Validator](#step-6--bond-as-a-validator)
- [Step 7 — Confirm You Joined the Active Committee](#step-7--confirm-you-joined-the-active-committee)
- [Monitoring the Node](#monitoring-the-node)
- [Useful Commands](#useful-commands)
- [Slashing Risks & Operational Checklist](#slashing-risks--operational-checklist)
- [Troubleshooting](#troubleshooting)
- [Firewall](#firewall)
- [A Note on Source Trust](#a-note-on-source-trust)
- [Legal Note](#legal-note)
- [About the Author](#about-the-author)

---

## Hardware Requirements

| Component | Minimum | Recommended |
|---|---|---|
| Operating System | Ubuntu 22.04+ / Debian 12+ | Ubuntu 24.04 |
| CPU | 2-core x86_64 or ARM64 (4-core per asentum.com/nodes) | 4 cores |
| RAM | 4 GB | 8 GB |
| Disk | 10–40 GB SSD (docs disagree — plan for 40 GB) | steady-state target < 20 GB |
| Network | Broadband, ≤ 50 GB/month target | 50 Mbps+ |

Also officially supported: macOS 12+/14+ and Windows 10+ (desktop app), and a Raspberry Pi 4 as the stated hardware floor.

---

## Network Endpoints

| Type | Endpoint |
|---|---|
| Chain ID | `1418` (`0x58a`) |
| RPC / Explorer | https://testnet.asentum.com |
| Docs | https://www.asentum.com/docs |
| Faucet | via explorer UI, or automatically during VPS/desktop install (100 test ASE, 1/minute/IP) |
| Validator set (on-chain truth) | https://testnet.asentum.com/validators |
| Telegram | https://t.me/asentum |

---

## Step 1 — System Verification

```bash
lsb_release -a
uname -r
lscpu | grep -E "Model name|CPU\(s\)|Thread|Socket|Core"
free -h
df -h
```

---

## Step 2 — System Update and Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git wget htop tmux jq unzip
```

Asentum's node bundle ships as a Node.js application (Node SEA / Node 22), so the installers below fetch Node.js for you if it is missing — there is no Go/Rust/Cosmos-style toolchain to install manually.

---

## Step 3 — Choose an Install Method

Asentum documents three working methods today (a fourth, "build from source," is not yet available — the chain repository is private pre-mainnet):

| Method | Best for | Status |
|---|---|---|
| Desktop app (Asentum Operator) | Non-technical / GUI users | GUI only — not applicable to a headless VPS |
| **VPS one-liner** | Headless servers (this guide's focus) | Shipped |
| Standalone CLI (`asentum` binary) | Developers who want manual control | Shipped |
| Build from source | Full auditability | Not yet public |

### ⚠️ A note on source trust before you run anything

While researching this guide, the install one-liners embedded on `asentum.com/docs` (the VPS installer script and the CLI installer script) could not be safely auto-extracted in full — the retrieval tooling flagged embedded content in those specific code blocks as a likely prompt-injection / suspicious-script pattern. The short, repeated summary command below appears consistently across two independent official pages and is most likely legitimate, but treat any `curl | bash` as untrusted until you've read it:

```bash
curl -fsSL testnet.asentum.com/install/validator -o /tmp/asentum-install.sh
less /tmp/asentum-install.sh      # read it before running anything
sudo bash /tmp/asentum-install.sh
```

This "download, read, then run" habit is good practice for any one-liner installer — it just matters more here given what this session's tooling flagged on Asentum's own docs pages. See [A Note on Source Trust](#a-note-on-source-trust) below for more detail.

---

## Step 3A — VPS One-Liner (recommended for servers)

Per the official docs, this single script:

1. Installs Node.js 22 if not present.
2. Downloads the validator bundle (~45 MB) to `/opt/asentum/chain`.
3. Downloads a chain snapshot (~32 MB) to `/opt/asentum/data` (skips full replay from block 0; only syncs remaining blocks from peers).
4. Generates a Dilithium3 validator keypair at `/opt/asentum/data/validator-key.json`.
5. Creates and starts a systemd service (`asentum-validator`).
6. Shows a live sync progress bar until caught up (usually under a minute).
7. Polls the faucet to fund the validator address with enough ASE to bond.
8. Signs and submits a `bond(pubKey)` transaction.
9. Installs the `asentum-validator` CLI wrapper on `PATH`.

```bash
curl -fsSL testnet.asentum.com/install/validator -o /tmp/asentum-install.sh
less /tmp/asentum-install.sh
sudo bash /tmp/asentum-install.sh
```

> ℹ️ Downloaded and reviewed the script first, rather than piping `curl` straight into `bash` — see the trust note in Step 3.

---

## Step 3B — Standalone CLI (manual control)

If you would rather manage the node/wallet yourself instead of the fully-automated installer:

- The `asentum` CLI is a single self-contained Node SEA binary (~55 MB) — no separate Node.js install needed.
- Published for macOS, Linux, and Windows, pre-configured against `testnet.asentum.com`.
- Config: `~/.asentum/config.json`. Wallets: `~/.asentum/keystore/`. Chain data (only if you run a node): `~/.asentum/chaindata/`.
- The exact one-line installer text lives on the [Install via CLI](https://www.asentum.com/docs/getting-started/install-cli) doc page — apply the same "download, read, then run" advice from Step 3.

After installing, verify:

```bash
asentum --version
asentum status
```

`status` hits the configured RPC and prints the current block height, validator count, and chain id. If it errors, your machine can't reach the testnet — check DNS and firewall.

---

## Step 4 — Verify Sync

```bash
asentum-validator status
# or, if using the standalone CLI:
asentum status
```

Cross-check against the public explorer at [testnet.asentum.com](https://testnet.asentum.com).

---

## Step 5 — Create / Fund a Wallet (Faucet)

The VPS one-liner creates and funds a wallet automatically (see Step 3A, item 7). To check balance or top up manually:

```bash
asentum-validator balance
```

Manual faucet use (e.g. via the standalone CLI or explorer): paste your 20-byte Asentum address at [testnet.asentum.com](https://testnet.asentum.com) for 100 test ASE (rate-limited to once/minute/IP).

---

## Step 6 — Bond as a Validator

The VPS installer bonds automatically. The underlying on-chain call is `bond(pubKey)` against the staking system contract, attaching ASE via `msg.value`:

- **Minimum self-bond:** docs disagree — `500 ASE` per the latest Network Parameters reference, `50,000 ASE` per the older Run-a-Validator guide. Check `https://testnet.asentum.com/validators` for the live minimum before sending funds.
- **Maximum bond:** no hard cap (a 10% per-block reward cap discourages stake concentration instead of a cap).
- **Activation:** ~10 blocks after bonding.
- **Increase stake:** call `bond(pubKey)` again.

---

## Step 7 — Confirm You Joined the Active Committee

New validators are typically **dormant for ~2 hours** after bonding — the active signing committee rotates in epochs of roughly 1,440 blocks, and a fresh bond only enters the signing set in the epoch *after* its entry transaction lands. This is expected, not a bug.

```bash
asentum-validator status
```

If it has been more than ~4 hours and you are still not in the set, confirm your node's RPC/broadcast port (default `8545`) is reachable from outside:

```bash
curl http://YOUR-SERVER-IP:8545 -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'
```

If that doesn't return a JSON response, open inbound TCP `8545` on the firewall / cloud security group.

---

## Monitoring the Node

```bash
asentum-validator status     # node + validator status
asentum-validator balance    # wallet balance
asentum-validator earnings   # rewards earned
asentum-validator logs       # live log stream
journalctl -u asentum-validator -f --no-pager
```

- **Explorer:** check your validator address at [testnet.asentum.com](https://testnet.asentum.com) for proposed blocks and transaction history.
- **RPC:** `GET /validators` on your local node or the public RPC returns the full validator set and your stake.

---

## Useful Commands

```bash
asentum-validator status
asentum-validator restart      # restart the systemd service
asentum-validator update       # pull latest chain code + snapshot, restart
sudo systemctl status asentum-validator
sudo systemctl restart asentum-validator
```

`update` is the one-shot upgrade path after a new chain release: it downloads the fresh validator bundle, verifies integrity, wipes local `blocks/` + `state/`, extracts the latest snapshot, and restarts the systemd service — your `validator.key` and on-chain bond are untouched.

---

## Slashing Risks & Operational Checklist

Only two things are actually slashable:

| Offense | Penalty | Tombstone (permanent ban) |
|---|---|---|
| Double-signing (same key, two machines) | ~5% of bonded stake burned | Yes |
| Prolonged downtime during your committee rotation | ~0.01% per window | No |

**Not** slashable: brief downtime outside your rotation, being slow but on time, voting against a proposal, running older-but-compatible software, mempool/peer-gossip misbehavior (throttled, not slashed).

Operational checklist:

- One signing key, one machine — never run it twice, even briefly, for "HA failover" or "backup" reasons. BFT consensus gets no benefit from two active replicas.
- Never restore a validator onto a new box without confirming the old one is fully stopped first (kill the systemd service, unplug it if you have to).
- Never copy the data directory to another machine and start it "just to check."
- Back up the operator wallet's recovery phrase separately from the 32-byte validator signing key (`/opt/asentum/data/validator.key` and its `validator.key.hex` copy) — anyone with the key file can sign as you.
- Set up alerts on missed blocks, peer count, and sync lag.
- Test any infrastructure change on testnet before touching a live bonded validator.
- Automatic OS security patches: fine. Automatic node-software upgrades: run `asentum-validator update` manually, don't cron it blindly.

---

## Troubleshooting

### Validator bonded but showing as dormant / not signing
Normal for ~2 hours after a fresh bond (see Step 7). Check again after two block-hours; if still absent after 4 hours, check `broadcastUrl` / port `8545` reachability.

### After `asentum-validator update`, "validator status: not in set"
The status field lags while the node re-syncs the gap between the snapshot height and current head (usually 1–5 minutes). Re-check after sync catches up, or verify the on-chain truth directly at `testnet.asentum.com/validators`.

### Installer aborts at "Downloading validator bundle"
Usually a truncated/interrupted download (`gzip: stdin: unexpected end of file`). Wait ~10 minutes and retry — a fresh bundle is shipped whenever this surfaces. Persistent failures: report on Telegram (`t.me/asentum`).

### "Couldn't read validator address from node metadata"
The service likely crashed before printing its address. Run `journalctl -u asentum-validator` to see why — often a corrupted data directory from a prior failed install. Remove `/opt/asentum/data` and re-run the installer.

### Recovering after a chain reset / hard fork
Run `asentum-validator update` — it pulls the latest snapshot (containing the new genesis), wipes the local fork, and restarts. If you bonded before the reset, your bond should already be reflected on the post-reset chain; otherwise, re-run the installer from scratch.

---

## Firewall

```bash
sudo ufw allow 8545/tcp comment "asentum RPC / broadcast"
# If your install reports a different P2P/broadcast port, open that one instead.
```

---

## A Note on Source Trust

`asentum.com`'s own documentation is internally inconsistent (see the Network Facts table in `README.md` — conflicting minimum bond amounts, conflicting block times, and conflicting consensus descriptions across pages; one reference page explicitly calls its own numbers "placeholders"). Several of its "one-liner install" code blocks also could not be safely auto-extracted during research for this guide, because they were flagged by this session's tooling as containing suspicious embedded content. None of this necessarily means the project is malicious — but it does mean, in practice:

- Do not run any install script from this project blind. Always download it and read it first (Step 3).
- Treat every numeric claim (minimum stake, block time, unbonding period) as unconfirmed until verified live against `testnet.asentum.com` yourself.
- This is a **testnet**. Do not point production keys, real funds, or a primary/shared infrastructure box at it without isolating it — a separate user, a separate VM/container, or at minimum a dedicated unprivileged system account.

---

## Legal Note

Running an Asentum testnet validator is operating experimental, pre-mainnet, open-source network infrastructure — a technical service, not an investment or security. `ASE` on testnet has no monetary value. This guide is technical documentation only, not financial or legal advice.

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
