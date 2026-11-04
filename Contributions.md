# Open Source Contributions

Contributions across vector databases, web frameworks, and community projects, with the work and PR links in one place.

| Project | Role | Area | Status |
|---|---|---|---|
| [LanceDB](#lancedb) | Contributor | Embedded vector DB: Rust client, Python SDK, docs | 3 PRs merged |
| [Litestar](#litestar) | Contributor | Python ASGI web framework | 1 merged, 1 in review |
| [CanonForces (OpenLake)](#canonforces--openlake) | Lead maintainer | Full-stack competitive programming platform | Ongoing |
| [SimPPL / C4GT](#simppl--code-for-govtech-c4gt) | Contributor (Sprint 3) | Social media data extraction pipeline | Completed |

---

## LanceDB

[lancedb/lancedb](https://github.com/lancedb/lancedb): open-source embedded vector database for AI applications.

### 1. Reject pagination cycles in `RemoteDatabase::list_jobs`
- **Issue:** [#4138](https://github.com/lancedb/lancedb/issues/4138) | **PR:** [#4139](https://github.com/lancedb/lancedb/pull/4139)
- **Problem:** If the server returned the same `page_token` repeatedly, `list_jobs` kept requesting the same page until the 100-page limit. It then returned `Ok(...)` with about 100 duplicate jobs, and the only sign was a client-side log warning the caller couldn't act on.
- **Fix:** Track seen tokens in a `HashSet<String>` and return an error on a repeated token, matching how `list_functions` in the same file already behaves. No public API change.
- **Test:** Regression test modeled on `test_list_functions_rejects_a_page_token_cycle`, using a mocked `POST /v1/jobs/list` that returns the same token on every call.

### 2. Fix timestamp precision loss in `AsyncTable.list_versions`
- **PR:** [#4426](https://github.com/lancedb/lancedb/pull/4426) (merged Oct 5) | **Issue:** [#4404](https://github.com/lancedb/lancedb/issues/4404)
- **Problem:** Nanosecond timestamps (~1.8e18) were split with float division/modulo (`ts // 1e9`, `ts % 1e9`). Floats can't represent every integer at that magnitude, so microseconds drifted (`.274000` came back as `.273999`).
- **Fix:** Switched to integer arithmetic with `divmod(ts, 1_000_000_000)` and integer `// 1000`.
- **Test:** Added `test_list_versions_timestamp_precision` using the value from the issue (`1790989429274000000`), expecting `microsecond == 274000`.
- **Scope decision:** Kept the PR to the precision fix. Timezone-aware UTC (option 1 in the issue) was deferred pending maintainer preference.

### 3. Actionable error for `tokenize()` with jieba/lindera tokenizers
- **PR:** [#4519](https://github.com/lancedb/lancedb/pull/4519) (merged) | **Issue:** [#4440](https://github.com/lancedb/lancedb/issues/4440) (option 2)
- **Problem:** `tokenize()` builds the FTS analyzer on the client. For `jieba/*` and `lindera/*` this needs model files under `LANCE_LANGUAGE_MODEL_HOME`. When they were missing, users only saw a raw `Invalid directory path` error, even though server-side search worked.
- **Fix:** The error now keeps the original cause and explains that `tokenize()` runs on the client, which directory the model files must be in, and that for remote tables they must match the server's.
- **Also:** Updated the tokenize docs in Rust, Python, TypeScript and the generated JS docs to say a model mismatch can't be detected, so tokens may differ from the server's index.
- **Tests:** Rust test with mocked `jieba/default` index details and an empty model dir, a test that other tokenizer errors don't get the hint, and Python sync/async tests.
- **Scope decision:** Left out mismatch detection, which needs a server-side tokenize endpoint (option 1).

---

## Litestar

[litestar-org/litestar](https://github.com/litestar-org/litestar): Python ASGI web framework. My fork: [aviralsaxena16/litestar](https://github.com/aviralsaxena16/litestar).

### 1. Forward `name` argument in `LitestarGroup.add_command()`
- **PR:** [#5048](https://github.com/litestar-org/litestar/pull/5048) (merged Sep 6)
- **Fix:** `LitestarGroup.add_command()` now forwards its `name` argument instead of dropping it.
- **Tests / docs:** [TODO: add if you wrote a test]

### 2. Reject `q=0` media types in `Accept.best_match()`
- **PR:** [#5073](https://github.com/litestar-org/litestar/pull/5073) (open, opened Sep 12, awaiting review; all 22 checks passing)
- **Problem:** Under HTTP content negotiation, `q=0` means a media type is explicitly *not acceptable*, but `best_match()` could still select it.
- **Fix:** `best_match()` now rejects media types with `q=0`.
- **Tests / docs:** [TODO: add if you wrote a test]

---

## CanonForces | OpenLake

[OpenLake/canonforces](https://github.com/OpenLake/canonforces) | [Live](https://canonforces.openlake.in/)

Gamified competitive programming platform (Next.js, Firebase, Redis, Judge0, Socket.IO) with 200+ active users.

**As maintainer**
- Reviewed 50+ PRs and onboarded 30+ contributors.
- Defined contribution workflows, triaged issues, and kept CI/CD green on GitHub Actions.
- Project mentor for FOSS Overflow and Hacktoberfest.

**Engineering contributions**
- Built the 1v1 matchmaking engine on Socket.IO with Redis atomic queues, eliminating race conditions.
- Resolved a production Firestore quota exhaustion incident under real user load with caching, pagination and deferred rendering.

---

## SimPPL | Code for GovTech (C4GT)

[aviralsaxena16/Bluesky_Data_extraction](https://github.com/aviralsaxena16/Bluesky_Data_extraction)

Sprint 3 (July - Aug 2025). Selected from 500+ applicants to build open Digital Public Infrastructure.

- Built a multi-threaded Python data pipeline that works around browser-extension limits to extract data concurrently.
- Tuned the extraction models to process and archive 15,000+ posts, including deeply nested comment and reply threads from complex queries.

---

## Other
- GSSoC: 30+ PRs merged.
