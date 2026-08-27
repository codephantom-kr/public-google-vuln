# Details

## Description
This report discloses a critical Business Logic Flaw & State Management Defect in Google Search AI Mode (Gemini Chat Interface). When a user prompt triggers a safety filter violation, the backend correctly issues a hard block and mandates a session reset. However, this safety state is strictly bound to the primary chat input box and fails to synchronize with frontend tree mutation events.

By clicking the "Edit" button on the blocked prompt bubble and appending or modifying the text with a simple single-word emotional token (e.g., "하..."), the backend safety lock is entirely overridden. Instead of maintaining the block state or sanitizing the session memory, the backend forces an "Unauthorized Context Flashback"—re-evaluating and restoring the previously restricted context buffer and outputting a long-form response. This allows an attacker to invalidate session-reset state flags and override system security controls via client-side UI manipulation.

