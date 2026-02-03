---
name: ubiquitous-language
description: >
  Enforces consistent domain terminology across code, documentation,
  and conversation. Use when naming variables, functions, database
  columns, API endpoints, tRPC procedures, or writing documentation.
  Use when user says "what should I call this", "naming suggestion",
  "rename this", "is this the right term", or reviews code for
  naming consistency. Also use when generating new code, schemas,
  or types to ensure domain terms are applied correctly.
---

# Ubiquitous Language

Enforce consistent domain terminology in all generated code, schemas, types, and documentation. When multiple synonyms exist for a concept, always use the canonical term.

## How to Apply

1. Before generating code or naming anything, consult `docs/GLOSSARY.md` for the canonical term.
2. If a user uses a non-canonical synonym, gently clarify the canonical term and use it in output.
3. When reviewing code, flag terms that deviate from the glossary.

## Naming Conventions by Layer

| Layer | Format | Example |
|---|---|---|
| TypeScript types/interfaces | PascalCase | `Conversation`, `Participant` |
| Variables & functions | camelCase | `getConversation`, `participantList` |
| tRPC procedures | camelCase verb-first | `createConversation`, `listParticipants` |
| Database tables | snake_case plural | `conversations`, `participants` |
| Database columns | snake_case | `created_at`, `participant_id` |
| URL paths / API routes | kebab-case plural | `/conversations`, `/participants` |
| Environment variables | SCREAMING_SNAKE | `DATABASE_URL`, `SUPABASE_ANON_KEY` |

## Rules

- Never invent new terms for existing glossary concepts.
- If a concept has no glossary entry, propose one to the user before proceeding.
- Prefer domain terms over generic ones (e.g. `Conversation` not `Thread` or `Chat`, if that's what the glossary says).
- Compound names should read naturally: `conversationParticipant` not `participantOfConversation`.
- Boolean fields/variables: prefix with `is`, `has`, `can`, `should` (e.g. `isArchived`, `hasUnreadMessages`).

## Glossary

Maintain the glossary in `docs/GLOSSARY.md`. Each entry follows this format:

```
## Term (PascalCase)
- **Definition**: What it represents in the domain.
- **Not**: Synonyms to avoid.
- **Used in**: Where it appears (DB table, type, procedure, etc.)
```
