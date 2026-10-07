# Missed Call Alert — SMS content

Companion to `IVR_Missed_Call_Alert_PIN_PRD_v2.md` (v2.0 signed off 29 Sep 2026). Hand this to the Message orchestrator / DLT team for template registration.

**Context.** The feature goes live **7 Oct 2026**. Push notifications already fire on a missed IVR call to both the customer and the CSP. Now the CT event payload also carries the masked calling DID and the callee's own PIN, so an SMS can be sent in parallel that gives the callee a one-tap callback — number + PIN.

The push notifications live today are the anchor for tone and language mix:

| Side | Current push (today) |
|---|---|
| **Customer** | *You missed a call from our team member.*<br>*Please call back.*<br>*आपने हमारे टीम का कॉल मिस कर दिया है।*<br>*कृपया उनसे बात करने के लिए वापस कॉल करें।* |
| **CSP** | *आपने कनेक्शन का कॉल मिस कर दिया।*<br>*कृपया वापस कॉल करें।* |

The SMS below mirrors that style, plus the callback number and the PIN.

**Placeholders used in the drafts** — these map to the fields the SMS campaign binds from the CT event payload (`IVR_Missed_Call_Alert_PIN_PRD_v2.md` §3 R1):

| Placeholder | Source field | Example |
|---|---|---|
| `{masked_number}` | the masked calling DID the callee dials to reach IVR | `+918046161110` |
| `{customer_pin}` | `customer_pin` on `dnp_customer_missed_call_alert` | `234491` |
| `{csp_pin}` | `csp_pin` on `call_dnp_missed_call_alert` | `087275` |

---

## Customer SMS — sent when the customer missed the call

### Option 1 — English only *(1 SMS segment, cleanest; closest to the CEO sample)*

> You missed a call from Wiom. Call back on **{masked_number}**, PIN: **{customer_pin}**. — Wiom

### Option 2 — Bilingual, short *(matches current push style; ~2 segments)*

> You missed a Wiom call. Call back on **{masked_number}**, PIN: **{customer_pin}**.
>
> आपने Wiom का कॉल मिस किया। **{masked_number}** पर कॉल करें, PIN **{customer_pin}** डालें।

---

## CSP SMS — sent when the CSP missed the call

### Option 1 — English only *(1 SMS segment)*

> You missed a call from your connection. Call back on **{masked_number}**, PIN: **{csp_pin}**. — Wiom

### Option 2 — Bilingual, short *(~2 segments)*

> Missed call from your connection. Call back on **{masked_number}**, PIN: **{csp_pin}**.
>
> आपके कनेक्शन का कॉल मिस। **{masked_number}** पर कॉल करें, PIN **{csp_pin}** डालें।

### Option 3 — Hindi only *(1 segment, matches the current CSP push exactly)*

> आपने कनेक्शन का कॉल मिस किया। वापस कॉल करें **{masked_number}**, PIN **{csp_pin}**।

---

## Recommended pair for launch

- **Customer: Option 2 (Bilingual).** The current push is bilingual; keeping SMS bilingual means the Verma Parivar archetype (first-time Wi-Fi users in non-metro India) sees the Hindi half.
- **CSP: Option 3 (Hindi only).** The current CSP push is Hindi only — mirroring that keeps the SMS to a single segment and matches the CSP app's Hindi-first copy.

---

## Notes for the DLT / Message orchestrator team

1. **DLT templates.** Each SMS variant needs its own DLT template registration. English-only variants (Option 1 for both) register faster if the goal is to launch today and iterate on bilingual later.
2. **Sender header.** Wiom's 6-char DLT header appears above the body; the "— Wiom" sign-off in the English drafts is optional and can be dropped to save characters once the header is confirmed.
3. **Terminology — CSP side.** The CSP-side SMS uses **"connection"**, not "customer". That matches both the current push notification wording and the Wiom Operational Terminology rule that **CSPs have connections, never customers** (section 5.1 of the terminology file). Please do not substitute "customer" during template registration.
4. **PIN field format.** `ivr_pin_registry.pin` is a character column; today's PINs render as 4–6 digits. The template placeholder works for either length. If a format change is planned, flag it before registration.
5. **Suppression contract (G4).** The PRD guarantees the SMS campaign will never see an event with an empty PIN — if the PIN row is missing or deallocated at emit time, the CT event is suppressed upstream rather than fired with a blank PIN. So the template can treat `{customer_pin}` / `{csp_pin}` as always populated.

---

## Change log

| Date | Change |
|---|---|
| 2026-10-07 | Initial drafts. Live launch same day. |
