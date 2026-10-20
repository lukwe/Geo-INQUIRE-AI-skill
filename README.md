# Geo-INQUIRE OGC API Processes skill

This skill helps an LLM-based AI agent prepare and submit volcanology model requests to the
Geo-INQUIRE OGC API Processes service at `https://voice.pi.ingv.it/geoinquire`.
It covers SOLWCAD, CONDUIT, and PYBOX. It requires an agentic client with
network access and an HTTP-capable tool such as a terminal with `curl`.

## Install

the user only needs to unpack the current release zip file and move it to the correct location as requested by the Agentic IDE he/she is planning to use.
The complete skill is in `.agents/skills/ogc-geoinquire/`. Keep `SKILL.md` and
its `references/` directory together. The folder follows the open
[Agent Skills specification](https://agentskills.io/specification), so clients
that support Agent Skills can use the same files without conversion. The whole release is based on textual markdown files detailing when and how to use all the end-points of the original remote API. The LLM agent will ingest all the instructions and act accordingly.

| Client                                                                                                           | Install for all local projects     | Install in one project           |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------------- | -------------------------------- |
| [OpenAI Codex](https://learn.chatgpt.com/docs/build-skills) (CLI or IDE extension)                               | `~/.agents/skills/ogc-geoinquire/` | `.agents/skills/ogc-geoinquire/` |
| [Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/agent-skills) (GitHub Copilot agent) | `~/.agents/skills/ogc-geoinquire/` | `.agents/skills/ogc-geoinquire/` |
| [Cursor](https://cursor.com/help/customization/skills)                                                           | `~/.agents/skills/ogc-geoinquire/` | `.agents/skills/ogc-geoinquire/` |
| [Gemini CLI](https://geminicli.com/docs/cli/using-agent-skills/)                                                 | `~/.agents/skills/ogc-geoinquire/` | `.agents/skills/ogc-geoinquire/` |
| [Claude Code](https://code.claude.com/docs/en/skills)                                                            | `~/.claude/skills/ogc-geoinquire/` | `.claude/skills/ogc-geoinquire/` |

For Codex, VS Code with GitHub Copilot, Cursor, and Gemini CLI, the folder is
already in the project location when you open this repository as your
workspace. If you run the Codex extension inside VS Code, use the Codex row.
For Claude Code, copy it to one of the Claude Code locations above. OpenAI's
ChatGPT desktop app also supports standalone skills; check its Skills sidebar
after local setup.

For example, on macOS or Linux, from this repository:

```sh
mkdir -p ~/.agents/skills
cp -R .agents/skills/ogc-geoinquire ~/.agents/skills/
```

For Claude Code, replace `~/.agents/skills` in those commands with
`~/.claude/skills`. On Windows, copy the same complete folder into the client
location shown by that client's documentation. When updating, select a newer
tagged release and replace the installed folder deliberately. Restart the
client or refresh its skill list if the new version is not visible.

### Other agentic IDEs

If your IDE supports Agent Skills, find its documented user or project skill
directory and place the complete `ogc-geoinquire/` folder there. The standard
defines the `SKILL.md` and supporting-file layout; each IDE decides where to
discover skills and how to invoke them. If your IDE does not discover skills,
you can still point its agent at `SKILL.md` and the bundled `references/`, but
automatic activation depends on that IDE's capabilities.

## Use

Select or mention the `ogc-geoinquire` skill in your client and ask a research
question. For example:

> Use ogc-geoinquire to prepare a SOLWCAD request for my measurements. Show the
> process description, required inputs, units, candidate payload, and any
> missing or ambiguous values before submitting anything.

To check installation without starting a model run, ask the agent to identify
the skill and read `references/api-overview.md`, then request the live process
list. The agent should report SOLWCAD, CONDUIT, and PYBOX without submitting
an execution request.

The skill requires a preflight review and explicit confirmation before a
process execution request. It preserves the job ID and raw results or artifact
links. The agent's ability to follow these instructions depends on the client,
model, and tool permissions; review the proposed payload before confirming.

## Releases and provenance

`SOURCE.json` records the approved development commit, the release generator
commit, and SHA-256 hashes of the distributed files. Use a tagged release when
reproducibility matters. Record the skill version, client and model, retrieved
process description, submitted payload, job ID, and raw outputs with your run.

This is a generated release repository. Skill and reference changes originate
in the development repository and are promoted here after validation. Do not
edit the distributed skill or references independently in this repository.

## License

The original material in this release is licensed under CC BY 4.0. See
[LICENSE.md](LICENSE.md). External services, model implementations, and linked
third-party materials retain their own terms.
