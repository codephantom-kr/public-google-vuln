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

PoC Video Proof: https://drive.google.com/file/d/10Yd77MTj8PkKeWmL3buzeMaIoYegf5j8/view?usp=sharing

Strategic Technical Countermeasures Against Expected Mitigation Claims

Countering "No Harmful Output" Claim (State Control Invalidation)

Expected Google Argument: The generated output does not contain explicitly harmful or illegal payloads (e.g., CSAM/CSAE facilitation), meaning no safety vulnerability exists.

Rebuttal: The core of this vulnerability is not the toxicity level of the final output, but rather the total invalidation of backend state enforcement. The backend explicitly declared a "Hard Block & Session Reset" flag. However, a client-side frontend UI action (Edit) completely overrode and bypassed this global safety lock. The ability for an end-user to force-unlock an enforced system state through simple tree mutation constitutes a definitive business logic flaw.

Countering "Normal Context Shift" Claim (Context Persistence Leak)

Expected Google Argument: The model simply shifted the conversation into a benign, empathetic de-escalation response based on the new token.

Rebuttal: In a properly isolated environment, modifying the prompt to a simple sigh ("하...") should only trigger a fresh generation corresponding strictly to that specific benign token. Instead, the application pulled the previously restricted, sensitive context buffer from memory and forced an unauthorized flashback, combining it into the new response. This clearly demonstrates a failure in backend memory isolation and context containment.

Countering "Isolated Safe Case" Claim (Attack Surface Escalation)

Expected Google Argument: This specific test case resulted in a non-hazardous, safe conversation flow.

Rebuttal: This specific PoC resulted in a de-escalated tone only because a highly minimized, neutral token was used. If a malicious actor utilizes this same structural frontend loophole to inject a targeted second-stage evasion payload, the completely unmonitored backend layer will process and output restricted data, serving as a critical stepping stone for severe security evasion.

## Remediation & Recommendations

Frontend-Level Mitigation (Immediate Fix): Disable or completely remove the "Edit" (수정) button functionality on any message node that has triggered a safety violation or a hard refusal response. Preventing the user from mutating blocked nodes will effectively close this attack vector at the frontend entry point.

Synchronize Tree Mutation Validation: The backend safety classifier must treat frontend message modifications with the exact same weight and strictness as a brand-new prompt. Enforce a global session check that maintains the block state regardless of the sub-tree path.

## Attack scenario

Who can exploit this vulnerability: Any unauthenticated or authenticated end-user with standard access to Google Search AI Mode (Gemini Chat Interface) through a standard web browser. No elevated system permissions, specialized accounts, or technical pre-requisites are required.

Security Impact after successful exploitation:

Total Invalidation of Global Safety Controls: Attackers can completely bypass enforced backend Hard Block states and Session Reset flags using simple client-side UI manipulation (Tree Mutation).

Unauthorized Context Retention & Memory Isolation Failure: The system fails to sanitize restricted context buffers upon issuing a safety block, allowing lingering sensitive/blocked conversation histories to be forcibly re-evaluated and leaked into subsequent responses.

Escalation Vector for Safety Evasion: Unlocks an unmonitored execution path where malicious actors can bypass initial safety filters and inject second-stage payloads to extract restricted information.
