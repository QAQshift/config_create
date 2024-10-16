# 配置生成并插入文件

使用了vue+ts+vite+elementPlus进行开发

netstat -aon | findstr :8080
taskkill /PID 13120 /F

<!--
1.首先点击添加配置项按钮
  会出现一个国家输入，
  一个用户类型输入，
  然后有三种操作选择，分别是：发送文字内容，发送文件，发送视频图片
  选择其中一种操作以后需要输入字符串，再点击添加操作，可以添加操作进入配置项
  由于可以有多个操作，所以可以多次选择输入并添加操作。
2.根据用户输入的内容，点击添加配置项，即可添加到到config当中。
  用户添加完多个配置项以后，上传js文件，点击导出可以将config往另上传的js中进行插入，并导出。
 -->
<!-- config的格式参考如下 -->

```js
const config = {
// 国家
  "AM": {
    // 用户类型
    "a": [
    // 执行操作
      {
        // 发送图片
        "func": sendMessage,
        "params": ["2222222222"],
      },
      {
        // 发送文件
        "func": sendFile,
        "params": ["2025MaxwellEventFurnitureCatalog.pdf"],
      },
      {
        // 发送图片
        "func": sendPicture,
        "params": ["测试"],
      },
    ],
    "b": [
        {
          // 发送图片
          "func": sendMessage,
          "params": ["2222222222"],
        },
        {
          // 发送文件
          "func": sendFile,
          "params": ["2025MaxwellEventFurnitureCatalog.pdf"],
        },
        {
          // 发送图片
          "func": sendPicture,
          "params": ["测试"],
        },
      ],
  },
  "MXG":{
    // 用户类型
    "b": [
        {
          // 发送图片
          "func": sendMessage,
          "params": ["2222222222"],
        },
        {
          // 发送文件
          "func": sendFile,
          "params": ["2025MaxwellEventFurnitureCatalog.pdf"],
        },
        {
          // 发送图片
          "func": sendPicture,
          "params": ["测试"],
        },
      ]
  }
};
```
