Query the database and show me the details of 
the 2 failed contacts from the latest run:

SELECT o.contact_id, o.member_gap_key, o.channel, 
o.status, o.generated_message, g.member_key, 
m.language_preference, m.digital_literacy_segment,
p.preferred_channel, p.sms_allowed, p.email_allowed,
p.do_not_contact_flag
FROM fact_nba_outreach_plan o
JOIN fact_member_gap g ON o.member_gap_key = g.member_gap_key
JOIN dim_member m ON g.member_key = m.member_key
JOIN dim_member_channel_pref p ON m.member_key = p.member_key
WHERE o.status = 'FAILED'
ORDER BY o.created_timestamp DESC
LIMIT 5

Show me the full output.
