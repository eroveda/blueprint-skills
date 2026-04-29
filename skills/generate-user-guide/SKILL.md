---
name: generate-user-guide
description: |
  Creates USER_GUIDE.md for end-users (not developers).
  Use when preparing onboarding material or end-user documentation.
  Trigger with "generate user guide" or "document for end users".
---

# Generate User Guide

You create USER_GUIDE.md that helps actual users learn how to use the system.

## Document Structure

```markdown
# User Guide — [Project Name]

## Welcome
One paragraph: what the system is for and who should use it.

## Getting Started

### First Login
1. Step-by-step from first access to first useful action
2. Include screenshots/examples if relevant
3. End with: "You're ready to..."

### Setting Up Your Account
- Profile setup
- Initial configuration
- Inviting teammates (if applicable)

## Daily Usage

### For [Role 1]
Common tasks for this role, ordered by frequency:

#### [Task Name]
1. Go to [section]
2. Click/tap [action]
3. Fill in [fields]
4. Confirm with [action]
5. You'll see [result]

(Repeat for each common task)

### For [Role 2]
(Same structure)

## Understanding the System

### Statuses and What They Mean
For each status in the system:
- **[Status Name]** — Plain explanation, when to use it
- Example scenario

### Permissions
Plain-language explanation of who can do what:
- [Role 1] can: ...
- [Role 2] can: ...
- [Role 1] cannot: ...

## Common Scenarios

### Scenario: [Realistic situation]
"You need to [goal]. Here's how:"
1. ...
2. ...
3. ...

(3-5 realistic scenarios that cover 80% of usage)

## Troubleshooting

### "I can't [action]"
Probably because: [reason]
To fix: [steps]

### "I see the error: [message]"
This means: [explanation]
To fix: [steps]

## FAQs

### Question 1?
Answer in plain language.

### Question 2?
Answer.

## Getting Help

- Where to find help (link/email)
- Response time expectations
- What information to provide when asking for help
```

## Style Rules

- Write for someone who has never used the system before
- Use "you" not "the user"
- Short paragraphs (max 3 sentences)
- One action per step
- Avoid jargon, but if unavoidable, explain in parentheses
- Use realistic examples (real names, real scenarios)
- Include error recovery for every flow

## How to Generate

1. Read FUNCTIONAL_FLOWS.md if it exists (reuse content)
2. Identify the most common 5-10 user actions
3. Write each as a step-by-step
4. Include screenshots/mockups placeholders: `[Screenshot: action result]`
5. Add troubleshooting based on validation errors in code
6. Save as USER_GUIDE.md

## Quality Checklist

- [ ] A new user can complete their first task without external help
- [ ] Every common task has a step-by-step
- [ ] Permissions are explained per role
- [ ] At least 3 troubleshooting entries for common errors
- [ ] FAQs cover real questions, not theoretical ones
- [ ] No technical jargon without explanation
- [ ] Tone is friendly but professional
