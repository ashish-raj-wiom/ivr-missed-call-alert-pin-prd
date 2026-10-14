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

## Customer WhatsApp — sent when the customer missed the call

> आपने हमारे engineer का कॉल मिस कर दिया है।
>
> वापस कॉल करने के लिए इस नंबर पर कॉल करें: *{{1}}*
> PIN: *{{2}}*
>
> — Wiom

---

## CSP WhatsApp — sent when the CSP missed the call

> आपने कनेक्शन का कॉल मिस कर दिया है।
>
> वापस कॉल करने के लिए इस नंबर पर कॉल करें: *{{1}}*
> PIN: *{{2}}*
>
> — Wiom

---

## WhatsApp-specific capabilities worth using

Two things WhatsApp gives us that SMS did not. Both are optional for day-one launch but worth asking the Message orchestrator team to turn on:

1. **"Call phone number" button.** WhatsApp Utility templates can carry a CTA button that opens the dialer pre-filled with the masked calling DID. One tap, no number copy-paste. Button label: **"कॉल करें"** / **"Call Wiom"**. Behaviour: dials `{{1}}` — the same masked DID in the body. The callee still needs to enter the PIN on the IVR prompt, but the number-dialling friction is gone.
2. **Bold emphasis on the PIN.** WhatsApp's `*text*` renders as bold. The drafts above already use it on the number and the PIN. This is the main readability win over SMS; keep it.

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
