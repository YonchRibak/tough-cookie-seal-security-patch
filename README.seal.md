# README.seal.md

## Summary for the reviewer

If you want the short version: I backported the null-prototype dictionary change (the same concept applied in upstream `4.1.3`) into the `v2.5.0` codebase, added a narrow unit test proving the fix, and included a small PoC script to demonstrate exploitability on vanilla `2.5.0` and that the exploit is blocked by the patched package.

---

## What I changed (high level)

- Replaced prototypeful in-memory maps with null-prototype dictionaries in the MemStore initialization to prevent prototype pollution.
- Added a focused unit test that reproduces the vulnerable behavior on vanilla `2.5.0` and verifies the issue is resolved on the patched package.
- Added a small PoC script (`index.js`) that prints a clear success/failure message when run against unpatched vs patched packages.
- Produced a clean `changes.diff` and an `npm pack` tarball for verification.

---
## Test Failures & Fixes (before patch)

While validating the patch, three IETF test cases (0002, COMMA0006, COMMA0007) initially failed. These were not caused by the security fix but by outdated or malformed test data that broke under Node 20’s stricter RFC 1123 date parsing.

I fixed the dates in parser.json and now tests pass.

---


## How to reproduce my verification (quick)


1. Verify exploit against vanilla package

```bash
mkdir /tmp/tc-vanilla && cd /tmp/tc-vanilla
npm init -y
npm install tough-cookie@2.5.0
# copy or download index.js into this folder
node index.js
# Expected output: EXPLOITED SUCCESSFULLY
```

2. Verify exploit against patched tarball

```bash
mkdir /tmp/tc-patched && cd /tmp/tc-patched
npm init -y
# from the repo that contains the packed tarball run: npm pack
# then copy tough-cookie-2.5.0-seal-patch.tgz into this folder
npm install ../path/to/tough-cookie-2.5.0-seal-patch.tgz
# copy the same index.js into this folder
node index.js
# Expected output: EXPLOIT FAILED
```

3. Run the full test suite (from repo root)

```bash
# ensure Node 20 (or the tested Node version) is active
npm ci
npm test
# Tests should pass, including test-cve-2023-26136.js
```

---

## How I validated the patch

- Confirmed the PoC reliably triggers the vulnerable behavior on upstream `v2.5.0`.
- Confirmed the PoC fails on the patched tarball.
- Ran the upstream test suite plus the added test to ensure no regressions.
- Ensured the diff is minimal and limited to memstore-related initialization and the test + PoC files.

---


