# 🔒 Triage Record — nodejs/undici #1373

> **Date:** June 30, 2026 | **By:** [SNTL84](https://github.com/SNTL84)

## Issue Details

| Field | Value |
|-------|-------|
| **Repo** | [nodejs/undici](https://github.com/nodejs/undici) |
| **Issue** | [#1373 — Default fetch timeout too short](https://github.com/nodejs/undici/issues/1373) |
| **Label** | `enhancement` · `good first issue` |
| **Priority** | Medium |
| **Effort** | ~3 hours |
| **Category** | Correctness / Regression Safety / Documentation |
| **Triage Comment** | [View Live →](https://github.com/nodejs/undici/issues/1373#issuecomment-4842427744) |
| **PR Linkback Comment** | [View Live →](https://github.com/nodejs/undici/issues/1373#issuecomment-4842448990) |
| **Pull Request** | [PR #5467 →](https://github.com/nodejs/undici/pull/5467) |
| **Fork** | [SNTL84/undici](https://github.com/SNTL84/undici) |
| **Branch** | [`fix/issue-1373-default-timeout-test`](https://github.com/SNTL84/undici/tree/fix/issue-1373-default-timeout-test) |

---

## Problem Statement

The original reporter flagged that undici's default `fetch` timeout was **30 seconds**, causing failures for slow-responding servers. Chrome's default is **300 seconds**. This mismatch caused real-world breakages documented in related issue [#1248](https://github.com/nodejs/undici/issues/1248).

**Impact:**
- Any Node.js application using undici's `fetch` to call slow upstream APIs (e.g., large file downloads, slow database-backed endpoints, third-party AI APIs) would receive `HeadersTimeoutError` after just 30 seconds
- No regression test existed to prevent the fix from silently reverting in future releases
- Documentation in `getting-started.md` was ambiguous about the current default

**Ecosystem Cost Estimate:**
| Cost Type | Impact |
|-----------|--------|
| Developer time lost debugging spurious timeouts | High — silent failure, hard to trace |
| Production incidents from 30s cutoff on slow APIs | Medium-High |
| Trust in Node.js fetch compatibility with browser | Medium |
| OSS maintenance debt (no regression test) | Medium |

---

## Revenue / Monetisation Angle

| Opportunity | Detail |
|-------------|--------|
| **Consulting** | Teams hitting this issue in production often need paid help diagnosing HTTP client config |
| **OSS Visibility** | nodejs/undici is a Tier-1 Node.js core dependency — contributor credit is high-signal |
| **Bounty Potential** | No direct bounty, but contributor reputation on core Node.js infrastructure is bankable |
| **Profile Authority** | Demonstrates timeout/HTTP internals knowledge — relevant for backend consulting leads |

---

## Root Cause

The `headersTimeout` and `bodyTimeout` defaults in [`lib/dispatcher/client.js`](https://github.com/nodejs/undici/blob/main/lib/dispatcher/client.js) were historically set to `30e3` ms. The fix to `300e3` was applied but:

1. No regression test was added to lock the fix in
2. The `fetch()` global surface (via `setGlobalDispatcher`) did not explicitly document whether it inherits these defaults
3. `getting-started.md` timeout section lacked explicit default value callout

**Relevant implementation files:**
- [`types/client.d.ts`](https://github.com/nodejs/undici/blob/main/types/client.d.ts) — `Default: 300e3 ms` in JSDoc
- [`docs/docs/api/Client.md`](https://github.com/nodejs/undici/blob/main/docs/docs/api/Client.md) — `bodyTimeout` default `300e3`
- [`lib/dispatcher/client-h1.js`](https://github.com/nodejs/undici/blob/main/lib/dispatcher/client-h1.js) — runtime `kHeadersTimeout` enforcement
- [`lib/core/symbols.js`](https://github.com/nodejs/undici/blob/main/lib/core/symbols.js) — `kHeadersTimeout` / `kBodyTimeout` symbols

---

## Solution Delivered

### 1. Maintainer-Grade Triage Comment

Posted a full analysis on the issue thread:
- Confirmed the fix exists in current codebase with citations to 4 source files
- Identified the missing regression test as the remaining gap
- Provided a ready-to-run test case following undici's own test conventions
- Offered to open a PR — then did

### 2. Regression Test — `test/issue-1373-default-timeout.js`

```js
'use strict'
// Regression test for https://github.com/nodejs/undici/issues/1373
const { createServer } = require('node:http')
const { once } = require('node:events')
const { test, after } = require('node:test')
const assert = require('node:assert/strict')
const { fetch } = require('..')

test('default headersTimeout is 300s — no timeout for 35s slow server (issue #1373)', async (t) => {
  const server = createServer((req, res) => {
    setTimeout(() => {
      res.writeHead(200, { 'content-type': 'text/plain' })
      res.end('ok')
    }, 35_000) // 35s — exceeds old 30s default, within new 300s default
  })
  server.listen(0)
  await once(server, 'listening')
  after(() => server.close())
  const { port } = server.address()
  const res = await fetch(`http://localhost:${port}/`)
  assert.equal(res.status, 200)
  assert.equal(await res.text(), 'ok')
})
```

**Why this works:**
- Fails loudly if `headersTimeout` ever reverts to 30s (throws `HeadersTimeoutError`)
- Passes silently under the correct 300s default
- Uses `node:test` + `node:http` — exact same pattern as `test/issue-5087.js` in the repo

---

## Session Timeline — Jun 30, 2026

| Time (IST) | Action | Link |
|------------|--------|------|
| 3:58 PM | 🔍 Issue read — root cause analysis, codebase scan | [Issue #1373](https://github.com/nodejs/undici/issues/1373) |
| 4:00 PM | 💬 Triage comment posted — citations, test case, PR offer | [Comment →](https://github.com/nodejs/undici/issues/1373#issuecomment-4842427744) |
| 4:02 PM | 🔎 Code search — confirmed `300e3` defaults in `types/client.d.ts`, `Client.md`, `symbols.js` | [Search →](https://github.com/nodejs/undici/blob/main/types/client.d.ts) |
| 4:04 PM | 🍴 Fork initiated — `nodejs/undici` → `SNTL84/undici` | [Fork →](https://github.com/SNTL84/undici) |
| 4:04 PM | 🌿 Branch created — `fix/issue-1373-default-timeout-test` | [Branch →](https://github.com/SNTL84/undici/tree/fix/issue-1373-default-timeout-test) |
| 4:04 PM | 🧪 Regression test pushed — `test/issue-1373-default-timeout.js` | [File →](https://github.com/SNTL84/undici/blob/fix/issue-1373-default-timeout-test/test/issue-1373-default-timeout.js) |
| 4:06 PM | 🚀 PR opened → `nodejs/undici:main` | [PR #5467 →](https://github.com/nodejs/undici/pull/5467) |
| 4:06 PM | 📝 PR linkback comment posted on issue thread | [Comment →](https://github.com/nodejs/undici/issues/1373#issuecomment-4842448990) |
| 4:11 PM | 📋 Triage record added to open-issue-triage repo | [This file →](https://github.com/SNTL84/open-issue-triage/blob/main/triages/nodejs-undici-1373-default-timeout-regression.md) |

---

## Acceptance Criteria Status

- [x] Root cause identified and cited with live source file links
- [x] Triage comment posted with test case + implementation references
- [x] Fork created from `nodejs/undici`
- [x] Regression test written following undici's `node:test` convention
- [x] PR #5467 opened against `nodejs/undici:main`
- [x] PR linked back in the issue thread
- [x] Triage record added to `open-issue-triage`

---

## All Links — undici Session

| Asset | Link |
|-------|------|
| 📌 Issue | [nodejs/undici #1373](https://github.com/nodejs/undici/issues/1373) |
| 💬 Triage Comment | [#issuecomment-4842427744](https://github.com/nodejs/undici/issues/1373#issuecomment-4842427744) |
| 📎 PR Linkback | [#issuecomment-4842448990](https://github.com/nodejs/undici/issues/1373#issuecomment-4842448990) |
| 🍴 Fork | [SNTL84/undici](https://github.com/SNTL84/undici) |
| 🌿 Branch | [fix/issue-1373-default-timeout-test](https://github.com/SNTL84/undici/tree/fix/issue-1373-default-timeout-test) |
| 🧪 Test File | [test/issue-1373-default-timeout.js](https://github.com/SNTL84/undici/blob/fix/issue-1373-default-timeout-test/test/issue-1373-default-timeout.js) |
| 🚀 Pull Request | [nodejs/undici #5467](https://github.com/nodejs/undici/pull/5467) |

---

## Contributor Signature

> | **SNTL 84** | Agentic AI Workflow Professional |
> Lead Generation · Fulfillment Automation · Full-Stack Builds · AI Workflows · Supply Chain BI
>
> 🌐 [desidevloper.com](https://desidevloper.com) · 💬 [WhatsApp](https://wa.me/919727413309) · 🔗 [LinkedIn](https://linkedin.com/in/sntl2784) · 💻 [GitHub](https://github.com/SNTL84) · 📸 [@desibiztrade](https://www.instagram.com/desibiztrade) · 🔴 [YouTube @SNTL84](https://youtube.com/@SNTL84)
>
> 🚀 *Follow for practical AI automation insights & founder systems.*
