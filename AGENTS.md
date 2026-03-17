# Repository Guidelines

## Project Structure & Module Organization
This repository is a small collection of reusable agent skills. The root contains a minimal [README.md](/Users/adspower/Documents/data/www/mix_skills/README.md). Skill content lives under `mix_web/`, with one directory per skill, for example `mix_web/git-diff-tdd/` and `mix_web/operate-log-guide/`. Each skill is defined by an uppercase `SKILL.md`; supporting examples should sit beside it in an `examples/` folder, such as `mix_web/git-diff-tdd/examples/example-output.md`.

## Build, Test, and Development Commands
There is no compiled build step in this repository. Use lightweight content-review commands instead:

- `rg --files` lists all tracked skill files quickly.
- `sed -n '1,160p' mix_web/<skill>/SKILL.md` reviews a skill without opening an editor.
- `git status --short` confirms only intended files changed.
- `git diff --stat` summarizes the scope of documentation edits before commit.

## Coding Style & Naming Conventions
Write in clear Markdown with short sections, ordered steps, and fenced code blocks using language tags like ```php``` or ```json```. Keep directory names lowercase and hyphenated, for example `operate-log-guide`; keep the skill entry file name exactly `SKILL.md`. Prefer direct instructions, concrete paths, and copy-pasteable commands. When updating existing skill content, preserve its language and formatting conventions unless the change requires a full rewrite.

## Testing Guidelines
This repo does not currently include an automated test suite. Treat validation as documentation QA: verify headings render correctly, examples match the described workflow, referenced file paths exist, and commands are internally consistent. If a skill includes sample output, update the adjacent `examples/` file in the same change.

## Commit & Pull Request Guidelines
Recent history uses very short lowercase subjects such as `fix` and `ok`. Keep commits small and focused, but use slightly more specific imperative subjects when possible, for example `fix: clarify git diff step`. Pull requests should describe the affected skill, summarize behavior or wording changes, and include before/after snippets or screenshots only when formatting changed materially.

## Agent-Specific Notes
Before editing a skill, read its local `SKILL.md` fully and preserve any execution contract or required response markers. Avoid cross-skill drift: update only the relevant skill directory unless shared guidance truly needs a repository-wide change.
