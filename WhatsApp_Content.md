# Missed Call Alert — WhatsApp content

Companion to `IVR_Missed_Call_Alert_PIN_PRD_v2.md` (v2.0 signed off 29 Sep 2026). Hand this to the Message orchestrator / WhatsApp Business team for template submission.

**Context.** The feature's callback-alert channel is **WhatsApp**, not SMS (changed 14 Oct 2026). Push notifications fire on a missed IVR call to both the customer and the CSP as they do today; the WhatsApp message carries the masked calling DID and the callee's own PIN so the callee can call back in one shot.

The push notifications live today anchor tone and language mix:

| Side | Current push (today) |
|---|---|
| **Customer** | *You missed a call from our team member.*<br>*Please call back.*<br>*आपने हमारे टीम का कॉल मिस कर दिया है।*<br>*कृपया उनसे बात करने के लिए वापस कॉल करें।* |
| **CSP** | *आपने कनेक्शन का कॉल मिस कर दिया।*<br>*कृपया वापस कॉल करें।* |

**Placeholders** — map to the fields the campaign binds from the CT event payload (`IVR_Missed_Call_Alert_PIN_PRD_v2.md` §3 R1):

| Placeholder | Source field | Example |
|---|---|---|
| `{{1}}` = masked calling DID | the number the callee dials to reach IVR | `+918046161110` |
| `{{2}}` = PIN | `customer_pin` on customer-side event, `csp_pin` on CSP-side event | `234491` |

*(WhatsApp Business templates use `{{n}}` numbered variables. Each language variant is a separate template.)*

---

## CSP WhatsApp — sent when the CSP missed the call *(PENDING RE-SUBMISSION — supersedes template 8909340)*

| Component | Content |
|---|---|
| **Body** *(text, variables `{{1}}` and `{{2}}`)* | नमस्ते,<br><br>आपके Wiom कनेक्शन ने आपको कॉल किया था, लेकिन आपसे बात नहीं हो पाई।<br><br>वापस कॉल करने के लिए इस नंबर पर कॉल करें: {{1}}<br><br>कॉल के दौरान पूछे जाने पर {{2}} दबाएं। |

**Why this change** — template 8909340 was approved by Meta on 15 Oct 2026 with the body *"आपके Wiom इंटरनेट कनेक्शन के संबंध में हमारी टीम ने आपको कॉल किया था …"*. That reads as **Wiom's team** having called the CSP, but on the CSP side the caller is actually the **subscriber (connection)** — the CSP missed a call from their own connection via Wiom's masked-calling line. The approved copy misattributes the caller, so the recipient wouldn't understand *who* they are calling back. The fix swaps the actor clause from "हमारी टीम ने" to "आपके Wiom कनेक्शन ने", matching today's CSP push (*"आपने कनेक्शन का कॉल मिस कर दिया"*) and the Wiom Operational Terminology rule that CSPs have **connections**, never customers.

Everything else — the structural shape, the "कॉल के दौरान पूछे जाने पर {{2}} दबाएं" dialpad framing, the greeting, the softened-miss clause, the two-paragraph separation of `{{1}}` from `{{2}}` — is byte-for-byte the approved shape, so Meta's classifier should still see the Utility pattern. Mirror the Header, Footer and Button components exactly as they were on template 8909340 — do not change any component shape.

Submit as a **new template** (same Utility category, same language, suggested name `ivr_2_missed_call_csp_hi_v2`). Keep template 8909340 unused in prod — do not route sends to it. Once the new template is approved, deprecate 8909340.

**Sample values** — `{{1}}` = `+918046161110` · `{{2}}` = `087275`

---

## Customer WhatsApp — sent when the customer missed the call

Mirror of the approved CSP template above, with the single swap `हमारी टीम` → `हमारे engineer` (per PM: customers understand "engineer" from last-mile service providers).

| Component | Content |
|---|---|
| **Body** *(text, variables `{{1}}` and `{{2}}`)* | नमस्ते,<br><br>आपके Wiom इंटरनेट कनेक्शन के संबंध में हमारे engineer ने आपको कॉल किया था, लेकिन आपसे बात नहीं हो पाई।<br><br>वापस कॉल करने के लिए इस नंबर पर कॉल करें: {{1}}<br><br>कॉल के दौरान पूछे जाने पर {{2}} दबाएं। |

Match the Header, Footer and Button components to template 8909340 — same shape as the approved CSP template.

**Sample values** — `{{1}}` = `+918046161110` · `{{2}}` = `234491`

---

## Why the approved shape clears INCORRECT_CATEGORY

The approved CSP body (and the mirrored customer body) sidestep every signal that triggered the earlier Utility rejections. For future reference — don't lose these:

| Signal that previously tripped the classifier | How the approved shape dodges it |
|---|---|
| Phone-number variable + code variable in close pairing | Separated across two paragraphs — breaks the "code delivery" structural pattern |
| The word "PIN" / "code" / "कोड" | Not used anywhere. The second variable is unlabelled |
| Imperative "enter" / "डालें" | **"दबाएं"** instead — "press", the standard IVR-dialpad verb. Service pattern, not auth pattern |
| Short, terse message (OTP-shaped) | Longer, conversational — opens with "नमस्ते", gives service context ("Wiom इंटरनेट कनेक्शन के संबंध में"), softens the miss with "आपसे बात नहीं हो पाई" instead of "आपने कॉल मिस कर दिया" |
| Bold formatting on the code | None |
| "Code is conditional on the call" framing missing | **"कॉल के दौरान पूछे जाने पर"** — explicit condition. `{{2}}` is positioned as a *response to a prompt*, not a code being delivered |

The key structural shift is **"कॉल के दौरान पूछे जाने पर {{2}} दबाएं"** — "when asked during the call, press {{2}}". The second variable reads as a dialpad response to an IVR prompt, not as a verification code. That's what moved the template from Authentication-pattern to Utility-pattern in Meta's classifier.

---

## Rejection history

### 14 Oct 2026 — first submission (template id 8904784) — rejected

Original body ended with `PIN: {{2}}`. Two format issues:
1. **Variable at end of Body** — Meta rejects templates whose Body ends with a variable with no trailing text.
2. **PIN pattern read as OTP** — "PIN: *{{2}}*" at the end reads as code delivery.

### 14 Oct 2026 — second submission — rejected with `INCORRECT_CATEGORY`

Second body was `… {{1}} पर कॉल करें और PIN {{2}} डालें।` — fixed the trailing-variable issue, but Meta's classifier returned `INCORRECT_CATEGORY`. The classifier thought the template belonged in Authentication category because the signals it looks for were still present:
- The word **PIN**
- The imperative **डालें** ("enter")
- Bold formatting on the code
- Short overall message

Authentication category is not an option — its mandated format ("`{{1}}` is your verification code") does not fit a callback routing PIN. The fix was to rewrite the body so none of those signals are left, keeping the template in Utility.

### Third submission — rewrote to "कॉल रेफरेंस" + "मिस्ड कॉल" header — rejected again with `INCORRECT_CATEGORY`

Changes attempted:
1. "PIN" → "कॉल रेफरेंस" (removed OTP keyword, used ticket-reference pattern instead).
2. Removed "डालें" imperative; variable in a declarative sentence.
3. Header "वापस कॉल करें" → "मिस्ड कॉल".

Still bounced. Meta's classifier was reading the structural pattern — short message + phone-number variable + short-code variable — as Authentication regardless of specific keywords.

### Fourth submission — approved on CSP side (template id 8909340, 15 Oct 2026) — later retired for wrong actor framing

Approved body (CSP side) as submitted on 15 Oct 2026:
> नमस्ते,
>
> आपके Wiom इंटरनेट कनेक्शन के संबंध में हमारी टीम ने आपको कॉल किया था, लेकिन आपसे बात नहीं हो पाई।
>
> वापस कॉल करने के लिए इस नंबर पर कॉल करें: {{1}}
>
> कॉल के दौरान पूछे जाने पर {{2}} दबाएं।

Cleared Meta's Utility classifier — the structural shifts that worked are documented in the **"Why the approved shape clears INCORRECT_CATEGORY"** section above. The customer template was then mirrored off this body, swapping `हमारी टीम` → `हमारे engineer`.

**Content bug discovered on 15 Oct 2026 (same day).** On the CSP side the caller is the subscriber, not Wiom's team, so "हमारी टीम ने आपको कॉल किया था" misattributes the caller. Fixed by swapping the actor clause to "आपके Wiom कनेक्शन ने" and dropping the "के संबंध में" preface, which preserves the Utility shape intact. The customer template is unaffected — on the customer side the caller **is** the engineer, so "हमारे engineer ने आपको कॉल किया था" is correct as written.

### Fifth submission — CSP, actor-corrected (pending)

Body being submitted:
> नमस्ते,
>
> आपके Wiom कनेक्शन ने आपको कॉल किया था, लेकिन आपसे बात नहीं हो पाई।
>
> वापस कॉल करने के लिए इस नंबर पर कॉल करें: {{1}}
>
> कॉल के दौरान पूछे जाने पर {{2}} दबाएं।

Only two edits vs the 8909340-approved body: `आपके Wiom इंटरनेट कनेक्शन के संबंध में हमारी टीम ने` → `आपके Wiom कनेक्शन ने`, and dropped "इंटरनेट" and "के संबंध में" to keep the sentence natural with the connection as the subject. The six-signal Utility structural pattern (greeting · softened miss · phone variable in its own paragraph · "कॉल के दौरान पूछे जाने पर {{2}} दबाएं" dialpad framing · no "PIN"/"कोड"/"डालें" · no bold on `{{2}}`) is intact.

---

## Utility-approval submission checklist (for Gupshup)

This template belongs in Meta's **Utility** category, not Authentication and not Marketing. The message is a service notification tied to a specific user action (the recipient missed a call); the PIN is a **callback routing code**, not a verification code. Fill the Gupshup submission form as follows.

| Field | Value |
|---|---|
| **Category** | Utility |
| **Language** | Hindi |
| **Template name** *(suggestion)* | `ivr_2_missed_call_customer_hi` *(and `..._csp_hi` for the CSP template)* |
| **Header type** | Text — no variables |
| **Header text** | `वापस कॉल करें` |
| **Body text** | *(see the Customer / CSP tables above — exact text)* |
| **Body sample `{{1}}`** | `+918046161110` |
| **Body sample `{{2}}`** | `234491` *(customer template)* · `087275` *(CSP template)* |
| **Footer text** | `Wiom` |
| **Buttons** | None *(Gupshup does not allow a variable in the Call-phone-number button, and the DID is per-message)* |
| **Use case / description** *(if asked)* | "Service notification sent after a user misses an IVR call from the counterparty on Wiom's masked-calling line. The message carries the Wiom masked DID and the recipient's own routing PIN so the recipient can call back and reconnect to the original counterparty. Not a verification code. Not marketing." |

**Why this fits Utility, not Authentication**
- The PIN does not verify the recipient's identity. It routes an inbound call back to the right counterparty inside Wiom's masked-calling layer.
- The message is triggered by a user action (missing a call), not by an authentication request.
- No "verification code", "OTP", "confirm your account" or "secure code" phrasing anywhere in the body. The second variable is framed as a service identifier ("कॉल रेफरेंस"), not an entry code.

**Why this fits Utility, not Marketing**
- No promotion of a plan, offer, product, discount or feature. The message delivers an operational outcome of a specific event the recipient initiated.

### Why we are not submitting under Authentication

Resubmitting as Authentication would likely auto-approve faster, but it comes with four real downsides:

1. **Rigid mandated format.** Authentication templates must follow a pre-approved pattern like *"`{{1}}` is your verification code"*. **Only one variable is allowed** — just the code. We would lose the masked DID as a dynamic variable, lose the "मिस्ड कॉल" header, lose the custom footer, and lose the missed-call context. The body collapses to a code-delivery line and nothing else.
2. **"Copy code" button replaces our UX.** Authentication templates auto-include a *Copy code* button that puts the code on the clipboard. That matches an app-login flow — not our dial-in flow. Users may think the task is done after copying and never make the call.
3. **Different pricing bucket and quality metrics.** Authentication is priced and rate-limited differently on Meta's conversation-based billing, and it is measured against OTP-specific complaint rates (users flagging codes they did not request). A missed-call PIN that the user did not actively request a code for sits awkwardly against those metrics.
4. **Audit risk — Meta can revoke later.** Meta's AI monitors live traffic against the declared category. Our PINs are not one-time (they can be reused across the ticket's lifetime), not short-lived like OTPs, and not reactive to a login attempt. If live traffic does not match verification-code behaviour, Meta can mark the template as miscategorised and revoke approval — any volume uplift makes that audit path fire more aggressively.

Trade in plain terms: faster approval today for a messier UX and audit exposure later. Use Authentication only as a last resort if every Utility rewrite is exhausted.

**If the resubmit is still rejected**
- **Rejection notes `INCORRECT_CATEGORY` again** (second time): swap `कॉल रेफरेंस` → `कॉल आईडी` or `कॉल नंबर`, and remove the bold formatting on `{{2}}` so the classifier sees it as a plain identifier, not a code.
- **Rejection notes "Marketing content"**: remove the Footer `Wiom`.
- **Rejection notes "mixed language / script"**: replace `engineer` with `इंजीनियर` on the customer template (CSP template is already single-script).
- **Rejection notes anything else**: paste the exact rejection text here or ask the Message orchestrator team to open a Gupshup support ticket citing WABA `918880322222` and the template name.

Keep each resubmit as one minimal change — do not stack fixes, so the next rejection note stays unambiguous.

---

## WhatsApp component notes

- **Header, Footer, Body limits.** Header text ≤ 60 chars, Footer text ≤ 60 chars, Body text ≤ 1024 chars. Both drafts above fit well inside.
- **Variables.** Variables (`{{1}}`, `{{2}}`) live only in the Body. Header and Footer are fixed strings — "static-header, static-footer" is the easiest Utility template shape to get approved.
- **Body must not end with a variable.** The first rejection on 14 Oct hit this rule. The current body ends with "है।" after `{{2}}` — fix applied.
- **Bold.** WhatsApp renders `*text*` as **bold**. Used on the number and the PIN — the two things the recipient needs to read.
- **No Call-to-action button.** Gupshup's template builder does not allow a variable in the Call-phone-number button, and the masked DID is per-message (different `{{1}}` on every send). Button omitted. If a Gupshup build later supports variables in the Call button, add it back with Label **कॉल करें** · Phone `{{1}}`.

---

## One terminology trade-off to flag

The customer-side messages above use **"engineer"**. Wiom's Operational Terminology bans "engineer" and calls for **Technician** (section 3.1 of the terminology file). This is a conscious deviation on PM instruction (14 Oct 2026): customers in the Verma Parivar archetype recognise "engineer" from how BSNL, cable and other last-mile service providers describe their field staff; "Technician" reads as unfamiliar in a WhatsApp context. The deviation applies **only to customer-facing WhatsApp copy** — every internal doc, log, event and screen keeps "Technician".

If the WhatsApp Business review team flags "engineer" or asks for a canonical term, swap to **"team member"** (which the current push uses) rather than "Technician", to keep the warmth of the colloquial phrasing.

---

## Notes for the Message orchestrator / WhatsApp Business team

1. **Template category.** Utility (triggered by a specific user action — a missed call). Not Marketing. Not Authentication.
2. **Language.** Hindi only, per PM. Two templates — one customer-side, one CSP-side.
3. **No DLT required.** WhatsApp Business templates go through Meta's own approval, not the TRAI DLT pipeline. Expect approval within ~24 hours for Utility templates.
4. **Fallback.** If a recipient's WhatsApp is unreachable, the orchestrator should fall back to the SMS templates from the previous draft (now superseded) — or let the push notification stand on its own. Confirm which with the orchestrator team.
5. **G4 contract (unchanged from the signed-off PRD).** The CT event will never carry an empty PIN — if the PIN row is missing or deallocated at emit time, the event is suppressed upstream. The template can treat `{{2}}` as always populated.
6. **PIN field format.** `ivr_pin_registry.pin` is a character column; PINs today render as 4–6 digits. Variable placeholder works for either length.
7. **"Connection" on the CSP side.** Please keep "कनेक्शन" / "connection" on the CSP-side templates — it matches today's push and the Wiom Operational Terminology (CSPs have connections, never customers). Do not substitute "customer" during template submission.

---

## Change log

| Date | Change |
|---|---|
| 2026-10-07 | Initial SMS drafts (superseded). |
| 2026-10-14 | Channel changed to WhatsApp. Customer-side wording updated per PM: "आपने हमारे engineer का कॉल मिस कर दिया है" replaces "You missed a call from our team member". Added WhatsApp capabilities section (Call button, bold emphasis). Added terminology trade-off note for "engineer". |
| 2026-10-14 | Hindi only, per PM — English and bilingual alternatives removed. |
| 2026-10-14 | Restructured into WhatsApp template components (Header · Body · Footer · Button) per Meta's template submission shape. Header "वापस कॉल करें" + Footer "Wiom" added for both templates. Previews included. |
| 2026-10-14 | First submission rejected (template id 8904784). Body rewritten so it does not end with `{{2}}` — now ends with "डालें।" Call button removed: Gupshup's template builder does not allow a variable in the Call-phone-number button, and the masked DID is per-message. |
| 2026-10-14 | Added a Utility-approval submission checklist for Gupshup: category, name convention, sample values, use-case description, and a stepped fallback sequence for the next possible rejection notes (Authentication / Marketing / mixed-script). Each fallback is a single minimal change. |
| 2026-10-14 | Second submission rejected with `INCORRECT_CATEGORY`. Rewrote body: "PIN *{{2}}* डालें" → "आपका कॉल रेफरेंस *{{2}}* है"; Header "वापस कॉल करें" → "मिस्ड कॉल". The three remaining Authentication-triggering signals (PIN keyword, imperative "डालें", action-driven header) are all removed; the template now reads as a factual service notification with a reference identifier, the standard Utility shape. |
| 2026-10-14 | Added "Why we are not submitting under Authentication" section — four downsides (one-variable format, Copy-code button UX, different pricing / quality metrics, audit-revocation risk) with the plain-terms trade. Captured so this doesn't need to be re-derived next time someone asks "why not Authentication?" |
| 2026-10-15 | **CSP template approved by Meta** (id 8909340). The approved body uses a longer, conversational Utility shape — greeting + service context + softened miss framing + "कॉल के दौरान पूछे जाने पर {{2}} दबाएं" (press when asked during the call). The key structural shift is positioning `{{2}}` as a dialpad response to an IVR prompt rather than a code being delivered. The three earlier rejections failed because Meta's classifier reads "phone-number variable + short-code variable + enter instruction" as Authentication regardless of specific keywords. Customer template rewritten to mirror the approved CSP body, swapping `हमारी टीम` → `हमारे engineer`. Section added: "Why the approved shape clears INCORRECT_CATEGORY" — captures the six structural signals and how the approved shape dodges each. |
| 2026-10-15 | **CSP body actor-fix.** Approved template 8909340 misattributes the caller — it says *"हमारी टीम ने आपको कॉल किया था"* (our team called you), but on the CSP side the caller is the subscriber (connection), not Wiom's team. Reworded to *"आपके Wiom कनेक्शन ने आपको कॉल किया था"*, matching today's CSP push (*"आपने कनेक्शन का कॉल मिस कर दिया"*) and the Wiom Operational Terminology rule that CSPs have connections. Dropped "इंटरनेट" and "के संबंध में" so the connection reads naturally as the subject. Everything else is byte-for-byte the approved shape, so the Utility classifier should still clear. Template 8909340 to be retired once the re-submission is approved; customer template is unaffected (the engineer **is** the caller on the customer side). |
