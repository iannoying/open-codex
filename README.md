<h1 align="center">Open Codex CLI</h1>
<p align="center">Lightweight coding agent that runs in your terminal</p>

<p align="center"><code>npm i -g open-codex</code></p>

> **Important Note**: This is a fork of the [original OpenAI Codex CLI](https://github.com/openai/codex) with expanded model support and changed installation instructions. The main differences in this fork are:
>
> - Support for multiple AI providers (OpenAI, Gemini, OpenRouter, Ollama)
> - Uses the [Chat Completion API instead of the Responses API](https://platform.openai.com/docs/guides/responses-vs-chat-completions) which allows us to support any openai compatible provider and model.
> - All other functionality remains similar to the original project
> - You can install this fork globally with `npm i -g open-codex`

---

<details open>
<summary><strong>Build from source</strong></summary>

```bash
# Clone the repository and navigate to the CLI package
git clone git@github.com:iannoying/open-codex.git
cd open-codex/codex-cli

# Install dependencies and build
npm install
npm run build

# Get the usage and the options
node ./dist/cli.js --help

# Run the locally‑built CLI directly
node ./dist/cli.js

# Or link the command globally for convenience
npm link
```

</details>

<details>
<summary><strong>Table&nbsp;of&nbsp;Contents</strong></summary>

- [Experimental Technology Disclaimer](#experimental-technology-disclaimer)
- [Quickstart](#quickstart)
- [Why Codex?](#whycodex)
- [Security Model \& Permissions](#securitymodelpermissions)
  - [Platform sandboxing details](#platform-sandboxing-details)
- [System Requirements](#systemrequirements)
- [CLI Reference](#clireference)
- [Memory \& Project Docs](#memoryprojectdocs)
- [Non‑interactive / CI mode](#noninteractivecimode)
- [Recipes](#recipes)
- [Installation](#installation)
- [Configuration](#configuration)
- [FAQ](#faq)
- [Contributing](#contributing)
  - [Development workflow](#development-workflow)
  - [Writing high‑impact code changes](#writing-highimpact-code-changes)
  - [Opening a pull request](#opening-a-pull-request)
  - [Review process](#review-process)
  - [Community values](#community-values)
  - [Getting help](#getting-help)
  - [Releasing `codex`](#releasing-codex)
- [Security \& Responsible AI](#securityresponsibleai)
- [License](#license)
- [Zero Data Retention (ZDR) Organization Limitation](#zero-data-retention-zdr-organization-limitation)

</details>

---

## Experimental Technology Disclaimer

Codex CLI is an experimental project under active development. It is not yet stable, may contain bugs, incomplete features, or undergo breaking changes. We're building it in the open with the community and welcome:

- Bug reports
- Feature requests
- Pull requests
- Good vibes

Help us improve by filing issues or submitting PRs (see the section below for how to contribute)!

## Quickstart

Install globally:

```shell
npm install -g open-codex
```

Next, set your API key as an environment variable (shown here with OpenAI, but other providers are supported):

```shell
export OPENAI_API_KEY="your-api-key-here"
```

> **Note:** This command sets the key only for your current terminal session. To make it permanent, add the `export` line to your shell's configuration file (e.g., `~/.zshrc`).
>
> **Tip:** You can also place your API key into a `.env` file at the root of your project:
>
> ```env
> OPENAI_API_KEY=your-api-key-here
> ```
>
> The CLI will automatically load variables from `.env` (via `dotenv/config`).

Run interactively:

```shell
open-codex
```

Or, run with a prompt as input (and optionally in `Full Auto` mode):

```shell
open-codex "explain this codebase to me"
```

```shell
open-codex --approval-mode full-auto "create the fanciest todo-list app"
```

That's it – Codex will scaffold a file, run it inside a sandbox, install any
missing dependencies, and show you the live result. Approve the changes and
they'll be committed to your working directory.

---

## Why Codex?

Codex CLI is built for developers who already **live in the terminal** and want
ChatGPT‑level reasoning **plus** the power to actually run code, manipulate
files, and iterate – all under version control. In short, it's _chat‑driven
development_ that understands and executes your repo.

- **Zero setup** — bring your API key and it just works!
- **Multiple AI providers** — use OpenAI, Gemini, OpenRouter, or Ollama!
- **Full auto-approval, while safe + secure** by running network-disabled and directory-sandboxed
- **Multimodal** — pass in screenshots or diagrams to implement features ✨

And it's **fully open-source** so you can see and contribute to how it develops!

---

## Security Model \& Permissions

Codex lets you decide _how much autonomy_ the agent receives and auto-approval policy via the
`--approval-mode` flag (or the interactive onboarding prompt):

| Mode                      | What the agent may do without asking            | Still requires approval                                         |
| ------------------------- | ----------------------------------------------- | --------------------------------------------------------------- |
| **Suggest** <br>(default) | • Read any file in the repo                     | • **All** file writes/patches <br>• **All** shell/Bash commands |
| **Auto Edit**             | • Read **and** apply‑patch writes to files      | • **All** shell/Bash commands                                   |
| **Full Auto**             | • Read/write files <br>• Execute shell commands | –                                                               |

In **Full Auto** every command is run **network‑disabled** and confined to the
current working directory (plus temporary files) for defense‑in‑depth. Codex
will also show a warning/confirmation if you start in **auto‑edit** or
**full‑auto** while the directory is _not_ tracked by Git, so you always have a
safety net.

Coming soon: you'll be able to whitelist specific commands to auto‑execute with
the network enabled, once we're confident in additional safeguards.

### Platform sandboxing details

The hardening mechanism Codex uses depends on your OS:

- **macOS 12+** – commands are wrapped with **Apple Seatbelt** (`sandbox-exec`).

  - Everything is placed in a read‑only jail except for a small set of
    writable roots (`$PWD`, `$TMPDIR`, `~/.codex`, etc.).
  - Outbound network is _fully blocked_ by default – even if a child process
    tries to `curl` somewhere it will fail.

- **Linux** – there is no sandboxing by default.
  We recommend using Docker for sandboxing, where Codex launches itself inside a **minimal
  container image** and mounts your repo _read/write_ at the same path. A
  custom `iptables`/`ipset` firewall script denies all egress except the
  OpenAI API. This gives you deterministic, reproducible runs without needing
  root on the host. You can use the [`run_in_container.sh`](./codex-cli/scripts/run_in_container.sh) script to set up the sandbox.

---

## System Requirements

| Requirement                 | Details                                                         |
| --------------------------- | --------------------------------------------------------------- |
| Operating systems           | macOS 12+, Ubuntu 20.04+/Debian 10+, or Windows 11 **via WSL2** |
| Node.js                     | **22 or newer** (LTS recommended)                               |
| Git (optional, recommended) | 2.23+ for built‑in PR helpers                                   |
| RAM                         | 4‑GB minimum (8‑GB recommended)                                 |

> Never run `sudo npm install -g`; fix npm permissions instead.

---

## CLI Reference

| Command                                   | Purpose                             | Example                              |
| ----------------------------------------- | ----------------------------------- | ------------------------------------ |
| `open-codex`                              | Interactive REPL                    | `codex`                              |
| `open-codex "…"`                          | Initial prompt for interactive REPL | `codex "fix lint errors"`            |
| `open-codex -q "…"`                       | Non‑interactive "quiet mode"        | `codex -q --json "explain utils.ts"` |
| `open-codex completion <bash\|zsh\|fish>` | Print shell completion script       | `codex completion bash`              |

Key flags: `--model/-m`, `--approval-mode/-a`, and `--quiet/-q`.

---

## Memory \& Project Docs

Codex merges Markdown instructions in this order:

1. `~/.codex/instructions.md` – personal global guidance
2. `codex.md` at repo root – shared project notes
3. `codex.md` in cwd – sub‑package specifics

Disable with `--no-project-doc` or `CODEX_DISABLE_PROJECT_DOC=1`.

---

## Non‑interactive / CI mode

Run Codex head‑less in pipelines. Example GitHub Action step:

```yaml
- name: Update changelog via Codex
  run: |
    npm install -g open-codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    open-codex -a auto-edit --quiet "update CHANGELOG for next release"
```

Set `CODEX_QUIET_MODE=1` to silence interactive UI noise.

## Tracing / Verbose Logging

Setting the environment variable `DEBUG=true` prints full API request and response details:

```shell
DEBUG=true open-codex
```

---

## Recipes

Below are a few bite‑size examples you can copy‑paste. Replace the text in quotes with your own task. See the [prompting guide](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md) for more tips and usage patterns.

| ✨  | What you type                                                                   | What happens                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "Refactor the Dashboard component to React Hooks"`                       | Codex rewrites the class component, runs `npm test`, and shows the diff.   |
| 2   | `codex "Generate SQL migrations for adding a users table"`                      | Infers your ORM, creates migration files, and runs them in a sandboxed DB. |
| 3   | `codex "Write unit tests for utils/date.ts"`                                    | Generates tests, executes them, and iterates until they pass.              |
| 4   | `codex "Bulk‑rename *.jpeg → *.jpg with git mv"`                                | Safely renames files and updates imports/usages.                           |
| 5   | `codex "Explain what this regex does: ^(?=.*[A-Z]).{8,}$"`                      | Outputs a step‑by‑step human explanation.                                  |
| 6   | `codex "Carefully review this repo, and propose 3 high impact well-scoped PRs"` | Suggests impactful PRs in the current codebase.                            |
| 7   | `codex "Look for vulnerabilities and create a security review report"`          | Finds and explains security bugs.                                          |

---

## Installation

<details open>
<summary><strong>From npm (Recommended)</strong></summary>

```bash
npm install -g open-codex
# or
yarn global add open-codex
```

</details>

<details open>
<summary><strong>Build from source</strong></summary>

```bash
# Clone the repository and navigate to the CLI package
git clone https://github.com/ymichael/open-codex.git
cd open-codex/codex-cli

# Install dependencies and build
npm install
npm run build

# Get the usage and the options
node ./dist/cli.js --help

# Run the locally‑built CLI directly
node ./dist/cli.js

# Or link the command globally for convenience
npm link
```

</details>

---

## Configuration

Codex looks for config files in **`~/.codex/`** (either YAML or JSON format).

```json
// ~/.codex/config.json
{
  "model": "o4-mini", // Default model
  "provider": "openai", // Default provider
  "approvalMode": "suggest", // or auto-edit, full-auto
  "fullAutoErrorMode": "ask-user" // or ignore-and-continue
}
```

You can also define custom instructions:

```md
# ~/.codex/instructions.md

- Always respond with emojis
- Only use git commands if I explicitly mention you should
```

### Alternative AI Providers

This fork of Codex supports multiple AI providers:

- openai (default)
- gemini
- openrouter
- ollama
- xai

To use a different provider, set the `provider` key in your config file:

```json
{
  "provider": "gemini"
}
```

OR use the `--provider` flag. eg. `codex --provider gemini`

Here's a list of all the providers and their default models:

| Provider   | Environment Variable Required | Default Agentic Model        | Default Full Context Model |
| ---------- | ----------------------------- | ---------------------------- | -------------------------- |
| openai     | OPENAI_API_KEY                | o4-mini                      | o3                         |
| gemini     | GOOGLE_GENERATIVE_AI_API_KEY  | gemini-2.5-pro-preview-03-25 | gemini-2.0-flash           |
| openrouter | OPENROUTER_API_KEY            | openai/o4-mini               | openai/o3                  |
| ollama     | Not required                  | User must specify            | User must specify          |
| xai        | XAI_API_KEY                   | grok-3-mini-beta             | grok-3-beta                |

#### When using an alternative provider, make sure you have the correct environment variables set.

```bash
export GOOGLE_GENERATIVE_AI_API_KEY="your-gemini-api-key-here"
```

---

## FAQ

<details>
<summary>What's the difference between this and the original OpenAI Codex CLI?</summary>

This is a fork of the original OpenAI Codex CLI project with expanded support for multiple AI providers beyond just OpenAI. The installation package is also different (`open-codex` instead of `@openai/codex`), but the core functionality remains similar.

</details>

<details>
<summary>How do I stop Codex from touching my repo?</summary>

Codex always runs in a **sandbox first**. If a proposed command or file change looks suspicious you can simply answer **n** when prompted and nothing happens to your working tree.

</details>

<details>
<summary>Does it work on Windows?</summary>

Not directly. It requires [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) – Codex has been tested on macOS and Linux with Node ≥ 22.

</details>

<details>
<summary>Which models are supported?</summary>

The default is `o4-mini`, but pass `--model gpt-4o` or set `model: gpt-4o` in your config file to override.

You can also use models from other providers like Gemini and OpenRouter. See the [Configuration](#configuration) section for more details.

</details>

---

## Zero Data Retention (ZDR) Organization Limitation

> **Note:** Codex CLI does **not** currently support OpenAI organizations with [Zero Data Retention (ZDR)](https://platform.openai.com/docs/guides/your-data#zero-data-retention) enabled.

If your OpenAI organization has Zero Data Retention enabled, you may encounter errors such as:

```
OpenAI rejected the request. Error details: Status: 400, Code: unsupported_parameter, Type: invalid_request_error, Message: 400 Previous response cannot be used for this organization due to Zero Data Retention.
```

**Why?**

- Codex CLI relies on the Responses API with `store:true` to enable internal reasoning steps.
- As noted in the [docs](https://platform.openai.com/docs/guides/your-data#responses-api), the Responses API requires a 30-day retention period by default, or when the store parameter is set to true.
- ZDR organizations cannot use `store:true`, so requests will fail.

**What can I do?**

- If you are part of a ZDR organization, Codex CLI will not work until support is added.
- We are tracking this limitation and will update the documentation if support becomes available.

## Funding Opportunity

We’re excited to launch a **$1 million initiative** supporting open source projects that use Codex CLI and other OpenAI models.

- Grants are awarded in **$25,000** API credit increments.
- Applications are reviewed **on a rolling basis**.

**Interested? [Apply here](https://openai.com/form/codex-open-source-fund/).**

## Contributing

This project is under active development and the code will likely change pretty significantly. We'll update this message once that's complete!

More broadly we welcome contributions – whether you are opening your very first pull request or you're a seasoned maintainer. At the same time we care about reliability and long‑term maintainability, so the bar for merging code is intentionally **high**. The guidelines below spell out what "high‑quality" means in practice and should make the whole process transparent and friendly.

### Development workflow

- Create a _topic branch_ from `main` – e.g. `feat/interactive-prompt`.
- Keep your changes focused. Multiple unrelated fixes should be opened as separate PRs.
- Use `npm run test:watch` during development for super‑fast feedback.
- We use **Vitest** for unit tests, **ESLint** + **Prettier** for style, and **TypeScript** for type‑checking.
- Before pushing, run the full test/type/lint suite:

  ```bash
  npm test && npm run lint && npm run typecheck
  ```

```bash
# Watch mode (tests rerun on change)
npm run test:watch

# Type‑check without emitting files
npm run typecheck

# Automatically fix lint + prettier issues
npm run lint:fix
npm run format:fix
```

### Writing high‑impact code changes

1. **Start with an issue.** Open a new one or comment on an existing discussion so we can agree on the solution before code is written.
2. **Add or update tests.** Every new feature or bug‑fix should come with test coverage that fails before your change and passes afterwards. 100 % coverage is not required, but aim for meaningful assertions.
3. **Document behaviour.** If your change affects user‑facing behaviour, update the README, inline help (`codex --help`), or relevant example projects.
4. **Keep commits atomic.** Each commit should compile and the tests should pass. This makes reviews and potential rollbacks easier.

### Opening a pull request

- Fill in the PR template (or include similar information) – **What? Why? How?**
- Run **all** checks locally (`npm test && npm run lint && npm run typecheck`). CI failures that could have been caught locally slow down the process.
- Make sure your branch is up‑to‑date with `main` and that you have resolved merge conflicts.
- Mark the PR as **Ready for review** only when you believe it is in a merge‑able state.

### Review process

1. One maintainer will be assigned as a primary reviewer.
2. We may ask for changes – please do not take this personally. We value the work, we just also value consistency and long‑term maintainability.
3. When there is consensus that the PR meets the bar, a maintainer will squash‑and‑merge.

### Community values

- **Be kind and inclusive.** Treat others with respect; we follow the [Contributor Covenant](https://www.contributor-covenant.org/).
- **Assume good intent.** Written communication is hard – err on the side of generosity.
- **Teach & learn.** If you spot something confusing, open an issue or PR with improvements.

### Getting help

If you run into problems setting up the project, would like feedback on an idea, or just want to say _hi_ – please open a Discussion or jump into the relevant issue. We are happy to help.

Together we can make Codex CLI an incredible tool. **Happy hacking!** :rocket:

### Releasing `codex`

To publish a new version of the CLI, run the release scripts defined in `codex-cli/package.json`:

1. Open the `codex-cli` directory
2. Make sure you're on a branch like `git checkout -b bump-version`
3. Bump the version and `CLI_VERSION` to current datetime: `npm run release:version`
4. Commit the version bump (with DCO sign-off):
   ```bash
   git add codex-cli/src/utils/session.ts codex-cli/package.json
   git commit -s -m "chore(release): codex-cli v$(node -p \"require('./codex-cli/package.json').version\")"
   ```
5. Copy README, build, and publish to npm: `npm run release`
6. Push to branch: `git push origin HEAD`

---

## Security & Responsible AI

Have you discovered a vulnerability or have concerns about model output? Please e‑mail **security@openai.com** and we will respond promptly.

---

## License

This repository is licensed under the [Apache-2.0 License](LICENSE).

Original project: [OpenAI Codex CLI](https://github.com/openai/codex)
