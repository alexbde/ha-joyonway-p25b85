# Migration Plan: ha-joyonway-p25b85 → ha-joyonway

> Move all content to a fresh repo `alexbde/ha-joyonway` with clean squash-merged PRs on `main`.

## Strategy

1. Create fresh repo `alexbde/ha-joyonway` on GitHub (no fork relation).
2. Push the upstream fork snapshot as the initial commit on `main`.
3. Apply the remaining changes in 7 logical PRs (replay non-merge commits by range onto feature branches, squash-merge each).
4. After all PRs are merged, rename the integration domain (`joyonway_p25b85` → `joyonway`) in a final PR.

This migration is **content-only replay**: exact commit SHAs/authorship/timestamps are not preserved on the new repo's `main` branch.

Original commit history remains visible in GitHub PR commit views from the old repo references. Keeping migration branches in the new repo is **not required**; they may be deleted after squash merge. The `main` branch stays clean (one squash commit per PR).

During migration, treat this old repo as **read-only**. Do not add new commits here unless the new repo (`alexbde/ha-joyonway`) is fully created, pushed, and ready to receive replay branches.

---

## Step 0: Create the new repo

```bash
gh repo create alexbde/ha-joyonway --public --description "Home Assistant integration for Joyonway spa controllers (P25B85)" --clone
cd ha-joyonway
```

## Step 1: Initial commit — upstream fork snapshot

This snapshots all 18 upstream commits (`20be69d`..`d20d912`) + the planning commits (`0c4f324`, `feeb2ae`, `f95d74e`) into one initial commit in the new repo.

**Boundary commit:** `558d0bd` (Add Joyonway P25B85 integration) — this is the first commit that has the new integration scaffolding. Actually, to include the full planning phase, we take everything up to and including `32e4e9a` (Merge PR #1).

**Range:** root..`32e4e9a` (commits 1–22)

Use the archive-based flow in **Execution Commands** below. This keeps the old repo read-only and creates the initial commit directly in the new repo.

---

## Step 2: PRs (7 total)

Each PR is created from a branch containing replayed commits from the old repo. Replay **non-merge commits only**. On GitHub, use **squash & merge** to keep `main` clean.

### PR 1: Core integration + protocol capture (Phase 1–4)
**Commits:** `051c05b`..`b344095`
**Scope:** Translations, icons, implementation of capture findings, entity metadata, Phase 4 captures, write support (light + pump replay), CRC reverse-engineering tools, temperature capture tooling.

```
051c05b Remove redundant custom component
87a8841 Update readme
0e05d35 Add translations and icon
aa1ecd6 Update plan.md with P25B85 integration details and protocol summary
95aa45d Implement capture findings
4a80f9a Review implementation
169e898 Ignore .local dir
3a12ab4 Fix entity metadata and bridge connectivity
c5800f3 Refresh docs and UV ozone labels
3793989 data: Phase 4 command frame captures
e71a9b4 feat: Phase 4 analysis and CRC reverse-engineering tools
5385c17 feat: write support — light switch and pump buttons
c5dadea feat: temperature command capture tool
c41053e docs: update plan with Phase 4 findings
b344095 docs: clarify CRC status
```

### PR 2: Climate, fan, sensors & Phase 5 (CRC cracking)
**Commits:** `da74e4f`..`e912ff4`
**Scope:** Temperature capture completion, climate entity, fan platform, heater/pump state sensors, spa clock, Phase 5 capture, heater/blower switches, CRC cracked + dynamic command generation becomes possible, error handling & tests.

```
da74e4f Adjust temperature commands capture script
b9d664f feat: enhance temp capture script
73eb4a7 feat: CRC analysis tools and temperature lookup table
ef94f36 feat: implement climate entity
1f19e35 feat: refactor entity descriptions + fan platform
51f2d53 feat: heater and pump state sensors
a3dfded feat: review and fix findings
5613203 feat: timezone-aware spa clock
a1aaca6 feat: Phase 5 extended command frame capture
7cbd46d feat: spa clock timezone rendering
7ad7078 feat: datetime labels localization
35fd0f8 chore: gitignore capture bin files
a58a5a2 feat: heater and blower switch entities
230e1f1 tools: CRC capture + analysis tools
76e68e5 docs: Phase 5 results
9a1efbe feat: CRC capture with config commands
3e1d7e7 feat: Crack the CRC
d60548c docs: Update plan & readme
8bb05f3 docs: CRC analysis documentation
18d454f docs: Clean up protocol documentation
a99b48a docs: CRC capture notes
e912ff4 feat: error handling and test coverage
```

### PR 3: Schedule, DateTime, dynamic commands & Phase 6
**Commits:** `46799a1`..`df2f342`
**Scope:** Schedule time entities, clock sync button, dynamic CRC-based command building (removes all replay constants), ozone entity, schedule flags byte, Phase 6 capture + analysis, options flow, entity renaming for consistency, icons.

```
46799a1 feat: schedule time entities + enable switches
9cad9e4 feat: sync clock button entity
ab2575c docs: schedule broadcast encoding
01672d0 docs: README schedule + clock sync
e1de54d docs: EW11/schedule debug tools
592eb68 docs: schedule slot enable encoding
2db0e80 docs: implementation rules for schedule
94a0fa6 refactor: rename entities/data keys for consistency
247c811 feat: dynamic state-dependent icons
095a41f test: align tests with renamed data keys
e79f816 feat: live test plan + guided write test
7f139b5 docs: entity refactor README
8c62ba9 tools: guided capture for phase 6
4ac5189 Add Phase 6 capture analysis
bd5d9c2 Adapter: schedule flags table
b14c066 Schedule enable/disable: flags byte
160c9c6 Tests: schedule flags verification
c3cfbb2 guided_write_test: flags byte
db8bfd6 protocol.md: Phase 6 findings
df2f342 Docs: update plan for Phase 6
```

### PR 4: Dynamic commands, ozone control & live write fixes
**Commits:** `8ae7cf1`..`92d0699`
**Scope:** Dynamic command builders (remove CMD_* replay constants), ozone entity with mode control, pump simplification, terminology alignment, live write-test fixes (temp 0x98, blower OFF, date-write 0x05), safety hardening (no auto writes on startup), community feedback fixes, options flow.

```
8ae7cf1 docs: Clean up redundant information
e27acc8 feat: Revised guided write test
1260a7a feat: build all commands dynamically + ozone entity
79ed2fa i18n: "Ozone" terminology
3bdeae9 docs+tests: dynamic commands + ozone entity
49aa398 refactor: rename disinfection → ozone
d2bec1e feat: options flow (ozone mode, auto clock sync)
25a989d fix: fan entity and pump transitions
96e1924 docs: update plan for live testing
e45b475 fix: icons placement for HACS
1ea5e0b fix: strip blower flag from heater byte
00af8ec fix: light toggle broadcast delay
0521c3c feat: overhaul guided write-test script
92f4740 docs: session 3 findings
2db860d fix: write-test stale buffer, frame parsing
11d4343 fix: temp 0x98, blower OFF 0x00, clock date
3742f66 feat: pump retry logic
2c0fab1 test: adapter tests for temp + blower
8bdd8d1 docs: session 4 live test results
9ed7ca6 docs: community feedback TODO
94767e3 chore: remove reply draft
1323269 docs: safety warning
0d5509d tools: date-write test script
59af81c docs: session 5 results
5357843 fix: date-write prefix 0x05
7a0e54e docs: DateTime prefix byte
a95bb5d docs: separate protocol from plan
a117a08 safety: no auto writes on startup
d842686 refactor: pump target-state commands
ab1926d feat: consistent logging
af2ac07 feat: ozone mode from broadcast byte 13
92d0699 docs: session 7 outcomes
```

### PR 5: Resilient UI — persistent connection + optimistic state
**Commits:** `77e15d8`..`85bef1c`
**Scope:** Persistent TCP connection with background reader loop, optimistic state for all writable entities, JoyonwayCoordinatorEntity base class, coordinator resilience tests, environment cleanup, runtime polish, blower/ozone UX refinements, community feedback consolidation.

```
77e15d8 docs: resilient UI implementation handoff
0229bbd docs: refresh project handoff priorities
170d62e feat: persistent TCP connection + background reader loop
678873b feat: optimistic state for all writable entities
246ebe5 refactor: JoyonwayCoordinatorEntity base for sensors
01c5e71 test: coordinator resilience + optimistic state tests
aa442c7 docs: resilient UI completion
ef61cbf chore: merge .venv-ha into .venv
8a868d4 Polish runtime behavior and entity internals
658e366 Update docs for strict connectivity
f8cd28a Disable blower switch by default
77d1204 Hide ozone switch in auto mode
0a140f3 Consolidate community feedback
85bef1c Polish blower terminology
```

### PR 6: Optimistic confirmation + schedule slot 2 fix + heating cycle
**Commits:** `7527779`..`a660a90`
**Scope:** Fork divergence analysis, optimistic state confirmation fix (snap-back bug), schedule freshness gating, standby status remap, schedule slot 2 write bug investigation + fix (force-write flags), heating cycle detection, capture tooling.

```
7527779 Add fork divergence checklist
14c4a61 fix: optimistic state confirmation before clearing pending
26e6ec3 feat: schedule freshness gating
5de78b7 docs: session 14 plan + README
80ac42f docs: repo rename todo
853eccc fix: remap 0x50 to standby
75bb3cb docs: standby status finding
bfda4bf docs: schedule slot 2 bug + capture tool
845e70d feat: detect circulation via byte 17 heating cycle flag
83a593d tools: heating cycle capture + analysis
a660a90 docs: session 17 plan + README
```

### PR 7: Schedule slot 2 fix + Intent Queue + final polish
**Commits:** `bccd484`..`fa4502a`
**Scope:** Schedule slot write tests, force-write `0x5A` implementation, schedule state/time flag split, live test matrix, intent queue (coalesce rapid actions, sequential drain, auto-cancel), removal of upstream leftovers, trademark notice.

```
bccd484 tools: schedule slot write test & capture scripts
879300b fix: always use 0x5A for both-disabled writes
8acb68e docs: schedule force-write flags
02b5807 test: verify 0x5A for all cases
a1d2eaa fix: correct s1-on/s2-off comment
71d0a82 Split schedule state/time write flags
7903250 Organize live test tooling and docs
a1d853f feat: Intent Queue (#2)
a1d02a3 chore: remove upstream packages/spa_consigne_lock.yaml
fa4502a docs: trademark disclaimer + liability notice
```

---

## Step 3: Domain rename PR (PR 8 — optional, after all above)

After all 7 PRs are merged, create one final PR that renames:
- `custom_components/joyonway_p25b85/` → `custom_components/joyonway/`
- Domain in `const.py`, `manifest.json`, `hacs.json`, `config_flow.py`, `__init__.py`
- All test imports
- All translation references
- Update README attribution

This is a clean, reviewable rename-only PR on the new repo.

---

## Execution Commands (step by step)

```bash
# === SETUP ===
# Create and clone new repo from GitHub
gh repo create alexbde/ha-joyonway --public \
  --description "Home Assistant integration for Joyonway spa controllers (P25B85)" \
  --clone

cd ha-joyonway

# Add old repo as a remote source for commit replay
git remote add old ../ha-joyonway-p25b85
git fetch old

# === STEP 1: Initial commit ===
# Export tree snapshot from old repo and import into new repo
cd /Users/alex/IdeaProjects/alexbde/ha-joyonway-p25b85
git archive 32e4e9a | tar -x -C ../ha-joyonway/

cd /Users/alex/IdeaProjects/alexbde/ha-joyonway
git add -A
git commit -m "Initial commit: upstream fork + P25B85 integration scaffold

Snapshot from alexbde/ha-joyonway-p25b85 (20be69d..32e4e9a).
Original upstream: KnapTheBuilder/ha-joyonway-p23b32.
P25B85 integration scaffolding added via PR #1."
git push origin main

# === STEP 2: PRs ===
# For each PR, create a branch and replay non-merge commits in order:

# PR 1 example:
git checkout -b pr/core-integration-phase1-4
git rev-list --reverse --no-merges 051c05b^..b344095 | xargs git cherry-pick
# Resolve any conflicts, then:
git push origin pr/core-integration-phase1-4
gh pr create --title "feat: Core integration + protocol capture (Phase 1–4)" \
  --body "Content replayed from old repo (non-merge commits only). See PR commit tab for full history context."
# On GitHub: squash & merge

# Repeat for PR 2–7 with respective ranges.
# After each merge, rebase the next branch on updated main.
```

---

## Branch names for PRs

All ranges below are replayed as **non-merge commits only**.

| # | Branch | Commit range (old repo) |
|---|--------|------------------------|
| 1 | `pr/core-integration-phase1-4` | `051c05b..b344095` |
| 2 | `pr/climate-fan-sensors-crc` | `da74e4f..e912ff4` |
| 3 | `pr/schedule-datetime-phase6` | `46799a1..df2f342` |
| 4 | `pr/dynamic-commands-ozone-safety` | `8ae7cf1..92d0699` |
| 5 | `pr/resilient-ui-persistent-tcp` | `77e15d8..85bef1c` |
| 6 | `pr/optimistic-confirmation-heating-cycle` | `7527779..a660a90` |
| 7 | `pr/schedule-slot2-intent-queue` | `bccd484..fa4502a` |
| 8 | `pr/domain-rename-joyonway` | (new commits in new repo) |

---

## Notes

- **Replay conflicts** are expected between PRs since later commits modify the same files. Each PR branch should be based on the previous PR's squash-merged `main`. Do them sequentially.
- **Merge commits:** replay non-merge commits only (for example with `git rev-list --reverse --no-merges <range> | xargs git cherry-pick`).
- **Alternative replay method:** Use `git format-patch` + `git am`:
  ```bash
  # In old repo, generate patches:
  git format-patch --no-merges 32e4e9a..b344095 -o /tmp/patches-pr1
  # In new repo, apply:
  git am /tmp/patches-pr1/*.patch
  ```
- **Tools directory** (`tools/`) contains capture scripts and data. These are included as-is — they document the reverse-engineering process.
- **Tests** should pass at each PR boundary (or at least at PR 7). Minor test breakage in intermediate PRs is acceptable since this is content-only replay.
- **History visibility:** commit-by-commit history is expected to be reviewed from GitHub PR commit tabs; migration branches can be deleted after merge.
- The domain rename (PR 8) is the only PR that modifies file paths — keep it separate and last for clean review.

