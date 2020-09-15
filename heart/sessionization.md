---
description: >-
  Разделение активности пользователя на периоды действия, так что в одном
  периоде расстояние между событиями меньше получаса/часа
---

# Sessionization

В кликзаусе вот так:

```text
SELECT
    user_id,
    created_at,
    event_type,
    runningAccumulate(s, user_id) AS session_id
FROM
(
    SELECT
        user_id,
        created_at,
        sumState(is_new) AS s,
        event_type
    FROM
    (
        SELECT
            created_at ,
            runningDifference(created_at) as rd,
            (rd > 1800) OR (rd < 0) AS is_new,
            user_id,
            event_type
        FROM
			(
			    SELECT
			        user_id,
			        created_at,
			        event_type
			    FROM uchi.page_events pe 
			    WHERE ptn_date = yesterday()
			    	and event_source = 'magic_math'
				ORDER BY
		            user_id ASC,
		            created_at ASC
			)
    )
    GROUP BY
        user_id,
        created_at,
        event_type
    ORDER BY
        user_id ASC,
        created_at ASC,
        event_type ASC
)
```

