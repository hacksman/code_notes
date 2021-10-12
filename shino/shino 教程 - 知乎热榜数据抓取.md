
## 基础环境
配置爬取的规则非常简单，但在开始之前，请确保你已经安装好 mysql、rabbitmq 和 redis 并已经正常启动它们。

我们启动数据库和消息队列后，可以通过校验程序测试一下服务是否正常。比如这里我们选择校验本地环境程序是否正常，可以这样键入命令

```shell
 python shino/check.py --env debug
```

如果出现以下字样，代表服务正常启动
```shell
2021-10-11 11:09:35.142 | SUCCESS  | __main__:db_is_ok:66 - mysql is ok
2021-10-11 11:09:35.144 | SUCCESS  | __main__:db_is_ok:106 - redis is ok
2021-10-11 11:09:35.162 | SUCCESS  | __main__:db_is_ok:141 - rabbitmq is ok
```

如果出现异常，请根据提示去检查数据库和消息队列是否被正常开启，且保证  `resources` 环境中的配置项正确。

## 初始化建表
shino 的运行依赖六张功能表，他们的功能如下所示

| 表名            | 功能       | 说明                   |
| --------------- | ---------- | ---------------------- |
| \_topic           | 主题       | 配置最终入库的字段     |
| \_parse_list      | 解析列表页 | 配置列表页被解析的规则 |
| \_parse_detail    | 解析详情页 | 配置详情页被解析的规则 |
| \_request_depend  | 请求依赖   | 配置请求的 url 所依赖的属性和字段         |
| \_seed_sch        | 种子调度   | 配置种子被调度和生成的规则                   |
| \_service_monitor | 服务监控   | 记录服务被调用次数信息                   |

我们可以通过键入一下命令一键创建所有表格
```shell
python shino/gentable.py --env local
```

这样我们就能在数据库中看到如下的六张表
![](https://gitee.com/hacksman/img_host/raw/master/img/20211011114311.png)

## 配置爬虫依赖表
### 配置主题表（\_topic）

这次的爬取，我们的最终目标是爬取知乎热榜的首页数据

![](https://gitee.com/hacksman/img_host/raw/master/img/20211009094516.png)

数据请求接口在这 👉 [戳这里查看](https://www.zhihu.com/api/v3/feed/topstory/hot-lists/total?limit=50&desktop=true)

![](https://gitee.com/hacksman/img_host/raw/master/img/20210929110752.png)

爬取之前我们得知道，我们需要哪些数据。比如这次我们的目标是爬取以下截图中的红色框的数据

![](https://gitee.com/hacksman/img_host/raw/master/img/20210929110920.png)

那么我们就需要在主题表中定义最终的数据字段名称、字段释义和字段类型。像下面这样

```json
{
  "_id": {
    "f_type": "INTEGER",
    "nullable": false,
    "primary_key": true,
    "auto_increment": true
  },
  "url": {
    "f_type": "String",
    "f_comment": "话题链接",
    "f_max_range": 256
  },
  "t_id": {
    "f_type": "String",
    "f_comment": "话题唯一标识id",
    "f_max_range": 64
  },
  "title": {
    "f_type": "String",
    "f_comment": "话题标题",
    "f_max_range": 256
  },
  "author": {
    "f_type": "JSON",
    "f_comment": "话题作者信息"
  },
  "t_type": {
    "f_type": "String",
    "f_comment": "话题类型",
    "f_max_range": 64
  },
  "created": {
    "f_type": "String",
    "f_comment": "话题创建时间",
    "f_max_range": 32
  },
  "excerpt": {
    "f_type": "Text",
    "f_comment": "话题摘录"
  },
  "detail_text": {
    "f_type": "String",
    "f_comment": "热度相关信息",
    "f_max_range": 256
  },
  "answer_count": {
    "f_type": "INTEGER",
    "f_comment": "话题回复数"
  },
  "comment_count": {
    "f_type": "INTEGER",
    "f_comment": "话题评论数"
  },
  "follower_count": {
    "f_type": "INTEGER",
    "f_comment": "话题关注数"
  }
}
```

那么最终 `_topic` 表中配置的字段和值应该是这样的



| 字段      | 字段释义          | 示例数据         |
| --------- | ----------------- | ---------------- |
| name      | 定义 topic 的名称 | 知乎             |
| desc      | topic 描述        | 知乎每日热榜信息 |
| table     | 最终入库的表名    | zhihu_hot_list   |
| schema |           定义的入库主题表结构信息        |     {"_id": {"f_type": "INTEGER", "nullable": false, "primary_key": true, "auto_increment": true}, "url": {"f_type": "String", "f_comment": "话题链接", "f_max_range": 256}, "t_id": {"f_type": "String", "f_comment": "话题唯一标识id", "f_max_range": 64}, "title": {"f_type": "String", "f_comment": "话题标题", "f_max_range": 256}, "author": {"f_type": "JSON", "f_comment": "话题作者信息"}, "t_type": {"f_type": "String", "f_comment": "话题类型", "f_max_range": 64}, "created": {"f_type": "String", "f_comment": "话题创建时间", "f_max_range": 32}, "excerpt": {"f_type": "Text", "f_comment": "话题摘录"}, "detail_text": {"f_type": "String", "f_comment": "热度相关信息", "f_max_range": 256}, "answer_count": {"f_type": "INTEGER", "f_comment": "话题回复数"}, "comment_count": {"f_type": "INTEGER", "f_comment": "话题评论数"}, "follower_count": {"f_type": "INTEGER", "f_comment": "话题关注数"}}            |                 |                  |

怎么样？很简单吧？

### 配置解析详情规则表 (\_parse_detail)

接下来我们只需要定义一下解析的规则把数据提取出来就可以了。我们观察一下这个知乎的数据结构，解析详情页的核心逻辑是要先进入 data 数组中，然后将其中的每个信息，解析出来成为一条单独的数据即可。那么我们可以配置一个嵌套的解析规则，首先进入 data 的部分可以这样配置

```
[
  {
    "$each": [],      				# 其中的每一条数据的解析规则
    "$name": "__datas__",			# 字段名
    "$parse_rule": "data",			# 解析规则
    "$value_type": "recursion",		# 值类型
    "$parse_method": "json"			# 解析方式
  }
]
```

然后在 `$each` 中单独配置每一条的解析规则，这里我们拿 `title` 和 `detail_text` 举例，他们分别存在于 `target` 层和外层，那么我们便可以这样配置

```
[
  {
	"$each": [],
	"$name": "title",
	"$parse_rule": "target.title",
	"$value_type": "raw",
	"$parse_method": "json"
  },
  {
	"$each": [],
	"$name": "detail_text",
	"$parse_rule": "detail_text",
	"$value_type": "raw",
	"$parse_method": "json"
  }
]
```

这个解析规则配完，我们补齐表中的其他信息就完成啦

| 字段      | 释义         | 示例数据                                                   |
| --------- | ------------ | ---------------------------------------------------------- |
| desc      | 解析规则描述 | 知乎热榜                                                   |
| topic_id  | 关联主题id   | 1                                                          |
| api       | 请求维度api  | https://www.zhihu.com/api/v3/feed/topstory/hot-lists/total |
| data_rule | 数据解析规则 | [{"$each": [{"$each": [], "$name": "url", "$parse_rule": "target.url", "$value_type": "raw", "$parse_method": "json"}, {"$each": [], "$name": "t_id", "$parse_rule": "target.id", "$value_type": "raw", "$parse_method": "json"}, {"$each": [], "$name": "title", "$parse_rule": "target.title", "$value_type": "raw", "$parse_method": "json"}, {"$each": [], "$name": "author", "$parse_rule": "target.author", "$value_type": "raw", "$parse_method": "json"}, {"$each": [], "$name": "t_type", "$parse_rule": "target.type", "$value_type": "raw", "$parse_method": "json"}, {"$each": [], "$name": "created", "$parse_rule": "target.created", "$value_type": "raw", "$parse_method": "json"}, {"$each": [], "$name": "excerpt", "$parse_rule": "target.excerpt", "$value_type": "raw", "$parse_method": "json"}, {"$each": [], "$name": "detail_text", "$parse_rule": "detail_text", "$value_type": "raw", "$parse_method": "json"}, {"$each": [], "$name": "answer_count", "$parse_rule": "target.answer_count", "$value_type": "raw", "$parse_method": "json"}, {"$each": [], "$name": "comment_count", "$parse_rule": "target.comment_count", "$value_type": "raw", "$parse_method": "json"}, {"$each": [], "$name": "follower_count", "$parse_rule": "target.follower_count", "$value_type": "raw", "$parse_method": "json"}], "$name": "__datas__", "$parse_rule": "data", "$value_type": "recursion", "$parse_method": "json"}]                                         |


### 配置种子调度表(\_seed_sch)
最后我们还需要配置一下种子表(_seed_sch)，方便它调度和启动
```
{
	# 种子名称
	"name": "知乎热榜",
	
	# 请求维度的 api
	"api": "https://www.zhihu.com/api/v3/feed/topstory/hot-lists/total",
	
	# 请求的初始种子 url
	"seed_url": "https://www.zhihu.com/api/v3/feed/topstory/hot-lists/total?limit=50&desktop=true",
	
	# 发起网络请求方式
	"method": "get",
	
	# 维度的优先级
	"priority": 4,
	
	# 是否开启抓取, 先关闭，等到我们启动服务后，再来开启
	"switch": "off",
	
	# 是否是一次性任务
	"is_once": 1
}

```


### 开始爬取
至此，我们的所有的准备工作已经完成了。接下来直接启动服务即可

```shell

# 进入到项目根目录，根据自己项目所在的路径进入
cd ~/shino

# 启动服务
sh start.sh debug
```

正常启动服务后，你会在命令行看到类似这样的提示
```shell
start check db
2021-10-12 11:33:21.739 | SUCCESS  | __main__:db_is_ok:66 - mysql is ok
2021-10-12 11:33:21.740 | SUCCESS  | __main__:db_is_ok:106 - redis is ok
2021-10-12 11:33:21.789 | SUCCESS  | __main__:db_is_ok:141 - rabbitmq is ok
db is ok
start create table system need
2021-10-12 11:33:21.944 | SUCCESS  | __main__:<module>:173 - table create success
table system need create success
start shino.saver
start shino.cleaner
start shino.extractor
start shino.downloader
start shino.seed_generator
```

接下来我们监测一下任务的日志

```shell
tail -f logs/*
```

出现如下的提示，证明服务启动成功
```shell
==> logs/cleaner.log <==
2021-09-29 10:24:31.479 | INFO     | shino.cleaner.clean_processor:do_job:55 - queue:「extract_info」 empty....
2021-09-29 10:24:35.505 | INFO     | shino.cleaner.clean_processor:do_job:55 - queue:「extract_info」 empty....

==> logs/downloader.log <==
2021-09-29 10:24:31.554 | INFO     | shino.downloader.download_processor:do_job:54 - queue:「download_req」 empty....
2021-09-29 10:24:35.579 | INFO     | shino.downloader.download_processor:do_job:54 - queue:「download_req」 empty....

==> logs/extractor.log <==
2021-09-29 10:24:31.479 | INFO     | shino.extractor.extract_processor:do_job:78 - queue:「download_rsp」 empty....
2021-09-29 10:24:35.495 | INFO     | shino.extractor.extract_processor:do_job:78 - queue:「download_rsp」 empty....

==> logs/saver.log <==
2021-09-29 10:24:31.649 | INFO     | shino.saver.save_processor:do_job:41 - queue:「clean_info」 empty....
2021-09-29 10:24:35.676 | INFO     | shino.saver.save_processor:do_job:41 - queue:「clean_info」 empty....

==> logs/seed_generator.log <==
2021-09-29 10:24:29.544 | INFO     | __main__:main:79 - main pid=67702

==> logs/extractor.log <==
2021-09-29 10:24:39.510 | INFO     | shino.extractor.extract_processor:do_job:78 - queue:「download_rsp」 empty....

==> logs/cleaner.log <==
2021-09-29 10:24:39.530 | INFO     | shino.cleaner.clean_processor:do_job:55 - queue:「extract_info」 empty....

==> logs/downloader.log <==
2021-09-29 10:24:39.603 | INFO     | shino.downloader.download_processor:do_job:54 - queue:「download_req」 empty....

==> logs/saver.log <==
2021-09-29 10:24:39.700 | INFO     | shino.saver.save_processor:do_job:41 - queue:「clean_info」 empty....

==> logs/extractor.log <==
2021-09-29 10:24:43.527 | INFO     | shino.extractor.extract_processor:do_job:78 - queue:「download_rsp」 empty....

==> logs/cleaner.log <==
2021-09-29 10:24:43.555 | INFO     | shino.cleaner.clean_processor:do_job:55 - queue:「extract_info」 empty....

==> logs/downloader.log <==
2021-09-29 10:24:43.626 | INFO     | shino.downloader.download_processor:do_job:54 - queue:「download_req」 empty....

==> logs/saver.log <==
2021-09-29 10:24:43.724 | INFO     | shino.saver.save_processor:do_job:41 - queue:「clean_info」 empty....

==> logs/extractor.log <==
2021-09-29 10:24:47.541 | INFO     | shino.extractor.extract_processor:do_job:78 - queue:「download_rsp」 empty....

==> logs/cleaner.log <==
2021-09-29 10:24:47.579 | INFO     | shino.cleaner.clean_processor:do_job:55 - queue:「extract_info」 empty....

==> logs/downloader.log <==
2021-09-29 10:24:47.649 | INFO     | shino.downloader.download_processor:do_job:54 - queue:「download_req」 empty....

==> logs/saver.log <==
2021-09-29 10:24:47.749 | INFO     | shino.saver.save_processor:do_job:41 - queue:「clean_info」 empty....
```

最后，我们来将种子表的开关打开，将 `_seed_sch` 中字段 `switch` 改成 `on` ，观察一下日志，出现如下的提示，证明抓取成功

```shell
==> logs/seed_generator.log <==
2021-09-29 10:28:29.784 | INFO     | shino.seed_generator.seed_gen_processor:__cycle_schedule:57 - start gen seed
2021-09-29 10:28:29.786 | INFO     | shino.seed_generator.seed_gen_processor:__cycle_schedule:74 - find sch work: task_key=202109291028_https://www.zhihu.com/api/v3/feed/topstory/hot-lists/total      appoint_crontab=None    is_once=1

==> logs/downloader.log <==
2021-09-29 10:28:31.038 | INFO     | shino.downloader.download_processor:do_job:47 - request_msg        {'batch_no': '202109291028_8846f7ab87c3ec1d2794be3fe067af41', 'api': 'https://www.zhihu.com/api/v3/feed/topstory/hot-lists/total', 'url': 'https://www.zhihu.com/api/v3/feed/topstory/hot-lists/total?limit=50&desktop=true', 'method': 'get'}
2021-09-29 10:28:31.039 | INFO     | shino.downloader.download_handle:download:25 - start_crawl url:https://www.zhihu.com/api/v3/feed/topstory/hot-lists/total?limit=50&desktop=true    method:get      download_type:None
2021-09-29 10:28:31.596 | INFO     | shino.downloader.downloader.downloader:download:23 - NoneType has download:https://www.zhihu.com/api/v3/feed/topstory/hot-lists/total?limit=50&desktop=true
2021-09-29 10:28:31.597 | INFO     | shino.downloader.download_handle:download:38 - result_code ==: 200
2021-09-29 10:28:31.599 | INFO     | shino.downloader.download_handle:download:50 - finish_crawl        use_time:0.5585441589355469     lens:95623      status:0        url:https://www.zhihu.com/api/v3/feed/topstory/hot-lists/total?limit=50&desktop=true
```


去数据库验证一下结果

![](https://gitee.com/hacksman/img_host/raw/master/img/20211009100122.png)