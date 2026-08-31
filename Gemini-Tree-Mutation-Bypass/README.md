# Google Search AI Mode — Session State Enforcement Finding (Responsible Disclosure Summary)

> ⚠️ This document intentionally omits reproduction steps, exact parameter values,
> and trigger content. It is a methodology summary for a finding that was formally
> reported to Google VRP.

## Overview

While using Google Search's AI Mode, I observed a point in the conversation flow
where the server issued a refusal under certain conditions and persisted that
refusal as part of the session state. I subsequently found that a client-side UI
action could be used to keep the same session context while eliciting a fresh
generative response.

## Methodology

1. **Network capture (HAR) collection**
   Captured the full conversation flow as HAR files via Chrome DevTools' Network tab.

2. **Diffing captures**
   Compared HAR captures from different points in time (initial state / normal
   flow / problematic flow) to track differences in the request list itself
   (which requests were newly issued) and changes in the session identifiers
   carried in each request.

3. **Session identifier tracking**
   Google's search infrastructure issues a unique identifier for each search
   session. I confirmed that a genuinely new topic query causes this identifier
   to be refreshed, accompanied by additional requests corresponding to
   page/session initialization.

4. **Response content comparison**
   Parsed each request's response body to confirm the presence and location
   (i.e., which session identifier it was attached to) of a refusal message
   rendered at a specific point in the flow.

## Key Findings (Summary)

- At a specific point in the conversation, the server generated an explicit
  refusal response, and this was confirmed to persist as part of that session's
  conversation history.
- When the client subsequently used a specific edit feature in the UI to modify
  and resend a prior message, the session identifier was not refreshed — the
  existing value was reused as-is.
- For a request resent through this path, the server returned a newly generated
  response despite a refusal already existing in the same session's history.
- The same pattern was reproducible on a separate account and in an incognito
  window, indicating this is not an account-specific issue but a structural
  flaw in session state handling.

## Why This Is an Application Logic Issue, Not an "AI Safety" Issue

The core of this issue is not about how the model probabilistically responds to
a given prompt. It is about **whether a refusal decision the server has already
made can be invalidated through client-side manipulation** — a deterministic
state-management problem. It is therefore addressable through a fix to the
application logic that validates session state, not through model retraining
or adjustments to a safety classifier.

Standard vulnerability classifications that appear relevant (offered as a
reference mapping, not an officially confirmed CWE designation):

- **CWE-841** (Improper Enforcement of Behavioral Workflow) — a structure in
  which the client can bypass the state-transition order the server expects.
- **CWE-226** (Sensitive Information in Resource Not Removed Before Reuse) — a
  structure in which prior session/context data is reused for a new request
  without being properly isolated or reset.

## Disclosure History

This issue was formally reported to Google VRP. The initial determination
classified it as a "safety guardrail bypass" and placed it out of scope for
the AI VRP program. I subsequently requested re-review twice, arguing that
this was a deterministic application state-management defect rather than
probabilistic model behavior. Google confirmed that the relevant team had
reviewed the report directly, but ultimately upheld the original
determination (Won't Fix).

One structural limitation that surfaced in this process is that infrastructure
or session-management defects that are superficially entangled with a content
safety issue may be routed wholesale into the AI-related VRP track — even when
the actual root cause sits at the application layer — without being
transferred to the Classic (traditional) web security VRP track. This kind of
scoping boundary issue seems unlikely to be limited to this single case, and
may recur structurally across vulnerability-reporting processes for web
services with integrated AI features. Reproduction steps and trigger content
are withheld given their potential for misuse.

## Lessons Learned

- A substantial amount of a production web application's session-management
  logic can be reverse-traced from browser network captures alone.
- Clearly distinguishing "confirmed facts" from "inference" in a report is
  essential to persuading a technical reviewer.
- Content safety issues and application security issues can overlap, but the
  scoping criteria of each program can differ completely.
