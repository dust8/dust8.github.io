---
title: js 直传文件到阿里云 oss
date: 2020-06-20 05:15:31
tags:
  - javascript
  - oss
---

js 从服务端获取 policy 后直传阿里云 oss.
实际并不需要用 demo 里面的插件, 原生 js 更容易理解.
服务端和客户端的 demo 都让人更糊涂, 估计是外包弄的, 水平不是一般的次.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>

  <body>
    <div>
      <input id="file" type="file" />
      <button id="upload" type="button">upload</button>
    </div>
    <script type="text/javascript">
      let button = document.getElementById("upload");
      button.addEventListener("click", (event) => {
        var formData = new FormData();
        var fileField = document.querySelector("input[type='file']");
        formData.append("success_action_status", 200);
        formData.append("OSSAccessKeyId", "");
        formData.append("policy", "");
        formData.append("signature", "");
        formData.append("key", "vid/course-video-application/a.zip");
        formData.append("file", fileField.files[0]);

        fetch("http://xx.oss-cn-shenzhen.aliyuncs.com", {
          method: "post",
          body: formData,
        });
      });
    </script>
  </body>
</html>
```

## 参考链接

- [服务端签名后直传](https://help.aliyun.com/document_detail/31926.html)
- [阿里云 oss js 直接上传方式](https://www.jianshu.com/p/0e9bf15650a7)
