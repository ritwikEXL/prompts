Redesign the Opportunities tab in dashboard/index.html 
completely from scratch. Keep all existing API calls and 
data sources but replace the entire layout with a clean 
product manager focused flow.

The new layout has 5 sections in this exact order.
Remove the existing budget optimizer, benchmark chart, 
Stars forecast, and financial cards layout entirely.
Replace with the following:

SECTION 1 — YOUR STARS SITUATION
This is the first thing the PM sees when they open 
the tab. One clean banner card with dark teal background.

Left side shows:
- Plan selector dropdown (default to lowest Stars plan)
- Current Stars rating shown large: "2.5★ Current"
- Target Stars rating: "3.0★ Target"  
- Gap label: "0.5★ to next tier"
- CMS bonus at risk label: "This gap costs you 
  $9M/year in CMS bonus payments"
  (calculated as: Stars gap x plan_revenue x 0.05)
- Measurement deadline: "147 days left in 
  measurement year" (calculate from today to 
  Dec 31 2026)

Right side shows 3 stat boxes:
- Open gaps: total open gap count for this plan
- Members at risk: unique members with open gaps
- Measures below benchmark: count of measures 
  where this plan is below national average

This section answers: WHERE AM I LOSING MONEY?

SECTION 2 — TOP OPPORTUNITIES FOR THIS PLAN
Below the banner, show the top 3 opportunities 
for the selected plan ranked by net return.

Each opportunity is a clean card with:

Left column:
- Measure name and code badge
- "X members have this gap open"
- Progress bar: plan compliance vs national benchmark
  e.g. "52% compliant vs 74% national average"
- Gap to close in plain English:
  "Close 18 more gaps to reach benchmark"

Middle column — REALISTIC ACHIEVABLE OUTCOMES:
This is the most important section.
Title: "What you can realistically achieve"

Show three member tiers with plain English labels:

LIKELY TO ACT (Tier 1)
These members have high propensity scores above 0.70
and high digital literacy. They just need a reminder.
Show: X members | Digital outreach only | $2/member
Expected: Y members will complete (60% rate)
Cost: $Z

PROBABLY WILL ACT (Tier 2)  
These members are reachable but need an incentive.
Show: X members | SMS + $15 gift card | $17/member
Expected: Y members will complete (35% rate)
Cost: $Z

NEED HIGH TOUCH (Tier 3)
These members have barriers — language, access, 
low digital literacy. Worth targeting if budget allows.
Show: X members | Call + $25 voucher | $45/member
Expected: Y members will complete (18% rate)
Cost: $Z

Below the tiers show:
"If you target Tier 1 + Tier 2 only:"
- Members contacted: X
- Expected completions: Y
- Total cost: $Z
- Stars improvement: +0.XX★
- CMS bonus gained: $XM
- Net return: $XM
- Highlighted in green: "Return on every $1 spent: $XX"

Right column — CONFIDENCE INDICATOR:
Show a simple confidence meter:

HIGH CONFIDENCE if:
- More than 20 eligible members
- Historical outreach data exists for this measure
- Plan compliance above 30%

MEDIUM CONFIDENCE if:
- 10 to 20 eligible members
- No historical data, using industry benchmarks

LOW CONFIDENCE if:
- Fewer than 10 eligible members
- Strongly recommend verifying with full plan data

Below confidence show:
"These projections are based on [synthetic demo data /
your uploaded data]. Connect real claims data for 
precise figures."

At bottom of each card:
A large teal button: "Launch This Campaign →"
That calls the existing session start flow.

SECTION 3 — PORTFOLIO VIEW (collapsed by default)
A collapsible section titled "See all opportunities 
for this plan"

When expanded shows a simple clean table with columns:
Measure | Open Gaps | Members | Est. Completions | 
Est. Cost | Est. Bonus | Confidence

No financial cards. No tier breakdowns. Just the table.
Sorted by Net Return descending.

At the bottom of the table:
"Select All" checkbox and "Launch Selected Campaigns"
button that starts a multi-campaign session.

SECTION 4 — PORTFOLIO BUDGET PLANNER
A simple clean section titled 
"Q3 Budget Planner — How to spend your outreach budget"

Show a budget input field (not a slider — a proper 
dollar input field with preset buttons: 
$50K, $100K, $250K, $500K, $1M, $2M)

When budget is entered or preset clicked show:

A simple two column layout:

Left: WHAT TO DO WITH THIS BUDGET
Ranked list of campaigns to fund:
1. [Measure] x [Plan] — $X cost — $XM return — Fund ✓
2. [Measure] x [Plan] — $X cost — $XM return — Fund ✓
3. [Measure] x [Plan] — $X cost — $XM return — Partial ◑
etc.

Right: WHAT YOU GET BACK
- Total campaigns funded: X of Y
- Total members contacted: X
- Realistic completions: X (not optimistic — 
  use weighted average of tier closure rates)
- Portfolio Stars improvement: +X.XX★ 
  (capped at 0.25 per plan per quarter)
- Total CMS bonus gained: $XM
- Total outreach cost: $XM  
- Net return: $XM
- Note: "Projections assume industry-average 
  response rates. Actual results depend on 
  your member population."

SECTION 5 — AI ANALYSIS (if API key is available)
At the very bottom, a collapsible panel titled
"AI Opportunity Analysis — powered by Claude"

When expanded shows the agentic loop analysis output
including Claude's reasoning about which opportunities
to prioritize and why.

If API key is not available show:
"AI analysis requires Anthropic API access. 
Contact your administrator to enable."

GENERAL DESIGN RULES:
- Remove all 999x ROI references entirely
- Remove all references to composite scores
- Never show a Stars improvement above 0.3 per plan
- Never show a CMS bonus above plan_revenue x 0.02 
  per single campaign
- All dollar figures must have scope labels:
  "(this measure only)" or "(this plan, all measures)"
  or "(full portfolio)"
- Use plain English everywhere — no jargon like 
  nba_propensity_score or T1/T2/T3 — use 
  Likely to Act / Probably Will Act / Need High Touch
- Every number that is a projection must have a 
  small gray label: "estimated" or "projected"
- Confidence indicators on every financial projection

Also fix these specific bugs while rebuilding:
- Max CMS Bonus summary stat must equal the actual 
  sum of per-opportunity CMS bonus values
- Stars forecast must either react to budget changes
  or be clearly labeled as independent of budget
- All scope labels must be consistent throughout

After rebuilding show me Section 1 and Section 2 
for Aurora MA-PD Signature (P005, 2.5 stars) with 
actual numbers from the database so I can verify 
they are correct before checking the rest.
