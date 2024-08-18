# 寻梦电影项目前端

## 后台管理界面

### 一.项目配置

#### 1.初始化项目 

```perl
pnpm create vite
```

#### 2.引入相关配置

#####       1.下载依赖

```perl
pnpm install
```



[【精选】面试突击：线程池有几种创建方式？推荐使用哪种？_线程中创建线程池,线程池大小怎样设置_Java面试那些事儿的博客-CSDN博客](https://blog.csdn.net/HongYu012/article/details/123331122?ops_request_misc=&request_id=&biz_id=102&utm_term=推荐怎样创建线程池&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-3-123331122.142^v70^wechat,201^v4^add_ask&spm=1018.2226.3001.4187)





#####       2.引入eslint

```perl
//1.导入eslint 
pnpm install eslint -D  

//2.初始化eslint配置文件
npm init @eslint/config

//下载eslint有关的插件
pnpm install -D eslint-plugin-import  eslint-plugin-vue  eslint-plugin-node eslint-plugin-prettier eslint-config-prettier eslint-plugin-node @babel/eslint-parser

```



##### 3.   .eslintrc.cjs文件配置

```json
module.exports = {
    env: {
        browser: true,
        es2021: true,
        node:true,
        jest:true
    },
    //指定如何解析语法
    parser:'vue-eslint-parser',
    //优先级低于parser的语法解析配置
    extends: [
        "eslint:recommended",
        // "standard-with-typescript",
        "plugin:vue/vue3-essential",
        "plugin:@typescript-eslint/recommended",
        "plugin:prettier/recommended"
    ],
    parserOptions: {
        ecmaVersion: "latest",
        sourceType: "module",
        parser: "@typescript-eslint/parser",
    },
    plugins: [
        "vue",
        "@typescript-eslint"
    ],
    /**
     * "off"或0  关闭规则
     * "warn"或1  打开规则作为警告(不影响代码执行)
     * "error"或2  规则作为一个错误(代码不能执行,界面报错)
     */
    rules: {
        "no-var": "error", // 要求使用 let 或 const 而不是 var
        "no-multiple-empty-lines": ["warn", { max: 1 }], // 不允许多个空行
        "no-console": process.env.NODE_ENV === "production" ? "error" : "off",
        "no-debugger": process.env.NODE_ENV === "production" ? "error" : "off",
        "no-unexpected-multiline": "error", // 禁止空余的多行
        "no-useless-escape": "off", // 禁止不必要的转义字符

// typeScript (https://typescript-eslint.io/rules)
        "@typescript-eslint/no-unused-vars": "error", // 禁止定义未使用的变量
        "@typescript-eslint/prefer-ts-expect-error": "error", // 禁止使用 @ts-ignore
        "@typescript-eslint/no-explicit-any": "off", // 禁止使用 any 类型
        "@typescript-eslint/no-non-null-assertion": "off",
        "@typescript-eslint/no-namespace": "off", // 禁止使用自定义 TypeScript 模块和命名空间。
        "@typescript-eslint/semi": "off",

// eslint-plugin-vue (https://eslint.vuejs.org/rules/)
        "vue/multi-word-component-names": "off", // 要求组件名称始终为 “-” 链接的单词
        "vue/script-setup-uses-vars": "error", // 防止<script setup>使用的变量<template>被标记为未使用
        "vue/no-mutating-props": "off", // 不允许组件 prop的改变
        "vue/attribute-hyphenation": "off" // 对模板中的自定义组件强制执行属性命名样式


    }
}

```



##### 4.eslintignore忽略文件

```perl
dist
node_modules
```



##### 5.引入prttier格式化工具

```perl
pnpm install -D eslint-plugin-prettier prettier eslint-config-prettier
```



##### 6.   .prttierrc.json文件配置

```json
{
  "singleQuote": true,
  "semi": false,
  "bracketSpacing": true,
  "htmlWhitespaceSensitivity": "ignore",
  "endOfLine": "auto",
  "tabWidth": 2,
  "trailingComma": "none"
}
//解释
1. "singleQuote": true
   - 这表示在代码中使用单引号而不是双引号

2. "semi": false
   - 这表示在代码行的末尾可以不使用使用分号 `;`。

3. "bracketSpacing": true
   - 这表示在对象字面量中的花括号 `{}` 内部添加空格。例如，使用 `{ key: value }` 而不是 `{key: value}`。

4. "htmlWhitespaceSensitivity": "ignore"

   - 这是与HTML代码相关的配置选项。它表示在解析HTML时忽略空格敏感性。换句话说，HTML中的空格将被视为不敏感，不会影响渲染或布局。
5. "endOfLine": "auto"
   - 这表示使用与当前操作系统相匹配的行尾字符。在Windows系统中，行尾通常是回车符加换行符 (`\r\n`)，而在Unix/Linux系统中，行尾通常是换行符 (`\n`)。设置为 "auto" 将自动适配操作系统的行尾字符。

6. "trailingComma": "none"
   - 当将 "trailingComma" 设置为 "none" 时，表示不在对象或数组的最后一个元素后面添加逗号。换句话说，不会使用尾逗号（trailing comma）。例如[1, 2, 3]而不是[1, 2, 3,]。

7. "tabWidth": 2
   - 这表示使用两个空格作为一个制表符的宽度。当你按下 Tab 键时，编辑器将插入两个空格字符。



```



##### 7.    .prettierignore忽略文件

```perl
/dist/*
/html/*
.local
/node_modules/**
**/*.svg
**/*.sh
/public/*
```



##### 8.引入stylelint

**[stylelint](https://stylelint.io/)为css的[lint](https://so.csdn.net/so/search?q=lint&spm=1001.2101.3001.7020)工具。可格式化css代码，检查css语法错误与不合理的写法，指定css书写顺序等。**

**项目中使用scss作为预处理器**

```perl
 pnpm add sass sass-loader stylelint postcss postcss-scss postcss-html stylelint-config-prettier stylelint-config-recess-order stylelint-config-recommended-scss stylelint-config-standard stylelint-config-standard-vue stylelint-scss stylelint-order stylelint-config-standard-scss  stylelint-config-html  stylelint-config-recommended-vue -D     
```



##### 9.    .stylelintrc.cjs配置文件

```js
// @see https://stylelint.bootcss.com/

module.exports = {
  extends: [
    'stylelint-config-standard', // 配置stylelint拓展插件
    'stylelint-config-html/vue', // 配置 vue 中 template 样式格式化
    'stylelint-config-standard-scss', // 配置stylelint scss插件
    'stylelint-config-recommended-vue/scss', // 配置 vue 中 scss 样式格式化
    'stylelint-config-recess-order', // 配置stylelint css属性书写顺序插件,
    'stylelint-config-prettier', // 配置stylelint和prettier兼容
  ],
  overrides: [
    {
      files: ['**/*.(scss|css|vue|html)'],
      customSyntax: 'postcss-scss',
    },
    {
      files: ['**/*.(html|vue)'],
      customSyntax: 'postcss-html',
    },
  ],
  ignoreFiles: [
    '**/*.js',
    '**/*.jsx',
    '**/*.tsx',
    '**/*.ts',
    '**/*.json',
    '**/*.md',
    '**/*.yaml',
  ],
  /**
   * null  => 关闭该规则
   * always => 必须
   */
  rules: {
    'value-keyword-case': null, // 在 css 中使用 v-bind，不报错
    'no-descending-specificity': null, // 禁止在具有较高优先级的选择器后出现被其覆盖的较低优先级的选择器
    'function-url-quotes': 'always', // 要求或禁止 URL 的引号 "always(必须加上引号)"|"never(没有引号)"
    'no-empty-source': null, // 关闭禁止空源码
    'selector-class-pattern': null, // 关闭强制选择器类名的格式
    'property-no-unknown': null, // 禁止未知的属性(true 为不允许)
    'block-opening-brace-space-before': 'always', //大括号之前必须有一个空格或不能有空白符
    'value-no-vendor-prefix': null, // 关闭 属性值前缀 --webkit-box
    'property-no-vendor-prefix': null, // 关闭 属性前缀 -webkit-mask
    'selector-pseudo-class-no-unknown': [
      // 不允许未知的选择器
      true,
      {
        ignorePseudoClasses: ['global', 'v-deep', 'deep'], // 忽略属性，修改element默认样式的时候能使用到
      },
    ],
  },
}

```



##### 10.   .stylelintignore忽略文件

```perl
/node_modules/*
/dist/*
/html/*
/public/*
```



### 二.项目集成

#### 1.集成Tailwind  CSS

#####       1.1 安装Tailwind  CSS

```perl
pnpm add -D tailwindcss postcss autoprefixer
```



#####       1.2  生成配置文件

```perl
pnpm dlx tailwindcss init -p
```



#####       1.3  在main.ts引入

```typescript
import 'tailwindcss/tailwind.css'
```



#####      1.4   引入daisyui

```perl
pnpm i -D daisyui@latest
```

#### 

#####     1.5 tailwind.config.js 配置

```js
export default {
  content: ['./index.html', './src/**/*.{vue,js,ts,jsx,tsx}'],
  // 禁用基础样式
  corePlugins: {
    preflight: false
  },
  theme: {
    extend: {}
  },
  plugins: [require('daisyui')],
  daisyui: {
    themes: [
      //定义主题
      'light',
      'dark',
      'cupcake',
      'synthwave',
      'retro',
      'valentine',
      'night'
    ]
  }
}

```



#### 2. src文件夹别名设置

```typescript
//vite.config.ts配置
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

export default defineConfig({
  plugins: [
      vue(),
  ],
    resolve: {
        alias: {
            '@': path.resolve('./src') //相对路径,，使用@代替src
        }
    }
})

```



```json
//teconfig.json配置    
    "baseUrl": "./" ,  //解析非相对模块的基地址，默认是当前目录
    "paths": {//路径映射，相对于baseurl
      "@/*": ["src/*"]
    }
```



#### 3.项目环境变量设置



#### 4.集成svg图标



#####         4.1   安装svg依赖插件

```perl
pnpm install vite-plugin-svg-icons -D      
```



#####        4.2   在vite.config.ts中配置插件

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { createSvgIconsPlugin } from 'vite-plugin-svg-icons'
import path from 'path'

export default defineConfig({
  plugins: [
    vue(),
    createSvgIconsPlugin({
      iconDirs: [path.resolve(process.cwd(), 'src/assets/icons')],
      symbolId: 'icon-[dir]-[name]'
    })
  ],
  resolve: {
    alias: {
      '@': path.resolve('./src') //相对路径,，使用@代替src
    }
  }
})

```



#####       4.3  main.ts文件中导入

```typescript
//将 SVG 图标注册为可用的组件
import 'virtual:svg-icons-register'
```



##### 4.4  svg图标的封装与使用

```vue
<template>
  <svg>
    <use :href="prefix + name" :fill="color"></use>
  </svg>
</template>

<script setup lang="ts">
//接收父组件传过来的参数
defineProps({
  //xlink:href属性值前缀
  prefix: {
    type: String,
    default: '#icon-'
  },
  //图标名字
  name: String,
  //颜色
  color: String
})
</script>

```



##### 4.5  注册成全局组件

```typescript
import SvgIcon from '@/components/SvgIcon/index.vue'
const app = createApp(App)
app.component('SvgIcon', SvgIcon)
```



##### 4.6  改进

```typescript
//自定义插件,注册整个项目的全局组件,不需要在main.ts中import全局组件
import SvgIcon from './SvgIcon/index.vue'
import App from '@/App.vue'
const allGlobalComponent = { SvgIcon }
export default {
  install(app: App) {
    // 注册项目全部的全局组件
    Object.keys(allGlobalComponent).forEach((key) => {
      app.component(key, allGlobalComponent[key])
    })
  }
}
```



```typescript
//main.ts
//引入自定义插件,注册整个项目的全局组件
import globalComponent from '@/components'
//安装自定义插件
app.use(globalComponent)
```





#### 5.集成element-plus

```perl
pnpm install element-plus
pnpm install @element-plus/icons-vue
```



#####   5.1  国际化

```vue
//App.vue
<template>
  <svg-icon></svg-icon>
  <el-config-provider :locale="local">
    //组件
  </el-config-provider>
</template>
<script setup lang="ts">
import zhCn from 'element-plus/dist/locale/zh-cn.mjs'
const local = zhCn
</script>
```



```ts
//import zhCn from 'element-plus/dist/locale/zh-cn.mjs' 不是ts文件
//要在vite.env.d.ts中加上declare module '...'
declare module 'element-plus/dist/locale/zh-cn.mjs'
```



#### 6. 集成路由

```perl
pnpm install vue-router
```



```json
qq:@gc03110018
```

