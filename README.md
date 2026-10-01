# NoPressure plugin for Grok Bot

Rehearse hard workplace conversations before they happen. [NoPressure](https://nopressure.io) turns a
situation into a live voice roleplay with a realistic character: delivering critical feedback, saying no,
a performance review, negotiating a raise, a first 1:1. You practise it in the browser, then go through the
debrief in the chat.

This is a [Cursor plugin](https://cursor.com/docs/plugins). Grok Bot installs plugins from the Cursor
Marketplace. The plugin contains no code. It holds a skill and the address of the hosted NoPressure MCP server.

## Install and connect

1. In Grok Bot, open **Plugins**, find **NoPressure** and add it. In Cursor, install it from the Marketplace.
2. Select **Authorize** (or **Connect**) and sign in to NoPressure. There is no API key to copy.

## Try it

```text
I have my performance review with my manager on Thursday and I want to ask for a raise. Help me rehearse it with NoPressure.
```

Grok Bot asks for anything it is missing, creates the simulation and sends you the practice link.

## How it works

1. Grok Bot notices a hard conversation coming up, from you or from your calendar, mail or chat. It gathers
   the specifics: who the other person is, how they behave, what was said last time, what is at stake.
2. It creates a simulation from that brief. Generation takes one to two minutes.
3. It sends you a practice link. You open it, sign in to NoPressure, and talk to the character by voice.
4. When you come back, Grok Bot fetches the feedback: the key insight, a rating on the core challenge,
   growth points and the transcript.

The simulation is created in your own private NoPressure workspace, and only you can see or run it.
NoPressure has no access to your calendar, mail or documents. It only sees the brief that Grok Bot writes.

## What's in the plugin

| Path | Contents |
|---|---|
| `.cursor-plugin/plugin.json` | Plugin manifest: name, version, description, author, links, license, keywords, logo, component paths |
| `mcp.json` | The remote NoPressure MCP server (`url`) |
| `skills/nopressure/SKILL.md` | The playbook: when to suggest a rehearsal, how to write the brief, how to create a simulation, send the link and fetch feedback |
| `assets/logo.svg` | Marketplace logo |
| `scripts/validate-plugin.mjs` | Checks the manifest, paths, frontmatter and release placeholders |

### MCP tools

The server is text-only: every tool result is plain text the model can read.

| Tool | Returns |
|---|---|
| `init_nopressure` | The playbook, for hosts that do not load skills |
| `create_simulation` | The `nopressure_simulation_id`. Generation runs in the background |
| `get_simulation` | Generating, ready or failed. When ready: the title, the description and the practice link |
| `get_feedback` | The debrief of the latest finished attempt, or "no finished attempt yet" |
| `get_simulation_data` / `update_simulation_data` | The editable simulation: title, description, character and situation |

Only two tools change anything, and only in your own NoPressure workspace. `create_simulation` adds a
simulation. `update_simulation_data` replaces the stored title, description and content of an existing one
with what it is sent. The other tools only read.

Signing in uses OAuth. The first time a tool is called, you are asked to sign in to NoPressure. Cursor
identifies itself with a Client ID Metadata Document (CIMD), so the plugin ships no OAuth client.

## Source of truth for the skill

`skills/nopressure/SKILL.md` is a verbatim copy of the skill served by the NoPressure MCP server. The server's
copy lives in the NoPressure monorepo and is the source of truth. Edit the skill there and copy it here
unchanged. Do not edit it in this repo only.

## Testing locally

Cursor loads unpublished plugins from `~/.cursor/plugins/local`. Cursor only follows a symlink there when
its target is a directory inside that folder, so copy the plugin in rather than symlinking your checkout.

```bash
mkdir -p ~/.cursor/plugins/local
rsync -a --delete --exclude .git ./ ~/.cursor/plugins/local/nopressure/
```

To test against a non-production deployment, edit the copy, not this repo. In
`~/.cursor/plugins/local/nopressure/mcp.json`, set `url` to the deployment's `/mcp` URL. Ask the NoPressure
team for it. Then run
**Developer: Reload Window** (or restart Cursor) and open **Customize** to check that the `nopressure` skill
and MCP server are listed.

- An installed Marketplace plugin with the same name takes priority over the local copy. Uninstall it first.
- On Teams and Enterprise plans, an admin has to turn on **Allow Local Plugin Imports**. It is off by
  default on Enterprise.

## Validating

```bash
node scripts/validate-plugin.mjs            # manifest, paths, frontmatter; placeholders are warnings
node scripts/validate-plugin.mjs --release  # the same, but any REPLACE_WITH_* placeholder is an error
```

`--release` has to pass before a release is submitted.

## Releasing

Every release is reviewed by Cursor before it is listed. Plugins are submitted at
[cursor.com/marketplace/publish](https://cursor.com/marketplace/publish).

1. Bump `version` in `.cursor-plugin/plugin.json`, for example to `0.1.1`.
2. If the skill changed in the monorepo, copy it over again.
3. Run `node scripts/validate-plugin.mjs --release`.
4. Test the plugin locally against the production server.
5. Commit with the version in the subject: `🔖 0.1.1: <what changed>`. 🔖 is the gitmoji for a release.
6. Tag the commit and push the branch, then the tag:

   ```bash
   git tag -a v0.1.1 -m "0.1.1"
   git push origin main
   git push origin v0.1.1
   ```

7. Submit the release.

## Privacy and terms

Using NoPressure is subject to the [NoPressure Terms of Service](https://nopressure.io/terms-of-service)
and the [JetBrains Privacy Policy](https://www.jetbrains.com/legal/docs/privacy/privacy/), both linked from
[nopressure.io](https://nopressure.io).

## License

[MIT](LICENSE)
