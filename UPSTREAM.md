# UPSTREAM.md — ts-sdk (@bsv/sdk)

_Last audited: 2026-05-29 (read-only git analysis)_

## Remotes (BOTH configured)
- **origin:**   `https://github.com/kryp2/ts-sdk.git`   (personal fork — the PR push target)
- **upstream:** `https://github.com/bsv-blockchain/ts-sdk.git`  (canonical BSV SDK)

The `kryp2` origin is currently a **verbatim mirror** of upstream: every
`refs/remotes/origin/*` SHA in this clone matches its `refs/remotes/upstream/*`
counterpart exactly (e.g. both `origin/master` and `upstream/master` = `76676e70`).
origin therefore adds no committed divergence today; it is just where you push
branches to open PRs against bsv-blockchain.

## State at audit time
- **Current branch:** `fix/validate-create-action-options-trustself`
- **Committed level:** branch HEAD = master = origin/master = upstream/master =
  `76676e70` (`Merge PR #504 ... dependabot/flatted-3.4.2`, by Ty Everett).
  **0 ahead / 0 behind upstream/master at the commit level.**
- **In-flight work lives in the WORKING TREE, not in a commit.** `git status`:
  ```
   M package-lock.json                            (-665 lines)
   M src/wallet/__tests/validationHelpers.test.ts (+10 lines)
   M src/wallet/validationHelpers.ts              (+1 line)
  ```
- Package version: **`@bsv/sdk` 2.0.14** (2.0.x line; latest tag in clone v2.0.13).
  (Earlier guesses of 1.7.x were wrong — this is 2.x.)

## Classification
**vendored-fork-with-mods** — at the commit level it is a clean, exactly-current
checkout of `bsv-blockchain/ts-sdk` master with a contribution fork (`kryp2`)
wired up, BUT there is real uncommitted local work staged for a PR. The work is
not yet a commit, so it is invisible to log/merge-base; treat the working tree
as the source of our delta until it is committed onto the feature branch.

## OUR MODS (uncommitted, in working tree)
Intended as the PR behind branch name `fix/validate-create-action-options-trustself`.
1. **src/wallet/validationHelpers.ts** (+1 line) — a tweak in/around
   `validateCreateActionOptions` (the `trustSelf` handling region, ~lines 440-466;
   `trustSelf: o.trustSelf` at 446, `trustSelf?: TrustSelf` at 466 are already
   present). The change validates/preserves the `trustSelf: 'known'` option.
2. **src/wallet/__tests/validationHelpers.test.ts** (+10 lines) — new test(s)
   covering trustSelf handling (existing tests at ~490 already check
   "preserves trustSelf when set to known" / "leaves trustSelf undefined").
3. **package-lock.json** (-665 lines) — lockfile shrunk. Likely an incidental
   regeneration / dependency pruning. NOT obviously part of the feature; review
   before committing so it does not piggyback on the trustSelf PR. (CLAUDE.md:
   never pin-strip without a deliberate reason.)

No fork-only commits exist; everything in git history is upstream
(bsv-blockchain contributors + dependabot + GitHub merges).

## Upstream activity
Active and maintained. GitHub `bsv-blockchain/ts-sdk`: pushed_at
2026-05-11, archived=false, disabled=false, open_issues=2, 70 stars,
default_branch master. npm `@bsv/sdk` latest = **2.1.4** (published 2026-05-26)
— so upstream has already moved past our local 2.0.14 onto the 2.1.x line.
The TypeScript SDK is the canonical first-class variant (not deprecated, not a
Go rewrite). Latest GitHub *release* tag is v2.0.0 "Chronicle" (releases lag the
npm/tag cadence).

## Recommendation: **track-and-rebase**
- Commit the trustSelf change as ONE isolated commit on
  `fix/validate-create-action-options-trustself`, push to the `kryp2` origin,
  open a PR against `bsv-blockchain:master`. Once merged, our delta goes to zero.
- Keep it a single additive commit so future rebases stay mechanical. The only
  real conflict surface is `src/wallet/validationHelpers.ts`.
- Decide deliberately about the `package-lock.json` -665 change — either drop it
  (`git checkout -- package-lock.json`) or split it into its own commit; do not
  bundle a silent lockfile shrink into the feature PR.
- We are slightly behind upstream (local 2.0.14 vs npm 2.1.4). Rebase the patch
  onto fresh `upstream/master` before opening the PR.

## How to sync next time (read-only first)
```bash
# 1. Fetch upstream (does not touch your branch / working tree)
git -C ts-sdk fetch upstream master < /dev/null

# 2. Confirm committed divergence (should be 0/0 unless you've committed the patch)
git -C ts-sdk rev-list --left-right --count upstream/master...fix/validate-create-action-options-trustself < /dev/null

# 3. Commit the working-tree patch onto the feature branch (review the lockfile first!)
git -C ts-sdk diff src/wallet/validationHelpers.ts < /dev/null
git -C ts-sdk checkout -- package-lock.json < /dev/null   # if the -665 is unwanted

# 4. Rebase the single feature commit onto current upstream
git -C ts-sdk rebase upstream/master fix/validate-create-action-options-trustself < /dev/null
#    Conflict surface limited to src/wallet/validationHelpers.ts.

# 5. Push to YOUR fork (origin = kryp2) and open the PR upstream
git -C ts-sdk push origin fix/validate-create-action-options-trustself < /dev/null
```
Reminder: every command in this repo must end with `< /dev/null` (the repo hangs
on open stdin otherwise).
