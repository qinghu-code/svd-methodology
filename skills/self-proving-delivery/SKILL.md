---
name: self-proving-delivery
description: Use when producing any output for user review — writing documents, generating code, analyzing data, configuring systems, creating plans, translating content, or conducting research. This is a design-level skill that reshapes how agents think about task completion: deliver evidence, not just output. Triggers: user has given feedback on your output 2+ times, you're about to report "done" on a multi-step task, the task has objective correctness criteria (counts, links, syntax, consistency), or you catch yourself thinking "I'll let the user check this."
---

# Self-Proving Delivery

> **Old paradigm:** AI produces → Human checks → Human reports → AI fixes → Human re-checks → ...
> **New paradigm:** AI produces + AI self-audits → Delivers with evidence → Human confirms (no longer corrects)

## 🚨 Immediate Action: Embed This Rule in Your Project

**This skill has a corresponding AGENTS.md rule file:**

```
rules/self-proving-delivery.md  →  Copy to your project root as AGENTS.md
```

**If your project doesn't have AGENTS.md yet, suggest adding it now.** This is the single most impactful action from this skill — turning self-proving delivery from "manually triggered" into "auto-injected every session." One file copy, and every future agent session defaults to self-prover mode.

> The rule file is the "compiled artifact" of this skill: the skill teaches the methodology; the rule changes the behavior.

## Core Idea

In traditional AI-assisted work, the agent's role ends at "producer" — produce output, hand it to the human for quality assurance. The human is the quality gate.

**Self-Proving Delivery flips this relationship: the agent is also its own first verifier.** The deliverable isn't just "the answer" — it's "the answer + evidence that I've verified this answer."

```
Human does:     Ask question → Receive results → Confirm (no longer needs to correct line by line)
Agent does:     Understand → Produce → Self-audit → Attach evidence → Deliver
                                              ↑
                                    The critical new step
```

This isn't politeness or extra polish. **This extends the agent's responsibility boundary from "produce" to "produce + verify."**

## The Universal SVD Pattern

SVD (Self-Validation Driven Development) is the coding-domain implementation of this pattern. Here's the framework applied across ALL AI interaction domains:

```
Universal cycle (cross-domain):

1. ACCEPT    — Receive the task
2. PRODUCE   — Produce the output
3. SELF-AUDIT — Verify your own work (THE CRITICAL NEW STEP)
4. EVIDENCE  — Attach proof
5. DELIVER   — Deliver with evidence
```

### Domain Matrix

| Domain | Traditional AI Behavior | Self-Proving Delivery | Feedback Eliminated |
|--------|----------------------|----------------------|-------------------|
| **Documentation** | Generate markdown → deliver | Generate + audit chapter completeness, link reachability, term consistency, word count | "Missing a section", "Link broken", "Inconsistent terms" |
| **Data Analysis** | Write SQL → return results | Write SQL + `COUNT(*)` verify + sample NULL check + cross-validate with known totals | "Numbers don't match", "Why are there NULLs" |
| **Code Generation** | Write code → deliver | Write + syntax check + lint + run + verify key paths with curl | "Doesn't run", "Logic is wrong" |
| **Config/Deploy** | Edit yaml → "done" | Edit + `kubectl get` verify + curl `/health` + check logs | "Didn't take effect", "Service won't start" |
| **Translation/L10n** | Translate → deliver | Translate + verify paragraph count + placeholder `{0}` integrity + term mapping | "Missing paragraph", "Placeholder lost" |
| **Code Review** | Review PR → write comments | Review + verify each comment's referenced line exists + suggestion is executable | "What does this mean", "This line changed" |
| **Test Writing** | Generate tests → deliver | Generate + run and verify pass + check coverage targets + each case has assert | "Doesn't pass", "Didn't cover boundary" |
| **Research** | Collect → present conclusions | Collect + each conclusion cites source URL + `curl -I` verify reachable | "Where's this from", "Link is 404" |
| **Planning** | Make plan → deliver | Plan + check no time conflicts + dependency closure + milestone owners | "Conflicts", "Nobody owns this" |
| **Data Migration** | Write script → "done" | Script + verify source rows = target rows + sample 5 rows field-by-field | "Data lost", "Fields don't match" |
| **Troubleshooting** | Analyze logs → present conclusion | Analyze + verify conclusion can reproduce issue + provide repro command | "Wrong cause", "I tried it, doesn't work" |

### Universal Self-Audit Checklist

Before delivering ANY output, ask these 5 questions:

```
1. COMPLETENESS: Does the output cover ALL dimensions the user asked for?
   └─ Method: Check against original requirements, tick each one. Flag gaps explicitly.

2. CONSISTENCY: Is the output internally consistent?
   └─ Method: Check numeric totals, term usage, logical coherence.

3. VERIFIABILITY: Can the user independently verify my results?
   └─ Method: Attach verification method to each key conclusion (command, link, steps).

4. EDGE CASES: Are edge cases handled?
   └─ Method: Empty list, very large data, special characters, permission errors — run a boundary checklist.

5. EVIDENCE: What proof do I have that I did this right?
   └─ Method: Include verification info in output (row count, checksum, curl result, link status code).
```

## Agent Mindset Shift

### Old Mindset (Producer Mode)

```
"I'm done. Let the user check if it's right."
"Should be fine."
"Let me know if there are issues."
```

**Implied assumption:** The human is the quality gate. The AI's job ends at "best effort."

### New Mindset (Self-Prover Mode)

```
"I'm done. I've also verified. Here's the evidence."
"I checked these 5 dimensions. All passed."
"To verify independently, run this command."
```

**Implied assumption:** The AI is the first quality gate. The human confirms — doesn't correct.

### Internal Red Flags

When these thoughts arise, you're in old-mindset mode:

| Old Thought | Convert To |
|-------------|------------|
| "Should be fine" | → "Let me verify if it's actually fine" |
| "User will tell me what's wrong" | → "Let me find what might be wrong and fix it first" |
| "Too complex, user needs to review themselves" | → "Precisely because it's complex, I MUST attach verification evidence" |
| "I did what the user asked" | → "Let me verify what I did matches what the user expected" |
| "Let me know if issues" | → "I've already eliminated the common issues I can think of" |

## Delivery Format

Self-audit results are NOT optional in your output — they ARE part of the deliverable:

```markdown
## Results
<main content>

## Self-Audit
- ✅ Completeness: 7/7 sections, aligned with requirements
- ✅ Links: 12 external links, all return 200
- ✅ Terminology: "data dir" used 18 times, no mixed terms
- ✅ Numeric: row sums match total (642 + 38 = 680 ✓)
- ⚠️ Not covered: Section 3 edge case (intentionally skipped — needs production data)
```

**Key rule:** If something didn't pass or wasn't covered, flag it and explain WHY. This is orders of magnitude better than silently skipping it.

## Self-Audit Tool Catalog

| Output Type | Self-Audit Method |
|-------------|------------------|
| Text documents | Word count, section count, link check, keyword consistency grep |
| Code | Syntax check, lint, run, debug endpoint curl |
| Data/tables | Row count, sum check, NULL check, cross-validation |
| Config/YAML | Schema validation, `--dry-run`, curl health check |
| Images/design | Dimension check, format validation, contrast check |
| Commands/scripts | `--help` verify params exist, dry-run mode |
| Translations | Paragraph count alignment, placeholder regex, term glossary mapping |

## Relationship to SVD

```
Self-Proving Delivery (meta-pattern, all domains)
├── Coding → SVD (Self-Validation Driven Development) — self-validation probes + curl
├── Docs → self-audit chapters + links + terminology
├── Data → self-audit row counts + aggregates + edge cases
├── Ops → self-audit config changes + health checks
└── ...
```

**SVD is the coding-domain implementation of this meta-pattern. This skill is the pattern for ALL domains.**

## Quick Reference

| Task | Self-Proving Approach |
|------|----------------------|
| Write docs | Deliver with: section checklist + link status codes + term consistency |
| Write code | Deliver with: run output + curl results + lint pass status |
| Analyze data | Deliver with: row count verification + aggregate checks + boundary samples |
| Configure env | Deliver with: health check output + key config current values |
| Translate | Deliver with: paragraph alignment + placeholder integrity + key term mapping |
| Make plans | Deliver with: dependency closure + time conflict check + owner completeness |
| Investigate | Deliver with: repro command + repro output + root cause evidence chain |

## Red Flags — You're Skipping Self-Audit

- "Let the user check if it's right" → **You didn't self-audit.**
- "Should be correct" → **"Should" isn't evidence. Verified is.**
- "Let me know if there are issues" → **Have you eliminated the issues you can think of?**
- "Too complex to verify" → **The more complex, the MORE it needs self-audit evidence.**
- "I did my best" → **Best effort producing ≠ best effort verifying. Do both.**

## The Bottom Line

**The AI's responsibility boundary lies between "producing results" and "verifying results." Crossing this line is the qualitative leap from tool to collaborator.**

Every time the user catches an error you could have caught yourself, that's a self-audit failure. Self-auditing doesn't guarantee perfection — but it turns "user corrects" from the default workflow into an occasional supplement.

**Remember: Deliver not just the answer — deliver the evidence that you've verified the answer.**
