# Phase 1: Foundation & API - Context

**Gathered:** 2026-01-19
**Status:** Ready for planning

<vision>
## How This Should Work

The feedback submission flow should be conversation-style — guiding users through providing details rather than a single form dump. The API uses a hybrid approach: it can provide optional guidance/prompts that host apps can display to users, but host apps can also ignore this and handle their own UX.

Apps integrate via a client SDK/library that handles the API communication. This makes integration dead simple — import the library, call a function, done.

</vision>

<essential>
## What Must Be Nailed

All three are equally critical for this phase:

- **Reliable data capture** — Never lose feedback. Data integrity is king.
- **Easy integration** — Dead simple for apps to start sending feedback via SDK.
- **Flexible structure** — Support different feedback types (bug, feature, general) cleanly.

</essential>

<specifics>
## Specific Ideas

- Provide an SDK/client library for apps to import (not just raw HTTP documentation)
- Conversation-style flow with optional guidance from the API
- Hybrid approach: API provides prompts, but apps can ignore them and do their own UX

</specifics>

<notes>
## Additional Context

The multi-step, conversation-style flow is a differentiator — most feedback APIs are simple POST endpoints. This approach guides users to provide better, more structured feedback.

The SDK is v1 scope, not v2. This is important for the user's integration vision.

</notes>

---

*Phase: 01-foundation-api*
*Context gathered: 2026-01-19*
