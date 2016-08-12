title: 破解startUML2
date: 2016-05-16 09:33:48
tags:
categories:
---


## StartUML2 软件示意图

<!-- more -->

![](http://7xrw2w.com1.z0.glb.clouddn.com/startUml.jpg)

### 找到安装目录下的LicenseManagerDomain.js文件

![](http://7xrw2w.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20160516093228.jpg)

### 编辑修改

```js
  ...
  function validate(PK, name, product, licenseKey) {
     var pk, decrypted;

     //add me
     return {
        name: "0xcb",
        product: "StarUML",
        licenseType: "vip",
        quantity: "coffee1993.com",
        licenseKey: "later equals never!"
    };

     try {
         pk = new NodeRSA(PK);
         decrypted = pk.decrypt(licenseKey, 'utf8');
     } catch (err) {
         return false;
     }
    ...
```

### 保存 重启StartUML2
- 输入验证
![](http://7xrw2w.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE20160516092610.jpg)


-
