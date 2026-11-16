# BUILD REPORT — datepoll

Clean-room rebuild of `com.doodates/mcp` v1.0.0
(remote-only MCP server at `https://doodates.com/mcp`), built
2026-10-28 as a forwarding client. Package: `@reneturcios/datepoll`
0.1.0, registry name `io.github.CloudCorpRecords/datepoll`.

## Surface built

All **3 captured tools built**; no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `create_date_poll` | `title`, `dates` |
| `get_poll_results` | `poll` |
| `close_poll` | `poll` |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, with `inputSchema.$schema` set to
`https://json-schema.org/draft/2020-12/schema`, exactly as captured.
The tool literals were generated programmatically from
`harness/fixtures/doodates-tools-original.json` by the script at
`/tmp/b43-datepoll-scratch/gen.py` (parse + re-serialise, placeholder
substitution into `src/forward.ts`) — never retyped — and a
post-generation check (the registration-integrity test suite, plus
the parity harness) confirmed the served literals deep-equal the
fixture's 3 tool objects, in order. Tools are registered via the MCP
SDK low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
titles, descriptions, required fields (`title`, `dates` for
create_date_poll; `poll` for get_poll_results and close_poll) and
property shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://doodates.com/mcp`.
- Env override `DOODATES_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on its remote: no auth
  header is ever sent, under any environment. The forwarding client
  has no code path that sets an `Authorization` header; tests assert
  its absence with an empty environment, with the endpoint
  overridden, across a reused session, and even when key-like env
  vars (`DOODATES_API_KEY`, `API_KEY`, `AUTHORIZATION`, …) are
  present in the environment, plus a never-`X-API-Key` check and an
  exact sent-header-set check (only `Content-Type`, `Accept`, and
  `Mcp-Session-Id` when issued).
- Error rendering `datepoll error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`.
- Parses as JSON; `name` == `@reneturcios/datepoll`.
- Package entries: **128**.
- sha256: `5b1eeee958cb3756e48e9bb345e4fa415ce6759a26bd429d18a3468f6427ee60`
  (re-verified unchanged after the subsequent offline `npm install`).

## server.json

- Description exactly: "Create group date polls in one call, read
  the votes, and close the poll when decided." —
  **85 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/datepoll:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/datepoll"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/datepoll`
  and its smoke test asserts the server name `datepoll`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **59 tests, 59 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 3
   tools, plus session handling/reuse, env override, the
   no-Authorization-header assertions under every environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/doodates-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/datepoll/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/datepoll/parity-out` → **verdict PASS, parity 100.0**
   (3/3 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
