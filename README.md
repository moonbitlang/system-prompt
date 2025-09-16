# MoonBit System Prompt

[`Agents.mbt.md`](./Agents.mbt.md) contains the MoonBit System Prompt, a
descriptive system prompt designed to enhance the performance of code agents
over MoonBit projects.

## How to use

We provide some instructions on how to use the MoonBit System Prompt in
different code agents.

### AGENTS.md

Many AI code agents now have adopted the [`AGENTS.md`](https://agents.md)
convention for providing guidance for agents. For such agents, you can simply:

1. Copy `Agents.mbt.md` into your project directory and require Codex to read it
   in the `AGENTS.md` file.
2. Append the content of `Agents.mbt.md` to your existing `AGENTS.md` file.

### Claude Code

[Claude Code](https://www.anthropic.com/claude-code) supports `CLAUDE.md` file.
Since `CLAUDE.md` supports `@path/to/import` syntax, we suggest coping
`Agents.mbt.md` into your project directory and mention it in `CLAUDE.md`. For
example, suppose you copied `Agents.mbt.md` to your project directory as
`moonbit.mbt.md`, then you can add the following line to your `CLAUDE.md` file:

```markdown
# MoonBit Language Reference
- @moonbit.mbt.md
```

### Codex CLI

Codex CLI supports [`AGENTS.md`](https://agents.md). You can either:

1. Copy `Agents.mbt.md` into your project directory and require Codex to read it
   in the `AGENTS.md` file.
2. Append the content of `Agents.mbt.md` to your existing `AGENTS.md` file.

Quoted from [openai/codex/docs/getting-started.md](https://github.com/openai/codex/blob/main/docs/getting-started.md#memory-with-agentsmd):

> You can give Codex extra instructions and guidance using `AGENTS.md` files.
> Codex looks for `AGENTS.md` files in the following places, and merges them
> top-down:
>
> 1. `~/.codex/AGENTS.md` - personal global guidance
> 2. `AGENTS.md` at repo root - shared project notes
> 3. `AGENTS.md` in the current working directory - sub-folder/feature specifics
>
> For more information on how to use AGENTS.md, see the [official AGENTS.md
> documentation](https://agents.md/).

### Cursor & Cursor CLI

[Cursor](https://cursor.com/) supports `AGENTS.md` files. See the
[`AGENTS.md`](#agentsmd) section above for details.

Also, one can use the `.cursor/rules` functionality of Cursor to include the
content of `Agents.mbt.md` file. You copied `Agents.mbt.md` to your project
directory as `.cursor/rules/moonbit.mdc`, and configure ways to include the
context in the front-matter of `.cursor/rules/moonbit.mdc`. For example:

```markdown
---
description: "MoonBit System Prompt"
globs:
alwaysApply: true
---
...
```

See <https://docs.cursor.com/en/context/rules> for more details.

[Cursor CLI supports the same rules system as the IDE](https://docs.cursor.com/en/cli/using#rules)

### Gemini CLI

[Gemini-CLI](https://github.com/google-gemini/gemini-cli) can be configured to
use `AGENTS.md` files, as per <https://agents.md>:

> **How do I configure Gemini CLI?**
>
> Configure Gemini CLI to use AGENTS.md in .gemini/settings.json:
>
> ```json
> { "contextFileName": "AGENTS.md" }
> ```

Also, one can also use `GEMINI.md` file in a pretty-much the same way as
`AGENTS.md`.

### GitHub Copilot

[GitHub Copilot](https://github.com/features/copilot) supports `AGENTS.md`
files. See the [`AGENTS.md`](#agentsmd) section above for details.
