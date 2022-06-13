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

### Через массивы

```
select
    reg_date,
    is_completed,
    sum(r[1]) N,
    sum(r[2]) one_day,
    sum(r[3]) three_day,
    sum(r[4]) week,
    sum(r[6]) two_weeks,
    sum(r[6]) days_30,
    sum(r[7]) days_60,
    sum(r[8]) days_90,
    sum(r[9]) days_180
FROM 
    (
	select 
            user_id,
            reg_date,
            is_completed,
            retention(res = 0,
		    	res = 1,
		        res = 3,
		        res = 7,
		        res = 14,
		        res = 30,
		        res = 60,
		        res = 90,
		        res = 180) r       
		from 
		(
			select
				user_id,
				min(ptn_date) as reg_date,
				arraySort( groupUniqArray(ptn_date) ) as dates,
				arrayFilter( x -> x in (0,1,3,7,14,30,60,90,180), arrayMap(x -> (x - reg_date), dates) ) as res
			from uchi.page_events pe
			where ptn_date >= '2021-01-18'
			group by user_id
			having reg_date between <Parameters.Parameter Start Date> and <Parameters.Parameter End Date>
--			having reg_date between '2021-02-01' and '2021-04-01'
		) ARRAY JOIN res
		GROUP BY user_id, reg_date, is_completed
group by reg_date, is_completed
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
