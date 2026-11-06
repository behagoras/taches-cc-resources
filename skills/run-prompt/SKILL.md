---
name: run-prompt
description: Ejecuta prompts guardados por número, nombre o namespace de pull request, sin volver a cargar el PR.
---

---
name: run-prompt
description: Delegate saved prompts by number, name, or case-insensitive pr-<number> namespace without reloading the pull request
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
- A pull request namespace such as "pr-23" in any letter case

**Multiple prompts:**

- Multiple numbers such as "005 006 007"
- An execution flag such as "005 006 007 --parallel" or "005 006 007 --sequential"
- If no flag specified with multiple prompts, default to --sequential for safety

<resolution_contract>

Resolve selectors only against active prompt files in `./.prompts/`. Search every category recursively and exclude `completed/` and `INDEX.md`. With no selector or `last`, choose the most recently modified prompt. Match numbers against filename prefixes and other text case-insensitively against filenames.

When an entire selector matches case-insensitive `(?i)^pr-[1-9][0-9]*$`, normalize it to lowercase `pr-<number>` and match `./.prompts/**/pr-<number>-*.md`. Treat it as a namespace, never as numeric prompt `<number>`. Do not load or inspect the pull request.

Run one match directly. Ask the user to choose among multiple matches. Multiple explicit filename stems may run together when each resolves uniquely and the user supplies `--parallel` or `--sequential`. For no match, report the selector, its normalized namespace when applicable, and the available active prompts.

</resolution_contract>
  </input>

<process>
<step1_parse_arguments>
Parse $ARGUMENTS to extract:
- Prompt numbers/names (all arguments that are not flags)
- Execution strategy flag (--parallel or --sequential)

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
4. Archive the prompt in the `completed/` directory inside its current category
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
4. Archive each prompt in the `completed/` directory inside its current category
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
4. Archive the first prompt in the `completed/` directory inside its current category
5. Read second prompt file
6. Spawn Task tool for second prompt
7. Wait for completion
8. Archive the second prompt in the `completed/` directory inside its current category
9. Repeat for remaining prompts
10. Confirm that every successful prompt was archived
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
✓ Archived to: ./.prompts/1xx-example/completed/005-implement-feature.md

<results>
[Summary of what the sub-task accomplished]
</results>
</single_prompt_output>

<parallel_output>
✓ Executed in PARALLEL:

- ./.prompts/005-implement-auth.md
- ./.prompts/006-implement-api.md
- ./.prompts/007-implement-ui.md

✓ Archived in each prompt's category

<results>
[Consolidated summary of all sub-task results]
</results>
</parallel_output>

<sequential_output>
✓ Executed SEQUENTIALLY:

1. ./.prompts/005-setup-database.md → Success
2. ./.prompts/006-create-migrations.md → Success
3. ./.prompts/007-seed-data.md → Success

✓ Archived in each prompt's category

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
