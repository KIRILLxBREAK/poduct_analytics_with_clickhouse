---
description: Вовлечение
---

# Engagement

Stickiness

1. DAU/MAU
2. WAU/MAU

```text
	select
		round( avg(wau*100/mau), 1) as stick
	from
			(
				-- Считаем MAU
				select
					toStartOfMonth(created_at) AS mnth,
					uniq(user_id) AS mau
				FROM events e
				WHERE toDate(created_at) >= yesterday()-365
					AND toDate(created_at) <= yesterday()
				GROUP BY mnth, grade_id 
			) montly
		inner join 
			(
				-- Считаем среднее DAU
				select
					grade_id,
					toStartOfMonth(event_date) AS mnth,
					avg(wau) AS wau
				FROM
					(
						select
							toMonday(s.created_at) AS event_date,
							grade_id,
							uniq(student_id) AS wau
						FROM uchi.sessions s
						WHERE grade_id in (1486)
							--AND toDate(s.created_at) >= toDate('2019-09-01')
							--AND toDate(s.created_at) <= toDate('2020-02-29')
							--AND toDate(s.created_at) >= toDate('2018-09-01')
							--AND toDate(s.created_at) <= toDate('2019-02-28')
							AND toDate(s.created_at) >= toDate('2020-04-01')
							AND toDate(s.created_at) <= toDate('2020-04-30')
						GROUP BY event_date, grade_id
				) daily
			group by grade_id, mnth
		) weekly
			using mnth
```

