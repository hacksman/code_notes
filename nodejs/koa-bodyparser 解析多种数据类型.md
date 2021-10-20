需要在 `koa-bodyparser` 的配置中，设置接受多种类型。

```javascript
const Koa = require('koa');

const bodyParser = require('koa-bodyparser');

const router = require('koa-router')();

const app = new Koa();


router.post('/test', async(ctx, next) => {

rawMsg = ctx.request.body

console.log(`rawmsg: ${rawMsg}`);

})


app.use(bodyParser({ "enableTypes": ['json', 'form', 'text'] }))

app.use(router.routes());
```

## 参考资料
[raw-body doesn't seem to work](https://github.com/koajs/bodyparser/issues/73)