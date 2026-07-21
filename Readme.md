Implement a real agentic financial analysis loop 
using the OpenRouter API key already in .env.
This replaces ALL hardcoded closure rates, tier 
thresholds, ROI calculations, and benchmark 
comparisons with genuine Claude reasoning.

IMPORTANT: The API key in .env is named 
ANTHROPIC_API_KEY but contains an OpenRouter key.
Use OpenAI client pointed at OpenRouter as already 
configured in the codebase.

STEP 1 — Create financial_analysis_loop.py

This is a new file separate from agent_loop.py
specifically for computing financial projections
from whatever data is in the database.

from openai import OpenAI
from dotenv import load_dotenv
import sqlite3
import json
import os

load_dotenv()

client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=os.getenv('ANTHROPIC_API_KEY'),
    default_headers={
        "HTTP-Referer": "https://careintel.exl.com",
        "X-Title": "CareIntel Financial Analysis"
    }
)

DB_PATH = os.getenv('DB_PATH', 'careintel.db')

Define these tools for Claude to use:

TOOL 1: get_measure_data
Input: measure_key (optional), plan_key (optional)
Queries careintel.db and returns:
- measure details (name, star_weight, clinical_description)
- plan details (name, revenue, total_members, 
  current_stars, target_stars)
- Gap distribution: total gaps, open gaps, 
  closed gaps, borderline, partial
- Member profiles: distribution of 
  digital_literacy, language_preference,
  socioeconomic_segment, age_band
- Channel consent: what % have email, sms, 
  call allowed
- Propensity distribution: min, max, mean, 
  percentile distribution (25th, 50th, 75th)
- Historical performance: if any previous 
  campaigns exist for this measure x plan, 
  return actual closure rates from 
  fact_nba_outreach_plan joined with 
  fact_member_gap
- Prior year gap rate: % of gaps that were 
  also open last year

TOOL 2: get_national_benchmarks
Input: measure_key
Returns from measure_benchmarks table:
- national_avg_rate
- top_quartile_rate  
- bottom_quartile_rate
If not in table, Claude should use its 
training knowledge of HEDIS benchmarks.

TOOL 3: get_plan_population
Input: plan_key
Returns from plan_population table:
- total_members
- plan_revenue
If not in table returns null and Claude 
should estimate from context.

TOOL 4: write_financial_analysis
Input: Full analysis object with these fields:
{
  measure_key: str,
  plan_key: str,
  analysis_version: str,
  
  eligible_population: int,
  current_compliance_rate: float,
  national_benchmark: float,
  gap_to_benchmark: float,
  open_gaps_count: int,
  
  tier_1: {
    count: int,
    definition: str,
    closure_rate: float,
    closure_rate_rationale: str,
    cost_per_member: float,
    cost_rationale: str,
    expected_closures: int
  },
  tier_2: {same structure},
  tier_3: {same structure},
  
  expected_total_closures: int,
  stars_improvement: float,
  stars_improvement_rationale: str,
  cms_bonus_impact: float,
  total_outreach_cost: float,
  net_return: float,
  return_per_dollar: float,
  
  confidence_level: str,
  confidence_rationale: str,
  key_risks: [str],
  key_opportunities: [str],
  recommended_approach: str,
  plain_english_summary: str
}

Writes to financial_analyses table in database.
Returns the written record.

TOOL 5: get_intervention_complexity
Input: measure_key
Claude uses this to reason about how hard 
this intervention is. Returns data from 
fact_member_gap about:
- Average days_open (longer = harder to close)
- Previous year gap flag rate (chronic non-closers)
- Average clinical risk score
- Channel distribution of members with this gap
Claude will use this to reason about 
realistic closure rates.

CREATE financial_analyses TABLE:

CREATE TABLE IF NOT EXISTS financial_analyses (
    analysis_id TEXT PRIMARY KEY,
    measure_key TEXT,
    plan_key TEXT,
    source_id TEXT DEFAULT 'demo',
    analysis_version TEXT,
    
    eligible_population INTEGER,
    current_compliance_rate REAL,
    national_benchmark REAL,
    gap_to_benchmark REAL,
    open_gaps_count INTEGER,
    
    tier_1_count INTEGER,
    tier_1_definition TEXT,
    tier_1_closure_rate REAL,
    tier_1_closure_rationale TEXT,
    tier_1_cost_per_member REAL,
    tier_1_cost_rationale TEXT,
    tier_1_expected_closures INTEGER,
    
    tier_2_count INTEGER,
    tier_2_definition TEXT,
    tier_2_closure_rate REAL,
    tier_2_closure_rationale TEXT,
    tier_2_cost_per_member REAL,
    tier_2_expected_closures INTEGER,
    
    tier_3_count INTEGER,
    tier_3_definition TEXT,
    tier_3_closure_rate REAL,
    tier_3_closure_rationale TEXT,
    tier_3_cost_per_member REAL,
    tier_3_expected_closures INTEGER,
    
    expected_total_closures INTEGER,
    stars_improvement REAL,
    stars_improvement_rationale TEXT,
    cms_bonus_impact REAL,
    total_outreach_cost REAL,
    net_return REAL,
    return_per_dollar REAL,
    
    confidence_level TEXT,
    confidence_rationale TEXT,
    key_risks TEXT,
    key_opportunities TEXT,
    recommended_approach TEXT,
    plain_english_summary TEXT,
    
    created_timestamp TEXT,
    claude_model_used TEXT
)

MAIN ANALYSIS FUNCTION:

def analyze_opportunity(measure_key: str, 
                        plan_key: str,
                        source_id: str = 'demo') -> dict:

    system_prompt = """
    You are a healthcare analytics expert specializing 
    in Medicare Advantage Stars improvement programs.
    You have deep knowledge of HEDIS measures, member 
    outreach effectiveness, and CMS payment structures.
    
    Your job is to analyze a specific care gap 
    opportunity and produce realistic, defensible 
    financial projections that a VP of Quality at 
    a health plan would trust.
    
    CRITICAL RULES:
    - Never guess or fabricate data. Use only what 
      the tools return.
    - Base closure rates on the actual member profiles 
      you see in the data — digital literacy, language, 
      age, propensity scores, channel consent.
    - Consider intervention complexity — a medication 
      refill is not the same as a colonoscopy.
    - If historical outreach data exists for this 
      measure and plan, weight it heavily over 
      generic benchmarks.
    - Explain your reasoning for every number you 
      produce — rationale fields must be specific 
      not generic.
    - Return realistic numbers. A plan with mostly 
      low-literacy elderly Spanish-speaking members 
      will have different closure rates than a plan 
      with young digitally-engaged members even for 
      the same measure.
    - Stars improvement formula:
      (expected_closures / eligible_population) 
      x star_weight x 0.5
      Cap at star_weight x 0.10 per campaign.
    - CMS bonus: stars_improvement x plan_revenue x 0.05
    - Cost per member must reflect actual channel 
      costs — email $1.50, SMS $0.50 + incentive, 
      call $8.00 + incentive. Do not use flat $2/$17/$45 
      unless the member mix justifies it.
    - Confidence level must be based on actual 
      data quality — eligible population size, 
      whether historical data exists, how representative 
      the synthetic data is.
    """
    
    messages = [{
        "role": "user",
        "content": f"""
        Analyze the gap closure opportunity for 
        measure {measure_key} on plan {plan_key}.
        
        Step 1: Get the measure and plan data 
        using get_measure_data tool.
        
        Step 2: Get national benchmarks for 
        this measure using get_national_benchmarks.
        
        Step 3: Get plan population data using 
        get_plan_population.
        
        Step 4: Understand intervention complexity 
        using get_intervention_complexity.
        
        Step 5: Based on everything you have seen, 
        produce a complete financial analysis.
        
        For tier definitions — look at the actual 
        member data you retrieved. Define tiers 
        based on what the data shows, not generic 
        cutoffs. For example if you see that 
        members with propensity above 0.65 AND 
        email consent AND high digital literacy 
        make up a distinct group, define Tier 1 
        as that group specifically.
        
        For closure rates — reason from the data.
        Consider:
        - How complex is this intervention?
        - What is the member profile of each tier?
        - Does historical data exist? If yes, 
          use those actual rates.
        - What channel mix is available for each tier?
        - What is the prior year gap rate? 
          High prior year rate means chronic 
          non-closers — lower your closure estimate.
        
        Write your complete analysis using 
        write_financial_analysis tool.
        
        In plain_english_summary write 3-4 sentences 
        a PM at Aetna would understand — what the 
        opportunity is, who to target, what it costs, 
        what they get back. Be specific with numbers.
        """
    }]
    
    Run the agentic loop until Claude calls 
    write_financial_analysis and returns.
    Return the written analysis.

def analyze_all_opportunities(source_id: str = 'demo'):
    
    Get all unique measure x plan combinations 
    from fact_member_gap for this source_id.
    
    For each combination call analyze_opportunity.
    
    Return list of all analyses.

STEP 2 — Add API endpoints

POST /financial/analyze/{measure_key}/{plan_key}
Triggers analyze_opportunity for this combination.
Returns the analysis result.
If analysis already exists and is less than 
24 hours old, return cached result.
If older than 24 hours or force=true parameter, 
re-run analysis.

POST /financial/analyze-all
Triggers analyze_all_opportunities in background.
Returns immediately with job_id.
Analysis runs asynchronously.

GET /financial/analyze-all/status
Returns how many analyses are complete vs pending.

GET /financial/analysis/{measure_key}/{plan_key}
Returns the most recent analysis for this combination.

GET /financial/analyses
Returns all analyses sorted by net_return descending.
This is what the Opportunities tab reads.

STEP 3 — Update GET /opportunities to use 
financial_analyses table

Instead of computing financial projections 
in Python on the fly, the opportunities 
endpoint should:

1. Check if financial_analyses has a recent 
   result (less than 24 hours) for this 
   measure x plan combination.
2. If yes: return the Claude-generated analysis.
3. If no: run analyze_opportunity on the fly, 
   store result, return it.

This means first load may be slower (Claude 
is analyzing each opportunity) but subsequent 
loads are instant from cache.

Show the PM that Claude is analyzing:
"AI is analyzing your opportunities... 
Analyzing EED × Aetna Premier... (3 of 14)"

STEP 4 — Update dashboard display

In the opportunity cards show Claude's 
actual reasoning not generic labels:

Instead of:
"Tier 1: High propensity, digital only, 60%"

Show what Claude actually determined:
"Tier 1: Members with propensity > 0.71, 
email consent, and high digital literacy 
(312 members). Eye exam requires specialist 
scheduling — 43% expected closure based on 
member engagement profile and low prior-year 
repeat rate."

Show Claude's plain_english_summary as 
the card description.

Show confidence with Claude's actual rationale:
"Medium confidence — 4,560 estimated eligible 
members, no historical outreach data for this 
measure on this plan. Using member profile 
analysis and HEDIS benchmarks."

Add a small "AI Analysis" badge on each card 
showing the Claude model used and timestamp.

Add an expandable "View Full Reasoning" section 
that shows Claude's complete rationale for 
tier definitions, closure rates, and 
Stars improvement calculation.

STEP 5 — Handle new data automatically

When a PM uploads new data via the Data Sources tab 
and activates it, automatically trigger 
POST /financial/analyze-all for the new source_id.

Show a banner: "AI is analyzing your data... 
This takes 2-3 minutes for a full portfolio analysis."

When complete refresh the Opportunities tab 
with Claude-generated projections for the 
uploaded data.

This means when a real PM at Aetna uploads 
their actual member and gap data, Claude reads 
their specific member profiles — age distribution, 
language mix, digital literacy, channel consent, 
historical response rates if available — and 
generates projections tailored to their 
actual population.

STEP 6 — Test with our existing data

After building run POST /financial/analyze-all
for source_id = 'demo'

Show me Claude's actual analysis output for:
1. MAD on P004 — easy measure, should show 
   high closure rates with Claude's reasoning
2. COL on P001 — hard measure, should show 
   low closure rates with reasoning
3. EED on P002 — medium, should show 
   medium rates

The three analyses should look genuinely 
different because Claude is reading different 
data for each one.

Show me the plain_english_summary Claude 
generates for each of these three opportunities.

These summaries should NOT look like 
templates — they should reflect what 
Claude actually found in the data.


Also the render link gives error saying make sure 8000 is running, and its not working. fix that too in the latest push 
