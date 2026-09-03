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

## 2. Agents with Above-Average Reopen Rates

```sql
WITH agent_rates AS (
    SELECT
        a.agent_id,
        a.name,
        a.team,
        SUM(CASE WHEN t.reopened_count > 0 THEN 1 ELSE 0 END) * 1.0 / COUNT(*) AS reopen_rate
    FROM agents a
    JOIN tickets t
        ON t.agent_id = a.agent_id
    GROUP BY a.agent_id, a.name, a.team
)
, rates_with_team_avg AS (
    SELECT
        agent_id,
        name,
        team,
        reopen_rate,
        AVG(reopen_rate) OVER (PARTITION BY team) AS team_avg_reopen_rate
    FROM agent_rates
)
SELECT
    agent_id,
    name,
    team,
    reopen_rate,
    team_avg_reopen_rate
FROM rates_with_team_avg
WHERE reopen_rate > team_avg_reopen_rate
ORDER BY team, reopen_rate DESC;
```

## 3. CSAT Trend by Category
```sql
SELECT
    DATE_TRUNC('month', c.submitted_at) AS month,
    t.category,
    AVG(c.score) AS avg_csat
FROM tickets t
JOIN csat_responses c
    ON c.ticket_id = t.ticket_id
WHERE c.submitted_at >= CURRENT_DATE - INTERVAL '3 months'
GROUP BY
    DATE_TRUNC('month', c.submitted_at),
    t.category
ORDER BY month, t.category;
```

