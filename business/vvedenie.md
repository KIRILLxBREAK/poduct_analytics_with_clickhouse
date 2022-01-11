---
description: метрики о деньгах
---

# Введение

MAU/DAU

```
select
    toStartOfYear(mtnh) as yr,
    uniqExactMerge(clients) as MAU,
    groupArray(finalizeAggregation(clients)) as DAY_array
from (
      select toStartOfMonth(date) as mtnh,
             uniqExactState(user_client_id) as clients
      from tracker.events e
      where date >= toDate('2021-10-01')
        and action_type = 'payment_success'
      group by mtnh
      order by mtnh
)
group by yr;
```



