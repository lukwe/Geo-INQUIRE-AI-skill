# Geo-INQUIRE OGC API Processes skill

This skill helps an agent prepare and submit volcanology model requests to the
Geo-INQUIRE OGC API Processes service at `https://voice.pi.ingv.it/geoinquire`.
It covers SOLWCAD, CONDUIT, and PYBOX. It requires an agentic client with
network access and an HTTP-capable tool such as a terminal with `curl`.

## Install

The complete skill is in `.agents/skills/ogc-geoinquire/`. Keep `SKILL.md` and
its `references/` directory together.

- Codex, Cursor, and Gemini CLI: copy that folder to
  `~/.agents/skills/ogc-geoinquire/` for use across projects, or keep this
  repository open as the project.
- Claude Code: copy that folder to `~/.claude/skills/ogc-geoinquire/`, or to
  `.claude/skills/ogc-geoinquire/` in a project.

For example, on macOS or Linux, from this repository:

```sh
mkdir -p ~/.agents/skills
cp -R .agents/skills/ogc-geoinquire ~/.agents/skills/
```

Copy the folder again when you deliberately update to a newer release. Restart
the client or refresh its skill list if the new version is not visible.

## Use

Select or mention the `ogc-geoinquire` skill in your client and ask a research
question. For example:

> Use ogc-geoinquire to prepare a SOLWCAD request for my measurements. Show the
> process description, required inputs, units, candidate payload, and any
> missing or ambiguous values before submitting anything.

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
