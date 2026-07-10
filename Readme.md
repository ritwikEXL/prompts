I'm working on CareIntel, a Medicare Stars Next Best Action platform.
Project folder: C:\Users\vmuser\Documents\NBA_Claude_Cli
Stack: dashboard/index.html (frontend), api.py (FastAPI backend, port 8000),
careintel.db (SQLite).

CONTEXT: The previous fixes to cohort scaling, ROI tiering, and per-card
CMS bonus calculation (in the Financial Opportunity Analysis section) are
working correctly and should NOT be touched — that section shows its math
transparently (e.g. "$887 cost · $201K bonus · 1,500 plan members @
$1050/mo PMPM") and the numbers are internally consistent there.

THE BUG: There are three places in the dashboard showing CMS bonus /
Stars improvement numbers that are NOT using that same validated
per-card calculation, and as a result are producing numbers that
contradict each other and are implausibly large:

1. Q3 Outreach Budget Optimizer top-line "Expected CMS Bonus" ($22.6M at
   a $55K partial-funding budget level) EXCEEDS the dashboard's own
   "MAX CMS BONUS" ceiling stat ($5.0M, labeled as the bonus if ALL gaps
   across ALL 35 measure-plan combinations were closed). It is
   mathematically impossible for a partially-funded subset of campaigns
   to produce more bonus than closing every single gap in the entire
   book. Find wherever "Expected CMS Bonus" is computed for the budget
   optimizer and make it sum from the same per-card bonus values used
   in the Financial Opportunity Analysis cards, only for cards that are
   actually funded at the current budget level — it should never be
   able to exceed "Max CMS Bonus."

2. The Budget Optimizer's "Expected Stars improvement per plan" section
   shows figures like MA-PD Signature +1.155 stars at a $55K quarterly,
   PARTIALLY-funded budget. But the separate "Stars Forecast — End of
   Measurement Year" panel, for the SAME plan (MA-PD Signature), shows
   "Full Campaign" (fully funded, all tiers, all measures, full year)
   maxing out at only +0.300 stars. A partial, quarterly scenario should
   never show ~4x the Stars gain of a full, annual, fully-funded scenario
   for the same plan. Find both calculations and reconcile them to use
   one shared, capped formula — the per-plan Stars gain shown anywhere
   in the app should be consistent regardless of which panel is
   displaying it, and should respect the same star_weight-based cap
   logic used elsewhere.

3. The "Stars Forecast" panel's three scenarios (Do Nothing / Digital
   First / Full Campaign) show cost figures that ARE internally
   consistent with the small per-tier costs seen in the Financial
   Opportunity Analysis cards (e.g. Digital First at $49 for a plan
   where individual measure cards show $2/member digital costs), but the
   BONUS figures for those scenarios are wildly inflated and disconnected:
   - Digital First: $49 cost -> $1.4M bonus implies a ~28,500x return,
     far beyond the ~200x ROI ceiling seen on the single best-performing
     card elsewhere in the dashboard for this same plan.
   - Full Campaign: $12K cost -> $2.7M bonus, on a plan with roughly
     500 members at $920/mo PMPM (~$5.52M annual plan revenue elsewhere
     in the dashboard), implies the bonus alone is ~49% of the plan's
     entire annual revenue, which is not a plausible CMS quality bonus
     payment scale (these are typically a modest single or low-double-digit
     percentage uplift over benchmark payments, not an amount comparable
     to half of total plan revenue).
   Rework the Stars Forecast panel's bonus calculation to derive from the
   same PMPM-based methodology as the Financial Opportunity Analysis
   cards, scaled to however many measures/gaps are included in each
   scenario (Digital First = T1-only gaps across all measures for this
   plan, Full Campaign = all tiers, all measures for this plan). The
   result should be directly reconcilable against summing the relevant
   individual Financial Opportunity Analysis cards for that same plan.

After the fix, show me: 
(a) the Budget Optimizer's Expected CMS Bonus at a couple of different
budget levels, confirmed to never exceed Max CMS Bonus,
(b) the per-plan Stars improvement figure from both the Budget Optimizer
and the Stars Forecast panel for the same plan, confirmed to match or be
clearly reconcilable, 
(c) the three Stars Forecast scenario numbers for one plan, with the
underlying PMPM math shown (cost, bonus, member count, PMPM rate) the
same way the Financial Opportunity Analysis cards already do.

Do NOT touch: the per-card Financial Opportunity Analysis calculations
(these are correct and validated), Run Session tab, Evaluation tab,
WhatsApp/Twilio, SendGrid, or agent_loop.py.
