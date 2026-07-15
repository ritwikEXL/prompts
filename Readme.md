Two fixes needed:

FIX A — Gap and member counts are still wrong

P005 is showing 762 open gaps and 420 members 
at risk. P004 is showing 2006 gaps and 1295 members.
These are impossible — the entire fact_member_gap 
table has only 250 rows total across all 5 plans.

Debug this by running these exact SQL queries 
directly against careintel.db and show me the 
raw results:

SELECT plan_key, COUNT(*) as total_gap_rows
FROM fact_member_gap
GROUP BY plan_key
ORDER BY plan_key

SELECT plan_key, 
COUNT(*) as open_gaps,
COUNT(DISTINCT member_key) as unique_members
FROM fact_member_gap
WHERE gap_status IN ('Open', 'Borderline', 'Partial')
AND is_suppressed = 0
GROUP BY plan_key

SELECT COUNT(*) as total_rows FROM fact_member_gap

Show me the raw numbers from these queries 
before fixing anything. Then fix the API endpoint 
that serves gap counts to use these exact queries 
and nothing else.

FIX B — Reseed fact_member_gap with star-correlated 
compliance rates

Update gap_status values in fact_member_gap so that 
each plan's compliance rate correlates with its 
Stars rating. Do this by updating gap_status to 
Closed for a proportion of gaps per plan:

P005 (2.5 stars) — close 38% of its gaps
P003 (3.0 stars) — close 50% of its gaps  
P001 (3.5 stars) — close 60% of its gaps
P002 (4.0 stars) — close 68% of its gaps
P004 (4.5 stars) — close 73% of its gaps

For each plan randomly select that proportion 
of gap rows and update gap_status to Closed.
Keep the rest as their current status mix 
of Open, Borderline, and Partial.

After reseeding show me:
SELECT plan_key, 
gap_status, 
COUNT(*) as count
FROM fact_member_gap
GROUP BY plan_key, gap_status
ORDER BY plan_key, gap_status

And confirm the compliance rates now make sense 
relative to each plan's Stars rating.

Also confirm the measures below benchmark count 
now shows correctly — P004 at 4.5 stars should 
show 1 or 2 measures below benchmark, not 7.
