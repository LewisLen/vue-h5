# vue-h5

基于vue-cli4.x的eslint+prettier模版的多页面移动端脚手架，开箱即用


## 安装和启动

```shell
# 全局安装vue-cli4.x
npm install -g @vue-cli
# 创建一个项目
vue create vue-h5
# 安装依赖启动项目
cd vue-h5
npm install
npm run serve
```


## 移动端适配

采用的的`lib-flexible`方案，搭配`postcss-pxtorem`插件可以直接在开发过程中用px做单位，插件会根据配置自动转化成rem

在 main.js 中引入 lib-flexible.js 或者直接在模板的 html 文件中引入该方案

```javascript
// .postcssrc.js文件
// 也可以是postcss.config.js
module.exports = {
  plugins: {
    "autoprefixer": {},
    "postcss-pxtorem": {
      "rootValue": 75, // 设计稿宽度的1/10
      "unitPrecision": 4, // 小数位
      "minPixelValue": 2, // 转换的最小单位
      "selectorBlackList": [], // 忽略的样式, 正则
      "propList": ["*"], // 需要做转化处理的属性，如`hight`、`margin`等，也可以正则匹配
      "exclude": /node_modules/
    }
  }
}
```

> 出现报错: Syntax Error: Error: PostCSS plugin postcss-pxtorem requires PostCSS 8.
> 解决方案，安装postcss-pxtorem@5.1.1版本: npm install postcss-pxtorem@5.1.1 -D


## eslint + prettier

VSCode 安装 ESlint 和 prettier 插件

主要依赖：

```javascript
// package.json
"devDependencies": {
  "@vue/cli-plugin-eslint": "~4.5.0",
  "@vue/eslint-config-prettier": "^6.0.0",
  "eslint": "^6.7.2",
  "eslint-plugin-prettier": "^3.3.1",
  "eslint-plugin-vue": "^6.2.2",
  "prettier": "^2.2.1",
}
// .eslintrc.js
extends: [
  "plugin:vue/essential", 
  "eslint:recommended", // 推荐eslint规则
  "@vue/prettier",
  "plugin:prettier/recommended", // 如果同时使用了eslint和prettier发生冲突了，会关闭掉与prettier有冲突的规则，也就是使用prettier认为对的规则
],
```

VSCode 主要设置

```javascript
// settings.json部分配置
"vetur.ignoreProjectWarning": true,
"vetur.format.defaultFormatterOptions": {
    "js-beautify-html": {
        "wrap_attributes": "force-expand-multiline"
    },
    "prettyhtml": {
        "printWidth": 100,
        "singleQuote": false,
        "wrapAttributes": false,
        "sortAttributes": false
    }
},
"explorer.confirmDragAndDrop": false,
"explorer.confirmDelete": false,
// eslint+prettier格式化代码
"eslint.alwaysShowStatus": true,
"eslint.run": "onSave",
// 自动修复代码
"editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
},
"eslint.probe": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact",
    "vue",
],
"eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact",
    "vue",
],
"prettier.useTabs": true,
```


## 封装axios

安装`axios`和`qs`，利用 interceptors 拦截器对 axios 请求进行封装

```javascript
// 引用方式一 main.js
import request from "./api/request";
Vue.prototype.$http = request;// 挂载到原型对象上
// 组件中
this.$http.get("api/productList").then((res) => {
  console.log(res);
});
// 引用方式二：每个请求按模块划分
// product.js
import request from "../request";
import QS from "qs";

export function getProductList(data) {
  return request({
    url: "api/productList",
    method: "get", // 默认是get
    data: QS.stringify(data),
  });
}
```

取消请求方式

```javascript
// 方式一
const cancelToken = axios.CancelToken;
const source = CancelToken.source();
axios.post('/productList',{code:'0001'},{
  cancelToken: source.token
})
source.cancel('取消请求')
// 方式二
const cancelToken = axios.CancelToken;
let cancel;
axios.get('/productList',{
  cancelToken: new cancelToken(function executor(c){
    cancel = c;
  })
})
cancel();// 取消请求
```

## 多环境配置

根目录下新增`.env.uat`文件，并在 package.json 中田添加对应的命令行

```javascript
// .env.uat
NODE_ENV = 'production'
VUE_APP_MODE = 'uat'
VUE_APP_baseURL = 'https://www.uat.com/'

// package.json
"uat": "vue-cli-service build --mode uat",
```