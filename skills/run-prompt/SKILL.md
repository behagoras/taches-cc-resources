---
name: run-prompt
description: Ejecuta prompts guardados por número, nombre o namespace de pull request en contextos de sub-agentes independientes.
---

---
name: run-prompt
description: Delegate one or more prompts to fresh sub-task contexts by number, name, or case-insensitive pr-<number> namespace with parallel or sequential execution
argument-hint: <prompt-number(s)-name-or-pr-namespace> [--parallel|--sequential]
allowed-tools: [Read, Task, "Bash(ls:*)", "Bash(mv:*)", "Bash(git:*)"]
---

<context>
Git status: !`git status --short 2>/dev/null || echo "(not a git repository — git steps will be skipped)"`
Prompt index: !`cat ./.prompts/index.json 2>/dev/null || echo "no index.json"`
Recent prompts: !`find ./.prompts -name '*.md' -not -path '*/completed/*' -not -name 'INDEX.md' | sort -t/ -k4 -rn | head -10`
</context>

<objective>
Execute one or more prompts from `./.prompts/` as delegated sub-tasks with fresh context. Supports single prompt execution, parallel execution of multiple independent prompts, and sequential execution of dependent prompts.
</objective>

<input>
The user will specify which prompt(s) to run via $ARGUMENTS, which can be:

**Single prompt:**

- Empty input runs the most recently created prompt by default
- A prompt number such as "001", "5", or "42"
- A partial filename such as "user-auth" or "dashboard"
- A pull request namespace in any letter case such as "pr-23", "PR-23", or "Pr-23"

**Multiple prompts:**

- Multiple numbers such as "005 006 007"
- An execution flag such as "005 006 007 --parallel" or "005 006 007 --sequential"
- If no flag specified with multiple prompts, default to --sequential for safety

<resolution_contract>

Resolve selectors only against prompt files already saved in `./.prompts/`. Do not load or inspect a pull request.

1. Search every category recursively. Exclude `completed/` and `INDEX.md`.
2. With no selector or with `last`, choose the most recently modified active prompt.
3. When an entire selector matches case-insensitive `(?i)^pr-[1-9][0-9]*$`, normalize it to lowercase `pr-<number>` and find active files matching `./.prompts/**/pr-<number>-*.md`. Treat this value as a namespace, never as numeric prompt `<number>`. Inputs such as `pr-0`, `pr-01`, and `pr-23-extra` follow the ordinary text-matching rule.
4. Match an ordinary number against the numbered filename prefix. Match ordinary text case-insensitively against filenames.
5. Use one match directly. If one selector matches several files, list the candidates and ask the user to choose. When the user supplies `--parallel` or `--sequential` with multiple explicit filename stems and every stem resolves uniquely, keep those files as separate selections and do not ask the user to choose among them.
6. If a selector has no matches, report that selector and list the available active prompts. For a pull request namespace, identify the normalized namespace in the report.
7. Archive every successfully executed prompt in the `completed/` directory inside its current category.

</resolution_contract>
  </input>

<process>
<step1_parse_arguments>
Parse $ARGUMENTS to extract:
- Prompt numbers/names (all arguments that are not flags)
- Execution strategy flag (--parallel or --sequential)
- Case-insensitive pull request namespaces, normalized according to `<resolution_contract>`

<examples>
- "005" → Single prompt: 005
- "005 006 007" → Multiple prompts: [005, 006, 007], strategy: sequential (default)
- "005 006 007 --parallel" → Multiple prompts: [005, 006, 007], strategy: parallel
- "005 006 007 --sequential" → Multiple prompts: [005, 006, 007], strategy: sequential
- "PR-23" → Pull request namespace: pr-23
- "pr-23-api pr-23-ui --parallel" → Multiple filename stems: [pr-23-api, pr-23-ui], strategy: parallel
</examples>
</step1_parse_arguments>

<step2_resolve_files>
Resolve every selector according to `<resolution_contract>`. Preserve the user's selector order after disambiguation.
  </step2_resolve_files>

<step3_execute>
<single_prompt>

1. Read the complete contents of the prompt file
2. Delegate as sub-task using Task tool with subagent_type="general-purpose"
3. Wait for completion
4. Archive the prompt according to `<resolution_contract>`
5. Commit all work:
   - Stage files YOU modified with `git add [file]` (never `git add .`)
   - Determine appropriate commit type based on changes (fix|feat|refactor|style|docs|test|chore)
   - Commit with format: `[type]: [description]` (lowercase, specific, concise)
6. Return results
   </single_prompt>

<parallel_execution>

1. Read all prompt files
2. **Spawn all Task tools in a SINGLE MESSAGE** (this is critical for parallel execution):
   <example>
   Use Task tool for prompt 005
   Use Task tool for prompt 006
   Use Task tool for prompt 007
   (All in one message with multiple tool calls)
   </example>
3. Wait for ALL to complete
4. Archive all prompts according to `<resolution_contract>`
5. Commit all work:
   - Stage files YOU modified with `git add [file]` (never `git add .`)
   - Determine appropriate commit type based on changes (fix|feat|refactor|style|docs|test|chore)
   - Commit with format: `[type]: [description]` (lowercase, specific, concise)
6. Return consolidated results
   </parallel_execution>

<sequential_execution>

1. Read first prompt file
2. Spawn Task tool for first prompt
3. Wait for completion
4. Archive first prompt according to `<resolution_contract>`
5. Read second prompt file
6. Spawn Task tool for second prompt
7. Wait for completion
8. Archive second prompt according to `<resolution_contract>`
9. Repeat for remaining prompts
10. Confirm that all successful prompts were archived according to `<resolution_contract>`
11. Commit all work:
    - Stage files YOU modified with `git add [file]` (never `git add .`)
    - Determine appropriate commit type based on changes (fix|feat|refactor|style|docs|test|chore)
    - Commit with format: `[type]: [description]` (lowercase, specific, concise)
12. Return consolidated results
    </sequential_execution>
    </step3_execute>
    </process>

<context_strategy>
By delegating to a sub-task, the actual implementation work happens in fresh context while the main conversation stays lean for orchestration and iteration.
</context_strategy>

<output>
<single_prompt_output>
✓ Executed: ./.prompts/005-implement-feature.md
✓ Archived to: ./.prompts/completed/005-implement-feature.md

<results>
[Summary of what the sub-task accomplished]
</results>
</single_prompt_output>

<parallel_output>
✓ Executed in PARALLEL:

- ./.prompts/005-implement-auth.md
- ./.prompts/006-implement-api.md
- ./.prompts/007-implement-ui.md

✓ All archived to ./.prompts/completed/

<results>
[Consolidated summary of all sub-task results]
</results>
</parallel_output>

<sequential_output>
✓ Executed SEQUENTIALLY:

1. ./.prompts/005-setup-database.md → Success
2. ./.prompts/006-create-migrations.md → Success
3. ./.prompts/007-seed-data.md → Success

✓ All archived to ./.prompts/completed/

<results>
[Consolidated summary showing progression through each step]
</results>
</sequential_output>
</output>

<critical_notes>

- For parallel execution: ALL Task tool calls MUST be in a single message
- For sequential execution: Wait for each Task to complete before starting next
- Archive prompts only after successful completion
- If the working directory is NOT a git repository (Git status shows "(not a git repository ...)"), SKIP all git staging/commit steps entirely — do not fail, just proceed without committing and note it in the output
- If any prompt fails, stop sequential execution and report error
- Provide clear, consolidated results for multiple prompt execution
  </critical_notes>
