---
layout: mypost
title: ant-design-vue组件设置中文
categories: [ ant-design-vue, 组件, 中文 ]
---

<br>

### 背景

- 在开发用户编辑生日功能的时候,使用到了ant-design-vue中的DatePicker组件,[DatePicker](https://2x.antdv.com/components/date-picker-cn/)

- 但是组件的默认语言是英文,所以需要将组件的语言切换为中文.

### 解决方案

- 官方推荐了两种解决方案:
    1. 全局配置语言
    2. 局部配置语言

#### 全局配置语言

- 我肯定首选是全局配置语言,因为只需要设置一次,不需要在每个页面都设置.

##### 官方方案

- 官方给的示例代码如下:
  ```vue
  
  <template>
    <a-config-provider :getPopupContainer="getPopupContainer">
      <app/>
    </a-config-provider>
  </template>
  <script>
    export default {
      methods: {
        getPopupContainer(el, dialogContext) {
          if (dialogContext) {
            return dialogContext.getDialogWrap();
          } else {
            return document.body;
          }
        },
      },
    };
  </script> 
  ```

- 这里的`a-config-provider`组件是用来全局配置语言的,它可以设置全局的语言,包括日期格式,时间格式,数字格式等.
- 我们只需要在`a-config-provider`组件中设置`locale`属性即可,它的值可以是`zh-CN`或者`en-US`,分别对应中文和英文.

##### 自选方案

- 我用的是组件式的方式
- 在`App.vue`中只需要引入`ConfigProvider`组件,并将BasicLayout组件包裹在`ConfigProvider`组件中即可.

  ```vue
  
  <script setup lang="ts">
  import {inject} from 'vue'
  import BasicLayout from '@/layouts/BasicLayout.vue'
  import {LoginUserStore} from '@/stores/LoginUserStore.ts'
  
  const locale = inject('locale')
  
  const loginUserStore = LoginUserStore()
  loginUserStore.fetchLoginUser()
  </script>
  
  <template>
    <div id="app">
      <a-config-provider :locale="locale">
        <BasicLayout/>
      </a-config-provider>
    </div>
  </template>
  
  <style>
    #app {
    }
  </style>
  ```

- 在`main.ts`中注册了`ConfigProvider`组件,并注入了`locale`属性.

  ```typescript
  import {createApp} from 'vue'
  import {createPinia} from 'pinia'
  
  import App from './App.vue'
  import router from './router'
  import Antd from 'ant-design-vue'
  import 'ant-design-vue/dist/reset.css'
  import './access.ts'
  import zhCN from 'ant-design-vue/es/locale/zh_CN' // 引入中文语言包
  import 'dayjs/locale/zh-cn' //引入dayjs的中文语言包
  
  const app = createApp(App)
  
  app.provide('locale', zhCN) // 注入locale属性
  app.use(createPinia())
  app.use(router)
  app.use(Antd)
  app.mount('#app') 
  ```
  
- 这样就可以全局配置语言了.