# Details

## Description
This report discloses a critical Business Logic Flaw & State Management Defect in Google Search AI Mode (Gemini Chat Interface). When a user prompt triggers a safety filter violation, the backend correctly issues a hard block and mandates a session reset. However, this safety state is strictly bound to the primary chat input box and fails to synchronize with frontend tree mutation events.

By clicking the "Edit" button on the blocked prompt bubble and appending or modifying the text with a simple single-word emotional token (e.g., "하..."), the backend safety lock is entirely overridden. Instead of maintaining the block state or sanitizing the session memory, the backend forces an "Unauthorized Context Flashback"—re-evaluating and restoring the previously restricted context buffer and outputting a long-form response. This allows an attacker to invalidate session-reset state flags and override system security controls via client-side UI manipulation.

## Attack Preconditions
The attacker must have access to Google Search AI Mode (Gemini Chat Interface) via a standard web browser.

No elevated permissions or special accounts are required.

## Reproduction Steps / POC
Trigger Guardrail: Navigate to Google Search AI Mode (https://www.google.com/search) and enter a prompt containing strictly restricted terms regarding sensitive contexts.

Observe System Mitigation: Confirm that the backend intercepts the input, presenting a hard refusal along with a mandatory session reset notification ("다음 질문 시 새롭게 검색이 시작됩니다."). (See Attached Screenshot)

Inject Evasion Payload via Tree Mutation: Do NOT use the primary bottom input box. Instead, click the "Edit" (수정) button directly on the blocked prompt bubble. Append or modify the text with a simple benign token such as 하... and resubmit. (See Attached Screenshot)

Observe State Override & Flashback: The backend global lock breaks completely. The system restores the previously restricted context buffer from memory and generates a detailed response addressing the prior blocked topic, confirming the state control override and memory isolation failure. (See Attached Screenshot)
