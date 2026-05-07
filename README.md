<div align="center">

<img src="https://raw.githubusercontent.com/danisampump/sampump-protocol/main/assets/header_banner.svg" alt="SamPump Protocol" width="860" />

<br/>

<img src="https://raw.githubusercontent.com/danisampump/sampump-protocol/main/assets/logo.png" alt="SamPump" height="70" />

[![Website](https://img.shields.io/badge/website-sampump.com-7c3aed?style=flat-square)](https://sampump.com)
[![Docs](https://img.shields.io/badge/docs-docs.sampump.com-2563eb?style=flat-square)](https://docs.sampump.com)
[![Discord](https://img.shields.io/badge/discord-join-5865F2?style=flat-square&logo=discord&logoColor=white)](https://discord.gg/AznjbUWb)
[![Twitter](https://img.shields.io/badge/twitter-@SamPumpofficial-1d9bf0?style=flat-square&logo=x)](https://x.com/SamPumpofficial)
[![Solana](https://img.shields.io/badge/solana-mainnet-9945FF?style=flat-square)](https://solana.com)
[![Built with Anchor](https://img.shields.io/badge/built_with-anchor-FC822E?style=flat-square)](https://anchor-lang.com)
[![License: MIT](https://img.shields.io/badge/license-MIT-22c55e?style=flat-square)](LICENSE)
[![Version](https://img.shields.io/badge/version-v1.0.0_May_2026-gray?style=flat-square)](https://github.com/danisampump/sampump-protocol)

</div>

---

## Overview

SamPump is a permissionless token launchpad built on Solana. Every token launched through SamPump requires the creator to lock a **SOL guarantee** into the protocol's on-chain vault at launch. This guarantee is held by the smart contract — not by any admin — and mathematically ensures buyers always have an exit.

There are no admin keys. No freeze authority. No upgradeable programs. The math and the code are the only authority.

<div align="center">
<img src="https://raw.githubusercontent.com/danisampump/sampump-protocol/main/assets/problem_solution.svg" alt="Problem vs Solution" width="860" />
</div>

---

## Why SamPump

Every other launchpad on Solana lets creators walk away with buyer funds. SamPump is the only protocol where this is structurally impossible — enforced by code, not by trust.

| Feature | Other Launchpads | SamPump |
|---|:---:|:---:|
| Anti-rug SOL vault | ✗ | ✓ |
| Buyers guaranteed exit via bonding curve | ✗ | ✓ |
| LP tokens burned on migration | Partial | ✓ |
| Immutable program (no upgrade authority) | ✗ | ✓ |
| No freeze authority on tokens | ✗ | ✓ |
| Creator fee sharing on every trade | ✗ | ✓ 0.30% |
| On-chain slippage protection | ✗ | ✓ |

---

## How It Works

```
[1] Create Token   →  [2] Presale      →  [3] Bonding Curve  →  [4] Migration     →  [5] Guarantee Released
Lock SOL guarantee    Optional fixed-      Automatic price       Auto-migrate to       Creator receives
on-chain              price phase          discovery             Raydium               locked SOL
```

### 1. Creator Deposits a Guarantee

At token creation, the creator deposits a minimum SOL amount directly into the protocol vault. This amount is determined at launch and is immutable. The vault is controlled exclusively by the on-chain program — no wallet, including the creator's, can withdraw it before migration completes.

### 2. Bonding Curve Pricing

Token price is determined by a constant-product AMM formula using virtual and real reserve pools. Virtual reserves provide price stability at launch without requiring large initial liquidity.

```
Price Formula:
  price = virtualSolReserves / virtualTokenReserves

Constant-product swap invariant:
  (x + Δx) · (y − Δy) = k

Where:
  virtualSolReserves    — virtual SOL depth for price stability at launch
  virtualTokenReserves  — virtual token depth for price stability at launch
  realSolReserves       — actual SOL collected on-chain
  realTokenReserves     — actual tokens remaining in the curve
```

### 3. The Anti-Rug Guarantee

<div align="center">
<img src="https://raw.githubusercontent.com/danisampump/sampump-protocol/main/assets/guarantee.svg" alt="Anti-Rug Guarantee" width="860" />
</div>

### 4. Presale Phase (Optional)

Creators can enable a presale before opening the public bonding curve. Presale pricing is fixed and stored in the `PresaleConfig` on-chain account. Governed entirely by the same program — no off-chain component, no admin override.

### 5. Migration to Raydium

When `realSolReserves` reaches the migration threshold, the protocol automatically:

- → Burns remaining bonding curve tokens
- → Creates a Raydium liquidity pool with the collected SOL
- → Burns all Raydium LP tokens — post-migration rug is impossible
- → Releases the creator's SOL guarantee
- → Marks the bonding curve as `isMigrated = true`

---

## Comparison

| Feature | SamPump | Competitor A | Competitor B | Competitor C |
|---|:---:|:---:|:---:|:---:|
| Anti-rug vault (creator SOL locked) | ✓ | ✗ | ✗ | ✗ |
| Immutable program (no upgrade authority) | ✓ | ✗ | ✓ | ✗ |
| LP tokens burned on migration | ✓ Automatic | ✓ Automatic | ✓ Automatic | ~ Partial |
| On-chain presale phase | ✓ | ✗ | ✗ | ✗ |
| Creator fee on every trade | ✓ 0.30% guaranteed | ~ variable (0.05–0.95%) | ✗ | ~ 0.10% fixed |
| No freeze authority | ✓ | ✓ | ✓ | ~ Varies |

---

## Core Guarantees

<div align="center">
<img src="https://raw.githubusercontent.com/danisampump/sampump-protocol/main/assets/core_guarantees.svg" alt="Core Guarantees" width="860" />
</div>

---

## Account Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    SamPump Program                      │
│        2q22G1KQBhzzYwE3bbNeJxoqGwNMR8ecZTS4HNPcyHhu    │
└──────────┬──────────────────┬──────────────┬────────────┘
           │                  │              │
┌──────────▼──────────┐  ┌────▼────────────┐  ┌──────────▼──────────┐
│   BondingCurve PDA  │  │ VaultAuthority  │  │  PresaleConfig PDA  │
│ [bonding_curve,mint]│  │     PDA         │  │   [presale, mint]   │
│ reserves, price,    │  │ [vault_authority│  │ fixed price, max    │
│ migration state     │  │ holds creator   │  │ tokens, sold count  │
│                     │  │ SOL (locked)    │  │                     │
└─────────────────────┘  └─────────────────┘  └─────────────────────┘
```

---

## On-Chain Program

**Program ID — Solana Mainnet**
```
2q22G1KQBhzzYwE3bbNeJxoqGwNMR8ecZTS4HNPcyHhu
```
[View on Solscan ↗](https://solscan.io/account/2q22G1KQBhzzYwE3bbNeJxoqGwNMR8ecZTS4HNPcyHhu)

| Account | Seeds | Description |
|---|---|---|
| BondingCurve | `[b"bonding_curve", mint]` | Virtual/real reserves, price state, migration status, creator info |
| PresaleConfig | `[b"presale", mint]` | Fixed presale price, max tokens, tokens sold, active flag |
| VaultAuthority | `[b"vault_authority"]` | PDA controlling the SOL guarantee vault — no external authority |
| MintAuthority | `[b"mint_authority"]` | PDA owning the token mint — revoked after creation |

---

## Security

| | |
|---|---|
| **No Admin Keys** | The program has no privileged instructions post-deploy. No wallet can override protocol behavior. |
| **No Freeze Authority** | Token accounts cannot be frozen. Buyers can always transfer or sell their tokens freely. |
| **Immutable Program** | The deployed program has no upgrade authority. What you audit is what runs — forever. |
| **On-Chain Slippage Protection** | Every swap enforces a minimum output parameter on-chain, preventing sandwich attacks. |
| **PDA-Controlled Vault** | The creator's guarantee SOL is locked in a PDA vault. Only the program can release it — on migration. |
| **LP Tokens Burned** | At migration, all Raydium LP tokens are burned. Post-migration rug is structurally impossible. |

---

## Fee Structure

| Action | Fee | Recipient |
|---|---|---|
| Buy (bonding curve) | 0.80% platform + 0.30% creator = **1.10% total** | Platform vault + creator wallet |
| Sell (bonding curve) | 0.80% platform + 0.30% creator = **1.10% total** | Platform vault + creator wallet |
| Token creation | 0.012 SOL (platform) + creator-set SOL guarantee | Platform wallet + locked in vault until migration |
| Presale purchase | 0.80% platform + 0.30% creator = **1.10% total** | Platform vault + creator wallet |
| Migration to Raydium | No fee | Automatic |

> **Note for Integrators:** The 1.10% total fee (0.80% platform + 0.30% creator) is deducted from the output amount. Slippage parameters should account for this when calculating expected outputs. All instructions are available at [docs.sampump.com](https://docs.sampump.com).

---

## Audit

> **Security Audit**
>
> Audit in progress. The program has no upgrade authority — the deployed bytecode on mainnet is final and cannot be modified. Independent audit results will be published in this repository upon completion.

---

## Documentation

Full technical documentation — bonding curve mathematics, vault mechanics, presale flow, and migration protocol — is available at [docs.sampump.com](https://docs.sampump.com).

---

## Links

<div align="center">

| | | | | | |
|:---:|:---:|:---:|:---:|:---:|:---:|
| [🚀 **Launch App**](https://sampump.com)<br/>sampump.com | [📖 **Docs**](https://docs.sampump.com)<br/>docs.sampump.com | [💬 **Discord**](https://discord.gg/AznjbUWb)<br/>discord.gg/AznjbUWb | [⬡ **On-Chain**](https://solscan.io/account/2q22G1KQBhzzYwE3bbNeJxoqGwNMR8ecZTS4HNPcyHhu)<br/>Solscan | [𝕏 **Twitter**](https://x.com/SamPumpofficial)<br/>@SamPumpofficial | [✉️ **Contact**](mailto:info@sampump.com)<br/>info@sampump.com |

</div>

---

<div align="center">

[![MIT License](https://img.shields.io/badge/license-MIT-22c55e?style=flat-square)](LICENSE)
[![Solana Mainnet](https://img.shields.io/badge/solana-mainnet-9945FF?style=flat-square)](https://solana.com)
[![Version](https://img.shields.io/badge/version-v1.0.0-gray?style=flat-square)](https://github.com/danisampump/sampump-protocol)

Built on Solana · Secured by mathematics · Launched 2026

**SamPump Protocol** — [sampump.com](https://sampump.com)

</div>
