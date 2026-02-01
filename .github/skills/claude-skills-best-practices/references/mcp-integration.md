# MCP Integration Guidance

## When to use MCP details

Use MCP tools when deterministic, structured actions are needed. Keep tool usage explicit and include prerequisites and validation steps.

## Rules

- Use fully qualified tool names: ServerName:tool_name
- Declare dependencies in frontmatter metadata
- Validate prerequisites before tool calls
- Include verification steps after tool calls

## Example workflow

1. Validate MCP connection and credentials
2. Run the tool with explicit parameters
3. Verify output and handle errors
4. Document results

## Common errors

- Tool not found: missing server prefix
- Authentication failure: invalid token or permissions
- Unclear tool selection: multiple servers with similar tool names
