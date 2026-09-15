# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Fork workflow — read this first

This is **pauldiee's fork** of `lcoscia/vcf-network-check` (remote `upstream`) — Leonardo Coscia's **VCF 9.1 Network Planner** (https://vcfplanner.lcoscia.fr/). The fork is a staging ground to preview changes before proposing them upstream as PRs and issues, same workflow as the `pauldiee/vcf-planner` fork.

- **This CLAUDE.md exists only on the fork's `main`** — it must never end up in a PR to upstream. To guarantee that, create feature branches from `upstream/main`, not from the fork's `main`: `git fetch upstream && git checkout -b <branch> upstream/main`. PR diffs then contain only the intended change.
- PRs target `lcoscia/vcf-network-check`; push branches to `origin` (the fork).
- Sync the fork with `git fetch upstream && git merge upstream/main` on `main` (the CLAUDE.md commit stays on top).

### Previewing changes — GitHub Pages on the fork

The fork hosts GitHub Pages from the **`pages-preview`** branch (root path) at **https://pauldiee.github.io/vcf-network-check/**. To preview a change live, merge its feature branch into `pages-preview` and push:

```bash
git checkout pages-preview && git merge <feature-branch> && git push origin pages-preview
```

- `pages-preview` intentionally has the `CNAME` file **deleted** — `main`'s CNAME points at upstream's custom domain (`vcfplanner.lcoscia.fr`) and would break the fork's Pages. Never re-add it when merging.
- `pages-preview` is a throwaway integration branch: it may contain several unmerged feature branches at once and lag or lead `main`. Never base PR branches on it.
- Local preview (alternative): serve over HTTP as described under Commands.

## Commands

No build step, no bundler — the website is `index.html` + native ES modules in `core/`.

```bash
# Serve the site (ES modules don't load over file://)
python -m http.server 8000        # → http://localhost:8000/index.html
```

There is no linter and no automated test suite in this repo.

## Architecture

Single-page network design tool for VCF 9.0/9.1 pre-deployment planning: from a handful of inputs (version, scenario, host counts, storage, NSX config, FQDN prefix/suffix) it generates the VLAN design, appliance IP/FQDN list, VIPs, validation results, and an Excel/JSON export.

- `index.html` — the entire UI: Alpine.js + Tailwind CDN (Tailwind is prefixed `tw-` to avoid class collisions; Alpine is loaded dynamically by the module script). Tabs: Overview, Management Domain, Platform Services, Workload Domains, VLAN Design, Appliances, VIPs, Validation, Export/Import, VCF Components.
- `core/` — native ES modules holding the engine (change logic here, not in the UI):
  - `reference.js` / `data.js` — reference tables and constants
  - `vlan.js` — VLAN table generation (incl. per-AZ rows for stretched topologies)
  - `appliances.js` — appliance list with per-component IP/FQDN requirements
  - `vips.js` — virtual IP allocation
  - `components.js` — the VCF Components card grid (per-component scope, IPs/unit, FQDNs/unit)
  - `sizing.js`, `summary.js` — counts and domain summary
  - `validation.js` — architectural rules with Blocker / Warning / Info severity
  - `excel.js` — the 5-sheet .xlsx export
  - `i18n.js` — **bilingual FR/EN** `translations` object; all UI labels and engine-generated notes flow through it. French content is never removed — both languages coexist; any new user-facing string needs both FR and EN entries.
  - `index.js` — module barrel

**Fidelity constraint:** the modeled rules trace to Broadcom TechDocs / the official VCF IP Allocation guidance (e.g. Services Runtime /28–/27 block with Identity Broker + Day-N Log Management/Real-time Metrics allocated *from* the block; VCF Automation's separate /29; NSX Manager VIP reserved in all modes; witness latency tiers). Changes to counts or rules should cite the source, mirroring the README's design-rules section.

Persistence is browser `localStorage` (project config + language choice).

## Relation to pauldiee/VCF9-DeploymentPlanning

Paul's public field guide (https://pauldiee.github.io/VCF9-DeploymentPlanning/) references this tool on its landing page and in Step 1 (`docs/01-network-dns-plan.md`) as a **generated starting point** for the network plan — generate here, then validate/own the result against that repo's sizing minimums, VM-Management carve-out, and network-team/architect hand-off. Findings made while using this tool with that flow are good candidates for upstream issues/PRs here.

## Customer data hygiene

Never commit example inputs/exports containing real customer names, site
names, IPs, hostnames, VLAN IDs, or FQDN prefixes/suffixes into this repo
(fork or upstream PRs) — use Rainpole-style placeholder values only.

**Customer data is never used with Claude on this repo, period, no
exceptions.** No real customer names, IPs, hostnames, credentials, or
other identifying details are ever entered into a Claude session while
working on this repo – not in chat text, not in a screenshot.
