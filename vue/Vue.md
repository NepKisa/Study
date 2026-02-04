# 初始化

全局安装依赖

```perl
npm install webpack -g
npm install -g vue-cli
```

初始化项目

```perl
npm create vue@latest infinity
或
vue create infinity
```

关闭eslint

```perl
npm remove -D eslint eslint-plugin-vue @vue/eslint-config-prettier vite-plugin-eslint
```

修改vue.config.js

```javascript
const { defineConfig } = require('@vue/cli-service')
module.exports = defineConfig({
  lintOnSave: false,
  transpileDependencies: true
})

```

# 目录规范

```perl
npm install axios vue-router
```

