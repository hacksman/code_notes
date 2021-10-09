#待编辑

```shell
SELECT author_id,room_id,exec_time FROM record_comment_task where _create_time >= (CURRENT_DATE() - INTERVAL 30 DAY)
```
