# Open Agent System: Troubleshooting Guide

A concise guide to diagnosing and fixing common issues.

---

## Diagnostic Approach

### Five-Question Method

When something goes wrong:

1. **Was the agent loaded?** Check if INSTRUCTIONS.md was read
2. **Was the right agent triggered?** Check routing logic
3. **Did inputs validate?** Check file existence, formats, permissions
4. **Did code tools work?** Check script execution, errors
5. **Did LLM processing succeed?** Check output completeness

### Getting Information

Ask the AI:
- "What instructions did you load?"
- "Which agent definition are you following?"
- "Does the file `path/to/file` exist?"
- "What's in directory `open-agents/output-drafts/`?"

---

## System Not Loading

### Symptom: AI doesn't recognize Open Agent System

**Check:** Does `AGENTS.md` have mandatory read directive?

**Expected:**
```markdown
**CRITICAL: Read `open-agents/INSTRUCTIONS.md` immediately.**
```

**Common mistakes:**
- Missing directive
- Wrong path (`open_agents/`, `openagents/`)
- Wrong filename case (`instructions.md`)
- File doesn't exist

**Fix:** Create/correct `AGENTS.md` and `open-agents/INSTRUCTIONS.md`

---

## Agent Not Triggering

### Symptom: User request doesn't activate any agent

**Check routing table** in `open-agents/INSTRUCTIONS.md`:
- Is there a trigger for this request?
- Is the trigger clear and specific?

**Problem:** Generic triggers like "research" match too broadly

**Fix:** Use specific phrases:
```markdown
| "research the history of" | Researcher |
| "convert article to HTML" | HTML Generator |
```

**Test:** Say "Use the [agent name] agent to..."
- Works → Routing issue, update triggers
- Fails → Agent definition issue

---

## Wrong Agent Triggered

### Symptom: Different agent than expected responds

**Check for overlapping triggers:**

**Problem:**
```markdown
| "create article" | Researcher |
| "create HTML" | HTML Generator |
```
User: "create article HTML" → Ambiguous!

**Fix:** More specific triggers
```markdown
| "research and write article" | Researcher |
| "convert markdown to HTML" | HTML Generator |
```

**Check trigger order:** More specific before general

**Add negative triggers** in agent definition:
```markdown
## When to Use

Use when: User asks to "research a topic"

Do NOT use when: 
- User asks to "research a codebase"
- User asks to "research a file"
```

---

## Agent Execution Errors

### Symptom: Agent starts but fails during execution

**Identify error stage:**
- Validation failed? (missing input)
- Code tool failed? (script error)
- LLM failed? (incomplete output)
- File write failed? (permissions)

### Input Validation Issues

**Common:**
- Input file doesn't exist
- Input file wrong format
- Required fields missing

**Fix:** Add pre-flight checks to agent:

```markdown
## Pre-flight Checks (Code)

1. Verify input exists
2. Check format/structure
3. Ensure output directory exists
4. Check disk space
```

### Code Tool Failures

**Common:**
- Script doesn't exist
- Not executable
- Syntax errors
- Wrong parameters

**Debug:**
- Check script exists and location
- Check executable permissions
- Run manually to see errors
- Verify parameters

**Fix:**
- Make executable: `chmod +x script.sh`
- Add error handling to scripts
- Test independently

### LLM Output Issues

**Common:**
- Incomplete (token limit)
- Wrong format
- Truncated

**Debug:**
- Check if output file created
- Check file size
- Verify structure

**Fix:**
- Break into sections
- Use explicit templates
- Validate with code after generation

---

## Deterministic Operations Failing

### Symptom: Inconsistent results for same input

**This indicates LLM is being used for deterministic work.**

**Examples:**
- Parsing JSON varies between runs
- Calculations produce different results
- Format conversions inconsistent

**Fix:** Replace LLM reasoning with code tools

**Bad:**
```markdown
Parse the JSON and extract the title field.
```

**Good:**
```markdown
Extract title using:
```bash
jq -r '.title' input.json
```
```

**See:** [BestPractices.md#the-deterministic-rule](BestPractices.md#the-deterministic-rule) for comprehensive guidance

---

## Output Issues

### Wrong Location

**Check agent definition:**
```markdown
## Output Location

Save to: `open-agents/output-articles/`
```

**Fix:** Add directory creation and verification:
```markdown
1. Create directory: `mkdir -p open-agents/output-articles/`
2. Save file
3. Verify: Check file exists at expected path
```

### Wrong Format

**Check agent has clear template:**
```markdown
## Output Format

```markdown
# Title

## Section
...
```
```

**Fix:** Validate output with code after generation

### Missing/Incomplete

**Possible causes:**
- File write failed (permissions)
- LLM stopped early (token limit)
- Output generated but not saved

**Debug:**
- Check file exists
- Check file size (0 bytes?)
- Check parent directory permissions

**Fix:**
- Add explicit save step
- Verify write succeeded
- Create parent directories

---

## Performance Problems

### Symptom: Agent is slow or uses excessive resources

**Issue:** Using LLM for batch operations

**Bad:**
```markdown
For each file:
- Read file
- Analyze content
- Extract data
```

**Good:**
```markdown
Use code loop for iteration:
- LLM only for creative decisions
- Code for parsing/extraction
```

**Issue:** Regenerating same logic repeatedly

**Fix:** Create tool once, reuse

**Issue:** Loading too much context

**Fix:** Use progressive disclosure (built into Open Agent pattern)

---

## Tool/Command Problems

### Symptom: Commands don't work

**Check command files exist:**
- `.claude/skills/{domain}-{command}/SKILL.md`
- `.codex/prompts/{domain}-{command}.md`
- `.gemini/commands/{domain}-{command}.toml`

**Check command points to correct agent:**

**Expected:**
```markdown
Follow instructions in `open-agents/agents/researcher.md`.
```

**Common mistakes:**
- Wrong path (missing `open-agents/`)
- Wrong filename
- File doesn't exist

**Check format** matches tool requirements

---

## Context/Memory Issues

### Symptom: Agent "forgets" instructions or behaves inconsistently

**Issue:** Agent file not loaded

**Debug:** Ask "What agent definition are you following?"

**Fix:**
- Verify file path in command
- Verify file exists
- Manually instruct to read file

**Issue:** Context window exceeded

**Symptoms:**
- Strong at start, degrades
- Forgets earlier instructions
- Contradicts itself

**Fix:**
- Keep agent files focused (<500 lines)
- Use tools for deterministic operations
- Start new conversation for complex workflows

**Issue:** Conflicting instructions

**Fix:**
- Make agent file authoritative
- Remove conflicting instructions
- Be explicit about precedence

---

## Debugging Techniques

### Verbose Mode

Add to agent definition:
```markdown
## Debug Mode

When user says "debug mode on":
- Report each step before executing
- Show commands before running
- Display file contents after creating
- Explain reasoning
```

### Dry Run

Add to agent behavior:
```markdown
## Dry Run

When user says "dry run":
- Describe actions without executing
- Show what commands would run
- Explain what files would be created
- Ask for confirmation
```

### Step-by-Step

Say: "Execute step by step, pausing after each action"

### Minimal Reproduction

1. Create simplest possible input
2. Run agent
3. Identify exact failure point
4. Fix specific issue
5. Test again

---

## Emergency Recovery

### Agent Corrupted Its Own Definition

**Symptoms:** Erratic behavior, nonsensical instructions

**Recovery:**
```bash
# Restore from git
git checkout HEAD -- open-agents/agents/broken_agent.md
```

**Prevention:** Commit working agents immediately

### System Completely Broken

**Recovery:**
1. Verify `AGENTS.md` exists with read directive
2. Verify `open-agents/INSTRUCTIONS.md` exists
3. Start new conversation
4. Explicitly read: "Read `open-agents/INSTRUCTIONS.md`"

### Tools Directory Corrupted

**Recovery:**
```bash
# List current state
ls -la open-agents/tools/

# Remove broken scripts
rm open-agents/tools/broken_script.sh

# Restore from git
git checkout HEAD -- open-agents/tools/
```

**Prevention:** Test tools independently, version control

### Output Directories Filled with Junk

**Recovery:**
```bash
# Archive before deleting
mkdir -p open-agents/.archive/$(date +%Y%m%d)
mv open-agents/output-*/* open-agents/.archive/$(date +%Y%m%d)/
```

**Prevention:** Use timestamps, regular cleanup

---

## Quick Reference: Common Errors

| Error | Likely Cause | Fix |
|-------|--------------|-----|
| "File not found" | Input doesn't exist | Verify path, check spelling |
| "Permission denied" | Can't write | Check permissions |
| "Command not found" | Script missing | Check path, verify exists |
| "Not executable" | No execute permission | `chmod +x script.sh` |
| "Syntax error" | Script has errors | Run with debug flag |
| "Unknown agent" | Routing failed | Check triggers |
| Inconsistent results | LLM for deterministic | Use code tools |

---

## Getting Help

If this guide doesn't solve your problem:

1. Re-read [OpenAgentDefinition.md](OpenAgentDefinition.md) for expected behavior
2. Review [BestPractices.md](BestPractices.md) for proper patterns
3. Create minimal agent to isolate problem
4. Simplify until it works, add back incrementally

**Most problems are:**
1. Wrong file path (typos, case)
2. Using LLM for deterministic work
3. Missing validation
4. Conflicting instructions

Check these first.

---

**See also:**
- [OpenAgentDefinition.md](OpenAgentDefinition.md) - System architecture
- [BestPractices.md](BestPractices.md) - Effective patterns and principles
