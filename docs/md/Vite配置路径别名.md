<!--
 * @Author: tangdaoyong
 * @Date: 2021-06-24 14:07:32
 * @LastEditors: tangdaoyong
 * @LastEditTime: 2021-06-24 14:37:17
 * @Description: Vite配置路径别名
-->
# Vite配置路径别名

[Vite resolve-alias](https://cn.vitejs.dev/config/#resolve-alias)

## 旧版

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  alias: {
    'assets': resolve(__dirname, './src/assets')
  },
  ...
})
```

## 新版

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  alias: [
    {
      find: 'assets',
      replacement : resolve(__dirname, './src/assets')
    }
  ],
  ...
})
```

## vscode无法识别别名路径

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  alias: [
    {
      find: 'utils',
      replacement : resolve(__dirname, './src/utils')
    }
  ],
  ...
})
```
如上，定义了别名`utils`。但在`.ts`中如下使用时。
```ts
import { prefixToUpperCase, prefixToLowerCase } from 'utils/util'
```
提示：`Cannot find module 'utils/util' or its corresponding type declarations.ts(2307)`表明编辑器并没有识别到路径别名，这时候需要在`tsconfig.json`中更新`ts`相关配置。
```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "jsx": "preserve",
    "sourceMap": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "lib": ["esnext", "dom"],
    "paths": {
      "utils/*": ["./src/utils/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"]
}
```
添加了：
```js
"paths": {
  "utils/*": ["./src/utils/*"]
}
```
这时就能够正常识别了，就是在`ts`的配置中，也配置一下路径别名，使其能够识别。

**注意**`js`则配置 `jsconfig.json`

## 注意

**注意**修改了配置文件需要`重新运行服务`。

另外在`vite`中导入`.vue`文件时后缀不能省，配置`extensions`也不行，[官方已经不允许不使用`vue`文件后缀导入了](https://github.com/vitejs/vite/issues/178)