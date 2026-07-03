3 failed contacts total. Here's the breakdown:

Record 1 — CT_162441_004

Member: MBR0008 · EN · High digital literacy · preferred EMAIL
Gap: G00055 · BCS · Channel assigned: SMS
Message generated: ✓ (Hi MBR0008, reminder: complete your BCS…)
Root cause: Twilio SMS delivery failed — likely the same SSL cert issue or a Twilio API error. Note the member prefers EMAIL but was assigned SMS (possible cohort channel mismatch).
Record 2 — CT_162441_006

Member: MBR0013 · EN · High digital literacy · preferred EMAIL
Gap: G00058 · BCS · Channel assigned: SMS
Message generated: ✓ (same BCS template)
Root cause: Same as Record 1. Again, preferred channel is EMAIL but routed via SMS.
Record 3 — CT_043822_001

Member: MBR0004 · EN · High digital literacy · preferred EMAIL
Gap: G00024 · COL · Channel: EMAIL
Message generated: ✓ (full email template)
Root cause: This is the original SSL cert failure from an earlier session (before the SSL fix was applied). Now resolved.
Two actionable findings:

Records 1 & 2 — Both are High digital literacy, EMAIL-preferred members being routed to SMS by the cohort logic. The C1_DIGITAL_HIGH_PROP branch prefers EMAIL first (chan.email_allowed==='true'?'EMAIL':...) so something in the campaign config overrode this. Worth checking whether the campaign's fallback channel was set to SMS and why it landed on these two members.
Record 3 — Historical pre-fix failure, no action needed. The same contact was re-sent successfully in a later run.
Want me to retry the two SMS failures, or investigate why those EMAIL-preferred members were assigned SMS?
