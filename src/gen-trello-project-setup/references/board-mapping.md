# Board Mapping Reference

## Board IDs

| Board Type | Board ID | Short URL | Usage |
|------------|----------|-----------|-------|
| **BE** (Backend) | `bbEZeABT` | `bbEZeABT` | Backend projects |
| **FE** (Frontend) | `IYi0tcMo` | `IYi0tcMo` | Frontend projects |
| **PM** (Project Management) | `STdWiajz` | `STdWiajz` | Project management and general projects |

## Board Selection Logic

1. **Explicit specification**: If user specifies "BE", "FE", or "PM", use the corresponding board
2. **Default**: If not specified, use PM board (`STdWiajz`)
3. **Context-based**: If project name suggests frontend/backend, use appropriate board

## Board ID Format

Trello board IDs can be:
- Short URL identifiers (e.g., `bbEZeABT`, `IYi0tcMo`, `STdWiajz`)
- Full board IDs (longer alphanumeric strings)

The MCP Trello functions accept both formats. Use the short URL format for consistency.
