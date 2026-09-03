---
name: network-troubleshooting
description: Structured method for troubleshooting infrastructure, networks and systems — initial scoping, zero assumptions, IS/IS NOT (Kepner-Tregoe), the scientific method, bottom-up OSI, 5 whys, an incident file, time-boxing and post-incident review. Use when investigating incidents, failures, outages, slowness, intermittent or unexpected behavior of a network/service/server, or whenever something is reported as "broken", "down" or "acting weird".
---

# Structured troubleshooting

An investigation flow in 5 phases: always-on master rules → opening → investigation tools → time and escalation → closeout. The Part 1 rules apply in every phase, regardless of which Part 3 tool is in use.

---

# PART 1 — Master rules (always active)

The first seven rules (1.1–1.7) apply to every investigation, regardless of which Part 3 tool is in use — they are not an alternative to those tools, they are the discipline underneath all of them. The last three (1.8–1.10) are the state you carry from start to finish of the investigation.

## 1.1 Zero assumptions

Never treat a config, value or cause as fact without confirmation from the user or real command output from the environment. This holds at every step: investigation, verbal explanation, summary, ticket notes.

- **Hypothesis vs. fact:** in any text, separate confirmed evidence from hypothesis ("a possible cause is…", "not confirmed, but…"). Never write "the machine was powered off" when all you know is "the policy has not applied for 3 weeks" — the cause stays unknown until investigated.
- **Stating a hypothesis as fact in an explanation is as damaging as in the diagnosis** — it contaminates records and steers effort toward the wrong cause.
- Validate every premise before building a hypothesis or recommendation on it.

## 1.2 Gathering information: questions vs. commands

**Judgment/context questions** (finite, plausible answers): use a structured multiple-choice format, not prose. Include a "Don't know" option when the user may not have the information. When a tool caps it (e.g. 4 questions / 4 options per call), split into groups.

**Collecting command output:** ask the user to paste the output directly into the chat. Always give the command as a copyable code block, numbered if there is more than one, with the target machine stated explicitly. Never collect command output through a structured question — it breaks the flow of someone running several commands in a row and pasting everything at once.

**Never mix, in the same turn, a pending command and a structured question** — a pending question forces the user to cancel it before pasting free text. Priority: command > question. If both seem necessary, send only the command now and hold the judgment question for after the result.

## 1.3 The Questioner — continuous critical reasoning

Not questioning is the shortest path to the wrong diagnosis.

**About what the user reports:**
- "How do they know that?" — it may be an interpretation of output, a relay from a third party, or an inference from a secondary symptom. Ask for the evidence before using it as a premise.
- "Are they describing what they saw or what they concluded?" — "the machine didn't update" is an observation; "the machine was powered off" is a conclusion. Never treat the user's conclusion as fact.
- "Is the sample representative?" / "Is 'all' literally all?" — behavior on one machine does not imply all of them; ask how many were checked.
- "Does this match what was already collected?" — new information that contradicts earlier observations should be investigated, not dismissed.

**About the absence of evidence:**
- "Absence from the log = it didn't happen?" — logs can be truncated or not cover the event; confirm the scope.
- "Zero result = real absence, or wrong scope/permission/syntax?" — try an alternative before concluding.
- "Is the silence trustworthy here?" — silent failure is common; absence of an error does not confirm success.

**Before acting:**
- "Do I have enough information, or is this impatience?" — premature action destroys evidence.
- "If this action fails, what will I learn?" — every action should produce information, not just attempt a fix.
- "Did I ask for a command, or accept a verbal description?" — verbal descriptions are imprecise; prefer concrete output.

## 1.4 Non-destructive investigation first

Collect the full state (logs, config, processes, tasks, services) **before** any change — the troubleshooting action itself can erase the evidence of the real cause. Snapshot if possible; only then change anything.

## 1.5 Reproducibility: consistent vs. intermittent

- **Consistent** → investigate static config and logs.
- **Intermittent** → passive collection before acting; do not force reproduction without understanding the trigger. Treating intermittent as consistent yields a false diagnosis.

**Passive collection does not replace an active test.** "Wait for the capture/log to catch an event" is a valid next step only if there is a specific question it answers that the current data does not. Ask: "what will this capture tell me that I don't already know?" If it only re-confirms something already evidenced, it is not progress — prioritize the active test available now (baseline, a direct command).

## 1.6 Collecting real data

Ask for a real command whenever the state is uncertain. Ambiguous output (empty, error) does not confirm absence — it may be wrong permission/syntax/scope; try an alternative before concluding. On a remote host, prefer redirecting output to a file and reading it afterward — a direct pipe can fail silently.

## 1.7 Step-by-step progression

Collect state → form a hypothesis → propose a verification → act only after confirming → verify the result with a command. Never assume the action worked.

## 1.8 Troubleshooting objective — define it and keep it in sight

Decide up front: **resolve the symptom**, or (when that is not yet possible) **explain the cause**. Record it in the "Objective" section of the incident file and use it as the filter for every new lead.

- **Test for every new lead:** "if this hypothesis is confirmed 100%, does that explain or resolve the original symptom?" An answer of "not necessarily" or "don't know" → say so **before** investing more time, not after several rounds.
- **Classify the lead before going deeper than 1–2 rounds:** (a) clear mechanistic link → dig in; (b) plausible but untested link → first propose the test that confirms/refutes it; (c) likely tangent → ask the user whether to open it as a separate track.
- **Scope creep signal:** new, real findings surfacing without anyone asking "does this serve the objective?". A real finding ≠ a relevant finding.
- The objective changes only by an explicit user decision — never by silent drift.

## 1.9 Hypothesis discipline

Every hypothesis makes a verifiable prediction and predicts **where and how often** the symptom should appear. Test that prediction against data already collected, not just against the case in hand.

- **Mechanism ≠ correlation:** require an explicit causal chain (X causes Y, step by step), not "they seem related" or "it's the same system". Symptoms on the same component can have independent causes.
- **The hypothesis scope must match the symptom scope:**
  - *Under-generalization* — proposing an action for 1 instance when the data already shows several affected; the action does not cover the real scope.
  - *Over-generalization* — explaining it as "known behavior of the component" without comparing rate/frequency against another instance of the same component. A documented precedent is a starting point, not confirmation.
- **Apply the scope test BEFORE investing effort in a lead.** An anomaly whose maximum reach is 1 device cannot explain a symptom affecting several instances — log it as an observation (it may serve another problem) and move on, without spending a round of questions/commands on it.
- **Before presenting a conclusion:** did I list every affected instance? does the hypothesis explain why these and not others (IS/IS NOT)? do I have real comparative data or only a single precedent? does the proposed action cover the whole scope? If it fails the test, don't discard it outright — go find the missing comparative data.
- **A "strong" or documented hypothesis ≠ a confirmed root cause.** Fixing a config based on a plausible hypothesis is a low-risk action worth keeping, but it does not close the investigation — that continues until there is direct evidence or the user decides to stop. Beware the bias of stopping at the first satisfying explanation.

## 1.10 Incident file — continuous logging

Create it at the start of every active investigation (does not apply to short, linear tasks such as migration/provisioning) and update it **at every discovery**, not only at the end.

**Why:** long conversations get compacted by the context limit, and the resulting summary loses tactical detail (what was tried/discarded, the current hypothesis, commands with critical data). The file survives compaction and lets you resume without loss.

**Keep it lean — replace, don't accumulate.** The file is a snapshot of the current state of the investigation, not a diary of everything ever considered. "Update at every discovery" means **rewrite the affected section**, not append a new paragraph below the old one.

- **Never create chronological reasoning sections** ("Finding 2", "Turning point 3", "Current hypothesis (updated)", "Current hypothesis (revised 2x)"). There is **one** "Current hypothesis" section and it gets overwritten.
- When a hypothesis falls, it leaves "Current hypothesis" and becomes **one line** in "Tried and discarded" (hypothesis → why it fell). All the supporting text that came with it — predictions, justification paragraphs, data that only served it — is **deleted**.
- A fact that turned out wrong (e.g. "the format is 2000/2003", "the backup is usable") is **corrected in place**, not left with a "~~strikethrough~~" and an addendum. If the wrong path teaches something reusable, that becomes one line in "Tried and discarded" or a short note — not the full history of how the mistake was found.
- The "Timeline" logs **actions and observed results** (terse: "ran X → got Y"), never the evolution of the reasoning.
- Before saving an update, reread the whole file: if a section contradicts another, or two blocks discuss the same thing at different moments, consolidate now. A file that has grown past ~2 pages is probably accumulating dead ends — compact it.
- The detail of "how we got here" lives in the conversation and in `git`/history; the file carries only what is needed to resume and to write the post-incident review.

**Location:** the project's incident directory, e.g. `incidents/incident-YYYYMMDD-short-description.md`.

**Template:**

```markdown
# Incident: [short description] — YYYY-MM-DD

**Status:** Open | Resolved | Waiting
**Priority:** P1 / P2 / P3
**Affected:** [systems, users, services]

## Initial symptom
[What was reported / observed — raw, not interpreted]

## Objective
[Resolve what, or explain what — the criterion for evaluating new leads]

## IS / IS NOT
What:    IS: … | IS NOT: …
Where:   IS: … | IS NOT: …
When:    IS: … | IS NOT: …
Extent:  IS: … | IS NOT: …

## Timeline
- HH:MM — [action taken + observed result — never the evolution of the reasoning]

## Current hypothesis
[ONE live hypothesis at a time — overwrite when it changes, don't version it]

## Tried and discarded
- [hypothesis/action] → [why it fell / negative result] — one line; delete the supporting text that only served the fallen hypothesis

## Next steps
- [ ] …

## Resolution
**Root cause:**
**Workaround applied:**
**Permanent fix:**
**Preventive actions:**
```

For network/connectivity problems, include an "OSI coverage" section (see 3.2). At the end, the completed file is the direct basis for the post-incident review (5.2).

---

# PART 2 — Opening the investigation

## 2.1 Initial scoping

Before any hypothesis or command: ask 2–4 multiple-choice questions about the categories relevant to the case (not all of them), including "Don't know". Skip anything already answered in the initial report.

**Possible categories:** scope of action (what may/may not be touched); architecture/topology (dependencies, protocol, layers, site/network); environment specifics (exceptions, technical debt, drift between "identical" instances); symptom (raw vs. interpreted); temporal (new/recurring, active/past); recent change; access (remote, credentials); precedent/runbook; triage (impact × urgency).

**Urgency bypass:** if the report already signals P1 (service down, many affected), skip straight to scope + access and restore first; full scoping comes afterward.

## 2.2 Triage: Impact × Urgency (ITIL)

|                  | High urgency          | Low urgency            |
|------------------|-----------------------|------------------------|
| **High impact**  | P1 — restore now      | P2 — handle in hours   |
| **Low impact**   | P3 — handle same day  | P4 — schedule          |

**Impact:** users/systems affected, criticality. **Urgency:** how fast it worsens without intervention.

## 2.3 Incident vs. Problem (ITIL)

Distinct workflows; mixing them wastes time and worsens impact.

- **Incident:** restore fast; a workaround is a valid solution; **now is not the time for root cause.**
- **Problem:** root cause, after the service is restored (or proactively).
- Common trap: deep diagnosis during an active incident, prolonging the outage.

## 2.4 What changed recently?

The first question in a problem investigation — a recent change is the most common cause in a stable environment. Correlate the onset of the problem with the ticketing system, event logs, deploys, changelog.

## 2.5 Kepner-Tregoe — IS / IS NOT

| Dimension    | IS                              | IS NOT                                          |
|--------------|---------------------------------|------------------------------------------------|
| **What**     | What exactly fails?             | What could fail but doesn't?                    |
| **Where**    | On which systems/locations?     | Similar locations where it does not occur?      |
| **When**     | Since when, under what conditions? | When it does not occur — what is different?   |
| **Extent**   | How many affected?              | Who is not affected despite being similar?      |

The "IS NOT" columns eliminate hypotheses without testing — the IS/IS NOT difference often points at the root cause.

---

# PART 3 — Investigation tools

The tools below act as the hypothesize → predict → test step of the scientific method: observe → hypothesize → predict ("if X, command Y should return Z") → test → conclude; never jump from observation straight to the corrective action. They are not mutually exclusive alternatives — which one to use depends on the shape of the problem, and more than one can apply to the same case.

**Quick triage:**

| Symptom looks like... | Tool |
|---|---|
| Networking, VoIP, directory auth, system-to-system communication | Bottom-up OSI model (3.2) |
| N instances affected, unsure if it's universal or local-state-dependent | Divide and conquer (3.1) |
| Technical mechanism already clear, still need the process/system-level cause | Five whys (3.3) |
| A known-good instance or version exists to compare against | Known-good baseline (3.4) |
| Don't even know where to start | IS/IS NOT (2.5) first, then pick from above |

## 3.1 Divide and conquer

Test at the midpoint between "works" and "fails", halving the possibility space each round, until you isolate the minimal component. E.g. N machines affected → test 1 first to learn whether it is universal or dependent on local state, before scaling the test.

## 3.2 The OSI model — bottom-up (connectivity)

For networking, VoIP, directory authentication or system-to-system communication, work bottom-up — do not start at the application:

| Layer | What to check |
|-------|---------------|
| 1 — Physical | Cable, patch panel, link up/down, Wi-Fi/fiber signal, switch LED |
| 2 — Data link | MAC, VLAN, STP/loops, interface errors |
| 3 — Network | IP, mask, gateway, routing, firewall, ping |
| 4 — Transport | TCP/UDP port, stateful firewall, established connection |
| 5–6 — Session/Presentation | TLS/certificate, session negotiation |
| 7 — Application | Protocol (SIP, LDAP, HTTP, RDP), authentication, app log |

**Why bottom-up:** a lower-layer problem produces a symptom that looks like an application issue — diagnosing L7 when it is L3 wastes hours. E.g. VoIP (NAT/firewall before codec), AD (DNS/Kerberos before LDAP), containers (bridge/iptables before the app).

**Do not stop mid-climb because you already have a satisfying explanation** — a plausible cause at L4/L7 does not rule out L1/L2, it only means they have not been checked yet. On a managed switch, look at port error counters (CRC, collision, discard, runt/giant) before calling physical/data-link clear.

**Keep one compact line per layer in the incident file**, with a verdict and a finding, updated at each check. That makes the gap obvious at a glance: a layer with no line = not yet checked. E.g.:

```
L1/L2 switch — clean (0 errors). L1 AP radio — RSSI/SNR good.
L3/L4 firewall — NAT ok, ALG ok, UDP timeout fixed. L7 app — configs correct.
```

## 3.3 Five whys (RCA)

Ask "why" until the root cause; do not stop at the immediate symptom — but each "why" needs real evidence (log, config, command) before it becomes the basis for the next one, not a chained assumption. Without that, five whys is just five stacked guesses, violating 1.1.

- **It isn't always a single chain.** An answer can have more than one plausible cause — branch (investigate both arms) instead of picking one path off the top of your head and dropping the other without checking.
- **"Five" is a convention, not a target.** Stop when the cause becomes something actionable and within the control of whoever will fix it (a process, a config, a missing validation) — that can take 3 questions or 8. Counting on to 5 after you already have the root cause is noise; stopping at 2 because it "already makes sense" is the bias from 1.9 (stopping at the first satisfying explanation).
- **Root cause ≠ immediate cause.** The last answer tends to point at a process or system ("missing validation X"), not a one-off event ("the file was moved") — a one-off event almost always still has a why behind it (why did moving the file break something without anyone noticing?).

Example: service won't start → config not found → path points to the old directory → deploy did not update the reference → deploy does not validate paths after moving a file → **root cause:** the deploy process lacks an integrity check. Fixing only the symptom leaves the root cause intact — it returns on the next update. This chain, with the evidence for each step, is the direct basis for the incident file's "Resolution" section (1.10) and the PIR (5.2).

## 3.4 Compare against a known-good baseline

Compare config/state against a known-good instance or the last working state (version, config, services, env vars, policies). Caution: the same declared config ≠ the same real state — there can be drift or a prior manual intervention.

---

# PART 4 — Time and escalation

## 4.1 Time-boxing and the escalation trigger

Set a time budget per step. No measurable progress (new evidence, a hypothesis confirmed or ruled out — not "keep investigating") → escalate, do not push the same path.

**When to escalate:** the problem is outside the available domain, the impact demands a faster response than the investigation allows, or the cause is in another domain. Escalating is not failure — it is bringing in the right resource at the right time.

**Long investigation:** every "next step" question should include a pause option ("let me think") — the incident file preserves the state so you can resume later.

---

# PART 5 — Closeout and record

## 5.1 Workaround vs. root cause (ITIL)

A workaround resolves the symptom without eliminating the cause; a permanent fix eliminates the cause. Every workaround ships with a deadline and an associated fix plan.

## 5.2 Post-incident review (PIR)

After a significant incident, run a blameless review: what happened (timeline), impact, root cause (5 whys), what resolved it, preventive actions. Focus on the system and the process, not on people. Always for P1/P2; for P3 if the cause is new or recurring.

## 5.3 Factual summary for records (tickets, emails)

Only executed actions and observed results — never inference as fact. Separate "observed" from "hypothesis".

## 5.4 Documenting exceptions

A runbook that did not apply to the case: record the exception in the original document or the ticket, not just in the conversation.
