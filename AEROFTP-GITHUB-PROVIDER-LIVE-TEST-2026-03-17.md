# AeroFTP GitHub Provider Live Test Report

Date: 2026-03-17
Repository under test: `axpnet/aeroftp-test-playground`
Local source commit: `2cf2b7ab328a04885f4292d57fae3c8e06f066c9`
Binary under test: `src-tauri/target/debug/aeroftp-cli`
Authentication mode: `gh auth token` piped via `--password-stdin`

## Scope

This report records the live playground checks executed against the AeroFTP GitHub provider after the review-driven fixes applied on 2026-03-17.

The goal was to verify that the provider behaves correctly for:

- GitHub URL parsing
- connection establishment
- repository root listing
- file metadata retrieval
- file content reading
- repository write path, using this report itself as the uploaded artifact

## Executed live checks

### 1. Connect

Command pattern:

```bash
printf '%s\n' "$TOKEN" | "$CLI" --password-stdin connect 'github://token@axpnet/aeroftp-test-playground'
```

Result: PASS

Observed values:

- Repository: `axpnet/aeroftp-test-playground`
- Branch: `main`
- Write mode: `direct push`
- Visibility: `public`
- Reported size: `3 KB`
- Remaining API quota during run: `4867/5000`
- Connection time: about `972 ms`

### 2. List repository root as JSON

Command pattern:

```bash
printf '%s\n' "$TOKEN" | "$CLI" --password-stdin ls 'github://token@axpnet/aeroftp-test-playground' / --json
```

Result: PASS

Observed entries at test time:

- `README.md`
- `TESTS.md`

JSON summary returned:

- `total = 2`
- `files = 2`
- `dirs = 0`
- `total_bytes = 4784`

### 3. Stat README.md

Command pattern:

```bash
printf '%s\n' "$TOKEN" | "$CLI" --password-stdin stat 'github://token@axpnet/aeroftp-test-playground' /README.md --json
```

Result: PASS

Observed values:

- `name = README.md`
- `path = README.md`
- `is_dir = false`
- `size = 295`
- `owner = axpnet`

### 4. Read README.md content

Command pattern:

```bash
printf '%s\n' "$TOKEN" | "$CLI" --password-stdin cat 'github://token@axpnet/aeroftp-test-playground' /README.md
```

Result: PASS

The provider returned the expected Markdown content beginning with:

```md
# AeroFTP Test Playground
```

and the repository purpose bullets were read successfully.

### 5. Upload this report to the playground repository

Command pattern:

```bash
printf '%s\n' "$TOKEN" | "$CLI" --password-stdin put \
  'github://token@axpnet/aeroftp-test-playground' \
  'docs/dev/audit/PLAYGROUND-GITHUB-LIVE-TEST-2026-03-17.md' \
  '/AEROFTP-GITHUB-PROVIDER-LIVE-TEST-2026-03-17.md'
```

Result: PASS

Observed values:

- Remote path: `/AEROFTP-GITHUB-PROVIDER-LIVE-TEST-2026-03-17.md`
- Uploaded size: `3878 bytes`
- Transfer speed observed by CLI: about `5.2 KB/s`
- Transfer time observed by CLI: about `0.7 s`

This step was the intentional live write verification. The report itself is now the proof artifact stored in the playground repository.

### 6. Verify uploaded report via stat and cat

Command patterns:

```bash
printf '%s\n' "$TOKEN" | "$CLI" --password-stdin stat \
  'github://token@axpnet/aeroftp-test-playground' \
  '/AEROFTP-GITHUB-PROVIDER-LIVE-TEST-2026-03-17.md' --json
```

```bash
printf '%s\n' "$TOKEN" | "$CLI" --password-stdin cat \
  'github://token@axpnet/aeroftp-test-playground' \
  '/AEROFTP-GITHUB-PROVIDER-LIVE-TEST-2026-03-17.md'
```

Result: PASS

Observed values from remote verification:

- `name = AEROFTP-GITHUB-PROVIDER-LIVE-TEST-2026-03-17.md`
- `path = AEROFTP-GITHUB-PROVIDER-LIVE-TEST-2026-03-17.md`
- `is_dir = false`
- `size = 3878`
- `owner = axpnet`

The first 30 lines were read back successfully from GitHub after upload.

## Notes

- The live verification was performed after fixing GitHub CLI parsing for `github://owner/repo@branch`.
- Root JSON listing no longer emits double-slash paths after rebuilding the real CLI binary.
- Protected-branch behavior is now surfaced honestly as read-only until a real branch workflow or PR flow is implemented.
- Release assets are now wired through a virtual releases directory in the provider instead of remaining dead code.

## Expected final outcome

The playground repository now contains this file:

`/AEROFTP-GITHUB-PROVIDER-LIVE-TEST-2026-03-17.md`

That file is intended to remain in the repository as a permanent, auditable artifact of the GitHub provider live test session.

## Final status

Live playground verification completed successfully.

- Read path: PASS
- Metadata path: PASS
- Content read path: PASS
- Write path: PASS
- Remote artifact verification: PASS