---
title: "Root Cause Analysis"
date: 2025-04-22
draft: false
---

## What Is Root Cause Analysis (RCA)? {#what-is-root-cause-analysis--rca}

Root cause analysis (RCA) is a structured problem-solving process used to identify the underlying cause(s) of an incident, defect, or failure, rather than only addressing its symptoms. A **root cause** is a causal factor that, if removed or mitigated, prevents the same class of failure from recurring under similar conditions.

In practice, RCA usually produces:

-   a clear problem statement and timeline,
-   validated causal chain(s) from symptoms to causes,
-   concrete corrective and preventive actions (CAPA),
-   follow-up checks to ensure the fix is effective.


## When To Use RCA {#when-to-use-rca}

-   Repeated incidents, high-severity outages, safety issues, or costly defects.
-   Failures with unclear ownership across teams or components.
-   Situations where quick mitigations exist but recurrence risk remains high.


## Typical RCA Workflow {#typical-rca-workflow}

1.  Define the problem: impact, scope, success criteria (what “fixed” means).
2.  Collect evidence: logs, metrics, traces, configs, releases, and human timeline.
3.  Build a timeline: what changed, when symptoms started, how it propagated.
4.  Form hypotheses: enumerate plausible causes and disprove aggressively.
5.  Validate causes: reproduce, isolate variables, run experiments, compare baselines.
6.  Decide actions: mitigations (short-term) + fixes (long-term) + detection improvements.
7.  Verify and learn: confirm no regression; update runbooks, monitors, and processes.


## Common Techniques {#common-techniques}

-   5 Whys (iteratively ask “why?” to uncover deeper causes)
-   Ishikawa / fishbone diagram (categorize contributing factors)
-   Fault tree analysis (logic decomposition from failure event)
-   Change/rollback analysis (diff recent changes and validate correlation vs causation)
-   Post-incident review (blameless retrospectives with evidence-based conclusions)


## Related Notes {#related-notes}

-   [OpenRCA: Can Large Language Models Locate the Root Cause of Software Failures?]({{< relref "2025-04-22-090257-openrca_can_large_language_models_locate_the_root_cause_of_software_failures.md" >}})


## References {#references}

-   Xu, J., Zhang, Q., Zhong, Z., He, S., Zhang, C., Lin, Q., ... &amp; Zhang, Q. (2025). _OpenRCA: Can Large Language Models Locate the Root Cause of Software Failures?_ (ICLR). See also: [OpenRCA note]({{< relref "2025-04-22-090257-openrca_can_large_language_models_locate_the_root_cause_of_software_failures.md" >}}).
