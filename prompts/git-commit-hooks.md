You are a professional Bun + Git Hooks configuration expert.

I am using **Bun** as my package manager (no npm or yarn in the project). Please set up **Husky + Commitlint** for me with the following strict requirements:

1. Use Conventional Commits specification
2. Force the subject to start with a **lowercase letter** and be **entirely in English**
3. Keep all standard types: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert

You may reference the latest official documentation if needed:
- Husky official docs: https://typicode.github.io/husky/
- Commitlint official docs: https://commitlint.js.org/
- Commitlint Getting Started: https://commitlint.js.org/guides/getting-started.html

Please generate the complete, ready-to-run configuration by strictly following these steps:

### Required Steps:
1. **Install dependencies**: Provide the correct `bun add` command (only the necessary packages)
2. **Initialize Husky**: Use the Bun-compatible way (Husky v9+ recommended — see official docs)
3. **Create the commit-msg hook**: Provide the full content of the `.husky/commit-msg` file and explain how to make it executable on macOS/Linux/Windows
4. **Create the Commitlint config file**: Generate `commitlint.config.js` with these rules:
   - `subject-case`: force lowercase (`lower-case`)
   - `header-max-length`: 72
   - `subject-empty`: cannot be empty
   - `type-empty`: cannot be empty
   - Extend `@commitlint/config-conventional`

5. **Provide test commands** so I can verify the setup works immediately

6. **Additionally provide**:
   - 8 example commits that fully comply with the rules (subject must be lowercase + English only)
   - Command to bypass the check in emergencies
   - Recommendation for team members (how to auto-install hooks after `bun install` using package.json prepare script)

Output everything in clear Markdown format:
- Number each step clearly
- Show every file to create or modify in complete, ready-to-copy code blocks
- Use only `bun` and `bunx` commands — never use npx or npm
- Keep explanations concise but precise

Start generating the full configuration step by step right now!
