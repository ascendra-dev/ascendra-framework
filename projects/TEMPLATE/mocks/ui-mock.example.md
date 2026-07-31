# UI Mock — Worked Example

This example uses the same fictional project as the framework's other worked examples: **Harborview Consulting Ltd — Invoice Management System** (see `projects/TEMPLATE/brds/brd.example.md`).

Harborview has one portal only — Finance Manager and Director share the same internal app; Payers never log in, they only follow an emailed payment link (BRD Section 1.5). So architecture Section 3.1.3.1 would list a single portal row: **Harborview Staff App**. This example shows that one portal's mock, generated from:

- **REQ-010/REQ-011** (Director approval above £10,000) → drives the approval action's visibility condition
- **SEC-003** ("Finance Manager must not be able to approve or reject invoices... enforced server-side") → drives the action-gating caption
- **ROL-001/ROL-002** (Finance Manager, Director) → drives the nav conditions

---

**Portal:** Harborview Staff App
**Source:** `projects/HARBORVIEW-INV-001/screens/screen-design.md` (base pass); `projects/HARBORVIEW-INV-001/architecture/arch-v1.md` Section 3.1.3.4 (patch pass — action gating shown below)

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<title>Harborview Staff App — UI Mock</title>
<style>
  /* Real values read from ascendra-ui/app/globals.css — not approximated. */
  :root {
    --background: oklch(1 0 0);
    --foreground: oklch(0.2928 0.0163 285.35);
    --card: oklch(1 0 0);
    --border: oklch(0.9437 0.0027 286.35);
    --primary: oklch(0.5573 0.2543 283.67);
    --radius: 0.625rem;
    --muted-foreground: oklch(0.4913 0.0254 285.43);
  }
  body { margin: 0; background: var(--background); color: var(--foreground); font-family: system-ui, sans-serif; }
  .shell { display: flex; min-height: 100vh; }
  .sidebar { width: 240px; border-right: 1px solid var(--border); padding: 1rem; }
  .main { flex: 1; padding: 1.5rem; }
  .screen { border: 1px solid var(--border); border-radius: var(--radius); padding: 1.5rem; margin-bottom: 1.5rem; }
  .gated { font-size: 0.75rem; color: var(--muted-foreground); }
  button.primary { background: var(--primary); color: white; border: none; border-radius: var(--radius); padding: 0.5rem 1rem; }
</style>
</head>
<body>
  <div class="shell">
    <nav class="sidebar">
      <!-- Invoices — baseline, always visible -->
      <div>Invoices</div>
      <!-- Approvals — requires: Director -->
      <div>Approvals <span class="gated">(requires: Director)</span></div>
    </nav>
    <main class="main">
      <section class="screen">
        <h2>SCR-001 — /invoices</h2>
        <p class="purpose">Finance Manager creates and sends invoices; both roles can view the list.</p>
        <table>
          <tr><th>Invoice #</th><th>Payer</th><th>Amount</th><th>Status</th></tr>
          <tr><td>INV-2026-014</td><td>Riverside Ltd</td><td>£12,400.00</td><td>Pending Approval</td></tr>
        </table>
      </section>
      <section class="screen">
        <h2>SCR-002 — /invoices/[id]</h2>
        <p class="purpose">Invoice detail. REQ-010: invoices at or above £10,000 require Director approval before sending.</p>
        <p>INV-2026-014 — Riverside Ltd — £12,400.00 — Pending Approval</p>
        <!-- Action gating: SEC-003 — Approve/Reject is Director-only, enforced server-side -->
        <button class="primary">Approve</button>
        <span class="gated">(requires: Director — SEC-003)</span>
      </section>
    </main>
  </div>
</body>
</html>
```

Note what this example deliberately does *not* do: it does not add a Branch Manager persona, a multi-currency toggle, or any screen not traceable to a Harborview requirement — even though a real invoicing product often has those. Every element on the mock exists because a specific REQ/SEC/ROL ID put it there.
