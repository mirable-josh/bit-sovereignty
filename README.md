# BitSovereignty DAO

**BitSovereignty DAO** is a self-governing treasury protocol built on the Stacks blockchain, leveraging Bitcoin's security model. It enables decentralized, transparent fund allocation and proposal execution through time-bound governance voting, powered by staked STX.

Members earn voting power by locking STX, which mints governance tokens used to vote on fund disbursements and policy changes. This system aligns incentives, secures the treasury, and decentralizes financial governance.

---

## 🏗️ System Overview

* **Layer 1 (Bitcoin)**: Anchors the security and immutability of all transactions through Stacks' Proof of Transfer (PoX).
* **Layer 2 (Stacks)**: Hosts the Clarity smart contract logic for deposits, voting, and fund management.
* **DAO Members**: Stake STX to mint governance tokens, create/vote on proposals, and earn the ability to direct treasury funds.
* **Contract Treasury**: Controlled exclusively by successful DAO proposals, ensuring decentralized fund allocation.

---

## ⚙️ Contract Architecture

### 1. **Initialization**

The contract must be initialized once by the contract deployer:

```clarity
(define-public (initialize))
```

---

### 2. **Governance Token Logic**

Tokens represent voting power. They are minted on deposit and burned on withdrawal.

* **Mint/Burn Functions**

  * `mint-tokens`: Called during `deposit`
  * `burn-tokens`: Called during `withdraw`

* **Balances**

  * Stored in `balances` map
  * Total supply tracked via `total-supply` variable

---

### 3. **Deposits & Withdrawals**

Users stake STX to participate in governance. Deposits mint voting tokens, and withdrawals burn them after a lock period.

* **Deposit**

```clarity
(define-public (deposit (amount uint)))
```

* Minimum deposit required: `minimum-deposit` (default: 1 STX)

* Tokens are locked for `lock-period` (~10 days by default)

* **Withdraw**

```clarity
(define-public (withdraw (amount uint)))
```

* Only possible after lock period expires
* Tokens are burned and STX is returned to the user

---

### 4. **Proposals & Voting**

DAO members can create proposals and vote using their governance tokens.

#### Create Proposal

```clarity
(define-public (create-proposal (description ...) (amount ...) (target ...) (duration ...)))
```

* Duration must be within bounds:

  * Min: 1 day (`minimum-duration` = 144 blocks)
  * Max: 14 days (`maximum-duration` = 20160 blocks)
* Target must not be the DAO contract itself

#### Vote

```clarity
(define-public (vote (proposal-id uint) (vote-for bool)))
```

* Each voter can vote once per proposal
* Voting power is determined by current balance at time of vote

#### Execute Proposal

```clarity
(define-public (execute-proposal (proposal-id uint)))
```

* Can only execute after expiration
* Requires more YES than NO votes
* Transfers funds to target address from the DAO treasury

---

### 5. **Storage Layout**

| Data Type | Name              | Purpose                                   |
| --------- | ----------------- | ----------------------------------------- |
| `bool`    | `initialized`     | Flag to ensure single initialization      |
| `uint`    | `total-supply`    | Total governance tokens in circulation    |
| `uint`    | `proposal-count`  | Tracks total proposals created            |
| `uint`    | `minimum-deposit` | Minimum deposit required (default: 1 STX) |
| `uint`    | `lock-period`     | Lock duration for deposits (~10 days)     |
| `map`     | `balances`        | Governance token balances                 |
| `map`     | `deposits`        | User deposit and lock info                |
| `map`     | `proposals`       | Proposal details                          |
| `map`     | `votes`           | Individual vote records                   |

---

## 🔐 Access Control

| Functionality     | Access Type   | Check                    |
| ----------------- | ------------- | ------------------------ |
| Initialization    | Owner only    | `is-contract-owner`      |
| Proposal Creation | Token holders | Balance > 0              |
| Voting            | Token holders | Balance > 0, not voted   |
| Execute Proposal  | Anyone        | On-chain condition check |

---

## 🧭 Data Flow

### Deposit → Mint → Vote → Execute → Withdraw

1. **User deposits STX**

   * Triggers STX transfer to contract
   * Mints governance tokens
   * Locks tokens for fixed duration

2. **User creates or votes on a proposal**

   * Voting power derived from governance token balance
   * Votes stored with immutability and traceability

3. **Proposal execution**

   * After expiration and passing vote
   * Transfers funds from DAO to target address

4. **User withdraws STX**

   * After lock period
   * Tokens burned, STX returned

---

## 📖 Read-Only Functions

| Function           | Description                         |
| ------------------ | ----------------------------------- |
| `get-balance`      | Get user's governance token balance |
| `get-total-supply` | Total minted governance tokens      |
| `get-proposal`     | Get details of a proposal           |
| `get-deposit-info` | Get user deposit metadata           |
| `get-vote`         | Check if and how user voted         |

---

## 🧪 Error Handling (Constants)

Grouped by domain for clarity:

### Access Control

* `err-owner-only` - Only contract owner can call
* `err-unauthorized` - Unauthorized access or insufficient privileges

### Deposits/Withdrawals

* `err-insufficient-balance`
* `err-zero-amount`
* `err-below-minimum`
* `err-locked-period`

### Proposals/Voting

* `err-invalid-proposal-id`
* `err-invalid-description`
* `err-invalid-target`
* `err-invalid-duration`
* `err-already-voted`
* `err-invalid-vote`
* `err-proposal-not-found`
* `err-proposal-expired`

### Transfers

* `err-transfer-failed`

---

## ✅ Security & Safeguards

* **Time Locks**: Prevent premature withdrawal and ensure governance integrity.
* **Minimum Deposit**: Deters spam proposals and encourages serious participation.
* **Immutable Voting**: Prevents vote tampering via mapping per user-proposal.
* **Owner Restrictions**: Only initialization is gated by the owner; all governance is fully decentralized afterward.
* **Proposal Target Validation**: Prevents self-draining contract via self-targeted proposals.

---

## 🚀 Future Extensions

This core DAO module can be extended to support:

* Reward/rebalancing mechanisms
* Treasury diversification (via SIP-010 tokens or Bitcoin)
* Multi-signature proposal execution
* Slashing or penalties for malicious actors

---

## 📜 License

MIT License. This project is open for extension, modification, and integration into other Bitcoin-secured governance protocols on the Stacks ecosystem.

---

## 🧠 About

Developed by experienced Clarity engineers with the goal of creating a resilient, Bitcoin-secured governance layer for DeFi and public goods funding.
