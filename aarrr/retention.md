---
description: Разные виды
---

# Retention

Retention N-дня

```text
SELECT distinct user_id 
	from
		(
									 SELECT user_id, min(ptn_date) as min_date
									 FROM uchi.page_events pe
									 where event_source = 'magic_math'
										and toDate(created_at) >= toDate('2020-09-02')
									 GROUP BY user_id
		) t1
		join
		(
		select
			user_id, 
			ptn_date
		from uchi.page_events t
		where event_source = 'magic_math' 
			and ptn_date >= toDate('2020-09-02')
		group by user_id, ptn_date 
		) t2
	using (user_id)
where ptn_date  =  (min_date + 2)
```

или так:

```text
select
	count(user_id)
from 
(
	SELECT 
		user_id,
		count(ptn_date) as cnt
	from
		(
									 SELECT user_id, min(ptn_date) as ptn_date
									 FROM uchi.page_events pe
									 where event_source = 'magic_math'
										and toDate(created_at) >= toDate('2020-09-02')
									 GROUP BY user_id
									 UNION ALL
									 SELECT user_id, min(ptn_date)+2 as ptn_date
									 FROM uchi.page_events pe
									 where event_source = 'magic_math'
										and toDate(created_at) >= toDate('2020-09-02')
									 GROUP BY user_id
		) t1
		join
		(
		select
			user_id, 
			ptn_date
		from uchi.page_events t
		where event_source = 'magic_math' 
			and ptn_date >= toDate('2020-09-02')
		group by user_id, ptn_date 
		) t2
		using (user_id, ptn_date)
	group by user_id 	
)
where cnt = 2
```

