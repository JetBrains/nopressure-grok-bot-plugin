# NoPressure plugin for Grok Bot

Rehearse hard workplace conversations before they happen. [NoPressure](https://nopressure.io) turns a
situation into a live voice roleplay with a realistic character: delivering critical feedback, saying no,
a performance review, negotiating a raise, a first 1:1. You practise it in the browser and get the feedback
there as soon as you hang up.

## Install and connect

1. In Grok Bot, open **Plugins**, find **NoPressure** and add it. In Cursor, install it from the Marketplace.
2. Select **Authorize** (or **Connect**) and sign in to NoPressure. There is no API key to copy.

## Try it

```text
I have my performance review with my manager on Thursday and I want to ask for a raise. Help me rehearse it with NoPressure.
```

Grok Bot asks for anything it is missing, creates the simulation and sends you its scene image and the
practice link.

## How it works

1. Grok Bot notices a hard conversation coming up, from you or from your calendar, mail or chat. It gathers
   the specifics: who the other person is, how they behave, what was said last time, what is at stake.
2. It creates a simulation from that brief. Generation takes one to two minutes.
3. It sends you the scene image and a practice link. You open the link, sign in to NoPressure, and talk to
   the character by voice.
4. When you hang up, the same page shows the feedback: the key insight, a rating on the core challenge,
   growth points and the transcript. Another attempt uses the same link.

The simulation is created in your own private NoPressure workspace, and only you can see or run it.
NoPressure has no access to your calendar, mail or documents. It only sees the brief that Grok Bot writes.

## What's in the plugin

| Path | Contents |
|---|---|
| `.cursor-plugin/plugin.json` | Plugin manifest: name, version, description, author, links, license, keywords, logo, component paths |
| `mcp.json` | The remote NoPressure MCP server (`url`) |
| `skills/nopressure/SKILL.md` | The playbook: when to suggest a rehearsal, how to write the brief, how to create a simulation and send the scene image and the practice link |
| `assets/logo.svg` | Marketplace logo |
| `scripts/validate-plugin.mjs` | Checks the manifest, paths, frontmatter and release placeholders |

### MCP tools

The server is text-only: every tool result is plain text the model can read.

| Tool | Returns |
|---|---|
| `init_nopressure` | The playbook, for hosts that do not load skills |
| `create_simulation` | The `nopressure_simulation_id`. Generation runs in the background |
| `get_simulation` | Generating, ready or failed; it waits up to about 45 seconds for generation to finish. When ready: the title, the description, the scene image and the practice link |
| `get_simulation_data` / `update_simulation_data` | The editable simulation: title, description, character and situation |

Only two tools change anything, and only in your own NoPressure workspace. `create_simulation` adds a
simulation. `update_simulation_data` replaces the stored title, description and content of an existing one
with what it is sent. The other tools only read.

Signing in uses OAuth. The first time a tool is called, you are asked to sign in to NoPressure. Cursor
identifies itself with a Client ID Metadata Document (CIMD), so the plugin ships no OAuth client.

## Validating

```bash
node scripts/validate-plugin.mjs            # manifest, paths, frontmatter; placeholders are warnings
node scripts/validate-plugin.mjs --release  # the same, but any REPLACE_WITH_* placeholder is an error
```

`--release` has to pass before a release is submitted.

## Privacy and terms

Using NoPressure is subject to the [NoPressure Terms of Service](https://nopressure.io/terms-of-service)
and the [JetBrains Privacy Policy](https://www.jetbrains.com/legal/docs/privacy/privacy/), both linked from
[nopressure.io](https://nopressure.io).

## License

[MIT](LICENSE)
