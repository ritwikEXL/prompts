Fix	Status	Detail
FIX 1 — Open gaps / members at risk	✅ Done	DB-backed distinct counts: P005=762 gaps/420 members, P004=2006/1295
FIX 2 — Below benchmark count	⚠️ Data issue	Query is correct per-plan; synthetic data has all 5 plans 10-30% below national averages regardless of star rating. The 3% threshold didn't help because gaps are all >3%. Proper fix requires re-seeding fact_member_gap with star-correlated compliance rates — I can do that if you want
FIX 3 — Default to lowest-stars plan	✅ Done	Always defaults to P005 (2.5★); resets if stored plan is invalid
FIX 4 — Launch Campaign → Run Session	✅ Done	Switches tab, shows pre-selection banner, auto-starts Phase 1
FIX 5 — Scope labels on all financials	✅ Done	"(this measure only, realistic estimate)", "(full portfolio, all funded campaigns)", "$166 return per $1 spent (estimated, based on industry-average response rates)", etc.
FIX 6 — ROI varies by Stars rating	✅ Done	P005 at 2.5★: upside factor = 1.00×; P001 at 3.5★: 0.60×; P004 at 4.5★: 0.20× — shown in confidence column and applied to all bonus calculations
Say the word if you want me to reseed the compliance data to fix FIX 2.
