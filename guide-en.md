<div align="center">

# 🔐 Asentum Testnet Full Node & Validator Setup Guide

**A complete guide to running an Asentum testnet node and registering as a validator**

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04+-E95420?style=flat-square&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Asentum](https://img.shields.io/badge/Asentum-Testnet-6C4DF6?style=flat-square)](https://www.asentum.com)
[![Chain ID](https://img.shields.io/badge/Chain%20ID-1423-blue?style=flat-square)](https://www.asentum.com/docs/reference/network-parameters)
[![Release](https://img.shields.io/badge/Validator%20release-0.8.4-green?style=flat-square)](https://explorer.asentum.com/install/validator-release.json)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[hazennetworksolutions.com](https://hazennetworksolutions.com)

</div>

---

> **Author:** HazenNetworkSolutions
> **Network:** Asentum Testnet (Chain ID: `1423` / `0x58f`)
> **Node software:** `asentum-validator` (VPS installer, release `0.8.4`) / `asentum` CLI
> **Last Updated:** October 2026

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
- [Updating the Node (v0.8.4)](#updating-the-node-v084)
- [Memory / Heap Limit (shared servers)](#memory--heap-limit-shared-servers)
- [Unbonding](#unbonding)
- [Slashing Risks](#slashing-risks)
- [Troubleshooting](#troubleshooting)
- [Firewall](#firewall)
- [Legal Note](#legal-note)
- [About the Author](#about-the-author)

---

## Hardware Requirements

| Component | Minimum | Recommended |
|---|---|---|
| OS | Ubuntu 22.04+ / Debian 12+ | Ubuntu 24.04 |
| CPU | 2 cores | 4 cores |
| RAM | 4 GB | 8 GB |
| Disk | 40 GB SSD | 80 GB+ SSD (block store grows ~1–2 GB/day) |
| Network | Broadband | 50 Mbps+ |

---

## Network Endpoints

| Type | Endpoint |
|---|---|
| Chain ID | `1423` (`0x58f`) |
| RPC / Explorer | https://explorer.asentum.com (also served at https://testnet.asentum.com) |
| Docs | https://www.asentum.com/docs |
| Validators | https://explorer.asentum.com/validators |
| Faucet (airdrop dashboard) | https://airdrop.asentum.com |
| Telegram | https://t.me/asentum |

Check the live chain ID yourself:

```bash
curl -s https://explorer.asentum.com -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_chainId","params":[]}'
# {"jsonrpc":"2.0","id":1,"result":"0x58f"}
```

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

⚠️ The installer is a plain bash script that pulls the validator bundle, chain snapshot and the `asentum-validator` command from `explorer.asentum.com/install/` (same files as `testnet.asentum.com/install/`). Always download and read a script before piping it to `bash`:

```bash
curl -fsSL https://explorer.asentum.com/install/validator -o /tmp/asentum-install.sh
less /tmp/asentum-install.sh
sudo bash /tmp/asentum-install.sh
```

This is a testnet — don't point real funds or a shared production box at it without isolating it (separate user/VM). On a shared server read [Memory / Heap Limit](#memory--heap-limit-shared-servers) first.

---

## Step 4 — VPS One-Liner Install

What it does: installs curl, jq, ufw and Node.js 22, downloads the validator bundle + a light chain snapshot, generates a Dilithium3 validator key (`/opt/asentum/data/validator.key` + `validator.key.hex`), configures ufw, sets up the `asentum-validator` systemd service, syncs, funds the bond automatically and bonds.

```bash
curl -fsSL https://explorer.asentum.com/install/validator -o /tmp/asentum-install.sh
less /tmp/asentum-install.sh
sudo bash /tmp/asentum-install.sh
```

Paths after install:

| Path | Content |
|---|---|
| `/opt/asentum/chain` | Node code + `.env.testnet` (your config) |
| `/opt/asentum/data` | Chain data **and** `validator.key` / `validator.key.hex` |
| `/opt/asentum/VERSION` | Installed release |
| `/etc/systemd/system/asentum-validator.service` | systemd unit |

> ⚠️ The installer (0.8.4) also writes a heap drop-in that gives Node ~70% of the machine's RAM — see [Memory / Heap Limit](#memory--heap-limit-shared-servers).

Installing with a wallet you already have (moving to a new server): `ASENTUM_IMPORT_KEY_HEX=ask` asks for the hex key with hidden input. Stop the old validator **before** the new one starts — the same key on two servers is a double-sign.

```bash
curl -fsSL https://explorer.asentum.com/install/validator | ASENTUM_IMPORT_KEY_HEX=ask bash          # as root
curl -fsSL https://explorer.asentum.com/install/validator | sudo ASENTUM_IMPORT_KEY_HEX=ask bash     # as a sudoer
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
asentum-validator version
```

Cross-check the height at [explorer.asentum.com](https://explorer.asentum.com):

```bash
curl -s http://127.0.0.1:8545 -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'
```

---

## Step 7 — Wallet & Faucet

```bash
asentum-validator balance
```

- **Validator onboarding:** the VPS installer funds the bond automatically (one-time drip per address) and bonds — no manual faucet step.
- **Manual faucet:** sign in at [airdrop.asentum.com](https://airdrop.asentum.com), link your wallet, click *Faucet drip* — 5 test ASE per drip, 1 request / 60 min per IP and per wallet. There is no faucet page on the explorer.
- 5 ASE covers gas, not a bond.

---

## Step 8 — Bond as a Validator

The installer auto-bonds via `bond(pubKey)` on the staking contract.

- Min self-bond: `500 ASE` (chain-enforced) — makes you a **candidate**, not a signer.
- Max self-bond: `50,000,000 ASE`, plus a 10% per-validator cap on each block's rewards.
- Delegation is not live on testnet — self-bond only.
- Activation: candidate ~10 blocks after bonding.
- Add stake later:

```bash
asentum-validator bond 500     # or: bond all (keeps 1 ASE for gas)
```

---

## Step 9 — Confirm Active Committee

Signing needs a seat in the **100-seat signing committee**, refilled every epoch (`300` blocks, ~10 min at 2s blocks; longer when blocks are slower):

- **50 core seats** — the 50 largest stakes.
- **50 reserve seats** — stake-weighted lottery among the remaining candidates, drawn each epoch.

Below the core cutoff you sign only in the epochs where the lottery picks you; status shows `waiting` otherwise. The cutoff moves as people bond — check it on [explorer.asentum.com/validators](https://explorer.asentum.com/validators). Your node auto-submits an entry transaction each epoch window.

```bash
asentum-validator status
curl -s https://testnet.asentum.com/validators | jq '.validators[] | select(.address=="YOUR-ASE-ADDRESS") | {status, committeeState, stake, consecutiveMisses, lastVotedHeight}'
```

Messages like `[vote-ineligible] ... voterInCommittee=false` in the logs only mean you are not in the committee for that round.

Votes and blocks travel over port `9000`; other validators do **not** need your `8545`.

---

## Monitoring

```bash
asentum-validator status
asentum-validator balance
asentum-validator earnings
asentum-validator logs
journalctl -u asentum-validator -f --no-pager
```

A height that does not move for minutes is a stall — see [Troubleshooting](#troubleshooting).

---

## Useful Commands

```bash
asentum-validator restart
asentum-validator version            # installed release, node build, CLI version
asentum-validator update --check     # report only, touches nothing
asentum-validator update             # install latest release, keeps key, .env, bond
asentum-validator resync             # fresh snapshot only (node stuck at one height)
asentum-validator self-update        # update only the CLI (also writes the heap drop-in!)
asentum-validator bond <ase|all>
asentum-validator unstake <ase>      # queue unbond (1,296,000 blocks)
asentum-validator withdraw           # claim matured unbonds
asentum-validator rpc status         # who can use your 8545 (public / private <ips> / local)
asentum-validator firewall --fix     # remove rules that block 9000/8545 ahead of ufw
asentum-validator keys               # where the key file lives (do not export on shared boxes)
sudo systemctl status asentum-validator
```

---

## Updating the Node (v0.8.4)

Latest validator release: **`0.8.4`** (published 29 Oct 2026, chain `1423`). Official update flow:

```bash
asentum-validator version
asentum-validator update --check
sudo asentum-validator update
asentum-validator version
asentum-validator status
```

What `update` does:

1. Downloads the bundle and checks it before touching anything; does nothing if you already run it.
2. Stops the service, installs the new chain code — **`.env*` files are excluded**, your `.env.testnet` stays as it is.
3. Keeps your chain data, **unless** the chain changed (reset / new genesis), the release requires it, there is no local data, or the node is **>3000 blocks behind** — then it restores a fresh snapshot (`validator.key` always kept).
4. Starts the service, records `/opt/asentum/VERSION`, refreshes the `asentum-validator` command.

Notes:

- On a chain reset `update` changes only the `ASENTUM_CHAIN_ID` line in `.env.testnet` (the old file is kept next to it).
- When `update` restores a snapshot it **deletes** the old chain data. Want to keep it? Run `resync` first (it moves the old data aside), then `update`.
- `update` also writes the heap drop-in — read the next section before running it on a shared server.
- Backing up config before an update never hurts:

```bash
sudo cp -p /opt/asentum/chain/.env.testnet /opt/asentum/chain/.env.testnet.bak-$(date +%Y%m%d)
```

---

## Memory / Heap Limit (shared servers)

Since 0.8.4 the installer, `update`, `self-update` and `heap` write a systemd drop-in that raises Node's heap to **~70% of RAM** (min 3072 MB, max 12288 MB):

```
/etc/systemd/system/asentum-validator.service.d/10-heap.conf
Environment=NODE_OPTIONS=--max-old-space-size=<70% of RAM>
```

On a dedicated box that's fine. On a **shared server** (other nodes running) it can let Asentum eat the RAM the others need. Cap it with a `NODE_OPTIONS` line in `.env.testnet` — `EnvironmentFile=` wins over the drop-in's `Environment=`, and `update` never overwrites `.env*`:

```bash
cat /etc/systemd/system/asentum-validator.service.d/10-heap.conf 2>/dev/null
grep NODE_OPTIONS /opt/asentum/chain/.env.testnet

# example: cap heap at 5 GB
grep -q '^NODE_OPTIONS=' /opt/asentum/chain/.env.testnet || \
  echo 'NODE_OPTIONS=--max-old-space-size=5120' | sudo tee -a /opt/asentum/chain/.env.testnet
sudo systemctl restart asentum-validator
free -h
```

Pick the value from your own free RAM; below the ~4 GB Node default the node can hit `JavaScript heap out of memory`.

---

## Unbonding

```bash
asentum-validator unstake <ase>
asentum-validator withdraw     # after the window
```

- Window: **1,296,000 blocks** — ~30 days at 2s blocks; with recent 4–9s blocks roughly 2–3 months.
- Cannot be cancelled; unbonding ASE earns nothing. Matured ASE must be claimed with `withdraw`.
- A partial unbond must leave at least 500 ASE bonded.
- Not getting a committee seat? Unbonding makes it worse — bond more instead.

---

## Slashing Risks

| Offense | Penalty (testnet today) | Tombstone |
|---|---|---|
| Double-signing | Removed from the active set; no stake burned (burning planned) | Yes (permanent) |
| Prolonged downtime | Missed rewards only (penalty planned) | No |

- One signing key, one machine — never run it twice (no HA failover).
- `validator.key` lives in `/opt/asentum/data/` — the same directory as the chain data. Never delete that directory, never copy it to another server and start it.
- Back up the validator key (`/opt/asentum/data/validator.key` / `.hex`) offline, separately from the wallet phrase.
- Alert on missed blocks / peer count / sync lag.

---

## Troubleshooting

**Waiting / not signing after bonding** — compare your stake with the committee cutoff, see Step 9. Above the cutoff a couple of epochs' wait is normal.

**"not in set" after update** — status lags during re-sync (1–5 min), re-check or verify on-chain.

**"Downloading validator bundle" fails** — truncated download, wait ~10 min and retry.

**Node metadata read error** — check `journalctl -u asentum-validator`.

**Node stuck at one height / far behind / `REJECT unverifiable peer block`** — a restart won't fix diverged data. Use the official resync:

```bash
sudo asentum-validator resync
asentum-validator status
```

`resync` downloads a fresh snapshot, stops the service, **moves** the current chain data to `/root/asentum-resync-<timestamp>` (only `blocks state v3 epoch-committee genesis-spec.json liveness-state.json`), keeps your own vote record, and leaves `validator.key` and `.env.testnet` untouched. Delete the old copy once the node is healthy. Doing an update at the same time: `asentum-validator update --resync`.

> ⛔ **Do NOT `rm -rf /opt/asentum/data`.** `validator.key` and `validator.key.hex` live there — delete it and your validator identity is gone. Never copy `data/` (or the key) to another machine and start it: two nodes with one key = double-sign.

**Chain reset / hard fork** — run `asentum-validator update` (it detects the new chain ID / genesis and restores the snapshot).

**Ports closed on Oracle Cloud-style images** — `sudo asentum-validator firewall --fix`.

**`JavaScript heap out of memory`** — see [Memory / Heap Limit](#memory--heap-limit-shared-servers).

---

## Firewall

The installer configures ufw itself: `22` (SSH), `8545` (RPC, public by default), `9000` (block/vote gossip, open only to the network's core nodes).

```bash
sudo ufw allow 8545/tcp comment "asentum RPC/broadcast"
sudo ufw status
asentum-validator rpc private <your-ips>   # optional: restrict 8545, signing is not affected
```

---

## Legal Note

Testnet only, no monetary value. Technical documentation, not financial/legal advice.

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
