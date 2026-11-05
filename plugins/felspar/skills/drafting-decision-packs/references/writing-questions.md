# Writing questions Felspar answers well

Felspar answers each question about an evidence excerpt by choosing one of your answers, and holds the question for evidence when the text doesn't settle it. Your answers stay a closed list and never become yes/no. Wording decides whether a correct answer is read clearly enough to decide. `check_pack` and `check_readings` name each problem and its fix; these rules get most drafts clean the first time.

1. **One judgment per question.** Split a question that hides several.
2. **Describe acceptable text positively, never as an absence.** Not "makes no result claim" but "about something other than outcomes: a process, a team, a product, a status line, a label or logistics".
3. **Give every common kind of text its own answer,** including near misses and short fragments (a status line, a heading). Text with no answer of its own comes back unsure.
4. **Describe each answer the same way:** what it covers, what it doesn't, and examples. Use the definition's inclusions, exclusions and examples.
5. **Felspar already handles text that doesn't say.** It holds the question for evidence, so don't add an "unknown" or "not stated" answer. **But do add "none of these".** That's different from text that doesn't say: "none of these" means it clearly says something outside your list (a tool "for every team" when the answers are developers, consumers and finance teams). Without it, clear text comes back unsure and the case goes to review for nothing. Mark that answer `noneOfThese: true`, describe it positively per rule 2 ("names a different kind of user, such as every kind of team or students"), and add a rule for where it leads. `check_pack` blocks a multiple-choice question without one.
6. **Prefer several answers over yes/no for "must not contain" rules.** A positively worded choice is read far more reliably than yes/no.
7. **Several acceptable answers can fit one text.** That is not uncertainty about the rule; make sure the rules treat them alike where the document does.
8. **Keep answer order deliberate and the same across a pack.**
9. **Never put IDs in the wording.** Only the text is read.
10. **Quote the sentence that asks the question as its `meaning`,** not a heading or label.
11. **Put a source limit in `evidenceKinds`,** not in `whereToLook`, which only guides agents. If the document says a fact comes only from one source, give the question its own `evidence`, or a clear statement from the wrong source can still decide it. Felspar refuses an excerpt sent under a kind the question doesn't list. The agent declares the kind, so this stops mistakes, not a mislabeled excerpt. `check_pack` warns when it spots such a limit (`source_limit_not_enforced`); it can't catch every one.
12. **Test both sides before proposing:** text that should pass and text that should fail. If acceptable text comes back below the bar, fix the wording; don't ask to lower the bar.
13. **Say whose event it is, and whether it has happened.** "Lost their job" is read as a match for "my husband lost his job" unless an exclusion says otherwise. Name the person the answer is about, and put near misses in `exclusions`.
14. **Keep examples, and keep them natural.** Write what a real record would say, in the company's voice, and never add wording copied from a reading test or eval.
15. **Each excerpt is read alone.** Test the excerpts agents will really send, a bare note as well as a labeled field, and fix a weak reading in the definition.

## Example

Before: "Does this text present a prior-customer result as guaranteed?" with answers _prior result_, _case guarantee_, _no result claim_. Clean text came back below the bar.

After: "What does this paragraph say about outcomes for the reader's own company? Describe the paragraph as written; most paragraphs are about other topics." Answers:

- **case guarantee:** says the reader's company will get, or is promised, a specific result;
- **past example:** another named customer's past result, not promised to the reader;
- **general statement:** results vary or depend on the customer, with no promise;
- **other topic:** a process, team, product, decision, status line, label or logistics.
