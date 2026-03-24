# Specify Templates

## specs/prd.md

````markdown
# {{PRODUCT_NAME}} — Product Requirements

## User Stories

| ID | Title | Priority | Summary |
|----|-------|----------|---------|
| US-001 | {{title}} | P0 | {{one-line summary}} |

### US-001: {{Title}}

**As a** {{persona}}, **I want to** {{action}}, **so that** {{benefit}}.

**Acceptance Criteria:**

```gherkin
Given {{precondition}}
When {{action with concrete values}}
Then {{observable outcome with concrete values}}
```

**Example:**
- Input: {{concrete input}}
- Output: {{concrete output}}

---

## Non-Functional Requirements

| ID | Category | Requirement | Target |
|----|----------|-------------|--------|
| NFR-001 | Performance | {{requirement}} | {{measurable target}} |
| NFR-002 | Security | {{requirement}} | {{standard/level}} |
| NFR-003 | Accessibility | {{requirement}} | {{WCAG level}} |
````

## specs/architecture.md

````markdown
# {{PRODUCT_NAME}} — Architecture

## Tech Stack

| Layer | Choice | Rationale |
|-------|--------|-----------|
| Language | {{language}} | {{why}} |
| Framework | {{framework}} | {{why}} |
| Database | {{database}} | {{why}} |
| Hosting | {{provider}} | {{why}} |
| Auth | {{provider/method}} | {{why}} |

## Data Models

### {{ModelName}}

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK | {{description}} |
| {{field}} | {{type}} | {{constraints}} | {{description}} |

**Relationships:**
- {{ModelA}} has many {{ModelB}} (via {{foreign_key}})

## System Overview

```
[Client] --> [API Gateway] --> [App Server] --> [Database]
                                    |
                                    +--> [External Service]
```

**Component boundaries:**
- {{Component}}: {{responsibility}} ({{tech}})
````

## specs/services/api.md

````markdown
# {{PRODUCT_NAME}} — API Specification

## Base URL

`{{base_url}}`

## Authentication

All endpoints except {{public_endpoints}} require `Authorization: Bearer <token>`.

## Endpoints

### {{Resource}}

#### {{METHOD}} {{path}}

**Description**: {{what it does}}
**Auth**: {{required | public}}

**Request:**
```json
{{request_schema}}
```

**Response (200):**
```json
{{response_schema}}
```

**Errors:**
| Status | Code | Description |
|--------|------|-------------|
| 400 | {{code}} | {{description}} |
| 404 | {{code}} | {{description}} |

---

## Error Format

All errors follow:
```json
{
  "error": { "code": "{{CODE}}", "message": "{{Human-readable message}}" }
}
```
````

## specs/services/auth.md

````markdown
# {{PRODUCT_NAME}} — Authentication

## Auth Flow

1. User submits credentials to `POST /auth/login`
2. Server validates, returns `{ access_token, refresh_token }`
3. Client stores tokens in {{storage_mechanism}}
4. Client sends `Authorization: Bearer <access_token>` on requests
5. On 401, client uses refresh token via `POST /auth/refresh`

## Token Strategy

| Token | Type | Lifetime | Storage |
|-------|------|----------|---------|
| Access | {{JWT/opaque}} | {{duration}} | {{where}} |
| Refresh | opaque | {{duration}} | {{where}} |

## Roles & Permissions

| Role | Permissions |
|------|-------------|
| {{role}} | {{permissions list}} |

## Protected Routes

| Route Pattern | Required Role |
|---------------|---------------|
| {{pattern}} | {{role}} |
````

## specs/services/i18n.md

````markdown
# {{PRODUCT_NAME}} — Internationalization

## Supported Locales

| Locale | Language | Status |
|--------|----------|--------|
| en | English | Default |
| {{locale}} | {{language}} | {{status}} |

## Key Format

`{namespace}.{section}.{key}` — e.g., `auth.login.submit_button`

## Pluralization

Use ICU MessageFormat: `{count, plural, one {# item} other {# items}}`

## File Structure

```
locales/
├── en.json
└── {{locale}}.json
```
````

## specs/decisions.md (Append Format)

Each decision appended as:

````markdown
### DEC-{NNN}: {Decision Title}
**Date**: {{DATE}}
**Context**: {{what prompted this decision}}
**Decision**: {{what we chose}}
**Alternatives**: {{what we considered}}
**Rationale**: {{why this choice}}
**Affects**: {{list of spec artifacts impacted}}
````
