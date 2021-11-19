#待编辑 

## @click.prevent：阻止事件的默认行为
```typescript
<template>
  <h1>Hello Demo</h1>
  <a href="http://www.baidu.com" @click.prevent="log">百度一下</a>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  methods: {
    log() {
      console.log("鸡仔说他要百度一下");
    },
  },
});
</script>

<style lang="scss"></style>
```

这里阻止了页面向百度进行跳转，只执行设定的 `log` 函数

## 参考资料
[vue中@click与它的事件修饰符@click.prevent、@click.capture、@click.stop、@click.self](https://www.cnblogs.com/emmamayday/p/15386776.html)