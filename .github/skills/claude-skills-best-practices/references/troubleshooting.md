# Troubleshooting

## Skill won’t upload

**Error**: Could not find SKILL.md
- Cause: File not named exactly SKILL.md
- Fix: Rename to SKILL.md (case-sensitive)

**Error**: Invalid frontmatter
- Cause: YAML formatting issue
- Fix: Verify --- delimiters and indentation

**Error**: Invalid skill name
- Cause: Name has spaces or capitals
- Fix: Use kebab-case and match folder name

## Skill doesn’t trigger

- Too generic description
- Missing trigger phrases
- Relevant file types not mentioned

Debug: Ask when the skill should trigger and revise description accordingly.

## Skill triggers too often

- Add negative triggers
- Narrow description and include specific domain terms

## MCP connection issues

1. Confirm MCP server is connected
2. Verify credentials and permissions
3. Test MCP tool directly
4. Ensure tool names are correct and fully qualified

## Instructions not followed

Common causes:
- Too verbose
- Critical instructions buried
- Ambiguous language
- Missing validations

Fix by promoting key rules to top-level sections and adding validation steps or scripts.
