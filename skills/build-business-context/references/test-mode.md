<!-- overlay: add — derived-from: build-business-context/SKILL.md @ 7d279ce1dd2a -->

# Test mode (fetch + persist only)

This mode exists to exercise just the two Noibu integration points — the *fetch* in
Step 1 and the *persist* in Step 7 — without running the generative middle (research,
call mining, interview, assembly).

**Trigger it** when the user explicitly asks for "test mode" (or "persist-only test")
**and uploads a Markdown file into the chat** to use as the context. Do not infer test
mode; require the explicit ask.

In test mode:

1. **Run Step 1 as written** — resolve the domain, capture both the UUID and the
   integer `numId`, and call `noibu_get_business_context`. This is the fetch under test.
   **Skip the overwrite gate**: if a context already exists, just tell the user it will
   be overwritten and continue — don't ask permission, since overwriting is the point.
2. **Skip Steps 2–6 entirely** — no setup questions, no research, no call
   mining, no interview, no assembly. Use the **uploaded file's contents verbatim** as
   the context. Do not read from a path and do not edit, reformat, or summarize it.
3. **Run Step 7 as written** — call `noibu_insert_business_context` with the `numId`
   from Step 1 as `domainNumId` and the uploaded file's full contents as `fullContent`,
   exactly as uploaded. Then confirm it's saved to Noibu for `<domain>`.

Everything else in this skill is bypassed in test mode.

---
