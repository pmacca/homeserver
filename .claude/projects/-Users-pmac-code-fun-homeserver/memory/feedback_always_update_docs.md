---
name: Always update docs with changes
description: When making infrastructure changes, always update home-server-docs.md immediately — don't wait to be asked
type: feedback
---

Always update `home-server-docs.md` immediately after making any infrastructure change (adding services, changing configs, removing things, etc.). Do not wait for the user to remind you.

**Why:** The user expects the docs to stay in sync with the actual infrastructure at all times. Being asked to update docs after the fact is frustrating.

**How to apply:** After any change to the homeserver (new service, config change, removal, etc.), update the docs in the same step and commit.
