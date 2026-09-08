<div align="center">

# 🔐 Asentum Testnet — Full Node & Validator Setup Guide

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
> **Node type:** Consensus validator (bonded full node)
> **Last Updated:** September 2026

---

## 📚 Guides

| Type | Language | Link |
|------|----------|------|
| Full Node & Validator Setup | 🇬🇧 English | [guide-en.md](guide-en.md) |

---

## 📋 Overview

Asentum: post-quantum (Dilithium3) JavaScript L1. Native coin `ASE` (18 decimals). Public testnet live at chain ID `1418`.

- Docs: [asentum.com/docs](https://www.asentum.com/docs)
- RPC / Explorer: [testnet.asentum.com](https://testnet.asentum.com)
- Validator set: [testnet.asentum.com/validators](https://testnet.asentum.com/validators)
- Telegram: [t.me/asentum](https://t.me/asentum)

### Network Facts

| Field | Value |
|---|---|
| Chain ID | `1418` (`0x58a`) |
| Native token | ASE (18 decimals) |
| RPC / Explorer | `https://testnet.asentum.com` |
| Consensus | Aura + GRANDPA (newer ref) / Tendermint-style (older guide) — docs conflict |
| Block time | ~2s (docs conflict, some say 5s) |
| Min self-bond | `500 ASE` (new ref) vs `50,000 ASE` (old guide) — verify live |
| Unbonding | 14 days |
| Signatures | ML-DSA-65 / Dilithium3 |

> ⚠️ Official docs contradict themselves on min bond, block time, and consensus — verify live values before bonding. See `guide-en.md` for a source-trust warning on the install scripts.

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
