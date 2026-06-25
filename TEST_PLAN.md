# Test Plan — Privacy-Preserving AI Bounty Judge

## Coverage: 30 Test Cases

### Unit Tests (Solidity / Hardhat)

| # | Test Case | Expected Behavior | Critical? |
|---|-----------|-------------------|-----------|
| 1 | `createBounty` with valid params | Bounty created, event emitted | ✅ |
| 2 | `createBounty` with zero reward | Revert "reward required" | ✅ |
| 3 | `createBounty` with past deadline | Revert "deadline future" | ✅ |
| 4 | `submitCommitment` before deadline | Commitment stored, event emitted | ✅ |
| 5 | `submitCommitment` after deadline | Revert "deadline passed" | ✅ |
| 6 | `submitCommitment` duplicate submitter | Revert "already submitted" | ✅ |
| 7 | `submitCommitment` exceeds MAX_SUBMISSIONS | Revert "max submissions" | ✅ |
| 8 | `revealAnswer` with correct answer+salt | Answer stored, `revealed = true` | ✅ |
| 9 | `revealAnswer` with wrong answer | Revert "commitment mismatch" | ✅ |
| 10 | `revealAnswer` before submission deadline | Revert "not reveal phase" | ✅ |
| 11 | `revealAnswer` after reveal deadline | Revert "reveal active" | ✅ |
| 12 | Multiple reveal attempts (same user) | First succeeds, second reverts | ✅ |
| 13 | `judgeAll` with zero revealed submissions | Revert "no valid submissions" | ✅ |
| 14 | `judgeAll` before reveal deadline | Revert "reveal active" | ✅ |
| 15 | `judgeAll` called by non-owner | Revert "not bounty owner" | ✅ |
| 16 | `judgeAll` with LLM success | `judged = true`, `aiReview` set | ✅ |
| 17 | `finalizeWinner` with valid index | Reward transferred, event emitted | ✅ |
| 18 | `finalizeWinner` before judging | Revert "not judged yet" | ✅ |
| 19 | `finalizeWinner` invalid index | Revert "invalid index" | ✅ |
| 20 | `finalizeWinner` double-finalize | Revert "already finalized" | ✅ |

### TEE-Encrypted Track Tests

| # | Test Case | Expected Behavior | Critical? |
|---|-----------|-------------------|-----------|
| 21 | `submitEncryptedAnswer` before deadline | Payload stored, event emitted | ✅ |
| 22 | `submitEncryptedAnswer` duplicate submitter | Revert "already submitted" | ✅ |
| 23 | `judgeAll` with mixed commit-reveal + TEE | Both processed in single LLM call | ⚡ |
| 24 | TEE answer never exposed via view functions | `getSubmission` returns 0 for answer | ✅ |

### Integration Tests (Ritual Chain)

| # | Test Case | Notes |
|---|-----------|-------|
| 25 | Full lifecycle: create → commit → reveal → judge → finalize | E2E on-chain |
| 26 | Executor discovery via `TEEServiceRegistry` | Capability 0 (HTTP) |
| 27 | ECIES encryption with executor public key | `symmetricNonceLength = 12` |
| 28 | LLM JSON response parsing | `responseFormat` with `json_schema` |

### Edge Cases

| # | Scenario | Handling |
|---|----------|----------|
| 29 | Submitter never reveals | Silently excluded from judging |
| 30 | Bounty with 0 submissions | Revert "no submissions" |
| 31 | Same salt across different bounties | Safe — bountyId is in hash |
| 32 | Owner tries to judge during reveal phase | Reverted by modifier |

## Key Attack Vectors Tested

| Attack | Mitigation | Test # |
|--------|-----------|--------|
| **Front-running commitment** | Hash includes `msg.sender` | 9 |
| **Replay commitment across bounties** | Hash includes `bountyId` | 31 |
| **Submitter copying another's commitment** | Reveal fails (sender mismatch) | 9 |
| **Owner judges early (sees answers)** | `afterReveal` modifier | 14 |
| **Double-claim reward** | `finalized` flag checked | 20 |
| **Brute-force salt** | Salt is `bytes32` (2²⁵⁶ space) | — |
