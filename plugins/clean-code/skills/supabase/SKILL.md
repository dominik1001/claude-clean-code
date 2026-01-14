---
name: supabase
description: Interact with the databases using Supabase.
---

When interacting with the database using Supabase make sure to follow these guidelines:

- Use `.throwOnError()` for error handling.
- Use `.maybeSingle()` instead of `.single()` when null is a valid response.
