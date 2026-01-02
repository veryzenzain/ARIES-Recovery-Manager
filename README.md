# ARIES Recovery Manager (Write-Ahead Logging & Crash Recovery)

## Overview

This project is a **from-scratch implementation of ARIES-style database recovery** in Java. It implements **write-ahead logging (WAL)**, **fuzzy checkpointing**, **savepoints**, and **ACID-compliant crash recovery** using the full **Analysis → Redo → Undo** pipeline.

The implementation closely follows the ARIES algorithm and is designed to be **correct, efficient, and test-driven**, passing an extensive suite of recovery tests.

This project demonstrates my ability to reason about **low-level systems behavior**, **crash consistency**, and **state reconstruction**, and to translate formal algorithms into robust, production-quality code.


## Key Features

###  Write-Ahead Logging (WAL)

* Logs all transactional changes before they are applied to disk
* Ensures durability by flushing commit records before returning
* Supports page updates, page/partition allocation, and deallocation

### Transaction Lifecycle Management

* Correct handling of transaction states:

  * `RUNNING`
  * `COMMITTING`
  * `ABORTING`
  * `RECOVERY_ABORTING`
  * `COMPLETE`
* Proper cleanup and logging on commit, abort, and end

### Savepoints & Partial Rollback

* Implements SQL-style savepoints
* Supports:

  * `SAVEPOINT name`
  * `ROLLBACK TO SAVEPOINT name`
  * `RELEASE SAVEPOINT name`
* Uses WAL to undo only the portion of a transaction after a savepoint

### Fuzzy Checkpointing

* Implements non-blocking (fuzzy) checkpoints
* Splits checkpoint state across multiple records when needed
* Correctly merges Dirty Page Table (DPT) and Transaction Table during recovery
* Updates master record atomically

### Full ARIES Restart Recovery

#### 1. Analysis Phase

* Reconstructs:

  * Dirty Page Table (DPT)
  * Transaction Table
* Handles:

  * Page updates and frees
  * Transaction status transitions
  * End-checkpoint merging
* Correctly identifies:

  * Transactions to commit
  * Transactions to abort during recovery

#### 2. Redo Phase

* Starts from minimum recLSN in DPT
* Redoes only necessary actions based on:

  * PageLSN comparison
  * DPT membership
  * Record type
* Avoids unnecessary I/O

#### 3. Undo Phase

* Uses a priority queue (max-heap) to undo highest-LSN actions first
* Generates and applies Compensation Log Records (CLRs)
* Uses `undoNextLSN` to skip already-undone work
* Ensures crash-safe undo by logging before applying changes

---

## Technical Highlights

* Careful handling of LSN chaining (`prevLSN`, `undoNextLSN`)
* Correct interaction between WAL, buffer manager, and disk manager
* Efficient single-pass undo using a global LSN ordering
* Extensive use of invariants to maintain recovery correctness

---

## Code Structure

* `ARIESRecoveryManager.java` – core recovery logic
* `LogRecord` subclasses – typed WAL records
* `TransactionTableEntry` – per-transaction metadata
* `TestRecoveryManager.java` – comprehensive recovery test suite

---

## Testing

This implementation passes all recovery tests, including:

* Simple commit / abort / savepoint cases
* Nested savepoint rollbacks
* Multi-record fuzzy checkpoints
* Crash recovery at arbitrary points
* CLR correctness and redo safety

The tests validate both correctness and subtle edge cases, such as:

* Skipping already-undone records
* Correct DPT cleanup after redo
* Idempotent redo behavior


