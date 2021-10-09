pymysql 中数据库的 ddl 会自动生成建表语句

```shell
create table _parse_list
(
    _id                  int auto_increment
        primary key,
    _create_time         timestamp default CURRENT_TIMESTAMP not null,
    _update_time         timestamp default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP,
    _is_delete           tinyint   default 0                 null,
    start_page           int       default 1                 null comment '初始页码',
);

create index idx_create_time
    on _parse_list (_create_time);

create index idx_update_time
    on _parse_list (_update_time);
```

如果你将这段代码直接拿到 pymysql 中执行，会提示语法错误。这是因为建表语句可以依次执行，而 pymysql 中是一次性执行命令的，因此需要在定义建表语句的时候事先定义好索引。那么同样的建表，我们可以这样定义建表语句：

```shell
create table if not exists _parse_list
(
    _id                  int auto_increment
        primary key,
    _create_time         timestamp default CURRENT_TIMESTAMP not null,
    _update_time         timestamp default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP,
    _is_delete           tinyint   default 0                 null,
    start_page           int       default 1                 null comment '初始页码',
    index                idx_create_time (_create_time),
    index                idx_update_time (_update_time)
)
```

其中索引字段是直接定义在建表语句中的