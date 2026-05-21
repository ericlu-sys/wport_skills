# Label Mapping Reference

This document maps all available labels based on the reference card: [model card](https://trello.com/c/qcHo7xT3/18-model)

## Label Categories

### Level Labels (Work Hierarchy)

| Label Name | Color | Usage | Notes |
|------------|-------|-------|-------|
| `level:goal` | `black_light` | Main project goals | Used for top-level project cards |
| `level:delivery` | `black` | Deliverable items | Used for deliverable cards (FE/BE tasks) |
| `level:task` | `black_dark` | Individual tasks | Used for specific task-level cards |

### Role Labels (Team Assignment)

| Label Name | Color | Usage | Notes |
|------------|-------|-------|-------|
| `role:fe` | `purple` | Frontend work | Applied to cards containing "畫面" or frontend-related content |
| `role:be` | `orange` | Backend work | Applied to cards containing "api" or backend-related content |

### Design/Development Stage Labels

| Label Name | Color | Usage | Notes |
|------------|-------|-------|-------|
| `mockup` | `blue` | Mockup stage | Design mockups |
| `wireframe` | `blue_dark` | Wireframe stage | Initial wireframes |
| `prototype` | `blue_light` | Prototype stage | Working prototypes |

### Status/Type Labels

| Label Name | Color | Usage | Notes |
|------------|-------|-------|-------|
| `optimization` | `green` | Optimization work | Performance or code optimization |
| `bug` | `red` | Bug fixes | Bug-related cards |

## Label Resolution Logic

When creating labels, follow this priority:

1. **Check exact match**: Look for exact label name first
2. **Check alternatives**: If not found, check alternative names
3. **Reuse existing**: If alternative exists, use its ID (don't create duplicate)
4. **Create new**: Only create if neither exact nor alternative exists

## Label Usage in Workflow

### Main Goal Card
- **Required**: `level:goal` only
- **Do NOT use**: Role labels (role:fe, role:be) or deliverable labels

### Deliverable Cards
- **Required**: 
  - Role label (`role:fe` OR `role:be`) based on content
  - `level:delivery` label
- **Optional**: Design stage labels (mockup, wireframe, prototype) if applicable

### Label Detection Logic
- Cards containing "api" → `role:be` + `level:delivery`
- Cards containing "畫面" → `role:fe` + `level:delivery`

## Color Mapping

If creating new labels, use these colors:

- `level:goal`: `black_light`
- `level:delivery`: `black`
- `level:task`: `black_dark`
- `role:fe`: `purple`
- `role:be`: `orange`
- `mockup`: `blue`
- `wireframe`: `blue_dark`
- `prototype`: `blue_light`
- `optimization`: `green`
- `bug`: `red`

## Trello Color Values

Available color values for `mcp_trello_create_label`:
- `red`
- `blue`
- `green`
- `yellow`
- `orange`
- `purple`
- `pink`
- `sky`
- `lime`
- `black`
- `null` (no color)

Note: Trello also supports dark/light variants (`red_dark`, `red_light`, etc.) but these are typically set automatically based on the base color.

## Reference Card

For a complete example of all labels, see: [Trello Card - model](https://trello.com/c/qcHo7xT3/18-model)
