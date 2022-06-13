---
description: Разные виды
---

# Retention

Retention N-дня

```
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

```
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





Массивы



```
select 
       segment,
       grade_id,
       count(student_id) AS day_0,
       countIf(arrayExists(x -> x = 1, days)) as day_1,
       countIf(arrayExists(x -> x >= 1, days)) as day_1_and_more
    from (
          select * from 
             (--смотрим дни захода
                 select 
                        segment,
                        student_id,
                        grade_id,
                        arrayCumSum(arrayDifference(arraySort(groupUniqArray(toUnixTimestamp(ptn_date))))) as days
                 from uchi.sessions s 
                 where ptn_date between %(start_date)s and %(final_date)s
                 --and grade_id = 1847
                 group by segment, student_id, grade_id
            ) sess
          inner join 
            (-- cмотрим студентов
                select 
                      student_id, 
                      grade_id, 
                      min(ptn_date) as min_date
                from uchi.sessions s
                --where grade_id = 1847
                group by student_id, grade_id
                having min_date >= %(start_date)s
             ) studs
                   using student_id, grade_id 
        )
group by segment, grade_id
```
