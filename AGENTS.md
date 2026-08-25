# Conversion Agent Workspace

Use the `dotnet-to-go-converter` agent for .NET API conversion work.

Read [CONTEXT.md](./CONTEXT.md) before naming conversion concepts. The agent produces a Conversion manifest and route artifacts under the selected output workspace. Follow the stage approvals in the agent file; unresolved conflicts and unsupported behavior become Blockers.

Skills are in `.agents/skills/` and are invoked by stage:

- `dotnet-source-analysis`
- `go-template-profile`
- `dotnet-to-go-planning`
- `api-route-conversion`
- `api-parity-validation`
