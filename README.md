# MoonBit System Prompt

[`Agents.mbt.md`](./Agents.mbt.md) contains the MoonBit System Prompt, a
descriptive system prompt designed to enhance the performance of code agents
over MoonBit projects.

## How to use

The simplest way is to reference the file and ask the AI to read it in the prompt.

Here we provide some extra instructions on how to use the MoonBit System Prompt in
different code agents.

### AGENTS.md

Many AI code agents now have adopted the [`AGENTS.md`](https://agents.md)
convention for providing guidance for agents. For such agents, you may:

- Copy `Agents.mbt.md` into your project directory as `AGENTS.md`
- Append the content of `Agents.mbt.md` to your existing `AGENTS.md` file.

### Claude Code

[Claude Code](https://www.anthropic.com/claude-code) supports `CLAUDE.md` file.
Since `CLAUDE.md` supports `@path/to/import` syntax, we suggest copying
`Agents.mbt.md` into your project directory and mention it in `CLAUDE.md`. For
example, suppose you copied `Agents.mbt.md` to your project directory as
`moonbit.mbt.md`, then you can add the following line to your `CLAUDE.md` file:

```markdown
# MoonBit Language Reference
- @moonbit.mbt.md
```

See [Memory Management](https://docs.claude.com/en/docs/claude-code/memory)
on detailed configuration.

### Codex CLI

Codex CLI supports [`AGENTS.md`](https://agents.md).
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

Also, one can use [rules](https://docs.cursor.com/en/context/rules) to include the
content of `Agents.mbt.md` file. For project rule, you can copy `Agents.mbt.md`
to `.cursor/rules/moonbit.mdc`, and configure ways to include the
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

[Gemini-CLI](https://github.com/google-gemini/gemini-cli) reads from `GEMINI.md`
files for user-level/repository-specific instructions/contexts. Since
it supports `@` memory import, we suggest copying `Agents.mbt.md` into your
project directory and mention it in `GEMINI.md`. For example, suppose you
copied `Agents.mbt.md` to your project directory as `moonbit.mbt.md`, then you
can add the following line to your `GEMINI.md` file:

```markdown
# MoonBit Language Reference
- @moonbit.mbt.md
```

See [Memory Import Processor](https://github.com/google-gemini/gemini-cli/blob/1634d5fcca29e7c64d37f99e17e42d303e12062d/docs/core/memport.md) for more information.

Also, Gemini-CLI can be configured to use `AGENTS.md` files, as per
<https://agents.md>:

> **How do I configure Gemini CLI?**
>
> Configure Gemini CLI to use AGENTS.md in .gemini/settings.json:
>
> ```json
> { "contextFileName": "AGENTS.md" }
> ```

### GitHub Copilot for VS Code

VS Code Copilot [experimentally supports `AGENTS.md`](https://code.visualstudio.com/docs/copilot/customization/custom-instructions#_use-an-agentsmd-file-experimental).

When working with multi-language projects, you may want to limit the scope
of the system prompt to specifically MoonBit-related files. You can do this by
creating a `.github/instructions/moonbit.instructions.md` file with the
following content:

```markdown
---
applyTo: "**/*.mbt,**/*.mbti,**/moon.mod.json,**/moon.pkg.json,**/*.mbt.md"
---
[Content of Agents.mbt.md]
```
