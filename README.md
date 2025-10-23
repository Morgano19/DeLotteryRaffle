DeLotteryRaffle 🍀
==================

Overview
--------

I've created **DeLotteryRaffle**, a secure and decentralized smart contract system written in **Clarity** for the Stacks blockchain. This system enables transparent and fair operation of both **Lottery** and **Raffle** games. By leveraging blockchain technology, it ensures verifiable randomness, automatic prize distribution, and immutable game rules, significantly mitigating the fraud risks associated with traditional centralized systems.

* * * * *

Features
--------

### Core Functionality

-   **Decentralized Lotteries:** Users can purchase a single ticket to enter a lottery. A winner is selected from the pool of unique participants.

-   **Decentralized Raffles:** Users can purchase a single ticket to enter a raffle. The winner is selected from the pool of unique participants.

-   **Contract Ownership & Control:** Creation and drawing of games are restricted to the contract owner, ensuring controlled game lifecycle management.

-   **STX Prize Pool:** Ticket prices in STX are aggregated into a `total-prize` pool, which is automatically distributed to the winner upon drawing.

-   **Time-Bound Games:** Lotteries and raffles have predefined durations (`LOTTERY-DURATION` = 1440 blocks, `RAFFLE-DURATION` = 720 blocks) to prevent indefinite games.

-   **Minimum Participant Requirement:** A minimum of 2 participants (`MIN-PARTICIPANTS`) is required before a winner can be drawn, ensuring viability.

-   **Maximum Participant Cap:** A hard limit of 1000 participants (`MAX-PARTICIPANTS`) is enforced for both game types.

### Advanced Features

-   **Multi-Ticket Lottery Purchase:** A dedicated public function (`buy-multiple-lottery-tickets`) allows a single participant to purchase multiple lottery tickets in one transaction.

-   **Bulk Discount Mechanism:** An automatic **10% discount** is applied to the total cost when a user purchases **5 or more** lottery tickets in a single `buy-multiple-lottery-tickets` transaction.

-   **Transparent Randomness (Pseudo):** The private function `generate-random-number` uses the current `block-height` and game-specific data to generate a pseudo-random index for winner selection, which is a common, transparent, and auditable approach on Stacks for simple contracts.

* * * * *

Contract Details
----------------

### Constants

| Constant | Value | Description |
| --- | --- | --- |
| `CONTRACT-OWNER` | `tx-sender` | The principal that deploys the contract, with elevated privileges. |
| `MAX-PARTICIPANTS` | `u1000` | Maximum number of unique participants allowed in any game. |
| `MIN-PARTICIPANTS` | `u2` | Minimum number of unique participants required to draw a winner. |
| `LOTTERY-DURATION` | `u1440` | Duration (in blocks) for a lottery. |
| `RAFFLE-DURATION` | `u720` | Duration (in blocks) for a raffle. |

### Error Codes

| Error Code | Value | Description |
| --- | --- | --- |
| `ERR-OWNER-ONLY` | `u100` | Sender is not the contract owner. |
| `ERR-INVALID-PRICE` | `u101` | Ticket price must be greater than zero. |
| `ERR-INVALID-ENTRY` | `u102` | Game ID is invalid or entry conditions are not met. |
| `ERR-TRANSFER-FAILED` | `u103` | STX transfer failed. |
| `ERR-NOT-ACTIVE` | `u104` | Game is not in the "active" status. |
| `ERR-TOO-EARLY` | `u105` | Attempt to draw winner before the `end-block`. |
| `ERR-MIN-PARTICIPANTS` | `u106` | Minimum participant count not met for drawing. |
| `ERR-INSUFFICIENT-BALANCE` | `u108` | Contract doesn't hold enough STX for withdrawal. |
| `ERR-MAX-PARTICIPANTS` | `u109` | Maximum participant count reached. |

* * * * *

Private Functions
-----------------

Private functions are internal helpers used by the public and read-only functions to enforce game rules, manage state, and execute logic securely. They are **not** callable directly by external principals.

| Function | Arguments | Returns | Description |
| --- | --- | --- | --- |
| `generate-random-number` | `(lottery-id uint) (participant-count uint)` | `uint` | **Pseudo-Randomness.** Generates a pseudo-random index using the formula `(block-height + lottery-id + participant-count)` modulo `participant-count`. This is used to select the winner. |
| `is-lottery-active` | `(lottery-id uint)` | `bool` | Checks if a lottery is currently in the **"active"** state, has passed its `start-block`, and has not yet reached its `end-block`. |
| `is-raffle-active` | `(raffle-id uint)` | `bool` | Checks if a raffle is currently in the **"active"** state, has passed its `start-block`, and has not yet reached its `end-block`. |
| `validate-lottery-entry` | `(lottery-id uint) (participant principal)` | `bool` | Ensures all conditions are met for a new lottery participant: game is active, max participants is not reached, and the participant is not already entered. |
| `validate-raffle-entry` | `(raffle-id uint) (participant principal)` | `bool` | Ensures all conditions are met for a new raffle participant: game is active, max participants is not reached, and the participant is not already entered. |

* * * * *

Public Functions
----------------

### Game Creation (Owner Only)

| Function | Arguments | Returns | Description |
| --- | --- | --- | --- |
| `create-lottery` | `(ticket-price uint)` | `(response uint uint)` | Creates a new lottery, setting its duration and price. Only callable by the `CONTRACT-OWNER`. |
| `create-raffle` | `(ticket-price uint)` | `(response uint uint)` | Creates a new raffle, setting its duration and price. Only callable by the `CONTRACT-OWNER`. |

### Game Entry

| Function | Arguments | Returns | Description |
| --- | --- | --- | --- |
| `enter-lottery` | `(lottery-id uint)` | `(response bool uint)` | Transfers 1 ticket's price to the contract and registers the sender as a unique participant. |
| `enter-raffle` | `(raffle-id uint)` | `(response bool uint)` | Transfers 1 ticket's price to the contract and registers the sender as a unique participant. |
| `buy-multiple-lottery-tickets` | `(lottery-id uint) (ticket-count uint)` | `(response uint uint)` | Allows bulk purchase of lottery tickets with an optional **10% discount** for ≥5 tickets. Updates ticket count but only counts as **1** unique participant. |

### Winner Drawing (Owner Only)

| Function | Arguments | Returns | Description |
| --- | --- | --- | --- |
| `draw-lottery-winner` | `(lottery-id uint)` | `(response principal uint)` | Selects a winner, transfers the `total-prize` to them, and sets the game status to "completed." Must be called after `end-block` and only by the `CONTRACT-OWNER`. |
| `draw-raffle-winner` | `(raffle-id uint)` | `(response principal uint)` | Selects a winner, transfers the `total-prize` to them, and sets the game status to "completed." Must be called after `end-block` and only by the `CONTRACT-OWNER`. |

### Administrative

| Function | Arguments | Returns | Description |
| --- | --- | --- | --- |
| `emergency-withdraw` | `(amount uint)` | `(response bool uint)` | Allows the `CONTRACT-OWNER` to withdraw STX from the contract in an emergency. |

* * * * *

Read-Only Functions
-------------------

These functions provide transparent access to game state without requiring a transaction.

| Function | Arguments | Returns | Description |
| --- | --- | --- | --- |
| `get-lottery-info` | `(lottery-id uint)` | `(optional {owner: principal, ...})` | Retrieves all stored data for a specific lottery. |
| `get-raffle-info` | `(raffle-id uint)` | `(optional {owner: principal, ...})` | Retrieves all stored data for a specific raffle. |
| `is-lottery-participant` | `(lottery-id uint) (participant principal)` | `(optional bool)` | Checks if a principal is a unique participant in a lottery. |
| `is-raffle-participant` | `(raffle-id uint) (participant principal)` | `(optional bool)` | Checks if a principal is a unique participant in a raffle. |
| `get-lottery-ticket-count` | `(lottery-id uint) (participant principal)` | `uint` | Retrieves the number of tickets a principal holds for a lottery. Defaults to `u0`. |
| `get-raffle-ticket-count` | `(raffle-id uint) (participant principal)` | `uint` | Retrieves the number of tickets a principal holds for a raffle. Defaults to `u0`. |

* * * * *

Technical Analysis & Security
-----------------------------

### Randomness Vulnerability (Self-Correction/Mitigation)

The current implementation of `generate-random-number` uses `block-height` and game data. This is a **pseudo-random** method that is **predictable** by miners, as the `block-height` is known at the time the transaction is processed (known as **miner extractable value or MEV**).

**Mitigation:** For a production-grade system handling significant value, I would recommend integrating with a dedicated, verifiable on-chain randomness service or implementing a secure **Commit-Reveal Scheme**.

### Anti-Fraud & Validation

The contract implements several robust checks to maintain integrity:

-   **Owner Restriction:** `asserts! (is-eq tx-sender CONTRACT-OWNER)` is used on creation and drawing functions.

-   **Active Status Checks:** Private helpers ensure users can only enter ongoing games.

-   **Time Checks:** Prevents premature winner selection.

-   **Minimum/Maximum Participation:** Ensures game viability and adherence to caps.

-   **STX Transfer Safety:** The `as-contract` function is used for prize distribution to prevent re-entrancy issues.

* * * * *

Deployment and Usage
--------------------

### Prerequisites

-   A Stacks wallet (e.g., Leather, Xverse).

-   A Stacks 2.0+ network environment (e.g., testnet, mainnet).

-   Sufficient STX to cover transaction fees.

### Deployment Steps

1.  **Compile:** The contract is written in Clarity and can be compiled using the Clarity REPL or a Stacks development environment like the Clarinet CLI.

2.  **Deploy:** The contract must be deployed on the Stacks blockchain. The deploying principal automatically becomes the `CONTRACT-OWNER`.

3.  **Fund:** The contract does not require initial funding, as the prize pool is accumulated from ticket purchases.

### Common Flow

1.  **Owner creates game:** `CONTRACT-OWNER` calls `create-lottery`.

2.  **Users enter:** Participants call `enter-lottery` or `buy-multiple-lottery-tickets` before the `end-block`.

3.  **Owner draws winner:** After the `end-block` is reached and `MIN-PARTICIPANTS` is met, the `CONTRACT-OWNER` calls `draw-lottery-winner`.

4.  **Prize Distribution:** The entire `total-prize` STX is transferred to the winner's address.

* * * * *

Contributing
------------

I welcome and encourage contributions to improve the security, efficiency, and feature set of DeLotteryRaffle.

### Development Setup

1.  **Clone the Repository:**

    Bash

    ```
    git clone [Your Repository URL Here]
    cd DeLotteryRaffle

    ```

2.  **Install Clarinet:** Follow the official Stacks documentation to install the Clarinet CLI.

3.  **Testing:** Use `clarinet test` to run any existing unit tests, and write new ones to cover any added or modified logic.

### Guidelines for Contribution

-   **Clarity Best Practices:** All code must adhere to the official Clarity language style guide.

-   **Test Coverage:** Any new feature or bug fix must be accompanied by new or updated unit tests.

-   **Security First:** Any changes to the randomness mechanism or asset handling must be rigorously reviewed for potential exploits.

-   **Documentation:** Update the in-line comments and the `README.md` to reflect any changes.

Please submit a Pull Request with a clear description of the changes.

* * * * *

License
-------

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2025 DeLotteryRaffle.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

```
