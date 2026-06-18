# Claude Handoff — Topic04 S3~S4 Common Framework Package

**Created:** 20260616_0202_KST  
**Target Version:** v0.38 candidate  
**Purpose:** Claude Code review/merge handoff  

---

## Read First

Please read the following files in order:

```text
last_status.md
gpt_delta/v0.38_topic04_gpt_delta_5_0_common_framework_finalization.md
prompt/5_0_common_automation_framework.md
prompt/00_overview.md
readme_workflow.md
```

---

## Current GPT-side Package Update

This package includes a GPT-side update for Topic04 S3~S4 common framework finalization.

Added:

```text
prompt/5_0_common_automation_framework.md
gpt_delta/v0.38_topic04_gpt_delta_5_0_common_framework_finalization.md
CLAUDE_HANDOFF_20260616_0202_KST.md
```

Updated:

```text
prompt/00_overview.md
last_status.md
```

---

## Key Decisions to Review

```text
1. Chapter 5 should include 5.0 Common Automation Framework.
2. 5.1~5.6 details are deferred to the next step.
3. 5.x common philosophy is Environment First / Capability First / Proposal First / Open-source First / Battle-tested First / Simplicity First.
4. Adaptive Schema should prioritize open-source, stable, efficient, maintainable solutions.
5. Artifact output must use timestamp-based run folders.
6. Policy and execution are separated: Skill Layer owns policy, Workflow Layer consumes policy.
7. Skill Loading Strategy is required before individual skills expansion.
```

---

## Requested Claude Code Work

```text
1. Review the GPT delta.
2. Validate whether 5_0_common_automation_framework.md should be accepted as-is or adjusted.
3. Produce Claude Review Delta.
4. After user approval, produce Merge Delta and Decision Log.
5. Decide whether to generate master/L1_AI_Automation_Roadmap_v0.38.md or keep this as package/workflow-only update.
```
