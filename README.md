# Privacy-Preserving AI Bounty Judge

> **Ritual Chain Academy — Session 2 Assignment**
> A decentralized bounty system where AI judges submissions fairly — without participants copying each other's answers.

![Solidity](https://img.shields.io/badge/Solidity-0.8.24-363636?logo=solidity)
![Ritual Chain](https://img.shields.io/badge/Ritual%20Chain-1979-indigo)
![License](https://img.shields.io/badge/License-MIT-green)

## Overview

Two privacy tracks to secure the bounty process:

| Track | Contract | Privacy | Chain |
|-------|----------|---------|-------|
| **Required** — Commit-Reveal | `AIJudge.sol` | Hash commitments + delayed reveal | Any EVM |
| **Advanced** — Ritual TEE | `PrivacyBountyJudge.sol` | ECIES-encrypted to TEE, never exposed | Ritual Chain |

## Lifecycle

```
Phase 1: submitCommitment()  → Store keccak256(answer, salt, sender, bountyId)
Phase 2: revealAnswer()      → Verify hash, store plaintext
Phase 3: judgeAll()          → Batch LLM call via 0x0802 precompile
Phase 4: finalizeWinner()    → Send reward to winner
```

## Deployed Contract

| Network | Address | TX Hash |
|---------|---------|---------|
| Ritual Chain (1979) | `0x860A5cAf7a76F33c5Dc6CC8F49f2307482770B6F` | `0xd64d3bc4108589fc5e9c191d1d1dbaa55c41b01a626f473e14d4acffb496d552` |

## Repository Structure

```
📦 ritual-chain-workshop
├── 📄 ARCHITECTURE.md          — System architecture & design decisions
├── 📄 TEST_PLAN.md             — 30 test cases
├── 📁 hardhat/                 — Smart contracts + deployment
│   ├── contracts/
│   │   ├── AIJudge.sol         — Commit-reveal bounty (Required Track)
│   │   ├── PrivacyBountyJudge.sol — TEE-encrypted (Advanced Track)
│   │   └── utils/
│   └── README.md               — Full lifecycle documentation
└── 📁 web/                     — Frontend (Next.js + wagmi)
```

## Reflection

> *"What should be public, what should stay hidden, and what should be decided by AI versus by a human in a bounty system?"*

In a fair bounty system, **public data** should include the judging rubric, reward amount, deadlines, commitment hashes (proving timestamps without revealing content), and the final winner. These build trust without compromising fairness. **Hidden data** includes answer content during the submission window — in commit-reveal, answers stay hidden until the reveal phase; in the TEE-encrypted model, answers are permanently hidden from all human eyes as the LLM decrypts them inside the enclave. **AI decisions** are best for evaluating merit against a rubric: comparing all submissions in a single batch call and producing a structured ranking with reasoning. **Human decisions** should govern the system's boundaries: setting the rubric, defining the reward, choosing deadlines, and making the **final call** on the winner. The LLM provides a recommendation, but the bounty owner inspects the reasoning and manually calls `finalizeWinner` — preventing edge cases where the AI hallucinates a non-existent submission or misapplies the rubric. Humans set the rules; AI evaluates within them.
