
Business Logic Vulnerability – Inventory Race Condition Enables Overselling

#1. System Context

Feature: Checkout / order creation
Component: Backend inventory and order service
Constraint: Stock is a finite shared resource that must be enforced during order placement

The system must prevent stock exhaustion violations under concurrent execution.


#2. Security Invariant

Inventory allocation must be enforced atomically at order creation.

Formally:

Total successful order allocations must never exceed available stock under concurrent execution.

This requires stock validation and deduction to occur within a single atomic operation.

#3. Vulnerability Description

The system implements a non-atomic check-then-update flow for inventory allocation.

Under concurrent checkout requests for the same product, multiple requests can pass stock validation before any inventory update is committed. This creates a race condition window where stale stock values are used.

As a result, multiple orders can be successfully created even when available stock is insufficient.


#4. Exploitation Scenario

An attacker can send multiple parallel checkout requests for the same SKU during the inventory validation window.

Due to concurrent execution, each request observes the same pre-decrement stock value and proceeds to order creation, bypassing intended stock constraints.


#5. Observed Behavior
* Multiple concurrent checkout requests succeed for a limited-stock item
* Each request returns a valid order confirmation
* Total successful orders exceed available inventory
* Inventory state is only updated after order creation
* No request-level rejection occurs during concurrency window

#6. Root Cause
* Non-atomic check-then-update inventory flow
* Absence of transactional stock reservation at commit time
* Concurrent requests operating on stale shared state
* Lack of concurrency control (e.g. locking or conditional updates)

#7. Security Impact
Inventory constraints are violated under concurrent execution
System accepts more successful orders than available stock
Breaks core business invariant: finite resource enforcement
Creates financial and fulfillment inconsistency risk

#8. Remediation Guidance
Enforce atomic stock reservation during order creation
Use conditional updates for stock deduction (e.g. decrement-if-available pattern)
Or apply row-level locking within a transaction boundary
Ensure inventory state cannot be read-modified-wrote concurrently without synchronization

#9. Security Principle

Any system managing finite shared resources must enforce atomic allocation at the transaction boundary.
Non-atomic validation of shared state under concurrency introduces race conditions that violate core system invariants.
