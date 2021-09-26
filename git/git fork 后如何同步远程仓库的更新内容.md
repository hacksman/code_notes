
**查看远程状态**
```shell
git remote -v
```


**添加一个远程源仓库的地址**
```shell
git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
```

**fetch 源仓库的信息**
```shell
git fetch upstream
```

**把源仓库上的修改合并到本地分支**
```shell
git merge upstream/master
```

#### 参考连接
[gitlab或github下fork后如何同步源的新更新内容？ - HyG cs的回答](https://www.zhihu.com/question/28676261/answer/44606041)
