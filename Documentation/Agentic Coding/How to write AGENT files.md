---
tags:
  - llm
  - agents-md
erstellt: 2026-10-27
---

# How to write AGENT files

## [philschmid.de](https://www.philschmid.de/writing-good-agents)

### Less is more

- Auto-generated AGENT.md files are a mistake and decrease performance.
- Codebase overviews don't help agent navigate faster.
- Bad instructions make task harder -> more token usage.

### What to Include

- WHAT: Tech Stack, Project Structure, ...
- WHY: Purpose of project and key components
- HOW: How to build, test and verify (e.g. non-obvious tooling)

### What **NOT** to Include

- Detailed codebase overviews/directory listings
- Code style guidelines
- Task-specific Instructions
- Auto-generated content (**NO** `/init`)

### Important Patterns

- Keep it short (<300 lines).
- Use progressive disclosure i.e. separate into multiple files and list them in AGENTS.md file.
- Write it yourself, deliberately.
- Frontier models can reliably follow ~150-200 instructions and ~50 of those are used by the harness.

## [github.blog](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2,500-repositories/)

### Introduction

- New GitHub Copilot feature allows to build custom agents instead of one general assistant.
- Most agent files fail because they're too vague.

### What works in practice

- This knowledge comes from an analysis of over 2.500 `agents.md` files.
	- Shows a clear pattern of what works -> provide agent with specific job or persona, exact commands to run, well-defined boundaries to follow, and clear examples.
- Best-performing files use the following:
	- Put commands early.
	- Code examples are better than explanation.
	- Set clear boundaries (e.g. **Never commit secrets**).
	- Be specific about tech stack.
	- Cover six core areas:
		- Commands
		- Testing
		- Project structure
		- Code style
		- Git workflow
		- Boundaries

### Example of a great `agent.md` file

```markdown
---
name: docs_agent
description: Expert technical writer for this project
---

You are an expert technical writer for this project.

## Your role
- You are fluent in Markdown and can read TypeScript code
- You write for a developer audience, focusing on clarity and practical examples
- Your task: read code from `src/` and generate or update documentation in `docs/`

## Project knowledge
- **Tech Stack:** React 18, TypeScript, Vite, Tailwind CSS
- **File Structure:**
  - `src/` – Application source code (you READ from here)
  - `docs/` – All documentation (you WRITE to here)
  - `tests/` – Unit, Integration, and Playwright tests

## Commands you can use
Build docs: `npm run docs:build` (checks for broken links)
Lint markdown: `npx markdownlint docs/` (validates your work)

## Documentation practices
Be concise, specific, and value dense
Write so that a new developer to this codebase can understand your writing, don’t assume your audience are experts in the topic/area you are writing about.

## Boundaries
- ✅ **Always do:** Write new files to `docs/`, follow the style examples, run markdownlint
- ⚠️ **Ask first:** Before modifying existing documents in a major way
- 🚫 **Never do:** Modify code in `src/`, edit config files, commit secrets
```

#### Why this file works well

- States clear role
- Executable commands (early)
- Project knowledge (specific versions)
- Three tier boundaries

### How to build first agent

- **One Simple Task** (e.g. writing documentation, adding unit tests, ...)
- Start minimal and build from there with the following three things:
	- *Agent name*
	- *Description*
	- *Persona*
- Copilot can help generate (contradicts previous source).

### Starter Template

````markdown
---
name: your-agent-name
description: [One-sentence description of what this agent does]
---

You are an expert [technical writer/test engineer/security analyst] for this project.

## Persona
- You specialize in [writing documentation/creating tests/analyzing logs/building APIs]
- You understand [the codebase/test patterns/security risks] and translate that into [clear docs/comprehensive tests/actionable insights]
- Your output: [API documentation/unit tests/security reports] that [developers can understand/catch bugs early/prevent incidents]

## Project knowledge
- **Tech Stack:** [your technologies with versions]
- **File Structure:**
  - `src/` – [what's here]
  - `tests/` – [what's here]

## Tools you can use
- **Build:** `npm run build` (compiles TypeScript, outputs to dist/)
- **Test:** `npm test` (runs Jest, must pass before commits)
- **Lint:** `npm run lint --fix` (auto-fixes ESLint errors)

## Standards

Follow these rules for all code you write:

**Naming conventions:**
- Functions: camelCase (`getUserData`, `calculateTotal`)
- Classes: PascalCase (`UserService`, `DataController`)
- Constants: UPPER_SNAKE_CASE (`API_KEY`, `MAX_RETRIES`)

**Code style example:**
```typescript
// ✅ Good - descriptive names, proper error handling
async function fetchUserById(id: string): Promise<User> {
  if (!id) throw new Error('User ID required');
  
  const response = await api.get(`/users/${id}`);
  return response.data;
}

// ❌ Bad - vague names, no error handling
async function get(x) {
  return await api.get('/users/' + x).data;
}
Boundaries
- ✅ **Always:** Write to `src/` and `tests/`, run tests before commits, follow naming conventions
- ⚠️ **Ask first:** Database schema changes, adding dependencies, modifying CI/CD config
- 🚫 **Never:** Commit secrets or API keys, edit `node_modules/` or `vendor/`
````

### Key takeaways

- Building an effective custom agent isn't about writing vague prompts.
- Clear persona and detailed operating manual (must include executable commands)
- Remember the **rule of six**.

## [augmentcode.com](https://www.augmentcode.com/blog/how-to-write-good-agents-dot-md-files)

### The same file can help one task and hurt another by 30 %

- Single `AGENTS.md` isn't good or bad -> can improve one task and hurt another (e.g. boost best practices but reduce completeness).

### How we measured this

- Used AuggieBench, one of the internal eval suites to evaluate agents on internal dev work.
- Start with high-quality PRs from large repo that reflect typical day-to-day tasks.
- Then compare the output against the golden PR, the version that actually landed after review by multiple senior engineers.

### What works

- Progressive disclosure beats comprehensive coverage.
	- Treat `AGENTS.md` like a skill i.e. cover the common cases at high level and push details into reference files the agent can load.
- Procedural workflows take agents from failing to finishing.
	- Describing a task as numbered, multi-step workflow was one of the strongest patterns.
	- E.g. six-step workflow for deploying a new integration -> agent followed step by step and missing wiring files dropped from 40 % to 10 % and agent finished faster.
		- Correctness went up by 25 % and completeness by 20 %.
- Decision tables resolve ambiguity before the agent writes code.
	- When codebase has two or three reasonable ways to do something, decision tables force choice up front.
- Examples from the real codebase improve code reuse.
	- Short snippets of three to ten lines from actual production code improve reuse and pattern adherence.
- Domain-specific rules matter (e.g. use decimal instead of float for all financial calculations).
- Pair every „don't“ with a „do“.
- Keep your code modular, and `AGENTS.md` too.
	- Best-performing agent docs described relatively isolated submodules.
	- Mid-size modules around 100 core files, with a 100-150 line `AGENTS.md` saw 10-15 % cross-metric gains.
	- Cross-cutting, long `AGENTS.md` files at the repo root under-performed.

### Where `AGENTS.md` falls short

- **Overexploration Trap**
	- Most common failure mode (essentially context rot)
	- Too much architecture overview -> agent gets pulled into reading documentation files to „better“ understand architecture which causes too many tokens being loaded into context.
	- Excessive warnings -> big section of „don't“ without matching „do“ produces failure -> agent reads each instruction, tries to figure out whether it applies and starts verifying its solution against every single warning.

### Know what you're optimizing for

### How agents actually find your docs

- `AGENTS.md` gets discovered 100 % of the time.
- References out of `AGENTS.md` are loaded 90 % of the time.
- Directory level `README.md` files aren't auto-loaded, but the agent reads them 80 % of time **when working in that directory**.
- Nested `README.md` files (i.e. in subdirectories the agent isn't currently working in) are read only 40 % of the time.
- Orphan docs in `_docs/` get read in under 10 % of the cases.

## [tembo.io](https://www.tembo.io/blog/agents-md)

### Introduction

- Coding agents always have the same first problem: they show up in repo knowing nothing.
	- Doesn't know build command, test runner, naming conventions etc.
- This is solved by `AGENTS.md`.

### How to write an `AGENTS.md` file

```markdown
# AGENTS.md

## Project overview
Node.js monorepo for the Acme billing platform. TypeScript everywhere.
Services live in /services, shared libraries in /packages.

## Setup commands
- Install deps: `pnpm install`
- Start dev server: `pnpm dev`
- Run all tests: `pnpm test`

## Code style
- TypeScript strict mode
- Single quotes, no semicolons
- Prefer functional patterns; avoid classes for new code

## Testing instructions
- CI config lives in .github/workflows
- Run `pnpm vitest run -t "<test name>"` to focus a single test
- The commit should pass lint and all tests before merge

## Security considerations
- Never commit .env files or anything in /secrets
- Payment code in /services/billing requires a human review tag

## PR instructions
- Title format: [<service>] <description>
- Always run `pnpm lint` and `pnpm test` before committing
```

- Write commands, not prose.
- Tell it what you'd tell a new teammate.
- Keep it tight.

## Andere Quellen

- [Agent Rules Gallery](https://agent-rules-gallery.hijo-luna-2050.chatgpt.site/)
- [OpenCode: Rules](https://opencode.ai/docs/de/rules)
- [arXiv 2602.11988](https://arxiv.org/abs/2602.11988)
