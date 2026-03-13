---
name: writing-design-docs
type: pattern
description: Writes system design docs organized by features with Purpose/Behavior/Verify. This is reference material — consult when creating or updating design docs.
---

# Writing Design Docs

Design docs describe systems organized by features. Each feature uses Purpose/Behavior/Verify.

## Doc Locations

| Location | Contains | Examples |
|---|---|---|
| `docs/` | Feature design docs + repo-level architecture | Payment flows, enrollment steps, migration plans, config externalization |

> **Note:** After Turborepo conversion, feature docs will move to `apps/enroll/docs/`.

## Contents

* [Use Purpose/Behavior/Verify for Each Feature](#use-purposebehaviorverify-for-each-feature)
* [Write Behaviors as Plain-Language Scenarios](#write-behaviors-as-plain-language-scenarios)
* [Use Default + Exceptions for Country-Specific Behaviors](#use-default--exceptions-for-country-specific-behaviors)
* [Distinguish Features from Implementation Details](#distinguish-features-from-implementation-details)
* [Number Behaviors for Multiple Scenarios](#number-behaviors-for-multiple-scenarios)
* [Reference Behaviors in Verify Steps](#reference-behaviors-in-verify-steps)
* [End Document After Last Feature](#end-document-after-last-feature)
* [Avoid Implementation Code in Design Docs](#avoid-implementation-code-in-design-docs)
* [Doc-First Development Workflow](#doc-first-development-workflow)

## Use Purpose/Behavior/Verify for Each Feature

Every feature section follows the same three-part structure:

| Section | Type | Example |
|---|---|---|
| Purpose | Intent (WHO + WHY) | "Let customers pay with their preferred method so checkout completes" |
| Context | Rationale (optional) | Risks avoided, standards followed, alternatives rejected |
| Behavior | Outcomes (WHAT) | "Customer selects credit card → card form shown, payment processed" |
| Verify | Test steps (HOW) | Page, endpoint, or DB check with what to confirm |

**Format:**

- **Overview**: 1-2 sentences covering what the system does.
- **Features ToC**: Links to each feature section.
- **Features**: Each feature has:
  - **Purpose**: One sentence — what it does and why (compressed user story)
  - **Context** (optional): Risks avoided, standards followed, alternatives considered
  - **Behavior**: Outcome statements — what happens in each scenario
  - **Verify**: Actionable steps to confirm the feature works

**Why this works:**

- Purpose captures intent without lengthy problem descriptions
- Behavior is human-readable AND machine-verifiable
- Verify gives explicit test instructions for both manual and automated testing
- No implementation code — doc stays accurate as implementation evolves

## Write Behaviors as Plain-Language Scenarios

Behaviors follow the pattern: **trigger → outcome**. Both halves are required.

**Good behaviors** — reads like a conversation:

- Customer submits an order with an expired card → payment fails, customer sees "Payment declined" with retry option
- Customer changes shipping address after payment → new tax calculation runs, customer confirms updated total
- Customer selects a country not in the supported list → redirected to market selection page

**Bad behaviors** — too technical:

- POST /orders with duplicate externalId returns 409
- Order entity transitions from PENDING to FAILED state
- calculateTax() is invoked with updated shippingAddress payload

**Bad behaviors** — missing outcome:

- Customer enters their shipping address on the checkout page
- Admin adds a new country config to the system

**Rewritten with outcomes:**

- Customer enters their shipping address → tax recalculated for new address, updated total shown
- Admin adds a new country config → country appears in market selection, enrollment flow uses new config

**The test:** Read the behavior aloud. If it sounds like a product requirement, it's good. If it sounds like a code comment, rewrite it.

## Use Default + Exceptions for Country-Specific Behaviors

When behavior varies across markets or configurations, describe the default then call out exceptions:

**Good** — default + exceptions:

```markdown
**Behavior**:
1. Customer selects a payment method → available methods shown based on country config
2. Country supports credit card (default) → standard card form shown
3. Country config includes alternative methods (e.g., JP konbini, TH bank wire) →
   additional options shown alongside card
```

**Bad** — listing every country:

```markdown
**Behavior**:
1. Customer in US selects payment → credit card form shown
2. Customer in JP selects payment → credit card and konbini forms shown
3. Customer in TH selects payment → credit card and bank wire forms shown
4. Customer in KR selects payment → credit card form shown
... (30 more)
```

The doc should reflect how the code works — config-driven, not hardcoded per country. This keeps behaviors to 3-5 per feature instead of 30+.

When a country has truly unique logic (not just different config values), call it out as a specific exception behavior.

## Distinguish Features from Implementation Details

A feature describes an outcome users or stakeholders care about.

- **Feature**: "Customers see prices in their local currency" (Currency Display)
- **NOT a feature**: "Country config maps market codes to currency symbols" (implementation detail)

Ask: "Would a stakeholder care about this as a distinct capability?" If not, it belongs in the Overview.

## Number Behaviors for Multiple Scenarios

When a feature has multiple scenarios, number them so verify steps can reference them:

```markdown
**Behavior**:
1. Customer pays with a valid card → order confirmed, confirmation email sent
2. Customer pays with an expired card → payment declined, customer prompted to retry
3. Customer's bank requires 3DS → redirect to bank verification, order held until confirmed
4. 3DS verification times out → order cancelled, customer notified
```

Each behavior has two halves separated by `→`: what triggers it and what the system does.

## Reference Behaviors in Verify Steps

Verify format options:

- **Page**: `/route/path` — what to do, what to confirm
- **POST/GET/etc**: `/api/endpoint` — request shape, response shape
- **DB**: table/query — what to check

Verify guidelines:

- Reference which behaviors each step covers: "Covers B1, B3"
- One verify step can cover multiple behaviors
- Include routes/endpoints
- Describe value types, not exact payloads (`{ channelId, ... }` not full JSON)
- No "Last Verified" tracking — trust CI/tests

## End Document After Last Feature

| Include | Exclude |
|---|---|
| System overview | TypeScript code examples |
| Feature ToC | Database schemas |
| Purpose/Behavior/Verify | Function signatures |
| Architecture diagrams | Implementation details |
| SQL in Verify (brief) | Trailing reference/summary sections |
| | File lists or key file tables |
| | Property summary tables |

Document ends with the last feature. No trailing sections after the final `---` divider.

## Avoid Implementation Code in Design Docs

Design docs contain NO implementation code. They describe outcomes, not how the system achieves them.

**Visuals (optional):** Architecture diagrams for complex systems using Mermaid without quoted descriptions:

```
STATE_A --> STATE_B: Submit
```

not

```
"User submits" --> "System processes"
```

## Doc-First Development Workflow

Design docs are both the plan AND the acceptance criteria:

1. Write the design doc using this skill's format (Purpose/Behavior/Verify)
2. Review the doc — behaviors define what gets built, verify defines done
3. Spawn a team to implement against the doc as their spec
4. Agents use Verify steps as acceptance criteria to check their work

## Example

```markdown
# Payment Processing

## Overview

Handles customer payments at checkout, coordinating between payment gateways
and order confirmation across all supported markets.

## Features

- [Card Payments](#card-payments)
- [Duplicate Prevention](#duplicate-prevention)

---

### Card Payments

**Purpose**: Allow customers to pay with credit/debit cards so they can
complete checkout with their preferred payment method.

**Behavior**:
1. Customer submits a valid card → payment authorized, order confirmed,
   confirmation email sent
2. Customer submits an expired or declined card → payment fails, customer
   sees decline reason and can retry
3. Country config includes alternative methods (e.g., JP konbini) →
   additional options shown alongside card

**Verify**:
1. **Covers B1**: Complete a checkout with a test card
   - **Page**: `/register` → payment step
   - Enter test card, submit
   - Confirm: Order confirmation page shown, email received

2. **Covers B2**: Attempt checkout with declined card
   - **Page**: `/register` → payment step
   - Enter declined test card, submit
   - Confirm: Error message shown with retry option, no order created

3. **Covers B3**: Country with alternative payment
   - **Page**: `/jp/register` → payment step
   - Confirm: Konbini option visible alongside credit card

---

### Duplicate Prevention

**Purpose**: Prevent customers from being charged twice if they double-click
the pay button or experience network issues.

**Behavior**:
1. Customer clicks pay twice quickly → only one payment processed, second
   click ignored
2. Network timeout causes customer to retry → system recognizes original
   payment and shows result instead of charging again

**Verify**:
1. **Covers B1**: Double-click the pay button
   - **Page**: `/register` → payment step
   - Click pay button rapidly
   - Confirm: Only one charge in payment gateway dashboard

2. **Covers B2**: Retry after timeout
   - **Page**: `/register` → payment step
   - Submit payment, simulate network timeout, submit again
   - Confirm: Original payment result shown, no second charge
```
