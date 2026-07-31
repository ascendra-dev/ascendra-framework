# FW-012 — Artifact Co-location

| Field | Value |
|-------|-------|
| **ID** | FW-012 |
| **Date** | 2026-06-27 |
| **Status** | Decided — implemented |
| **Area** | Project folder skeleton, TEMPLATE, all commands |

---

## Decision

Playbooks and discovery state files are co-located with the artifact they produce, not stored in a separate `playbooks/` folder.

---

## Structure

| Artifact | Location |
|---|---|
| Domain knowledge | `projects/{CODE}/domain/{slug}-core.md` |
| Domain playbook | `projects/{CODE}/domain/{slug}-domain-playbook.md` |
| Domain discovery state | `projects/{CODE}/domain/domain-discovery-state.md` |
| BRD playbook | `projects/{CODE}/brds/{slug}-brd-playbook.md` |
| BRD discovery state | `projects/{CODE}/brds/discovery-state.md` |
| BRD | `projects/{CODE}/brds/brd-core-v1.md` |

The `playbooks/` folder is removed from the project skeleton and from `projects/TEMPLATE/`.

---

## Templates (TEMPLATE folder)

| Template | Location |
|---|---|
| Domain knowledge template | `projects/TEMPLATE/domain/domain.template.md` |
| Domain playbook template | `projects/TEMPLATE/domain/domain-playbook.template.md` |
| Domain discovery state template | `projects/TEMPLATE/domain/domain-discovery-state.template.md` |
| BRD playbook template | `projects/TEMPLATE/brds/brd-playbook.template.md` |
| BRD discovery state template | `projects/TEMPLATE/brds/brd-discovery-state.template.md` |
| BRD template | `projects/TEMPLATE/brds/brd.template.md` |

`projects/TEMPLATE/playbooks/` is removed.

---

## Rationale

A folder IS its artifact family. Everything needed to produce and track a BRD lives in `brds/`. Everything needed to produce and track domain knowledge lives in `domain/`. Commands know exactly where to look — no cross-folder searching. The relationship between a playbook and what it produces is self-evident from the filesystem.

The previous `playbooks/` folder held a mix of domain playbooks and BRD playbooks with no structural distinction between them. Co-location makes ownership explicit and removes the ambiguity.

---

## Implementation impact

- `init-project.md` — remove `playbooks/` from folder skeleton
- `projects/TEMPLATE/playbooks/` — remove folder; move `playbook.template.md` to `projects/TEMPLATE/brds/brd-playbook.template.md`
- `/gen-brd-playbook` (renamed from `/gen-playbook`) — output path: `brds/{slug}-brd-playbook.md`
- `/gen-domain-playbook` (new) — output path: `domain/{slug}-domain-playbook.md`
- `/run-brd-discovery` — discovery state path: `brds/discovery-state.md` (was project root)
- `/gen-brd` — discovery state path: `brds/discovery-state.md`
- All commands that reference `playbooks/` or `discovery-state.md` at project root — update paths
