---
name: agents
version: 1.1.1
description: |
  Multi-agent orchestrator — intelligently analyze tasks, auto-select and coordinate 1-3 AI agents in parallel, synthesize results.
  Use when: "multi-agent", "agents", "cross-review", "讓多個agent", "交叉審查", "協調", "第二意見",
  "幫我調動", "並行", "parallel", "multiple perspectives", "challenge this", "review with multiple",
  or when a task clearly benefits from multiple expert viewpoints.
  Modes: auto (smart routing), review (code cross-review), challenge (adversarial), consult (specify agents).
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
---

# Multi-Agent Orchestrator v1.1

Intelligent coordinator for parallel AI agents. One prompt → analysis → agent selection → parallel execution → synthesized report.

## Available Agents

| Agent | CLI | Specialty |
|-------|-----|-----------|
| **codex** | `codex exec` | Code review, refactoring, security, test generation |
| **gemini** | `gemini -p --approval-mode plan` | Architecture analysis, broad knowledge, design patterns |
| **kilo** | `kilo run` | Logic analysis, system design, complex reasoning |
| **opencode** | `opencode run` | Rapid prototyping, code completion, automation |
| **cline** | `cline --plan --json` | Structured analysis, implementation details |
| **qoder** | `qodercli -p` | Information retrieval, documentation, knowledge base (budget-limited) |

---

## 4 Modes

### Mode 1: Auto (Default)
**When**: User describes a task without specifying agents
**Action**: Analyze → classify → select best 1-3 agents → execute parallel → synthesize

**Task Classification Matrix**

| Task Type | Keywords | Primary | Secondary |
|-----------|----------|---------|-----------|
| **Code Review** | review, audit, lint, refactor, quality, improve | codex | gemini |
| **Feature Implementation** | implement, develop, create, build, fix, feature | cline | opencode |
| **Architecture Design** | design, architect, system, planning, decision | kilo | gemini |
| **Analysis/Explanation** | analyze, explain, summarize, compare, evaluate | gemini | kilo |
| **Rapid Prototyping** | prototype, sketch, quick, rapid, script, automate | opencode | cline |
| **Knowledge Lookup** | query, search, docs, reference, example, manual | qoder | gemini |
| **Security Audit** | auth, security, crypto, vulnerability, exploit | codex + cline | — |
| **Debugging** | bug, stuck, error, crash, investigation, root cause | codex | kilo |

**Selection Algorithm**:
1. **Extract keywords** from user prompt (case-insensitive, multi-language)
2. **Match against matrix** — each match scores 1-3 points per agent
3. **Select primary** (highest score) + optionally secondary (if score >1)
4. **Cap at 2-3 agents** for parallel execution
5. **Skip qoder** if budget seems tight (only 1 run per week)

**Example**:
```
User: "審查這個認證流程的代碼，看看有沒有漏洞"
→ Keywords: 審查(review) + 認證(auth) + 代碼(code) + 漏洞(security)
→ Matched: codex(3) + cline(2) + gemini(1)
→ Select: codex + cline (parallel)
```

---

### Mode 2: Review (Cross-Review)
**When**: User says `/agents review` or "幫我做 code review"
**Action**: codex + gemini in parallel, output: PASS | CONCERNS | BLOCK

```
## 🔍 Cross-Review Report

**Task**: [description]
**Agents**: codex + gemini
**Duration**: ~XXs

### Codex (Code Quality) [PASS|CONCERNS|BLOCK]
- 代碼品質: ...
- 性能: ...
- 安全: ...
- 建議: ...

### Gemini (Architecture) [PASS|CONCERNS|BLOCK]
- 架構合理性: ...
- 設計模式: ...
- 擴展性: ...
- 建議: ...

---

### 綜合判斷
**共識**: ...
**分歧**: ...
**結論**: [PASS | APPROVE_WITH_CHANGES | BLOCK]
**Claude 建議**: ...
```

---

### Mode 3: Challenge (Adversarial)
**When**: User says `/agents challenge` or "挑戰一下這個方案"
**Action**: Select 2 best agents for opposite views, run adversarial debate

```bash
# Agent A argues for the solution
# Agent B argues against it
# Report the blind spots and trade-offs
```

**Output**:
```
## ⚔️ Challenge Report

**Thesis**: [Your original solution/idea]

### Agent A: Pro Argument
[Agent 1 argues FOR]

### Agent B: Counter Argument
[Agent 2 argues AGAINST]

### Hidden Risks Exposed
- ...
- ...

### Trade-offs to Consider
- ...
```

---

### Mode 4: Consult (Manual Selection)
**When**: User says `/agents consult kilo,gemini` or "讓 kilo 和 gemini 看看"
**Action**: Run only specified agents, skip auto-selection

```bash
# Usage: /agents consult <agent1>,<agent2>,...
# Example: /agents consult kilo,gemini
# Available: codex, cline, gemini, kilo, opencode, qoder
```

---

## Auto-Trigger Scenarios

These are built into the orchestrator's judgment during normal conversation:

| Scenario | Auto-Action | Why |
|----------|------------|-----|
| User reviews code or PR | → Mode 2: Review | Multiple perspectives catch more issues |
| Commit/push with >100 line changes | → codex + gemini | High-risk changes need cross-check |
| Architecture decision / major refactor | → gemini + kilo | Design decisions need both vision & logic |
| User stuck on bug for 3+ turns | → codex or kilo | Fresh perspective needed |
| auth/crypto/secrets related changes | → codex + cline | Security → mandatory second opinion |
| User explicitly asks for "第二意見" | → best 1-2 agents for topic | Direct request |
| Analysis of foreign codebase | → gemini + kilo | Need both breadth and structure |

---

## Execution Framework

### Parallel Bash Execution

```bash
TASK="$1"
AGENTS=("codex" "gemini")  # Array of agent names
PIDS=()
OUTPUTS=()

# Spawn all agents in parallel
for agent in "${AGENTS[@]}"; do
  case $agent in
    codex)
      codex exec "$TASK" > /tmp/agents-$agent.txt 2>&1 &
      ;;
    gemini)
      gemini -p --approval-mode plan "$TASK" > /tmp/agents-$agent.txt 2>&1 &
      ;;
    kilo)
      kilo run "$TASK" > /tmp/agents-$agent.txt 2>&1 &
      ;;
    opencode)
      opencode run "$TASK" > /tmp/agents-$agent.txt 2>&1 &
      ;;
    cline)
      cline --plan --json "$TASK" > /tmp/agents-$agent.txt 2>&1 &
      ;;
    qoder)
      qodercli -p "$TASK" > /tmp/agents-$agent.txt 2>&1 &
      ;;
  esac
  PIDS+=($!)
done

# Wait for all to finish
for pid in "${PIDS[@]}"; do
  wait $pid || true  # Silent failure
done

# Gather results (skip missing)
for agent in "${AGENTS[@]}"; do
  [ -s /tmp/agents-$agent.txt ] && OUTPUTS+=("$agent")
done
```

### Silent Failure Protocol
- If an agent times out or crashes, skip it
- Continue with other agents' results
- If all fail, report which agents were attempted and why (rate limit, network, etc.)
- Never block the entire orchestration due to one agent's failure

---

## Output Report Template

```markdown
## 🤖 Multi-Agent Report

**Task**: [user's request summary]
**Mode**: Auto | Review | Challenge | Consult
**Agents Called**: [codex, gemini, ...]
**Duration**: ~XXs
**Status**: ✅ Success | ⚠️ Partial (N agents failed)

---

### Agent 1: [Name] [PASS|CONCERNS|BLOCK]

[Key findings, recommendations, caveats]

---

### Agent 2: [Name] [PASS|CONCERNS|BLOCK]

[Key findings, recommendations, caveats]

---

### Synthesis

**Consensus**:
Areas where both agents agree.

**Disagreement**:
Where agents diverge and why (different priorities, risk tolerance, etc.).

**Blind Spots Exposed**:
What neither agent considered.

**Trade-offs**:
Key decisions and their costs.

**Final Judgment**:
✅ PROCEED | ⚠️ PROCEED_WITH_CHANGES | 🛑 BLOCK

**Claude's Take**:
[Synthesize into a decisive recommendation, not just a summary of both views]
```

---

## Integration with Existing Workflows

### With /ship (Pre-Ship Code Review)
Suggest auto-trigger: When user runs `/ship`, automatically run `/agents review` on the diff before creating PR.

### With /investigate (Debugging)
When stuck >3 turns, suggest: "Context: Multiple turns without progress. Let me call `/agents` for fresh perspectives."

### With Commit Hooks (Future)
```bash
# In .claude/settings.json hook
if [ $(git diff --stat | wc -l) -gt 100 ]; then
  echo "Large commit detected. Running /agents review..."
fi
```

---

## Reliability & Cost

### Constraints Respected
- **Rate limits**: Space out agent calls, skip qoder on frequent runs
- **Context**: Keep prompts under 2000 chars per agent
- **Timeout**: Each agent max 60s, total parallel max 90s
- **Budget**: qoder has strict weekly limits (1-2 runs max)

### Degradation
- If 1 of 2 agents fails → use single agent's output + note the limitation
- If all agents fail → report failure reason (network, rate limit, timeout) and suggest retry later
- Never silence failures — always explain to user

---

## Example Invocations

### Example 1: Auto Mode
```
User: "我想審查一下這個 PR，有沒有什麼設計上的問題"
Claude: 自動分析 → Keywords: review + design → Select: codex + gemini
Output: Cross-review report with code quality + architecture checks
```

### Example 2: Challenge Mode
```
User: "/agents challenge 我們用 microservices 架構對嗎"
Claude: 自動選 gemini(pro) + kilo(con)
Output: Pro/con arguments exposing trade-offs
```

### Example 3: Consult Mode
```
User: "/agents consult kilo 這個系統設計會不會有瓶頸"
Claude: 單獨運行 kilo，專注系統分析
Output: Detailed bottleneck analysis
```

---

## Trigger Keywords (All Languages)

| 中文 | English | 日本語 |
|------|---------|--------|
| multi-agent, agents, 協調, 交叉審查, 第二意見, 並行, 多個視角, review, challenge, consult | multi-agent, agents, orchestrate, cross-review, second opinion, parallel, multiple perspectives, review, challenge, consult | マルチエージェント, 協調, クロスレビュー, 複数の視点 |

---

*Created: 2026-03-26*
*Ready for /triple-publish deployment to GitHub + npm + PyPI*
