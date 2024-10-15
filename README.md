# Vue 3 + TypeScript + Vite

This template should help get you started developing with Vue 3 and TypeScript in Vite. The template uses Vue 3 `<script setup>` SFCs, check out the [script setup docs](https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup) to learn more.

Learn more about the recommended Project Setup and IDE Support in the [Vue Docs TypeScript Guide](https://vuejs.org/guide/typescript/overview.html#project-setup).



我需要做一个前端页面，首先点击添加配置项按钮，会出现一个国家下拉选择，一个用户类型的下拉选择，然后有三种操作选择，分别是：发送文字内容，发送文件，发送视频图片，选择其中一种操作以后需要输入字符串，由于可以有多个操作，所以可以多次选择输入。根据用户输入的内容，点击添加，即可添加到到config当中，其次添加完要在页面上显示用户刚刚添加这个配置项的内容。用户添加完多个配置项以后，点击一个按钮，点可以将config往另外一个另一个本地的js中进行插入。最后有一个按钮点击可以导出合并的js文件到对应文件夹中。
目前使用了vue+ts+vite+elementPlus进行开发，以上需求希望可以过程进行优化，同时要充分考虑到页面布局的美观，页面交互性要优秀。
config的格式参考如下
const config = {
// 国家
  AM: {
    // 用户类型
    a: [
    // 执行操作
      {
        // 发送图片
        func: sendMessage,
        params: ["2222222222"],
      },
      {
        // 发送文件
        func: sendFile,
        params: ["2025MaxwellEventFurnitureCatalog.pdf"],
      },
      {
        // 发送图片
        func: sendPicture,
        params: ["测试"],
      },
    ],
    b: [
        {
          // 发送图片
          func: sendMessage,
          params: ["2222222222"],
        },
        {
          // 发送文件
          func: sendFile,
          params: ["2025MaxwellEventFurnitureCatalog.pdf"],
        },
        {
          // 发送图片
          func: sendPicture,
          params: ["测试"],
        },
      ],
  },
  MXG:{
    // 用户类型
    b: [
        {
          // 发送图片
          func: sendMessage,
          params: ["2222222222"],
        },
        {
          // 发送文件
          func: sendFile,
          params: ["2025MaxwellEventFurnitureCatalog.pdf"],
        },
        {
          // 发送图片
          func: sendPicture,
          params: ["测试"],
        },
      ]
  }
};


type Action = {
  // 操作函数
  func: 'sendMessage' | 'sendFile' | 'sendPicture';
  // 参数
  params: string[];
};

type UserType = {
  [key: string]: Action[];
};

type Config = {
  [countryCode: string]: UserType;
};