# Beyond the Default Stack: Build an Agentic Workflow That Fits You
### 45-Minute Workshop Outline

---

## Opening: The Agentic Moment (5 min)

- Quick show of hands: who has used an AI coding agent in the last month?
- The problem with "just install Copilot and go": default stacks aren't designed for *your* context
- Session roadmap and promise: leave with a mental model and a starting blueprint

---

## Part 1: The Coding Stack Landscape (8 min)

A rapid-fire survey so attendees know what's out there — without endorsing one toolchain

- **Cloud-based chat agents:** Claude, ChatGPT — great for reasoning and drafting
- **IDE-integrated agents:** GitHub Copilot, Cursor — embedded in your flow
- **Agentic coding tools:** Claude Code, Cursor Agent mode — autonomous, multi-step
- **Salesforce-native:** Agentforce for Developers, Einstein for Developers — org-aware but constrained
- Key point: **no single tool does everything** — the best workflows are composable
- Brief Higher Ed angle: licensing realities, FERPA/data concerns, and institutional procurement all affect which tools you can actually use

---

## Part 2: Blast Radius — The Constraint Mindset (10 min)

The most important concept in the session

- Define **blast radius**: how much damage can an agent do if it goes wrong?
    - File system access, repo permissions, org access, data exposure
- Agentic tools are powerful *because* they act autonomously — that's also the risk
- **Scratch orgs as your containment strategy:**
    - Isolated, disposable, no production data
    - Let the agent experiment freely without touching real records or metadata
    - Build → validate → promote, never the reverse
- Constraint layers that reduce blast radius:
    - Source-control-first workflows (agent touches code, not org directly)
    - Permission sets scoped to the task
    - Explicit file/folder boundaries in your agent config
- Higher Ed angle: student data, FERPA, and why blast radius is *especially* critical in education orgs — a misconfigured agent with broad org access in a Student Information System integration is a compliance event, not just a bug

---

## Part 3: The Salesforce Walled Garden — and the Gate Is Open (7 min)

- The classic walled garden: SLDS, Lightning Web Components, Apex, Flow — opinionated, structured, well-documented
    - This is *actually good* for agentic coding: AI loves well-defined constraints
    - Rich public documentation means models are well-trained on Salesforce patterns
- **But the wall has a gate:** Salesforce is headless now
    - Experience Cloud decoupled, Salesforce APIs consumed by external apps, LWC OSS outside the platform
    - Higher Ed angle: student portals, LMS integrations, ERP connectors — your Salesforce work likely crosses the wall already
- Implication: your agentic workflow needs to handle *both* worlds — opinionated platform code and freeform integration code

---

## Part 4: Extending the Agent — MCPs, Skills, and Terminal Tools (10 min)

Where workflows get truly personalized

- **What is an MCP (Model Context Protocol)?**
    - A standard way to give agents access to external tools and data sources
    - Examples relevant to Salesforce devs: Salesforce MCP, GitHub MCP, filesystem MCP
- **Skills / custom instructions:**
    - Reusable context bundles — your org's naming conventions, architectural rules, team standards
    - Think of them as "standing orders" that persist across every session
    - Higher Ed angle: encode your institution's data governance rules, integration patterns with Banner/Workday/Canvas as a skill
- **Terminal tools that punch above their weight:**
    - `sf` CLI — agent can run deploys, run tests, query orgs
    - `git` — branch, diff, commit as part of the agentic loop
    - `jq`, `curl` — lightweight API inspection
    - Shell scripts as agent-callable tools
- The composability principle: **MCP + Skills + Terminal = an agent that knows your stack**

---

## Part 5: Putting It Together — Your Workflow Blueprint (5 min)

- The minimal viable agentic workflow for a Salesforce developer:
    1. Scratch org for isolation
    2. Source-control-first (SFDX project, Git)
    3. Skill/context file loaded at session start
    4. MCP tools scoped to what the task needs
    5. Human review gate before any promotion
- You don't need all of this on day one — **start with blast radius, add layers**
- Higher Ed specific callout: document your workflow for your team — student-facing systems demand auditability

---

## Closing & Q&A (5 min)

- Key takeaways:
    - Know your blast radius before you give an agent access
    - Salesforce's opinionated stack is an *asset* for agentic work
    - MCPs and skills are how you make the agent yours
    - Scratch orgs are your safety net — use them aggressively
- Resources to share (slide): MCP registry, Salesforce DX docs, Claude Code docs, Agentforce for Developers
- Open Q&A

---

## Timing Summary

| Section | Time |
|---|---|
| Opening | 5 min |
| Stack Landscape | 8 min |
| Blast Radius & Constraints | 10 min |
| Walled Garden / Headless | 7 min |
| MCPs, Skills, Terminal Tools | 10 min |
| Blueprint + Close + Q&A | 5 min |
| **Total** | **45 min** |

---

## Notes

- The **blast radius concept** is your anchor — it's memorable, concrete, and genuinely useful regardless of which tools attendees use. It's worth returning to it in every section as a thread.
- The **Higher Ed callouts** are woven throughout rather than siloed, which should feel natural rather than bolted on.
- If Q&A runs long you can trim the Blueprint section — the mental model will already be in their heads by that point.
