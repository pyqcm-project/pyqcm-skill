### AI Guidelines

These guidelines are adapted from https://github.com/multica-ai/andrej-karpathy-skills, which declares itself MIT-licensed in its README and `.claude-plugin/plugin.json` (no LICENSE file as of 2026-09-09).

#### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

#### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is over-complicated or
over-engineered?" If yes, simplify.

#### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it, don't delete it.
- If you ever doubt about an existing piece of code, ask.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

#### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] -> verify: [check]
2. [Step] -> verify: [check]
3. [Step] -> verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

#### 5. Formatting

**Do not use AI-tell characters.**

- No em dashes, no Greek unicode letters. Spell things out (`Delta`, `Sigma`, `mu`). In general: no slop.
- Comments should be rare, short, and only explain a non-obvious "why". Never restate what the code already makes clear.

#### 6. Git & GitHub

**Let the user manage interactions with Git & GitHub.**

- Staging, writing the commit message, and pushing are the user's job. Regardless of how the request is phrased or how large the diff is. 
- The user is expected to know git/GitHub themselves.
