# Multi-Agent Orchestrator

Intelligently coordinate multiple AI agents in parallel for complex tasks. Auto-analyzes requirements, selects optimal agent combinations, and synthesizes results.

## Features

✅ **Smart Routing** — Analyzes task keywords to select 1-3 best agents
✅ **Parallel Execution** — Runs agents simultaneously, not sequentially
✅ **Silent Failure Handling** — Continues with available results if an agent fails
✅ **4 Operating Modes** — Auto, Review, Challenge, Consult
✅ **Synthesized Reports** — Combines multiple perspectives into decisive recommendations
✅ **Multi-Language Support** — Understands English, 中文, 日本語, 한국어

## Quick Start

### Default (Auto Mode)
User describes task → Claude auto-routes to best agents:

```
User: "審查這個認證流程，有沒有安全漏洞"
Claude: 分析 → 選擇 codex + cline (並行)
Output: Cross-review report
```

### Explicit Invocations

**Review Mode** — Code cross-review:
```
/agents review
```

**Challenge Mode** — Adversarial debate:
```
/agents challenge
```

**Consult Mode** — Specify agents:
```
/agents consult kilo,gemini
```

## Available Agents

| Agent | Specialty |
|-------|-----------|
| **codex** | Code review, refactoring, security, tests |
| **gemini** | Architecture, design patterns, broad knowledge |
| **kilo** | Logic, system design, complex reasoning |
| **cline** | Implementation details, structured analysis |
| **opencode** | Rapid prototyping, code completion, automation |
| **qoder** | Information retrieval, documentation, knowledge base |

## Task Classification

Automatically detects and routes:

- **Code Review** → codex + gemini
- **Feature Implementation** → cline + opencode
- **Architecture Design** → kilo + gemini
- **Analysis** → gemini + kilo
- **Prototyping** → opencode + cline
- **Knowledge Lookup** → qoder + gemini
- **Security Audit** → codex + cline
- **Debugging** → codex + kilo

## Installation

### Project Level
```bash
# Already installed at:
# ~/YD 2026/.claude/skills/agents/
```

### Global Installation (via triple-publish)
```bash
cd ~/YD\ 2026/.claude/skills/agents/
/triple-publish  # Deploy to GitHub + npm + PyPI
```

Or manually:
```bash
# Copy to global skills
cp -r ~/YD\ 2026/.claude/skills/agents/ ~/.claude/skills/agents/
```

## Output Format

All modes produce structured reports:

```markdown
## 🤖 Multi-Agent Report

**Task**: [description]
**Mode**: Auto | Review | Challenge | Consult
**Agents Called**: codex, gemini
**Duration**: ~45s

### Agent 1 [PASS|CONCERNS|BLOCK]
...

### Agent 2 [PASS|CONCERNS|BLOCK]
...

### Synthesis
**Consensus**: ...
**Disagreement**: ...
**Blind Spots**: ...
**Conclusion**: [PASS | APPROVE_WITH_CHANGES | BLOCK]
**Claude's Take**: ...
```

## Auto-Trigger Scenarios

The orchestrator automatically uses multiple agents when:

| Scenario | Trigger | Agents |
|----------|---------|--------|
| Code review / PR | Large changes (>100 lines) | codex + gemini |
| Architecture decision | Major refactor, new pattern | gemini + kilo |
| Security-sensitive changes | auth, crypto, secrets | codex + cline |
| Debugging | Stuck for 3+ turns | codex or kilo |
| User requests "second opinion" | Explicit mention | Best 1-2 agents |

## Reliability

### Constraints Respected
- ✅ Rate limit awareness (skip qoder on frequent runs)
- ✅ Context efficiency (keep prompts <2000 chars)
- ✅ Timeout protection (60s per agent, 90s total)
- ✅ Budget management (qoder: 1-2 runs/week)

### Failure Handling
- 1 agent fails → Use available agent's output + note limitation
- All agents fail → Report reason (timeout, rate limit, network)
- Never silently drop results

## Configuration

### Enable in Claude Code
The skill is available by default. To verify:

```bash
# Check installation
cat ~/.claude/skills/agents/SKILL.md | head -20

# Or in Claude Code
/agents consult gemini "what's your favorite algorithm?"
```

### Per-Project Settings
Add to `.claude/settings.json`:
```json
{
  "enabledPlugins": {
    "agents-skill@builtin": true
  }
}
```

## Examples

### Example 1: Auto Code Review
```
User: "Help me review this authentication module"
→ Auto-detects: code review task
→ Routes to: codex (code quality) + gemini (design)
→ Output: Combined report with security & architecture insights
```

### Example 2: Challenge Mode
```
User: "/agents challenge Should we use microservices?"
→ Auto-selects: gemini (pro) vs kilo (con)
→ Output: Debate exposing trade-offs and blind spots
```

### Example 3: Consult Specific Agents
```
User: "/agents consult kilo,cline Design this database schema"
→ Routes only to: kilo + cline
→ Output: Focused analysis on both architectural & implementation aspects
```

## Troubleshooting

### Agent not responding
- Check if agent is installed: `<agent> --version`
- Check authentication: run agent login script
- Check rate limits: some agents have weekly quotas
- Wait a moment and retry

### Slow execution
- Parallel execution normally takes 30-90s total
- If >120s, one agent likely timed out (check /tmp logs)
- Reduce task complexity for faster response

### Missing results
- Check if required agent is available: `/agents consult qoder test`
- Some agents have budget limits (qoder: 1-2/week)
- Skip unavailable agents and use others' results

## Architecture

```
SKILL.md
├── Mode 1: Auto (keyword matching → agent selection)
├── Mode 2: Review (codex + gemini parallel)
├── Mode 3: Challenge (adversarial debate)
└── Mode 4: Consult (manual agent selection)
    └── Parallel Bash Execution
        ├── Silent failure handling
        ├── Result aggregation
        └── Synthesis & reporting
```

## Future Enhancements

- [ ] Learning from past runs (which agents usually agree/disagree)
- [ ] Dynamic budget allocation (adjust agents based on available quota)
- [ ] Streaming output (show results as agents finish)
- [ ] Custom agent combinations via YAML config
- [ ] Integration with CI/CD for automatic code review

## Version History

**v1.1.1** (2026-03-27)
- Renamed GitHub repo to orchestrator-agents-skill
- Updated all URLs and references to new repo name
- All 6 agent CLIs upgraded to latest versions

**v1.1.0** (2026-03-27)
- Independent repo release with full triple-publish (GitHub + npm + PyPI)
- Synced local skill directory with all project files
- Documentation and version consistency improvements

**v1.0.0** (2026-03-26)
- Initial release
- 4 modes: Auto, Review, Challenge, Consult
- Support for 6 agents (codex, cline, gemini, kilo, opencode, qoder)
- Parallel execution with silent failure handling
- Multi-language keyword matching

## License

MIT — See LICENSE file

## Contributing

This skill is maintained as part of YD 2026 Project. To contribute:

1. Fork from `~/.claude/skills/agents/`
2. Test thoroughly (all 4 modes)
3. Submit changes via standard PR workflow
4. Deploy via `/triple-publish` when approved

---

**Ready to use**: Just describe your task and Claude will automatically coordinate the right agents!
