# 组合模式

## 认识

用小的子对象来构建更大的对象，而这些小的子对象本身也许是由更小的“孙对象”构成的。

组合模式最大的优点在于可以一致地对待组合对象和基本对象

## 基本案例

```javascript
var getCommand = function () {
    return {
        tasklist: [],
        add: function (task) {
            this.tasklist.push(task)
        },
        cale: function () {
            for (var i = 0, task; task = this.tasklist[i++];) {
                task.cale();
            }
        }
    }
}

// 获取a
var geta1 =  {
    cale: function() {
        console.log('我是a1')
    }
}
var geta = {
    cale: function() {
        console.log('我是a');
    },
    add: function() {
        throw Error('叶节点不能添加子节点')
    }
}
geta.add(geta1);


// 获取b
var getb1 =  {
    cale: function() {
        console.log('我是b1')
    }
}
var getb2 =  {
    cale: function() {
        console.log('我是b2')
    }
}
var getb = getCommand();
getb.add(getb1)
getb.add(getb2)


// 获取c
var getc1 =  {
    cale: function() {
        console.log('我是c1')
    }
}
var getc2 =  {
    cale: function() {
        console.log('我是c2')
    }
}
var getc3 =  {
    cale: function() {
        console.log('我是c3')
    }
}
var getc = getCommand();
getc.add(getc1)
getc.add(getc2)
getc.add(getc3)

var get = getCommand()
get.add(geta)
get.add(getb)
get.add(getc)
get.cale();
```

## 扫描文件的案例

```javascript
var Folder = function (name) {
    this.name = name;
    this.parent = null;
    this.files = [];
}
Folder.prototype.add = function (file) {
    file.parent = this;
    this.files.push(file)
}
Folder.prototype.remove = function (file) {
    if (!this.parent) return;
    for (var files = this.parent.files, l = files.length - 1; l >= 0; l--) {
        var file = files[l];
        if (file == this) {
            files.splice(l, 1);
        }
    }
}
Folder.prototype.scan = function () {
    console.log('开始扫描问价' + this.name);
    for (var i = 0, file; file = this.files[i++];) {
        file.scan();
    }
}


var File = function (name) {
    this.name = name;
    this.parent = null;
}
File.prototype.add = function() {
    throw new Error('不能添加在文件底下');
}
File.prototype.remove = function() {
    if (!this.parent) {
        return;
    }
    for (var files = this.parent.files, l = files.length - 1; l >= 0; l--) {
        var file = files[l];
        if (file == this) {
            files.splice(l, 1);
        }
    }
}
File.prototype.scan = function() {
    console.log('开始扫描文件：' + this.name);
}

var folder = new Folder('学习资料')
var folder1 = new Folder('JAVASCRIPT')
var file1 = new Folder('深入浅出Javascript')
folder1.add(new File('Javascript设计模式'));
folder.add(folder1)
folder.add(file1)

folder1.remove();
folder.scan();
```

