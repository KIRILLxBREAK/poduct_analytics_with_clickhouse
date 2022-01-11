---
description: >-
  Разделение активности пользователя на периоды действия, так что в одном
  периоде расстояние между событиями меньше получаса/часа
---

# Sessionization

В кликзаусе вот так:

```
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

Другой пример, с использованием массивов:

```
select
	round( sum(sessions_sum) / sum(sessions_count), 2) as avg_session_duration,
	round( avg(avg_sessions), 2) as normalising_avg_session_duration
FROM 
(
	select 
		user_id,
		arrayReduce('sum', sessions_time) as sessions_sum,
		arrayReduce('count', sessions_time) as sessions_count,
		arrayReduce('avg', sessions_time) as avg_sessions
	FROM
    (
			    SELECT
			        user_id,
			        groupArray((created_at,event_type)) as events,
			        arraySort(x -> x.1, events) as sort_events,
			        sort_events.1 as creates,
			        arrayMap(x -> assumeNotNull(toUnixTimestamp(x)), sort_events.1) as times,
					--arrayCumSum(if(x>1800,1,0), arrayDifference( times ) ) as session_id
					arraySplit((x,y) -> y > 1800, times, arrayDifference( times )) as tmp,
					arrayMap(x -> arrayReduce('max', x), tmp) as arr_max,
					arrayMap(x -> arrayReduce('min', x), tmp) as arr_min,
					arrayMap((x,y) -> x-y, arr_max, arr_min) as sessions_time
			    FROM uchi.page_events pe 
			    WHERE ptn_date >= '2021-01-18'
			    	and event_source = 'magic_math_paid_6_5'
				GROUP BY user_id
    ) users
	--where user_id = 24286460
) s
```
