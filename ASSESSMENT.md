# Voxgig SDK Generator assessment: Resend

Author: Caesar Mansour. AI assistance disclosure: Claude (Anthropic) ran the commands and drafted this report under my direction; I reviewed each checkpoint and decided every intervention. Time box: 30 min (16:50 to 17:20 Israel time, 2 Oct 2026).

## 1. What I attempted

Generate a TypeScript SDK for the Resend API from Resend's official OpenAPI spec using the official Voxgig toolchain, following the documented quickstart exactly, then test it and record the developer experience.

- Eligibility: scraped all 27 pages of the `voxgig-sdk` org (803/803 repos). No repo name contains "resend"; `voxgig-sdk/resend-sdk` returns 404.
- Spec: `resend/resend-openapi` `resend.yaml` (MIT), OpenAPI 3.1.2, info.version 1.5.1, 72 paths, 113 operations (all with `operationId`), one bearer scheme, `servers` present. sha256 `d75e4e44...94ceff4a`.
- Toolchain: `@voxgig/create-sdkgen` 0.30.6, `@voxgig/sdkgen` 4.34.0, `@voxgig/apidef` 8.22.1, `@voxgig/model` 12.0.0, `@voxgig/docgen` 0.30.1, Node 24.21.0.

## 2. What worked

- Scaffold, `target add ts`, `feature add test`, `npm run generate`, `npm install`, `npm run build` all exited 0. Total generator wall time about 45 s.
- OpenAPI 3.1 constructs (type arrays, `const`) parsed without errors.
- Generated offline test suite: 516 tests, 515 pass, 0 fail, 1 skipped.
- The `readme_examples` tests compile and run the documented code blocks (264 found, 250 executed in mock mode). Strong idea: docs cannot silently rot.
- Output includes an "unofficial, not affiliated" NOTICE, SECURITY.md, AGENTS.md, and a `.gitignore` that covers the documented `.env.local` key file.

## 3. What failed or was unexpected

No hard failures. Unexpected output:

1. **Entity fragmentation.** 18 Resend resource tags became 60 entities, about half named after response schemas, e.g. `RemoveContactResponseSuccessEntity`, `UpdateTopicResponseSuccessEntity`. `ContactEntity` has only `create, list, load`; `remove` and `update` live on separate entities and `list` is duplicated on `ListContactsResponseSuccessEntity`. A developer looking for `client.Contact().remove()` will not find it.
2. **Generator warnings:** `WARN require-missing ./cmp/ts/ReadmeFeatures_ts` and `./cmp/ts/AgentGuide_ts` during generate. Components referenced but not copied by `target add ts`.
3. **README quickstart numbering** jumps from "1. Create a client" to "3. Load an automationrun". Possibly related to warning 2 (not verified).
4. **Lead example is `automationrun`** (seemingly first alphabetically) rather than sending an email, Resend's primary use case.
5. **Action endpoints** (`/domains/{id}/verify`, `/broadcasts/{id}/send`, `/emails/{id}/cancel`) are mapped as `create({ $action: 'verify', ... })`. Reachable, but non-obvious.
6. `DEP0176 fs.F_OK` deprecation warnings under Node 24, the version the toolchain requires.
7. **Copyright assigned automatically, to the wrong party for this use.** The generated root `LICENSE` and `ts/LICENSE` read "Copyright (c) 2026 Voxgig", and `ts/package.json` sets author "Voxgig" under the `@voxgig-sdk` scope. The scaffolded template `.sdk/tm/LICENSE` names "Resend", the upstream provider, which contradicts the generated "not affiliated" NOTICE. None names the contributor, although the task states the contributor retains copyright. I found no option for the holder in the docs I read. Left unchanged, as generated.

## 4. Manual intervention

| What | Why | Where |
| --- | --- | --- |
| Installed Node 24 via `npm i -g node@24` | `@voxgig/sdkgen` declares `engines: node >=24`; sandbox had 22; docs only say "a recent LTS" | environment only |
| Added Resend's MIT notice for the bundled spec | Generator copies the spec into `.sdk/def/` without its upstream license notice | commit `manual: ...` |
| Git history built in sandbox, pushed by hand | No GitHub connector available to the AI | process only |
| Commit authorship rewritten to my Git identity | Commits were first created under an AI placeholder identity | `git rebase --root --reset-author`; file content unchanged (tree hashes verified) |

Generated code was not edited. Commit `generated: ...` is untouched generator plus build output; every later commit is manual. The root LICENSE is generated MIT with holder "Voxgig"; deliberately left unchanged (see section 3, item 7).

## 5. Experience using the generator

Fast and smooth when the docs are followed: under a minute from spec to a building, tested SDK, with no prompts or config. The rough edges are in the documentation layer (three different invocation styles, unstated Node floor, redundant steps) and in model quality for a resource-oriented spec: the SDK compiles and its own tests pass, but its shape does not match how a Resend developer thinks about the API. Fixing that means editing `.sdk/model/entity/*.aontu`, which I did not attempt within the time box.

## 6. Recommended DX improvements

1. State the Node floor (24+) in the create-sdkgen README and tutorial, and fail early in `create-sdkgen` with a clear message instead of an npm engines warning.
2. Use one invocation everywhere (`npm create @voxgig/sdkgen@latest`); the README's bare `create-sdkgen` refers to an unscoped package that does not exist on npm.
3. Align the tutorial with the scripts: `generate` already runs `tsc --build src`, and `target add ts` already adds `test`.
4. In apidef, group operations by path resource (or tag) before falling back to response schema names, and report "N operations mapped to M entities" with a list of suspicious entities after scaffold.
5. Treat `require-missing` for README components as an error, or have `target add` copy them.
6. Let the README lead example be configurable, or prefer the most-used tag/first `POST` over alphabetical order.
7. Carry the upstream spec license into NOTICE when copying the spec.
8. Default the API key env var to the provider's convention when detectable (`RESEND_API_KEY` vs generated `RESEND_APIKEY`), or document the mismatch.
9. Warn clearly that the live suite runs create/update/remove scenarios against the real account, and offer a read-only live mode.
10. Set the copyright holder at scaffold time (a flag, or default to `git config user.name`), apply it to every LICENSE and `package.json` author, and document how to change it. Today three files name two different holders, neither of them the contributor.

## 7. Testing performed

| Test | Result |
| --- | --- |
| `npm run build` (ts) | pass |
| Generated offline suite (`npm test`, in-memory mock) | 516 total, 515 pass, 0 fail, 1 skipped (`cost` feature not generated) |
| README/REFERENCE examples (subset of above) | 264 blocks, 250 executed, all pass |
| Live suite (`RESEND_TEST_LIVE`) | not run: writes to the real account |
| Live read-only smoke (`Domain().list()`) | see below |

Note: offline tests validate the SDK against its own generated mock, not against the real Resend API.

Live smoke result: NOT RUN AT TIME OF WRITING.

## 8. Repository

URL: URL: https://github.com/caesar-mansour/resend-voxgig-sdk

## Work log (Israel time)

- 16:50 start. GitHub API rate-limited; switched to HTML scrape of the org
- 16:53 eligibility confirmed (803/803 repos)
- 16:55 docs, npm versions, spec profile and apidef 3.1 handling checked
- 16:56 Node 24 workaround installed
- 16:56 scaffold (23 s, no warnings)
- 16:56 `target add ts`, `feature add test`
- 16:56 `npm run generate` (9 s, 2 WARN, 2 deprecations)
- 16:57 ts install + build pass
- 16:58 offline tests: 515 pass / 0 fail / 1 skip
- 16:58 inspected entities, LICENSE, gitignore, README, live-test config
- 16:59 no GitHub connector; git bundle workaround. Commits 1 and 2
- 17:01 report written
- 17:02 corrections: author name, copyright DX observation, authorship rewrite prepared
