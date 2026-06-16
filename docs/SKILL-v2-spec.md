# SKILL.md v2 Specification

## New Frontmatter Fields (v2.0.0)

Five fields added to every skill's YAML frontmatter. All are required in v2.

| Field | Type | Constraint |
|---|---|---|
| `depends_on` | array of strings | Empty array `[]` allowed; cross-bundle deps not supported |
| `tags` | array of strings | Min 2, max 8; lowercase kebab-case |
| `audience` | array of strings | Min 1; values from the valid-values list |
| `bundle` | string | Must match GitHub repo name exactly |
| `mcp_tools` | array of objects | Min 2, max 3; each needs name + description + input_schema |

### `depends_on` (array of strings)

Explicit skill dependency chain. The MCP skill-graph resolver loads these before
the current skill's content.

If a named dependency skill is not found in the current bundle, the MCP server logs a warning and continues without it — cross-bundle dependencies are not supported.

```yaml
depends_on:
  - healthcare-context
```

### `tags` (array of strings)

Searchable labels. Use lowercase kebab-case. Minimum 2, maximum 8.

```yaml
tags:
  - fhir
  - interoperability
  - r4
  - rest-api
```

### `audience` (array of strings)

Who this skill is for. Valid values: `qa-engineer`, `sdet`, `developer`,
`architect`, `devops`, `clinician`, `informatics-engineer`, `compliance-officer`.

```yaml
audience:
  - clinician
  - informatics-engineer
  - compliance-officer
```

### `bundle` (string)

Which repo/plugin this skill belongs to. Must match the GitHub repository name exactly (the value after `aks-builds/` in the repo URL, e.g., `quality-skills` for `github.com/aks-builds/quality-skills`). The CI validator compares this field against the known repo name at validation time.

```yaml
bundle: healthcare-skills
```

Valid values: `quality-skills`, `healthcare-skills`

### `mcp_tools` (array of objects)

Typed MCP tool declarations. Each tool must have `name`, `description`, and
`input_schema` (valid JSON Schema object).

Tool names must follow the pattern `<skill_name_underscored>_<action>`.

> **Constraints:** `minItems: 2` — every skill must declare at least 2 tools.
> `maxItems: 3` — no more than 3 tools per skill.

**Action vocabulary:**

| Action | Use for |
|---|---|
| `scaffold` | Generate starter code or configuration from scratch |
| `debug` | Diagnose a failure, error, or unexpected behaviour |
| `audit` | Review existing code/config for issues, anti-patterns, or gaps |
| `migrate` | Convert from one tool/version/format to another |
| `checklist` | Generate a compliance or readiness checklist |
| `validate` | Check a document, message, or resource against a specification |
| `generate` | Produce data, reports, or synthetic artifacts |
| `troubleshoot` | Step-by-step diagnosis of an integration or runtime issue |

> **Note on `default` in `input_schema`:** The `default` key in `input_schema`
> is a JSON Schema annotation (draft-07). It is passed through by the MCP server
> as metadata but is not validated by the CI schema checker. Validators using
> strict mode should treat it as an annotation-only keyword.

> **Note on `required`:** If all properties are optional, omit the `required`
> array entirely (do not write `required: []`). If at least one property must be
> provided, list those property names in `required`.

```yaml
mcp_tools:
  - name: fhir_validate
    description: "Check a FHIR resource against the R4 specification and profile constraints"
    input_schema:
      type: object
      properties:
        resource:
          type: string
          description: "FHIR resource JSON or YAML to validate"
        profile:
          type: string
          description: "Optional StructureDefinition URL to validate against"
      required: [resource]
  - name: fhir_troubleshoot
    description: "Step-by-step diagnosis of a FHIR REST API integration issue"
    input_schema:
      type: object
      properties:
        error:
          type: string
          description: "HTTP error response or OperationOutcome from the FHIR server"
        endpoint:
          type: string
          description: "The FHIR endpoint that was called (optional)"
      required: [error]
  - name: fhir_audit
    description: "Review a FHIR implementation for interoperability gaps and anti-patterns"
    input_schema:
      type: object
      properties:
        focus:
          type: string
          enum: [search, references, extensions, terminology, all]
          default: all
```

## Complete v2 Frontmatter Example

```yaml
---
name: fhir-integration
description: When the user wants to validate, troubleshoot, or audit a FHIR R4 REST API integration…
metadata:
  version: 2.0.0
  author: aks-builds
  license: MIT
depends_on:
  - healthcare-context
tags:
  - fhir
  - interoperability
  - r4
  - rest-api
audience:
  - clinician
  - informatics-engineer
  - compliance-officer
bundle: healthcare-skills
mcp_tools:
  - name: fhir_validate
    description: "Check a FHIR resource against the R4 specification and profile constraints"
    input_schema:
      type: object
      properties:
        resource: { type: string }
        profile: { type: string }
      required: [resource]
  - name: fhir_troubleshoot
    description: "Step-by-step diagnosis of a FHIR REST API integration issue"
    input_schema:
      type: object
      properties:
        error: { type: string }
        endpoint: { type: string }
      required: [error]
  - name: fhir_audit
    description: "Review a FHIR implementation for interoperability gaps and anti-patterns"
    input_schema:
      type: object
      properties:
        focus: { type: string, enum: [search, references, extensions, terminology, all], default: all }
---
```
