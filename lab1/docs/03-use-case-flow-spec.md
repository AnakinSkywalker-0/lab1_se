# 3. Use-Case Flow Specification — UC-07 Reserve Pickup Window

| Field | Detail |
|---|---|
| **Use Case ID / Name** | UC-07 Reserve Pickup Window |
| **Primary Actor** | Shelter Coordinator |
| **Secondary Actors** | Donor Partner (notified), Push Notification Service |
| **Related Requirements** | FR-002, FR-003, FR-004, NFR-002 |
| **Includes / Extended by** | «include» Generate Pickup Verification Code; «extend» Cancel Reservation |
| **Trigger** | Shelter Coordinator selects "Reserve" on a batch from the claim board |

## Preconditions
1. Shelter Coordinator is logged in and the shelter account is **verified** by a Platform Admin.
2. The selected batch has status **AVAILABLE** and its expiry time has not passed.
3. The shelter has no overlapping reservation for the chosen window.

## Postconditions
**Success:** Batch status = RESERVED for this shelter; the pickup window is locked; a single-use verification code is stored (hashed) and shown to the shelter; the donor is notified; the reservation auto-releases if the window lapses.
**Failure:** Batch status is unchanged, no code is issued, and the shelter sees a clear reason.

## Main Success Scenario
| Step | Actor | System |
|---|---|---|
| 1 | Shelter selects an available batch from the claim board. | Displays batch details: items, dietary tags, quantity, address, live expiry countdown, and available pickup windows. |
| 2 | Shelter picks a pickup window that ends before expiry. | Validates window ≤ expiry minus safety buffer (e.g., 30 min). |
| 3 | Shelter taps **Reserve**. | Checks shelter is verified (RBAC) and acquires an atomic lock on the batch (prevents double booking). |
| 4 | — | Sets batch to RESERVED and saves the reservation. |
| 5 | — | **«include»** Generate Pickup Verification Code: creates a random single-use code, stores only its hash, and ties it to this reservation. |
| 6 | — | Displays confirmation with the code, QR, address, and window; sends a push/email copy. |
| 7 | — | Notifies the Donor Partner: "Shelter X will collect between 18:00–18:30." |
| 8 | Use case ends. | Removes the batch from the public claim board. |

## Alternate Flow A1 — Batch no longer available (at step 3)
1. Another shelter reserved the batch first, **or** the batch expired while the user was deciding.
2. System rejects the lock, leaves the batch unchanged, and shows "This batch was just claimed / has expired."
3. System refreshes the claim board and suggests similar nearby batches (matching dietary tags).
4. Use case ends (failure postcondition).

## Alternate Flow A2 — Shelter withdraws (extension, after step 6)
1. Shelter selects **Cancel Reservation** («extend»).
2. System invalidates the code, sets batch back to AVAILABLE (if not expired), republishes it, and notifies the donor.
3. Use case ends.

## Exception — Account not verified (at step 3)
System denies the reservation (HTTP 403), shows "Awaiting admin verification," and the use case ends.

## Business Rules
- One active reservation per batch; a shelter may hold at most 3 concurrent reservations.
- Verification codes are single-use and expire with the reservation window.
