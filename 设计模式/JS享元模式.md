# 享元模式

## 认识

享元模式是一种用于性能优化的模式，享元模式的核心是运用共享技术有效支持大量细粒度的对象。

享元模式要求对象的属性划分为内部状态和外部状态。

享元模式是一种时间换空间的优化模式

## 实现享元模式

* 工厂模式创建享元对象

* 管理器记录对象相关外部状态，使外部状态通过某个钩子和共享对象联系起来

## 文件上传实例

```javascript
var Upload = function (uploadType) {
    this.uploadType = uploadType;
};
Upload.prototype.delFile = function (id) {
    uploadManager.setExternalState(id, this);
    if (this.fileSize < 3000) {
        return this.dom.parentNode.removeChild(this.dom);
    }
    if (window.confirm("确定要删除此文件吗？" + this.fileName)) {
        return this.dom.parentNode.removeChild(this.dom);
    }
};
```

```javascript
// 工厂模式创建享元对象
var uploadFactory = (function () {
    var createFlyWeightObjs = {};
    return {
        create: function (uploadType) {
            if (createFlyWeightObjs[uploadType])
                return createFlyWeightObjs[uploadType];
            return (createFlyWeightObjs[uploadType] = new Upload(uploadType));
        },
    };
})();
```

```javascript
// 管理器记录对象的外部状态
var uploadManager = (function () {
     // 保存所有upload对象外部状态
     var uploadDatabase = {};
     return {
         // 新增一个享元对象
         add: function (id, uploadType, fileName, fileSize) {
             var flyWeightObj = uploadFactory.create(uploadType);
             var dom = document.createElement("div");
             dom.innerHTML =
                 `<span>文件名称：${fileName}, 文件大小：${fileSize}</span>` +
                 `<button class="delFile">删除文件</button>`;
             dom.querySelector(".delFile").onclick = function () {
                 flyWeightObj.delFile(id);
             };
             document.body.appendChild(dom);
             // 保存所有upload对象的外部状态
             uploadDatabase[id] = {
                 fileName: fileName,
                 fileSize: fileSize,
                 dom: dom,
             };
             return flyWeightObj;
         },
         // 设置享元对象的外部状态
         setExternalState: function (id, flyWeightObj) {
             var uploadData = uploadDatabase[id];
             for (var i in uploadData) {
                 flyWeightObj[i] = uploadData[i];
             }
         },
     };
 })();
```

```javascript
var id = 0;
window.startUpload = function (uploadType, files) {
    for (var i = 0, file; (file = files[i++]); ) {
        var uploadObj = uploadManager.add(
            ++id,
            uploadType,
            file.fileName,
            file.fileSize
        );
    }
};

startUpload("plugins", [
    { fileName: "1.txt", fileSize: 1000 },
    { fileName: "2.html", fileSize: 2000 },
    { fileName: "3.php", fileSize: 3000 },
]);
```

