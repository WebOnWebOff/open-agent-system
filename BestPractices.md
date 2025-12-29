# Open Agent System: Best Practices

A concise guide to building effective, maintainable Open Agent Systems.

---

## Core Principle: The Deterministic Rule

**If an operation has one correct answer, use code tools. If it requires judgment, use LLM reasoning.**

| Task Type | Use | Why |
|-----------|-----|-----|
| Deterministic | Code tool | Consistent, fast, reliable, free |
| Creative/Ambiguous | LLM | Flexible, interpretive, contextual |

### Deterministic Operations (Use Code)

- Parsing structured data (JSON, XML, YAML, CSV)
- File format conversions (JSON→CSV, Markdown→HTML)
- Mathematical calculations (checksums, file sizes, statistics)
- Text validation (email format, URL format, regex patterns)
- File system operations (copy, move, rename, traverse)
- Batch processing with defined rules
- Template filling with known values

### Creative Operations (Use LLM)

- Summarizing content
- Writing prose
- Making editorial choices
- Interpreting ambiguous data
- Deciding tone, style, emphasis
- Making judgment calls

### Hybrid Approach

Some operations use both:
- **HTML generation**: LLM structures content, code produces valid syntax
- **Report creation**: LLM writes analysis, code formats and validates
- **Data extraction**: Code parses structure, LLM interprets ambiguous fields

---

## Agent Design Patterns

### Single Responsibility

Each agent should do ONE thing well.

**Good**: Separate agents for research, conversion, publishing  
**Bad**: One "super agent" that does everything

### Clear Contracts

Define exactly what goes in and what comes out.

```markdown
## Input Requirements
- Format, location, required fields

## Output Guarantees
- Format, location, naming pattern
```

### Fail-Fast Validation

Validate inputs BEFORE processing using code tools.

```markdown
## Pre-flight Checks (Code)

1. Verify input file exists
2. Check file format/structure
3. Ensure output directory exists
4. Validate disk space

If any check fails, report error and exit.
```

### Idempotency

Running an agent twice should be safe. Check before overwriting.

---

## Tool Creation Strategy

### When to Create a Tool

Create a script when:
- Operation will repeat (more than twice)
- Operation is deterministic
- Speed matters (batch operations)
- Consistency critical (same result every time)

### Tool Workflow

1. **First time**: Perform inline, document approach
2. **Second time**: Recognize repetition
3. **Create tool**: Write script, save to `open-agents/tools/`
4. **Document**: Add to agent's "Available Tools" section
5. **Use**: Reference tool instead of regenerating logic

### Tool Standards

Every tool should have:
- Header comment (purpose, usage)
- Input validation
- Error handling with clear messages
- Exit codes (0=success, non-zero=failure)

---

## Output Organization

### Progressive Stages

Use folders to represent workflow stages:

```
output-drafts/      # Raw, unvalidated
output-reviewed/    # Human-reviewed
output-final/       # Ready for use
```

### Consistent Naming

**Good**: `2025-12-29_article.md`, `history_of_aviation.md`  
**Bad**: `my article.md`, `article!final!.md`, `doc.md`

### Metadata Strategy

Include frontmatter for tracking:

```yaml
---
created: 2025-12-29
agent: researcher
version: 1
status: draft
---
```

---

## Performance Optimization

### Minimize LLM Calls

**Bad**: Call LLM for each file in batch  
**Good**: Use code loop, LLM only for creative decisions

### Parallelize When Possible

Use code to run independent operations concurrently.

### Cache Expensive Operations

If an operation is expensive:
- Save results to file
- Check cache before regenerating
- Use code to manage cache

---

## Maintainability

### Document Everything

Every agent needs:
- Clear purpose statement
- Trigger conditions
- Example inputs/outputs
- Known limitations

### Version Outputs

Include version info:

```yaml
---
agent_version: researcher_v2
spec_version: 1.0.0
---
```

### Keep Agents Focused

If an agent file exceeds 500 lines:
- Split into multiple agents
- Extract shared behaviors
- Create tool scripts for complexity

### Consistent Terminology

Pick terms and stick to them across all agents.

---

## Testing

### Create Test Inputs

- `test/inputs/valid/` - Should succeed
- `test/inputs/invalid/` - Should fail gracefully
- `test/expected/` - Known-good outputs

### Validation Checklist

- [ ] Output created in correct location
- [ ] Filename matches expected pattern
- [ ] Format is valid
- [ ] Required metadata present
- [ ] No errors or hallucinations
- [ ] Code tools executed successfully

---

## Common Pitfalls

### Pitfall: Using LLM for Deterministic Work

**Problem**: Asking LLM to parse JSON, calculate values, validate formats  
**Solution**: Write a simple script once, reuse forever

### Pitfall: Agents That Do Too Much

**Problem**: Single agent handles too many responsibilities  
**Solution**: Split into focused agents with clear boundaries

### Pitfall: Unclear Triggers

**Problem**: Multiple agents respond to same request  
**Solution**: Use explicit trigger phrases, maintain routing table

### Pitfall: No Error Handling

**Problem**: Agent crashes on missing/invalid input  
**Solution**: Validate with code before LLM processing

### Pitfall: Hardcoded Paths

**Problem**: Absolute paths that break on other systems  
**Solution**: Use workspace-relative paths

### Pitfall: No Version Control

**Problem**: Can't track agent behavior changes  
**Solution**: Git commit after modifications

---

## Advanced Patterns

### Agent Chaining

Design agents to feed each other with clear contracts:

```
Researcher → Converter → Publisher
     ↓
  Validator
```

### Self-Modification

Agents can improve themselves:

```markdown
If you encounter the same error twice:
1. Create validation tool to prevent it
2. Update this agent to use the tool
3. Commit changes
```

### Dynamic Tool Creation

```markdown
When you recognize a repeated operation:
1. Create script in `open-agents/tools/`
2. Test it
3. Document in this agent file
4. Use immediately
```

### Human-in-the-Loop

```markdown
After generating draft:
1. Save to `output-drafts/`
2. Prompt: "Review draft. Continue? (y/n)"
3. Proceed only on approval
```

---

## Summary

**Key Principles:**

1. **Deterministic → Code**: If it has one right answer, use a script
2. **Single Responsibility**: One agent, one job
3. **Validate Early**: Check inputs with code before LLM processing
4. **Create Tools**: Don't regenerate same logic repeatedly
5. **Document Everything**: Clear purpose, triggers, examples
6. **Test Thoroughly**: Known-good test cases
7. **Version Control**: Systematic commits

Following these practices creates systems that are fast, reliable, and maintainable.

---

**See also:**
- [OpenAgentDefinition.md](OpenAgentDefinition.md) - System architecture and structure
- [Troubleshooting.md](Troubleshooting.md) - Diagnosing and fixing common issues
