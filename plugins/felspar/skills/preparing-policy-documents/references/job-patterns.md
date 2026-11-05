# Job patterns

**These patterns are for spotting gaps and asking good questions. They are never policy.** The company's document decides. A typical number here is only a starting point for a recommended default the owner can accept, change or reject; until they say so, it isn't a rule. Use the company's own words for cases, categories and outcomes, not the ones below.

Each pattern lists what a case is, the questions the decision usually needs (numbers and dates marked), typical outcomes, thresholds and escalations, and the **gaps documents usually leave**, which is where to start asking.

## Contents

- Support: ticket routing, refunds
- Legal: NDA and contract review
- People: candidate screening, leave requests
- Finance: invoice approval, expense approval
- IT and security: access requests
- Engineering: code change review
- Content and trust: publishing review, moderation
- Procurement: vendor onboarding
- Sales: lead qualification

## Support: ticket routing

- **Case:** one customer request; the help desk ticket number.
- **Questions:** what the request is about (categories); the customer's plan or tier; whether it reports an outage or data loss; how long it's been open (number, hours).
- **Outcomes:** route to a team (billing, technical, account), urgent handling, self-service answer, review.
- **Thresholds:** response-time targets in hours by tier.
- **Escalations:** human review for unclear tickets; incident review for outages.
- **Gaps documents usually leave:** a category for requests that fit none of the teams; how to tell billing from account issues (near misses); what counts as "urgent"; what to do when a ticket raises two topics.

## Support: refunds

- **Case:** one refund request; the order number.
- **Questions:** delivery date and request date (dates); order value (number, currency); reason (categories: damaged, wrong item, changed mind, late); whether proof was sent.
- **Outcomes:** refund, partial refund or store credit, decline, review.
- **Thresholds:** a window in days from delivery; a value above which someone approves.
- **Escalations:** a support lead or finance review for exceptions and large refunds.
- **Gaps documents usually leave:** what happens after the window (decline, or review?); who reviews exceptions; whether the last day counts; which date starts the clock (order, shipping, delivery); what proof is enough for "damaged"; what the reply must and must never promise.

## Legal: NDA and contract review

- **Case:** one contract; the contract ID, under a matter.
- **Questions:** governing law (categories); mutual or one-way; term length (number); whether non-standard clauses appear (non-solicit, IP assignment).
- **Outcomes:** sign as is, sign with standard edits, send to legal review, decline.
- **Thresholds:** maximum term; maximum liability cap.
- **Escalations:** legal review; compliance review for sanctions or regulated counterparties.
- **Gaps documents usually leave:** which governing laws are acceptable and what happens with others ("none of these"); what makes a clause "non-standard"; whether the counterparty or the contract decides; who signs.

## People: candidate screening

- **Case:** one candidate for one opening; their email plus the requisition number.
- **Questions:** meets each required qualification (categories with near misses); years of experience (number); work authorization; location.
- **Outcomes:** advance, reject, hold for the hiring manager.
- **Thresholds:** minimum years of experience; required certifications.
- **Escalations:** hiring manager review; people team review for anything sensitive.
- **Gaps documents usually leave:** "required" versus "nice to have"; what counts as equivalent experience; how to handle missing information on the application; what a rejection message must never say.

## People: leave requests

- **Case:** one leave request; the request ID or the employee and start date.
- **Questions:** type of leave (categories); notice given (days between request and start); length (days); remaining balance (number, days).
- **Outcomes:** approve, decline, refer to manager, refer to the people team.
- **Thresholds:** notice in days; maximum consecutive days without extra approval.
- **Escalations:** manager review; people team review for protected leave.
- **Gaps documents usually leave:** working days or calendar days; what happens with short notice in an emergency; which leave types are always referred; what "extended leave" means in days.

## Finance: invoice approval

- **Case:** one supplier invoice; the invoice number plus the supplier.
- **Questions:** amount (number, currency); matching purchase order (yes, partial, none); invoice date and due date (dates); supplier on the approved list.
- **Outcomes:** approve for payment, hold for a match, reject, finance review.
- **Thresholds:** approval limits by amount; tolerance for a mismatch with the order (percent or amount).
- **Escalations:** finance review; a budget owner's approval above a limit.
- **Gaps documents usually leave:** what happens with another currency; the tolerance for small mismatches; invoices with no purchase order; whether the limit includes tax.

## Finance: expense approval

- **Case:** one expense claim; the claim ID.
- **Questions:** category (travel, meals, equipment, other); amount (number, currency); receipt attached; days since the expense (date).
- **Outcomes:** approve, decline, ask for a receipt, manager review.
- **Thresholds:** per-category limits; days allowed to submit; amount below which no receipt is needed.
- **Escalations:** manager review; finance review over a limit.
- **Gaps documents usually leave:** a category for expenses that fit none; whether alcohol, tips or upgrades count; submission deadline in days; what happens just past a limit.

## IT and security: access requests

- **Case:** one access request; the request ID, about a person and a system.
- **Questions:** which system (categories, including sensitive ones); the requester's role; the level of access (read, write, admin); how long it's needed (days).
- **Outcomes:** grant, grant for a limited time, decline, security review.
- **Thresholds:** maximum time for temporary access; which levels need extra approval.
- **Escalations:** security review; the system owner's approval.
- **Gaps documents usually leave:** what makes a system "sensitive" (rarely defined); what "temporary" means in days; who owns each system; what happens when the role doesn't match the request.

## Engineering: code change review

- **Case:** one proposed code change; the repository plus its pull request or change number.
- **Questions:** which areas the change touches (categories, including sensitive ones such as authentication, data migrations, payments or infrastructure); whether automated checks passed on the latest revision; what the reviewing agent or person found (categories: none, minor, blocking); whether a required specialist review covered the changed area.
- **Outcomes:** merge without human review, one approval, two approvals, block until fixed, specialist review.
- **Thresholds:** approvals required per risk level; how old a check or review may be relative to the latest revision.
- **Escalations:** specialist review (often security) for sensitive areas; a second approver for high-risk changes; human review when the risk can't be settled.
- **Gaps documents usually leave:** what makes an area "sensitive" (often only a folder list, though a filename or size rarely settles risk); who owns an area with no named owner; how a mixed change is rated (mostly low risk with one sensitive line); whether generated files, reverts or dependency updates follow the same rules; whether the author's own approval counts.

## Content and trust: publishing review

- **Case:** one piece of content; its draft ID, or the text itself when there's no ID.
- **Questions:** whether it makes claims about results, competitors or safety (categories); whether it cites a source; audience or region.
- **Outcomes:** publish, publish with edits, send to review, don't publish.
- **Thresholds:** age of a statistic or source (days).
- **Escalations:** legal review for claims; brand review.
- **Gaps documents usually leave:** what counts as a "claim" versus an opinion; which sources are acceptable and how old; required disclaimers, word for word; what must never be said.

## Content and trust: moderation

- **Case:** one reported post or message; its content ID.
- **Questions:** which rule it may break (categories with near misses); whether it targets a person; repeat reports for the same account (number).
- **Outcomes:** leave up, remove, restrict, trust and safety review.
- **Thresholds:** number of prior violations before a stronger action; time window for counting them (days).
- **Escalations:** trust and safety review; legal review for threats or legal requests.
- **Gaps documents usually leave:** the line between satire or quotation and the real thing; a "none of these" category for reports that break no rule; how many prior violations, over how many days.

## Procurement: vendor onboarding

- **Case:** one vendor; its web domain or registration number.
- **Questions:** what the vendor will access (categories: none, company data, customer data); annual spend (number, currency); security documents provided; country.
- **Outcomes:** approve, approve with conditions, security review, decline.
- **Thresholds:** spend above which more checks apply; age of a security report (days).
- **Escalations:** security review; legal review for contracts; finance review for spend.
- **Gaps documents usually leave:** what counts as "customer data"; how old a security report may be; what happens when documents are missing; renewals versus new vendors.

## Sales: lead qualification

- **Case:** one company; its web domain.
- **Questions:** company size (number, employees); industry (categories); buying intent (categories); region.
- **Outcomes:** qualify, nurture, disqualify, review.
- **Thresholds:** size bands; how recent an intent signal must be (days).
- **Escalations:** sales review for unclear cases.
- **Gaps documents usually leave:** what counts as real buying intent versus research; a "none of these" industry; how recent a signal must be; whether the size edge itself qualifies.
