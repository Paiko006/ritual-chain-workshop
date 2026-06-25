# Architecture — Privacy-Preserving AI Bounty Judge

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         Frontend (web/)                          │
│  ┌──────────┐ ┌────────────┐ ┌──────────┐ ┌────────────────┐   │
│  │ Wallet   │ │ Bounty     │ │ Submit   │ │ Judge &        │   │
│  │ Connect  │ │ Create     │ │ Answer   │ │ Finalize       │   │
│  └──────────┘ └────────────┘ └──────────┘ └────────────────┘   │
└──────────────────────────────┬──────────────────────────────────┘
                               │ viem / wagmi
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Smart Contract (hardhat/)                      │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    AIJudge.sol                           │    │
│  │                                                         │    │
│  │  Phase 1: submitCommitment()   ← keccak256(answer,salt, │    │
│  │                                   submitter,bountyId)   │    │
│  │  Phase 2: revealAnswer()       ← verify + store answer  │    │
│  │  Phase 3: judgeAll()           ← LLM precompile (0x0802)│    │
│  │  Phase 4: finalizeWinner()     ← send reward            │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              PrivacyBountyJudge.sol                      │    │
│  │  Same phases + TEE-encrypted submission path            │    │
│  │  submitEncryptedAnswer() → ECIES to executor pubkey     │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │           PrecompileConsumer.sol (utils/)                │    │
│  │  _executePrecompile() → handles 0x0802 async unwrap     │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Ritual Chain (ID 1979)                       │
│                                                                  │
│  ┌────────────┐  ┌──────────────┐  ┌────────────────────────┐   │
│  │ Consumer   │  │ RitualWallet │  │ LLM Precompile (0x0802)│   │
│  │ Contract   │  │ (fee mgmt)   │  │ (batch judging)        │   │
│  └────────────┘  └──────────────┘  └────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

## Two Privacy Tracks

### Track 1: Commit-Reveal (any EVM chain)
- Submitter computes `commitment = keccak256(answer, salt, msg.sender, bountyId)`
- Only hash stored on-chain during submission window
- After deadline, submitter reveals answer + salt
- Contract verifies hash matches
- LLM judges only revealed answers

### Track 2: TEE-Encrypted (Ritual Chain only)
- Submitter encrypts answer via ECIES to executor's public key
- Encrypted blob stored on-chain - NEVER decrypted outside TEE
- During judgeAll, the LLM precompile decrypts inside the enclave
- No reveal phase needed - answers hidden permanently

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| `msg.sender` in commitment hash | Prevents commitment stealing / front-running |
| Batch judging (single LLM call) | More efficient than N calls, enables comparison |
| Separate reveal deadline | Prevents LLM from seeing answers mid-reveal |
| Owner finalizes (not AI) | Safety: humans validate AI recommendation |
| Dual privacy modes | One contract supports both commit-reveal and TEE |
