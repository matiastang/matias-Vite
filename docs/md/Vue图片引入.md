<!--
 * @Author: tangdaoyong
 * @Date: 2021-06-24 14:24:35
 * @LastEditors: tangdaoyong
 * @LastEditTime: 2021-06-24 15:21:47
 * @Description: Vue图片引入
-->
# Vue图片引入

## Vue 引入前景图片

### 1. 直接引入

```html
<!-- 1. 相对路径直接引用 -->
<img src="../../assets/login-bottom.webp" class="bottom" />
<!-- 2. 根目录路径直接引用 -->
<img src="/src/assets/login-bottom.webp" class="bottom" />
<!-- assets是一个路径别名，在vite中配置。这种方式是不行的，html中不能识别别名 -->
<!-- <img src="assets/login-bottom.webp" class="bottom" /> -->
```

这种处置**问题**，开发环境可以正常引入，打包后却不能显示的问题。

### 2. 绑定路径变量

```js
<img :src="bottomImg" class="bottom" />

// <script lang="ts">
import { defineComponent } from 'vue'
// 1. 相对路径导入
// import bottomImg from '../../assets/login-bottom.webp'
// 2. 相对根路径导入
// import bottomImg from '/src/assets/login-bottom.webp'
// 3. 路径别名导入
import bottomImg from 'assets/login-bottom.webp'

export default defineComponent({
  name: 'Login', // 路由组件名称
  data() {
        return {
            bottomImg,
        }
  },
})
// </script>
```
如上，导入图片之后，定义到`data`中，然后在界面上绑定变量到路径。

## Vue 引入背景图片

### 1. 直接内联绑定路径

```js
<div class="container" :style="{ backgroundImage: 'url(' + containerImg + ')' }">
</div>
import containerImg from 'assets/login-container.webp'

export default defineComponent({
  name: 'Login', // 路由组件名称
  data() {
        return {
            containerImg,
        }
  },
})
```

### 2. 内联绑定style

```js
<div class="container" :style="containerStyle">
</div>

export default defineComponent({
  name: 'Login', // 路由组件名称
  data() {
        return {
            containerStyle: {
                backgroundImage: 'url(' + containerImg + ')',
                backgroundRepeat: 'no-repeat',
                backgroundSize: 'cover'
            },
        }
  },
})
```

## 问题

vite 官方默认的配置，打包后会把图片名加上 hash值，但是直接通过 :src="imgSrc"方式引入并不会在打包的时候解析，导致开发环境可以正常引入，打包后却不能显示的问题，示例如下：
打包后路径：

<img src="static/icon/123.jpg">
实际打包后的图片路径：static/icon/123.hash.jpg

可以尝试以下方法解决：

HTML：

<img :src="getSrc('123')">
JS:

const getSrc = (name) => {
    const path = `/static/icon/${name}.svg`;
    const modules = import.meta.globEager("/static/icon/*.svg");
    return modules[path].default;
  };