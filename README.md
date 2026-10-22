# Geo-INQUIRE OGC API Processes skill

This skill helps an LLM-based AI agent prepare and submit volcanology model requests to the
Geo-INQUIRE OGC API Processes service at `https://voice.pi.ingv.it/geoinquire`.
It covers SOLWCAD, CONDUIT, and PYBOX. It requires an agentic client with
network access and an HTTP-capable tool such as a terminal with `curl`.

## Install

The complete skill is in `.agents/skills/ogc-geoinquire/`. Keep `SKILL.md` and
its `references/` directory together. The folder follows the open
[Agent Skills specification](https://agentskills.io/specification), so clients
that support Agent Skills can use the same files without conversion. Choose
one of the installation routes below.

### 1. Quick install with the skills CLI

This route requires [Node.js](https://nodejs.org/en/download) **22.20 or newer**
and a working `npx` command from npm. Check both before installing:

```sh
node --version
npx --version
```

From the root of the project where you want the skill, run:

```sh
npx skills add lukwe/Geo-INQUIRE-AI-skill --skill ogc-geoinquire
```

The [skills CLI](https://github.com/vercel-labs/skills) lets you select your agentic clients. It installs in the current project by default.
To install for all your projects, run:

```sh
npx skills add lukwe/Geo-INQUIRE-AI-skill --skill ogc-geoinquire -g
```

In the same project, check for updates and install them with:

```sh
npx skills check
npx skills update ogc-geoinquire -p
```

To check or update a user-wide installation, add `-g` to the command. Use `-g` instead of `-p` for `update`.
The CLI updates from the current repository version. For a fixed research version, use a tagged release and record its tag.
The CLI collects anonymous usage telemetry. Set `DISABLE_TELEMETRY=1` to opt out. See its [configuration guide](https://github.com/vercel-labs/skills/blob/main/docs/troubleshooting.md).

### 2. Download a tagged release and install manually

Download **Source code (zip)** for a chosen version from this repository's
[Releases or Tags page](https://github.com/lukwe/Geo-INQUIRE-AI-skill/releases), then
extract it. Copy the complete nested `.agents/skills/ogc-geoinquire/` folder
into your client's skill location. This route does not require Node.js.

| Client                                                                                                           | Install for all local projects     | Install in one project           |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------------- | -------------------------------- |
| [OpenAI Codex](https://learn.chatgpt.com/docs/build-skills) (CLI or IDE extension)                               | `~/.agents/skills/ogc-geoinquire/` | `.agents/skills/ogc-geoinquire/` |
| [Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/agent-skills) (GitHub Copilot agent) | `~/.agents/skills/ogc-geoinquire/` | `.agents/skills/ogc-geoinquire/` |
| [Cursor](https://cursor.com/help/customization/skills)                                                           | `~/.agents/skills/ogc-geoinquire/` | `.agents/skills/ogc-geoinquire/` |
| [Gemini CLI](https://geminicli.com/docs/cli/using-agent-skills/)                                                 | `~/.agents/skills/ogc-geoinquire/` | `.agents/skills/ogc-geoinquire/` |
| [Claude Code](https://code.claude.com/docs/en/skills)                                                            | `~/.claude/skills/ogc-geoinquire/` | `.claude/skills/ogc-geoinquire/` |

Open the extracted repository as a project to use the skill in Codex, VS Code with GitHub Copilot, Cursor, or Gemini CLI.
In VS Code, use the Codex row for the Codex extension. For Claude Code, copy the folder to a Claude Code path above.
The ChatGPT desktop app also supports standalone skills. Check its Skills sidebar after local setup.

For example, on macOS or Linux, from this repository:

```sh
mkdir -p ~/.agents/skills
cp -R .agents/skills/ogc-geoinquire ~/.agents/skills/
```

For Claude Code, use `~/.claude/skills` instead of `~/.agents/skills` in the commands above.
On Windows, use the skill location in your client's documentation. To update, download a newer tagged release and replace the installed folder.
Restart your client or refresh its skill list if the new version is not visible.

### 3. Keep a Git checkout

If you prefer Git, clone this repository into a separate directory:

```sh
git clone https://github.com/lukwe/Geo-INQUIRE-AI-skill.git "$HOME/Geo-INQUIRE-AI-skill"
```

The checkout contains the skill at `.agents/skills/ogc-geoinquire/`. You can
open the checkout itself as a project for clients that read `.agents/skills/`,
or copy or link that nested skill folder into a client location in the table
above. For example, on macOS or Linux, a Claude Code user can keep the clone
in their home directory and link its skill folder for user-wide access:

```sh
mkdir -p "$HOME/.claude/skills"
ln -s "$HOME/Geo-INQUIRE-AI-skill/.agents/skills/ogc-geoinquire" "$HOME/.claude/skills/ogc-geoinquire"
```

Run `git pull` inside the checkout to update it. A linked installation follows
the checkout; a copied installation must be copied again after the pull.
Do not clone the whole repository directly into a client's `ogc-geoinquire/`
skill folder: `SKILL.md` would remain nested too deeply.

### Other agentic IDEs

If your IDE supports Agent Skills, put the complete `ogc-geoinquire/` folder in its documented project or user skill directory.
Each IDE chooses where to find skills and how to invoke them. If your IDE does not find skills automatically, point its agent to `SKILL.md` and `references/`.
Automatic activation depends on the IDE.

## Use

Select `ogc-geoinquire` and ask a research question. For example:

> Use ogc-geoinquire to prepare a SOLWCAD request for my measurements. Show the process description, required inputs and units, proposed payload, and missing or ambiguous values. Do not submit it yet.

To test installation without a model run, ask the agent to read `references/api-overview.md` and list the live processes.
It should report SOLWCAD, CONDUIT, and PYBOX without submitting an execution request.

The skill checks inputs and units during preflight. It requires your explicit confirmation before execution. It preserves the job ID and raw results or artifact links.
The client, model, and tool permissions affect how reliably the agent follows these instructions. Review the proposed payload before you confirm.

## Releases and provenance

`SOURCE.json` records the approved development commit, release generator commit, and SHA-256 hashes of the released files.
For reproducible research, use a tagged release. Record the tag, client, model, retrieved process description, payload, job ID, and raw outputs.

This release comes from the validated development repository.

## License

The original material in this release is licensed under CC BY 4.0. See
[LICENSE.md](LICENSE.md). External services, model implementations, and linked
third-party materials retain their own terms.
