# Repository guidance

This repository contains shared internal tooling, utilities, and AI agent skills
for the Serverpod team.

## General conventions

- Keep changes focused, simple, and useful across team members' environments.
- Avoid machine-specific paths, credentials, and assumptions about local checkouts.
- Document a tool's purpose, prerequisites, and usage alongside its implementation.
- Update the root README when adding tooling or skills so they remain discoverable.

## Skills

- Store each skill in `skills/<skill-name>/SKILL.md`, with supporting files in the
  same directory.
- Keep the directory name and the `name` in the skill's YAML frontmatter aligned.
- Keep instructions clear and actionable, and keep any agent metadata consistent
  with the skill.

## Validation

- Run checks appropriate to the tooling changed and document required commands
  alongside that tooling.
- For documentation or skill changes, verify relative links, referenced paths,
  and metadata. No runtime tests are needed for prose-only changes.

## Commit Messages

- Use the conventional commits format.
- Use the present tense.
- Use the active voice.
- Use the imperative mood.
- Always write in sentence case.
- Format terms with backticks when needed.
- No trailing period.

```
feat: Add new feature
fix: Fix a bug on the `feature_name` feature
chore: Update dependencies
refactor: Refactor code
test: Add tests
docs: Update documentation
style: Format code
perf: Improve performance
```
