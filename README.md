# datepoll

Find a date that works for the whole group. datepoll is a local forwarding client for the hosted DooDates MCP service: it answers `tools/list` on your own machine from the captured tool definitions and forwards every poll operation to DooDates, returning the results exactly as the service sends them back.

Participants vote with no account. Creating a poll returns a `vote_url` to share with the group and an `admin_url` — keep the admin link, because without an account it is the only way back in to read the results or close the poll.

## The tools

- **`create_date_poll`** — Create a scheduling poll so a group can vote on dates or time slots. `title` and `dates` are required; dates are concrete ISO dates (`YYYY-MM-DD`), and optional `time_slots` (or per-date `times_by_date`) turn all-day options into timed ones. The response carries the vote link, the admin link, and a ready-to-send share message.
- **`get_poll_results`** — Read the current results of a poll: votes per option, ranking and winner. `poll` is required (a slug, vote link or admin link), and access needs the admin link or its token — the vote link alone does not grant it.
- **`close_poll`** — Close a poll so it stops accepting votes, or reopen it with `reopen: true`. Closing is reversible and does not set a final date. Like results, it needs the admin link or token.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/datepoll mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "datepoll": {
      "command": "npx",
      "args": ["-y", "@reneturcios/datepoll", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/datepoll
docker run --rm ghcr.io/cloudcorprecords/datepoll get_poll_results '{"poll":"team-dinner"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
datepoll create_date_poll '{"title":"Team dinner","dates":["2026-11-07","2026-11-14"]}'
datepoll get_poll_results '{"poll":"https://doodates.com/poll/team-dinner/results#owner_token=..."}'
datepoll close_poll '{"poll":"team-dinner","admin_token":"..."}'
datepoll --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `DOODATES_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://doodates.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream's registry record declares no headers, so datepoll never sends an authentication header — there is no API key to configure.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `datepoll error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface, exercises the session handshake and its reuse, asserts that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
