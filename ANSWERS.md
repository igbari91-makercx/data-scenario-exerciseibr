# Support Data Scenario — Answers

## 1. First Response Time by Team

```sql
SELECT
    a.team,
    AVG(t.first_response_minutes) AS avg_first_response_minutes
FROM tickets t
JOIN agents a
    ON t.agent_id = a.agent_id
WHERE t.status = 'closed'
  AND t.closed_at >= CURRENT_TIMESTAMP - INTERVAL '30 days'
GROUP BY a.team
ORDER BY avg_first_response_minutes;
```
