# Result window is too large

ES 分页查询时，默认最大信息条数 10000 当size超过10000时报错 Result window is too large, from + size must be less than or equal to \[10000\]

此时只用设置

max\_result\_window 得值即可 

```text
PUT your-index/_settings
{"max_result_window" : "200000000"}
```

