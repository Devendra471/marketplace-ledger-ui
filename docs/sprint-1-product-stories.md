# Sprint 1 — Product Stories

**Product:** Marketplace Ledger UI
**Sprint window:** 2026-06-15 → 2026-06-26 (2 weeks)
**Status:** Planned

## Sprint goal

Convert the highest-value mocked flows in the prototype into real, auditable
behaviour — so an operator can run a trip from payment through POD-gated balance
release, see correct wallet/recovery movements, and pull the core finance
reports. Everything that moves money or releases a document must leave an
audit trail.

## Context

The current build (`index.html`) is a UI prototype on mock data. Many actions
(downloads, bulk document generation, POD preview, report drill-downs,
Zoho/Razorpay) only fire a toast. This sprint takes the most operationally
critical of those from "looks done" to "actually does the thing", and hardens
the money math that the rest of the product depends on.

## Legend

- **Priority:** P0 (committed) · P1 (committed, stretch-tolerant) · P2 (stretch)
- **Estimate:** story points (Fibonacci)
- Each story is **Ready** only when its acceptance criteria are testable.

---

## Epic A — Payments execution & POD gating

### LED-101 · Record an advance/balance payment that updates the ledger
**P0 · 5 pts**

> As an **operations user**, I want to record an advance or balance payment
> against a trip so that the party ledger and trip balance reflect it
> immediately.

**Acceptance criteria**
- "Pay Advance" / "Pay Balance" capture amount, mode, and UTR (auto-generated
  if blank).
- Posting a payment creates a line-item ledger entry and reduces the open
  balance for that trip and party.
- A duplicate payment for an already-paid stage is blocked with a clear reason.
- Every payment writes an audit entry (actor, trip, amount, stage, timestamp).

### LED-102 · Block balance release until POD is approved
**P0 · 3 pts**

> As a **finance approver**, I want the balance payment disabled until the POD
> is approved so that we never release funds against an unverified delivery.

**Acceptance criteria**
- For FO trips, "Pay Balance" is disabled while POD status ≠ Approved.
- The disabled button shows the blocking reason on hover and on click.
- The trip list shows a "blocked" indicator and reason on affected rows.

### LED-103 · POD review workflow (Submitted → Approved/Disputed)
**P1 · 5 pts**

> As an **audit user**, I want to move a POD through Submitted → Approved or
> Disputed with a remark so that downstream payment and document eligibility
> is driven by a real status, not a mock field.

**Acceptance criteria**
- Approve / Dispute actions update POD status and audit state.
- Disputed PODs require a remark and keep balance + documents blocked.
- Status change is captured in the trip audit log.

---

## Epic B — Wallet & recoveries

### LED-201 · Auto-apply wallet credit on payment
**P0 · 5 pts**

> As an **operations user**, I want available wallet credit to be applied to a
> party's next payment automatically so that outstanding recoveries are netted
> without a manual journal entry.

**Acceptance criteria**
- When a party has wallet credit, a payment first consumes the wallet bucket,
  then pays the remainder via the chosen mode.
- The wallet line is shown distinctly (e.g. mode = "Wallet Recovery") in the
  trip ledger.
- Wallet balance and net exposure update on the Wallet & Recoveries page.

### LED-202 · Replace mock "unsettled" with real allocation math
**P1 · 3 pts**

> As a **finance user**, I want the unsettled-receipts figure to be computed
> from receipts received minus amounts allocated to trips so that the number
> is trustworthy.

**Acceptance criteria**
- Remove the hard-coded 15% placeholder.
- Unsettled = received − allocated, per party and in aggregate.
- Page recomputes when an allocation or receipt changes.

### LED-203 · Cancelled trip moves payments to wallet
**P1 · 3 pts**

> As an **operations user**, I want a cancelled trip to move any paid amounts
> into the party wallet and reduce charges to cancellation-only so that money
> already paid is preserved as credit.

**Acceptance criteria**
- Cancelling a trip zeroes freight, keeps only cancellation charges, zeroes
  deductions, and credits paid amounts to the party wallet.
- A Credit Note becomes available for the cancelled trip.
- The cancellation is audited.

---

## Epic C — Documents

### LED-301 · Generate Bill of Supply / Credit Note / Loading Memo
**P1 · 5 pts**

> As an **operations user**, I want to open a real generated document for an
> eligible trip so that I can share or file it instead of seeing a toast.

**Acceptance criteria**
- Loading Memo available for non-cancelled trips; BoS gated on POD-approved;
  FO BoS on settled; Credit Note on cancelled.
- "View" renders a document preview from trip data.
- Ineligible documents show the specific eligibility reason.

### LED-302 · Bulk-generate eligible documents
**P2 · 3 pts**

> As an **operations user**, I want to bulk-generate documents for all eligible
> trips so that I don't open them one at a time.

**Acceptance criteria**
- "Bulk Generate (eligible)" processes only trips that pass eligibility.
- A summary reports generated vs. skipped (with reasons).
- Replaces the current "queued (mock)" toast.

---

## Epic D — Reports

### LED-401 · Payables (FO) & Receivables (LSP) reports with aging
**P0 · 5 pts**

> As a **finance user**, I want trip-wise payables and receivables with aging
> buckets so that I can see what we owe and what is owed to us.

**Acceptance criteria**
- Payables: trip-wise payable / paid / balance, grouped by FO.
- Receivables: receivable / received / outstanding, grouped by LSP & branch.
- Aging buckets: 0–30 / 31–60 / 61–90 / 90+.
- Each report opens a real view (no dead "Open →").

### LED-402 · Export the active report to CSV
**P1 · 2 pts**

> As a **finance user**, I want to export a report to CSV so that I can
> reconcile in a spreadsheet.

**Acceptance criteria**
- Export produces a CSV of the rows currently in view (respecting filters).
- Replaces mock "Download started" toasts on report/table downloads.

---

## Epic E — Admin & integrations

### LED-501 · Rules engine drives commission & TDS
**P1 · 3 pts**

> As an **admin**, I want commission slabs and TDS configured in the Rules
> Engine to actually drive trip math so that finance changes don't need a code
> change.

**Acceptance criteria**
- Editing commission slabs / TDS rate recomputes affected trip figures.
- Component master limits (max abs / max %) are enforced on manual entries.
- Changes are audited.

### LED-502 · Zoho / Razorpay integration status (read-only)
**P2 · 2 pts**

> As an **admin**, I want to see connection status and last-sync for Zoho and
> Razorpay so that I know whether postings and payouts are flowing.

**Acceptance criteria**
- Integration tab shows connected/disconnected and last-sync time per provider.
- Clearly labelled as status-only this sprint (no live calls).

---

## Sprint commitment summary

| Priority | Stories | Points |
|----------|---------|--------|
| P0 | LED-101, LED-102, LED-201, LED-401 | 18 |
| P1 | LED-103, LED-202, LED-203, LED-301, LED-402, LED-501 | 21 |
| P2 (stretch) | LED-302, LED-502 | 5 |
| **Total** | **12 stories** | **44** |

## Out of scope this sprint
- Live Zoho posting / Razorpay payout execution (status display only — LED-502).
- Real document storage / e-sign; previews are generated from trip data.
- Authn/RBAC enforcement beyond the existing Role × Section visibility config.
- Mobile / responsive layout pass.

## Definition of Done
- Acceptance criteria met and demoable on seeded data.
- Money-moving and document-releasing actions write audit entries.
- No remaining "(mock)" toast on any flow this sprint touched.
- Reviewed and merged to `main` via PR.
