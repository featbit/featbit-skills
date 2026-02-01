# Workflows and Feedback Loops

## Contents
- When to use workflows
- Checklist pattern
- Feedback loop pattern
- Conditional workflow pattern
- Template and examples patterns

## When to use workflows

Use workflows when tasks are multi-step, fragile, or need verification. Keep the steps explicit and include validation at each critical stage.

## Checklist pattern

Copy a checklist into responses to track progress:

Task Progress:
- [ ] Step 1: Gather inputs
- [ ] Step 2: Execute primary action
- [ ] Step 3: Validate output
- [ ] Step 4: Fix issues and re-validate
- [ ] Step 5: Finalize

## Feedback loop pattern

Common loop: validate → fix → repeat.

1. Perform the action.
2. Run validation or checklist.
3. If failures appear, correct them and re-validate.
4. Proceed only when validation passes.

## Conditional workflow pattern

Guide decision points explicitly:

1. Determine task type:
   - New content → follow Creation workflow
   - Update content → follow Editing workflow

2. Creation workflow:
   - Use a preferred library
   - Generate structure
   - Validate formatting

3. Editing workflow:
   - Modify existing content
   - Validate changes
   - Rebuild or save

## Template pattern

Provide a default output template and indicate how strict it is.

Strict template example:

# [Title]

## Summary
[One paragraph]

## Key findings
- Finding 1
- Finding 2

## Recommendations
1. Recommendation 1
2. Recommendation 2

## Examples pattern

Use input/output examples to lock in format:

Input: Fixed date formatting bug in reports
Output:
fix(reports): correct date formatting in timezone conversion

Use the same structure for other outputs.
