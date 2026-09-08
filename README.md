<div align="center">

# 🔐 Asentum Testnet — Full Node & Validator Setup Guide

**A complete guide to running an Asentum testnet node and registering as a validator**
*Post-quantum (Dilithium3) signatures, VPS one-liner install, staking/bonding, and monitoring — step by step.*

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04+-E95420?style=flat-square&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Asentum](https://img.shields.io/badge/Asentum-Testnet-6C4DF6?style=flat-square)](https://www.asentum.com)
[![Chain ID](https://img.shields.io/badge/Chain%20ID-1418-blue?style=flat-square)](https://www.asentum.com/docs/reference/network-parameters)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[hazennetworksolutions.com](https://hazennetworksolutions.com)

</div>

---

> **Author:** HazenNetworkSolutions
> **Network:** Asentum Testnet (Chain ID: `1418` / `0x58a`)
> **Node type:** Consensus validator (bonded full node)
> **Last Updated:** September 2026

---

## 📚 Guides

| Type | Language | Link |
|------|----------|------|
| Full Node & Validator Setup | 🇬🇧 English | [guide-en.md](guide-en.md) |

---

## 📋 Overview

Asentum is a post-quantum, JavaScript-native Layer-1 blockchain: every consensus signature (block proposals, prevotes/precommits) uses NIST FIPS 204 ML-DSA-65 (Dilithium3) instead of ECDSA, smart contracts are plain, immutable JavaScript source (no compiler, no upgrade key), and the network is explicitly tuned to run on consumer hardware — a Raspberry Pi 4 is the stated supported floor. The native coin is `ASE` (18 decimals, smallest unit wei).

The public testnet (`testnet.asentum.com`, chain ID `1418`) is live, producing blocks under BFT-style consensus. Anyone can run a validator today via the desktop app, a VPS one-liner, or the standalone CLI; building from source is not yet available since the chain repository is still private pre-mainnet.

- Official Site / Docs: [asentum.com/docs](https://www.asentum.com/docs)
- Public RPC / Explorer: [testnet.asentum.com](https://testnet.asentum.com)
- Faucet: built into the explorer (paste address, 100 test ASE, 1/minute/IP) and into the desktop app / VPS installer
- Node Types Reference: [Node Types](https://www.asentum.com/docs/getting-started/node-types)
- Validator Guide (official): [Run a Validator](https://www.asentum.com/docs/getting-started/validator)
- Network Parameters (official): [Reference](https://www.asentum.com/docs/reference/network-parameters)
- Telegram: [t.me/asentum](https://t.me/asentum) · X: [@asentum](https://x.com/asentum)

### Network Facts

| Field | Value |
|---|---|
| Chain ID | `1418` (`0x58a`) |
| Native token | ASE (18 decimals, base unit wei) |
| Public RPC / Explorer | `https://testnet.asentum.com` |
| Consensus (Network Parameters ref, updated 2026-09-04) | Aura (block production) + GRANDPA (finality), 2/3 stake-weighted |
| Consensus (older Run-a-Validator guide wording) | Tendermint-style BFT, liveness-aware committee (active if it proposed 1 of the last 30 blocks) |
| Target block time | 2 seconds per the Network Parameters reference (marketing copy elsewhere says "5-second blocks" — unconfirmed) |
| Minimum self-bond | Docs disagree: `500 ASE` (Network Parameters ref, newer) vs `50,000 ASE` (Run a Validator guide, older) — **verify live at testnet.asentum.com/validators before bonding** |
| Unbonding window | 14 days |
| Signatures | ML-DSA-65 / Dilithium3 (post-quantum) |
| Hashing / Serialization | BLAKE3 / SSZ |
| State | Sparse Merkle Tree over LevelDB |

> ⚠️ The official docs contain unresolved contradictions between pages (minimum self-bond, exact block time, consensus family). The Network Parameters page itself says: "Placeholders are clearly marked — final values are set via protocol governance before mainnet." Re-check the live values before bonding real stake. See `guide-en.md` for a note on script/source trust as well.

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
