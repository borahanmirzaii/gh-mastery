# `gh project field-list`

> **One-liner:** List all fields defined on a GitHub Projects v2 board, including their types, IDs, and — for single-select fields — the option IDs needed for `item-edit` and GraphQL mutations.

## When you reach for it

`field-list` is the **prerequisite read** before any field mutation. You need the field's node ID (`PVTF_…`) before you can call `item-edit --field-id`, `field-delete --id`, or the `updateProjectV2Field` GraphQL mutation (to rename Status options). It also gives you the single-select option IDs required by `item-edit --single-select-option-id`.

In short: whenever you're about to programmatically set a field value or change field configuration, run `field-list --format json` first to grab the right IDs.

## Key flags

- `<number>` — the project number (positional, required). Get it from `gh project list`.
- `--owner <login>` — who owns the project. Use `@me` for yourself. **Required.**
- `--format json` — emit JSON instead of the default table. Almost always what you want in scripts because the table view omits option IDs.
- `--jq <expr>` — filter the JSON inline. Useful for extracting a specific field's ID or options.
- `-L / --limit <n>` — max fields to return (default 30). Most projects have far fewer than 30 fields, but raise it if you've added many custom fields.

## Examples

```bash
# Human-readable table of all fields in project #14
gh project field-list 14 --owner borahanmirzaii

# Full JSON dump — name, type, id, and options for each field
gh project field-list 14 --owner borahanmirzaii \
  --format json --jq '.fields[] | {name,type,id,options}'

# Get just the Status field's node ID
gh project field-list 14 --owner borahanmirzaii \
  --format json \
  --jq '.fields[] | select(.name=="Status") | .id'

# Get the Status field's option IDs (needed for item-edit and GraphQL mutations)
gh project field-list 14 --owner borahanmirzaii \
  --format json \
  --jq '.fields[] | select(.name=="Status") | .options'

# Get the node ID of a custom field named "Sprint goal"
gh project field-list 1 --owner @me \
  --format json \
  --jq '.fields[] | select(.name=="Sprint goal") | .id'

# List only single-select fields (useful to find all fields with options)
gh project field-list 14 --owner borahanmirzaii \
  --format json \
  --jq '.fields[] | select(.type=="SINGLE_SELECT") | {name,id,options}'
```

## Gotchas

- **The table view omits option IDs for single-select fields.** The default output shows field names and types but not the option IDs you need for `item-edit`. Always use `--format json` when you need those IDs.

- **Built-in Status options cannot be edited via `gh project field-edit` — only custom fields can.** `field-edit` does not exist as a `gh project` subcommand (as of gh 2.92.0). To rename or reorder Status options, use the `updateProjectV2Field` GraphQL mutation with the `fieldId` you get from `field-list`. The `fieldId` is the `PVTF_…` node ID, **not** the project number. Example:

  ```bash
  # 1. Get the Status field ID
  FIELD_ID=$(gh project field-list 14 --owner borahanmirzaii \
    --format json --jq '.fields[] | select(.name=="Status") | .id')

  # 2. Get the project node ID
  PROJECT_ID=$(gh project view 14 --owner borahanmirzaii \
    --format json --jq '.id')

  # 3. Get existing option IDs (required — you must supply them all in the mutation)
  gh project field-list 14 --owner borahanmirzaii \
    --format json \
    --jq '.fields[] | select(.name=="Status") | .options'

  # 4. Mutation (replace option IDs and names as needed)
  gh api graphql -f query='
    mutation($pid:ID!,$fid:ID!) {
      updateProjectV2Field(input:{
        projectId:$pid, fieldId:$fid,
        singleSelectOptions:[
          {id:"<todo-option-id>",   name:"Backlog",     color:GRAY,   description:""},
          {id:"<inprog-option-id>", name:"In Progress", color:YELLOW, description:""},
          {id:"<done-option-id>",   name:"Done",        color:GREEN,  description:""}
        ]
      }) { projectV2Field { ... on ProjectV2SingleSelectField { name options {id name} } } }
    }' -F pid="$PROJECT_ID" -F fid="$FIELD_ID"
  ```

- **You must include ALL existing options in `updateProjectV2Field`.** Omitting an option from the array deletes it (and any item values set to that option). Always read all current options first and pass the full list, modifying only the ones you want to change.

- **Field node IDs start with `PVTF_` (or `PVTSSF_` for single-select fields) and are different from option IDs.** Option IDs (returned under `.options[].id`) are short hex strings like `f75ad846`. Regular field IDs look like `PVTF_lAHODj…` and single-select field IDs look like `PVTSSF_lAHODj…`. Don't mix them up in mutations.

- **`--owner` is required.** No shorthand from current directory, unlike `gh repo`.

## Concepts

- [Projects v2 data model](../../../concepts/projects-v2-data-model.md) — how fields, items, and single-select options relate in the GraphQL schema (forthcoming in Milestone 3).

## Sources

- Manual: https://cli.github.com/manual/gh_project_field-list
- Local: `gh project field-list --help` (gh 2.92.0)
