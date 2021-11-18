以 `.ts` 结尾的 [[typescript]] 文件，运行前通常需要用 [[tsc]] 先将它编译成 `.js` 的文件，然后才可以通过 `node` 调用运行，每次运行测试完成，还需要将编程出来的 `.js` 文件手工清除，步骤非常的繁琐且不优雅。比如像下面这样

```shell
tsc a.ts
```

它会在当前目录下生成一个同名的 `.js` 文件
```shell
>>>ls
a.ts	a.js
```

然后你可以通过 `node` 去调用这个编译后的 `.js` 文件
```
node a.js
```

最后，为了保证整洁，你还需要删除这个编译好的 `.js` 文件。
```
>>>rm -rf a.js
>>>ls
a.ts
```

发现没，这样做下来非常繁琐，其实我们完全不用这么麻烦，直接可以使用 `ts-node` 运行这个 `.ts` 文件。像这样
```shell
npx ts-node a.ts
```

非常方便且优雅

## 参考资料
[How to run TypeScript files from command line?](https://stackoverflow.com/questions/33535879/how-to-run-typescript-files-from-command-line)