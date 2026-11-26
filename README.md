# devfolio

Read a developer's portfolio without opening a browser. devfolio is a small forwarding client for a hosted portfolio MCP server: the tool list is answered on your own machine from captured definitions, and each call travels upstream, with the site's answer handed back exactly as it arrived, as markdown.

All six of the service's captured tools are exposed. Pull the profile — summary, work experience, education and contact links — read the hire information covering services, engagement options, frequently asked questions and a booking link, browse the full project list with its source and demo links, scan the technical article index, open one article in full, or fetch any top-level page of the site, including the reading list and the toolkit page. Everything here is a read of already-published pages; no tool sends, submits or books anything.

## The tools

- **`get_profile`** — The profile as markdown: summary, work experience with results, education and contact links. Takes no arguments.
- **`get_hire_info`** — The hiring page as markdown: services on offer, engagement options, FAQ and the booking link published on that page. Takes no arguments.
- **`list_projects`** — Every project with its description, year, technologies and links to source code, live demos and technical articles. Takes no arguments.
- **`list_posts`** — Every technical article with slug, title, description, publish date, tags and reading time. Takes no arguments; use a slug from here with `get_post`.
- **`get_post`** — The full text of one article as markdown. Requires `slug`, for example `sovereign-rag`.
- **`get_page`** — Any top-level page of the site as markdown. Requires `page`, one of `index`, `hire`, `projects`, `blog`, `books` or `toolkit`.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/devfolio mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "devfolio": {
      "command": "npx",
      "args": ["-y", "@reneturcios/devfolio", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/devfolio
docker run --rm ghcr.io/cloudcorprecords/devfolio get_profile '{}'
```

## CLI

A single binary covers both the command line and the MCP server. Every tool is a subcommand whose one argument is a JSON string of that tool's inputs:

```bash
devfolio get_profile '{}'
devfolio get_hire_info '{}'
devfolio list_projects '{}'
devfolio list_posts '{}'
devfolio get_post '{"slug":"sovereign-rag"}'
devfolio get_page '{"page":"books"}'
devfolio --version
```

Exit codes: `0` for a successful forwarded call, `1` when the result that comes back is an error, and `2` for usage mistakes like an unknown subcommand or a JSON argument that does not parse.

## Configuration

- `KRISHP_MCP_URL` — Replaces the upstream endpoint. Defaults to the pinned hosted endpoint, `https://krishparekhdev.com/mcp`. (This override variable is a reconstruction introduced by this rebuild.)

The registry record for the upstream remote lists no headers, so devfolio never attaches an authentication header — every call goes upstream keyless.

When the upstream cannot answer — it returns a JSON-RPC error, answers with a non-2xx status, sends a body that cannot be read, or the request itself fails — the caller receives an error result whose text begins with `devfolio error:`.

## How forwarding works

The first call of a process opens a session against the upstream: an `initialize` request (protocol version `2025-03-26`), a `notifications/initialized` notification, and then the `tools/call`. If the service issues an `Mcp-Session-Id` header during initialize, devfolio repeats it on subsequent requests and keeps reusing that one session until the process ends. Replies are understood in both plain JSON and server-sent-events framing. Successful results are passed through untouched, `isError` flag and any extra fields included.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The suite compares the served tool list against the captured surface tool for tool, walks the session handshake and its reuse, proves no `Authorization` header is sent under any environment, and confirms that arguments travel upstream unchanged and results return unchanged for every one of the six tools.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
