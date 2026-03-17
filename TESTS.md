# AeroFTP GitHub Provider — Test Results

> Date: 17 March 2026
> Provider version: Phase 1 MVP
> Tester: Claude Opus 4.6 (automated)
> Repository: axpnet/aeroftp-test-playground

---

## Test Environment

- **AeroFTP CLI**: v2.9.9 (debug build)
- **Rust**: 1.85+
- **OS**: Ubuntu Linux 6.17.0
- **GitHub API**: REST v3 + GraphQL
- **Auth**: Fine-grained PAT via `gh auth token`

---

## Test Results

### Test 1: List repository root

```bash
aeroftp ls "github://token@axpnet/aeroftp-test-playground" / -l
```

**Result: PASS**

```
-rw-r--r--     295 B  -  README.md

1 items (0 directories, 1 files) — 295 B total
```

---

### Test 2: Stat file (JSON)

```bash
aeroftp stat "github://token@axpnet/aeroftp-test-playground" /README.md --json
```

**Result: PASS**

```json
{
  "status": "ok",
  "entry": {
    "name": "README.md",
    "path": "README.md",
    "is_dir": false,
    "size": 295,
    "modified": null,
    "permissions": null,
    "owner": "axpnet"
  }
}
```

---

### Test 3: Cat file (read content to stdout)

```bash
aeroftp cat "github://token@axpnet/aeroftp-test-playground" /README.md
```

**Result: PASS**

Full file content printed to stdout, pipe-friendly (no extra output on stdout).

---

### Test 4: Upload file (creates Git commit)

```bash
echo "Test from AeroFTP CLI - mar 17 mar 2026, 13:03:47, CET" > /tmp/gh_test.txt
aeroftp put "github://token@axpnet/aeroftp-test-playground" /tmp/gh_test.txt /test/hello.txt
```

**Result: PASS**

```
/tmp/gh_test.txt → /test/hello.txt (55 B, 73 B/s, 0.7s)
```

Verified on GitHub: commit `12a8aa06` "Create hello.txt" created on `main` branch.

---

### Test 5: List subdirectory

```bash
aeroftp ls "github://token@axpnet/aeroftp-test-playground" /test/ -l
```

**Result: PASS**

```
-rw-r--r--      55 B  -  hello.txt

1 items (0 directories, 1 files) — 55 B total
```

---

### Test 6: Download file

```bash
aeroftp get "github://token@axpnet/aeroftp-test-playground" /test/hello.txt /tmp/gh_downloaded.txt
```

**Result: PASS**

```
/test/hello.txt → /tmp/gh_downloaded.txt (55 B, 70 B/s, 0.8s)
```

Downloaded content matches uploaded content byte-for-byte.

---

### Test 7: Delete file (creates Git commit)

```bash
aeroftp rm "github://token@axpnet/aeroftp-test-playground" /test/hello.txt
```

**Result: PASS**

```
Deleted: /test/hello.txt
```

Verified on GitHub: commit `6d28416a` "Delete hello.txt" created on `main` branch.

---

### Test 8: Connect (server info)

```bash
aeroftp connect "github://token@axpnet/aeroftp-test-playground"
```

**Result: PASS**

```
Connected to axpnet/aeroftp-test-playground (GitHub)
  User:     token
  Port:     443
  Protocol: GitHub
  Server:   GitHub axpnet/aeroftp-test-playground | Branch: main | direct push | Private: false | Size: 0 KB | API: 4925/5000 remaining
  Time:     971ms
```

Displays: branch, write mode, visibility, repo size, rate limit.

---

### Test 9: Tree

```bash
aeroftp tree "github://token@axpnet/aeroftp-test-playground" /
```

**Result: PASS**

```
/
└── README.md

0 directories, 1 files
```

---

## Unit Tests

```
cargo test --lib github
```

**Result: 24/24 PASS**

```
test providers::github::errors::tests::... ok (5 tests)
test providers::github::graphql::tests::... ok (7 tests)
test providers::github::rate_limit::tests::... ok (4 tests)
test providers::github::releases_mode::tests::... ok (6 tests)
test providers::github::repo_mode::tests::... ok (2 tests)

test result: ok. 24 passed; 0 failed; 0 ignored
```

---

## Clippy

```
cargo clippy --all-targets -- -D warnings
```

**Result: PASS** — zero warnings, zero errors.

---

## Git Commits Created During Tests

```
6d28416a Delete hello.txt
12a8aa06 Create hello.txt
d87e4ca3 Initial commit: test playground for AeroFTP GitHub integration
```

All commits are real Git commits on the `main` branch, created via the GitHub Contents API through AeroFTP CLI.

---

## Summary

| Test | Operation | Status |
|------|-----------|--------|
| 1 | List root directory | PASS |
| 2 | Stat file (JSON) | PASS |
| 3 | Cat file (stdout) | PASS |
| 4 | Upload → Git commit | PASS |
| 5 | List subdirectory | PASS |
| 6 | Download file | PASS |
| 7 | Delete → Git commit | PASS |
| 8 | Connect (server info) | PASS |
| 9 | Tree | PASS |
| — | 24 unit tests | PASS |
| — | Clippy | PASS |

**Overall: 9/9 live tests + 24 unit tests + clippy = ALL PASS**

---

*This repository is maintained as a test playground for AeroFTP GitHub provider development.*
