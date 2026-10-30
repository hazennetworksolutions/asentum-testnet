<div align="center">

# 🔐 Asentum Testnet — Full Node & Validator Setup Guide

**A complete guide to running an Asentum testnet node and registering as a validator**

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04+-E95420?style=flat-square&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Asentum](https://img.shields.io/badge/Asentum-Testnet-6C4DF6?style=flat-square)](https://www.asentum.com)
[![Chain ID](https://img.shields.io/badge/Chain%20ID-1423-blue?style=flat-square)](https://www.asentum.com/docs/reference/network-parameters)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[hazennetworksolutions.com](https://hazennetworksolutions.com)

</div>

---

> **Author:** HazenNetworkSolutions
> **Network:** Asentum Testnet (Chain ID: `1423` / `0x58f`)
> **Node type:** Consensus validator (bonded full node)
> **Validator release:** `0.8.4`
> **Last Updated:** October 2026

---

## 📚 Guides

| Type | Language | Link |
|------|----------|------|
| Full Node & Validator Setup | 🇬🇧 English | [guide-en.md](guide-en.md) |

---

## 📋 Overview

Asentum: post-quantum (Dilithium3) JavaScript L1. Native coin `ASE` (18 decimals). Public testnet live at chain ID `1423`.

- Docs: [asentum.com/docs](https://www.asentum.com/docs)
- RPC / Explorer: [explorer.asentum.com](https://explorer.asentum.com) (also served at [testnet.asentum.com](https://testnet.asentum.com))
- Validator set: [explorer.asentum.com/validators](https://explorer.asentum.com/validators)
- Faucet: [airdrop.asentum.com](https://airdrop.asentum.com)
- Telegram: [t.me/asentum](https://t.me/asentum)

### Network Facts

| Field | Value |
|---|---|
| Chain ID | `1423` (`0x58f`) |
| Native token | ASE (18 decimals) |
| RPC / Explorer | `https://explorer.asentum.com` |
| Consensus | Aura (block production) + GRANDPA (finality, 2/3 stake-weighted) |
| Block time | 2s target (real block time varies with load) |
| Epoch | 300 blocks |
| Min self-bond | `500 ASE` (chain-enforced, makes you a candidate) |
| Max self-bond | `50,000,000 ASE` |
| Signing committee | 100 seats per epoch: 50 core (largest stakes) + 50 stake-weighted lottery |
| Unbonding | 1,296,000 blocks (~30 days at 2s blocks; longer when blocks are slower) |
| Slashing | Tombstone only (double-sign); no stake burned on testnet today |
| Signatures | ML-DSA-65 / Dilithium3 |

> ⚠️ Values from the official [Network Parameters](https://www.asentum.com/docs/reference/network-parameters) page (Oct 2026). Never delete `/opt/asentum/data` — `validator.key` lives there; use `asentum-validator resync` for a stuck node. See `guide-en.md` for the v0.8.4 update flow, the heap-limit warning for shared servers, and the source-trust warning on the install scripts.

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
