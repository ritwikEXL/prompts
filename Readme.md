Fix the following, in order, and show me updated numbers after each before moving to the next:

1. BUG: The "Max CMS Bonus" summary stat (currently showing $1.2M) does not
   equal the sum of "CMS bonus (all closed)" across all 35 Financial
   Opportunity Analysis cards -- summing even 5 of the 35 cards already
   exceeds $1.2M. Find wherever "Max CMS Bonus" is computed and make it
   equal the actual sum of the all-gaps-closed bonus value across all 35
   cards, not a separately derived or cached figure.

2. BUG: "Expected Stars improvement per plan" in the Q3 Outreach Budget
   Optimizer shows an identical +0.300 for every single plan (MA-PD Choice,
   MA-PD Premier, MA Value, MA-PD Signature, DSNP Community) regardless of
   budget level or each plan's distinct gap profile and funded campaigns.
   This should show real variance per plan based on which of that plan's
   campaigns are actually funded at the current budget -- find why every
   plan is saturating at the same value and fix it to be data-driven.

3. CLARIFY/FIX: The Stars Forecast panel (Do Nothing / Digital First / Full
   Campaign) for a selected plan does not change when the portfolio-wide
   budget slider in the Q3 Outreach Budget Optimizer is moved. Confirm
   whether this panel is intentionally independent (a standalone per-plan
   simulator) or should be reading from the same funded-budget state as the
   optimizer. Whichever it is, make it explicit in the UI -- e.g. label it
   "Independent per-plan scenario, not tied to portfolio budget above" if
   that's the intended design, or wire it to the shared budget state if not.

4. LABELING: Add explicit scope labels next to every dollar and Stars
   figure on the dashboard so scope is never ambiguous -- e.g. "$123K
   needed (portfolio-wide, all 35 opportunities)" vs "$12K outreach cost
   (this plan only, all measures)" vs "$887 cost (this measure x plan
   combo only)". Right now multiple dollar figures at different scopes sit
   next to each other with no indication of what each one covers, which
   reads as contradictory even when the numbers are individually correct.

5. FEATURE: Add the ability to directly apply/run a selected Stars Forecast
   scenario (Do Nothing / Digital First / Full Campaign) for the chosen
   plan, rather than requiring the user to scroll down and act on each of
   the 35 individual opportunity cards separately.

6. FEATURE: Add a portfolio-level "Realistic Achievable Outcomes" summary
   showing, for the currently funded budget: (a) total probable member
   closures this quarter -- sum of "Expected closures" across all currently
   funded cards, driven by the real tier-propensity closure rates already
   used per card, and (b) total probable net profit -- sum of "Net return"
   across all currently funded cards. This should sit near the top of the
   Opportunities view so a PM sees one clear "here's what will realistically
   happen at this budget" figure without manually adding up 35 cards.

7. REDESIGN: The AI Opportunity Analysis panel (now running on the real
   agentic loop against the Anthropic API) is currently a small widget at
   the very top with only 3 ranked opportunities shown. Elevate it to be
   the primary entry point of the Opportunities view -- make it the lead
   experience a PM sees first, and make each recommendation's reasoning
   expandable so the underlying trace/audit trail is visible on demand.
   Do NOT delete or bury the detailed Financial Opportunity Analysis cards
   underneath -- keep them as the supporting "show your work" layer the AI
   recommendations are drawn from, since a PM will want to verify the AI's
   numbers against the real per-card data, not just trust a black box.

After each fix, show me the updated numbers so I can confirm before you
move to the next one.
