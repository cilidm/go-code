# 搜索elasticsearch的数据条数大于10000的情况解决

默认情况下当用elasticsearch进行深度分页查询时的size-from大于10000的时候，就会报错

```bash
### kibana里执行
PUT /us_tm_info_v1/_settings
{
  "index.max_result_window": "20000"
}
```

第二种方式 

在config/elasticsearch.yml文件中的最后加上

```bash
index.max_result_window: 100000000
```

这种方法要注意在最前面加上空格

