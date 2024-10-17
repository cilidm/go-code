# Es数据迁移

#### 数据转移

```bash
POST /_reindex
{
    "source": {
        "index": "source"
    },
    "dest": {
        "index": "dest"
    }
}
```

#### 设置别名

```bash
# 设置别名
POST /_aliases
{
    "actions":[
        {
            "add":{
                "index":"source",
                "alias":"new"
            }
        }
    ]
}
```



