# 如何给nwjs扩张nodejs模块

在桌面应用程序的开发过程中, 避免不了要跟硬件打交道, 这个时候, 系统提供的API就显得不够用了, 
必须要自己扩展nodejs模块来跟C++通讯,实现调用底层硬件设备的接口

## 快速开始

然而addons的实现是一个巨型大坑, 而且找不到有效的资料来填补, 
在一路的中英文对照中, 终于摸索出来一条可以用的道路, 其关键还是开发环境的部署上面

### 开发环境

* 1. win7 或者 winXP , 最好是32位的, 尽量不要太高的版本
* 2. python - v2.7版本, 最高不要超过v3.0
* 3. visual studio 2013 - 我这个版本通过了, 如果不行再尝试2010 
* 4. nodejs - v0.10.26版本, 也可以尝试别的版本, 如果别的版本不行, 那就乖乖的使用低版本
* 5. nwjs - 我用的是win32位的版本, 对应目标系统
* 6. nw-gyp - 用`npm install nw-gyp -g`装到全局
* 7. 命令行执行 `npm config set msvs_version 2013 --global`

## hello word 代码准备

### test.cpp - c++代码

```
#include <v8.h>
#include <node.h>
#include <iostream>
using namespace v8;
using namespace node;

void Method(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = Isolate::GetCurrent();
  HandleScope scope(isolate);
  args.GetReturnValue().Set(String::NewFromUtf8(isolate, "world"));
}

void init(Handle<Object> exports) {
  NODE_SET_METHOD(exports, "hello", Method);
}

NODE_MODULE(addon, init)
```

### binding.gyp - 一个json文件

```
{
    "targets": [{
        "target_name": "helloworld",
        "sources": [
            "test.cpp"
        ]
    }]
}
```

### index.html

```
<!DOCTYPE html>
<html lang="en" ng-app="CrosslightApp" ng-controller="MainController">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <title>Welcome to SimuCenter</title>
        <meta name="description" content="">
        <meta name="viewport" content="width=device-width, initial-scale=1">
		
		<script type="text/javascript" src="test.js"></script>
	</head>
	
	<body>
	<script type="text/javascript">
		var addon = require('helloworld');
		console.log(addon.hello());
		alert(addon.hello())
	</script>
	</body>
	
</html>
```

### package.json

```
{
  "name": "test",
  "version": "1.0.1",
  "main": "index.html"
}
```

## 编译addons

* 1. `win+r` 输入 `cmd` 启动命令行
* 2. 进入test.cpp文件所在的目录路径
* 3. `nw-gyp configure --target=<这里输入你的nwjs版本号,例如:0.12.3>`
* 4. `nw-gyp build` 进行编译, 如果成功, 会在当前目录的`build\Relase`文件夹下面找到helloworld.node
* 5. 把编译好的文件放到app项目的node_modules目录下

## 启动app

### 目录结构

app

--index.html

--package.json

--node_modules

  --helloworld.node

### 在当前目录执行`<nwjs所在目录>nw.exe .`

看到弹框, 说明成功了
