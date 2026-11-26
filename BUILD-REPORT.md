# BUILD REPORT — devfolio

Clean-room rebuild of `com.krishparekhdev/portfolio` v1.0.0
(remote-only MCP server at `https://krishparekhdev.com/mcp`),
built 2026-10-28 as a forwarding client. Package: `@reneturcios/devfolio`
0.1.0, registry name `io.github.CloudCorpRecords/devfolio`.

## Surface built

**FULL surface: all 6 captured tools built**, none omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `get_profile` | *(none — no `required` key in the capture, served verbatim)* |
| `get_hire_info` | *(none — no `required` key in the capture, served verbatim)* |
| `list_projects` | *(none — no `required` key in the capture, served verbatim)* |
| `list_posts` | *(none — no `required` key in the capture, served verbatim)* |
| `get_post` | `slug` |
| `get_page` | `page` |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema` — the capture carries a
`title` key per tool, so the fixture carries it and it is
served verbatim (titles: "Get profile", "Get hiring
information", "List projects", "List articles", "Read an
article", "Get a page as markdown"). Every `inputSchema`
is an object schema with `additionalProperties: false`;
the four no-argument tools carry empty `properties`;
`get_post`'s `slug` is a plain string; `get_page`'s
`page` enumerates `index`, `hire`, `projects`, `blog`,
`books`, `toolkit`. The tool literals were generated
programmatically from
`harness/fixtures/krishparekhdev-tools-original.json` by
the scripts at `/tmp/laneb9-devfolio-scratch/gen.py` and
`/tmp/laneb9-devfolio-scratch/splice.py` (parse +
re-serialise, spliced into `src/forward.ts` between the
TOOL_DEFS_GENERATED markers) — never retyped — and the
splice script's round-trip check confirmed the spliced
block parses back deep-equal to the fixture's 6 tool
objects, in order. Tools are registered via the MCP SDK
low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

The surface is read-only access to one person's published
portfolio site: profile (summary, experience, education,
contact links), hire info (services, engagement options,
FAQ, booking link — returned as published page content, a
read, not a send), projects, article list and full article
text, and top-level site pages. Nothing is sent, submitted
or booked by any tool.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools'
names, titles, descriptions, the `required` arrays (none
for `get_profile`, `get_hire_info`, `list_projects`,
`list_posts`; `slug` for `get_post`; `page` for
`get_page`), and the property shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://krishparekhdev.com/mcp`.
- Env override `KRISHP_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with a `KRISHP_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` /
  `KRISHP_TOKEN` variables present.
- Error rendering `devfolio error:` text for upstream failures
  — labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests, and no live call is
  claimed beyond the coordinator's tools/list capture.

## Deviations

**None.** No write-guard workaround was needed in this session:
the lockfile, `npm ci`, `tsc` build and test suite all ran
through the normal routes (direct `node_modules/.bin/tsc` /
`tsx` binaries), and `dist/` is the direct output of
`tsc` on the validated sources.

## Lockfile

Generated clean from birth (Route 3, clean dir, no
`node_modules`, no pre-existing lockfile):
`npm install --package-lock-only --ignore-scripts --offline`
→ success. Parses as JSON; `name` == `@reneturcios/devfolio`;
exactly **128** package entries; sha256
`3c6cd4f1410c90daea4225d1f396fee744d63343249935eb79e917c4ea19fa5d`.
`npm ci --ignore-scripts --offline` then succeeded and the
lockfile sha256 was re-verified **unchanged** afterwards.

## server.json

- Description exactly: "Read a developer portfolio: profile, hire info, projects, articles and site pages." — **82 characters** (measured from the written file).
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/devfolio:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/devfolio"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/devfolio`
  and its smoke test asserts the server name `devfolio`.

## Verification (run 2026-10-28, builder session)

1. `npm install --package-lock-only --ignore-scripts --offline`
   → success; lockfile parses, name `@reneturcios/devfolio`,
   128 package entries (see Lockfile).
2. `npm ci --ignore-scripts --offline` → success; lockfile
   sha256 unchanged.
3. `tsc --noEmit` (direct `node_modules/.bin/tsc`) → exit 0.
4. `tsc` build → exit 0; `node dist/cli.js --version` → `0.1.0`.
5. Full suite (`tsx --test test/*.test.ts`, direct
   `node_modules/.bin/tsx`) → **64 tests,
   64 pass, 0 fail** (registration-integrity suite deep-equals
   the served tools/list against the fixture's 6 tools, in
   order, at both the registry level and through an
   in-memory-transport MCP `tools/list`; plus per-tool schema
   detail assertions including the title checks, session
   handling/reuse, env override, the no-Authorization-header
   assertions under every environment, verbatim argument
   forwarding for all 6 tools and result pass-through
   including upstream isError, SSE parsing, error rendering,
   MCP-layer validation, and CLI behaviour).
6. Parity (harness, tools-file mode):
   `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/krishparekhdev-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/devfolio/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/devfolio/parity-out/`
   → **verdict PASS, parity 100.0** (6/6 identical, 0 missing,
   0 extra, no schema diffs). Output in `parity-out/`.
