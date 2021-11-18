## never
never 表示永远不存在值的类型

### 用例
#### 函数抛出异常
```javascript
function error(message: string): never {
	throw new Error(message)
}
```

#### 无返回值的函数
```javascript
function fool(): never {
	while (true) { }
};
```

#### 详细检查
```javascript
type Foo = string | number;

function controlFlowAnalysisWithNever(foo: Foo) {
	if (typeof foo === 'string') {
	} else if (typeof foo === 'number') {
	} else {
		const check:never = foo;
	}
}
```

如果将 Foo 修改为  `type Foo = string | number | boolean` ，而忘记在函数中新增 `typeof foo === 'boolean'` 的判断，这个时候 `else` 分支的 `foo` 类型被收窄为 `boolean` 类型。导致无法复制给 never 类型，就会产生一个编译错误，从而可以确保条件逻辑穷尽了所有可能。实现类型绝对安全的代码

## 参考资料
[TypeScript never 类型 | 全栈修仙之路](http://www.semlinker.com/ts-never-type/)