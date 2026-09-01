---
name: writing-style
description: Writing standard for technical documentation, replies and written communication — clarity, concision, objectivity and precision, cohesion, correctness, impersonality, completeness. Use when writing or reviewing any text (documentation, wiki page, README, procedure, summary, email, ticket, long reply).
---

# Writing standard

Applies to documentation, chat replies and any prose produced. Draws on official/administrative writing standards and plain-language guidance: the text must be understood on first reading, convey the most with the fewest words, and not depend on who wrote it. The rules are deliberately short; the final review is mandatory.

## Before writing

- Identify the reader: what they already know, what they will do with the text. Write for the reader, not for the writer.
- Identify the goal: inform, instruct, record, or support a decision. Format follows goal.
- State the main message in one sentence before drafting. It opens the text.

## 1. Clarity

- Direct order: subject, verb, object. Invert only for a real gain.
- The common word over the rare one. Technical terms only when needed, and defined on first use.
- No ornate phrasing, archaisms, in-house jargon, or foreign words that have a plain equivalent.
- Short sentences, one main idea each. A sentence with three commas or more becomes two.
- No ambiguity: every pronoun and every reference has one unmistakable antecedent.
- Restrained adjectives: an adjective only when it adds verifiable information.
- Prefer the positive form to the negative. Avoid double negatives.

## 2. Concision

- Conclusion first. The result or the answer opens the text; context comes after, only what is needed to act.
- Cut useless words, redundancy, and passages that add nothing. Concision is not terseness; it is removing excess.
- Cut warm-up openers and go straight to the information.
- Cut hedging that serves no purpose. When you know, state it; when you don't, say so.
- A roundabout phrase becomes a verb. A nominalization becomes a verb.
- A sentence that repeats the previous one goes. A word that can be removed without changing the meaning goes.
- Don't explain what the target reader already knows by definition.

## 3. Objectivity and precision

- Go straight to the point, with no preamble or throat-clearing.
- Fact, not opinion. A number, not an adjective of quantity or intensity.
- Concrete, not vague: name the file, the service, the host, the command, the version. An instruction with no concrete target is not an instruction.
- Precision: say exactly what you mean, leaving no room for a different reading.
- Separate the confirmed from the hypothesis, with an explicit label. Never present a guess as a fact.
- Assert only what evidence supports (a command, a log, a document). Don't generalize from a single case.
- No marketing tone and no superlatives.

## 4. Cohesion and coherence

- Logical progression: each paragraph follows from the previous one; the text has a beginning, a middle and an end.
- The connective that reflects the real relation between ideas (cause, contrast, conclusion, condition) — not a generic "and" for everything.
- Unambiguous reference: "this", "that", "the latter" only with a clear antecedent.
- No internal contradiction. Don't re-present information already given as if it were new.
- One idea per paragraph; the first sentence announces the paragraph's subject.

## 5. Correctness and consistency

- Correct grammar and spelling; standard usage. Proofread before delivering.
- One term per concept, from start to finish — no swapping in a synonym. Applies to component names, commands and statuses.
- A verifiable value (path, port, hostname, version, flag name) is checked at the source, not written from memory.
- Consistent formatting: capitalization of proper nouns, the way commands/files/paths are cited, punctuation in lists.
- Stable verb tense within a section: system behavior in the present, completed action in the past.
- Numbers and units in the same style throughout.

## 6. Impersonality and formality

- The text speaks on behalf of the role or the team, not the person. No first-person singular, no personal impressions, no "in my opinion".
- Active voice, with the agent explicit. Imperative for instructions, not a passive or roundabout construction.
- Present tense to describe the system; past tense for a completed action.
- Formal register without pomposity: neither casual nor over-ceremonious.
- Courtesy and neutrality in tone; no irony and no judgment of people.
- Standardization: documents of the same kind share the same structure, the same headings and the same terms.

## 7. Structure

- Inverted pyramid: the most important first — across the whole document and within each section.
- A heading that states the content, not a generic label.
- Useful order: a prerequisite before the step that depends on it; cause before effect.
- Use a list for steps or parallel items; running text for reasoning.
- Parallel items in parallel form (same part of speech, same sentence structure).

## 8. Completeness

- The text carries everything the reader needs to understand and act — and nothing beyond that.
- A procedure: prerequisites, steps in order, expected result, what to do if it fails.
- Don't make the reader look elsewhere for what is essential to this text.

## Review — before delivering

Re-read with some distance from the moment of writing. When possible, read aloud: a sentence that stumbles on reading needs a rewrite.

- [ ] Reader and goal clear; is the main message right at the start?
- [ ] Direct order, short sentences, no ornate phrasing or needless jargon?
- [ ] Can ~20% of the words be cut without losing information? If so, cut.
- [ ] Has every adjective of quantity or intensity become a number or a concrete fact?
- [ ] Fact and hypothesis separated and labeled?
- [ ] One term per concept throughout; abbreviations defined on first use?
- [ ] Do connectives reflect the real relation between ideas; are references unambiguous?
- [ ] No first person and no personal impressions; active voice; instructions in the imperative?
- [ ] Paths, ports, names and versions checked at the source?
- [ ] Does each paragraph hold one idea, and is the order logical?
- [ ] Does the text answer everything the reader needs in order to act?
