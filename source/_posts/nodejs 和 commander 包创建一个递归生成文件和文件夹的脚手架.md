---
title: nodejs 和 commander 包创建一个递归生成文件和文件夹的脚手架
date: {{ date }}
tags:  [ nodejs, commander]
categories: 前端
copyright: true
top:
description: 我们采用 nodejs 和 commander 包创建一个递归生成文件和文件夹的脚手架。可以通过 create file 和 create dir 等命令就可以创建文件或者文件夹
---

## 先看下预览：

查看帮助：

```bash
$ create -h

Usage: index [options] [command]

Options:
  -v, --version                                output the version number
  -h, --help                                   display help for command

Commands:
  dir|d <directory_name>                       create a directory
  file|f [options] <file_name> [file_content]  create a file
  help [command]                               display help for command
```

递归创建文件夹：

```bash
$ create dir test/s1
```

成功在当前文件夹下创建 test/s1 文件夹

递归创建文件：

```bash
$ create file test/s1.js
```

成功在当前文件夹下创建 test/s1.js 文件

## 使用 [commander](https://www.npmjs.com/package/commander) 包创建命令行初始化

npm 初始化，新建 index.js 和 createFile.js 文件，安装 commander 包

```bash
$ npm init -y
$ npm install commander --save

$ touch index.js
$ touch createFile.js
```

修改 package.json

```json
{
  "bin": {
    "create": "index.js"
  }
}
```

在 index.js 中写入以下代码，commander 具体属性可以参考 [这里](https://www.npmjs.com/package/commander)

```javascript
#!/usr/bin/env node

const program = require("commander");

const package = require("./package.json");

program.version(package.version, "-v, --version");
program.parse(process.argv);
```

使用 `npm link` 把包预发布到全局（ `npm link` 命令详解可以参考 [这里](https://docs.npmjs.com/cli/link.html) ）

```bash
$ npm link
```

如果提示以下结果代表预发布成功

```bash
npm WARN test@1.0.0 No description
npm WARN test@1.0.0 No repository field.

audited 1 package in 1.064s
found 0 vulnerabilities

C:\Users\Administrator\AppData\Roaming\npm\test -> C:\Users\Administrator\AppData\Roaming\npm\node_modules\test\index.js
C:\Users\Administrator\AppData\Roaming\npm\node_modules\test -> C:\Users\Administrator\Desktop\test
```

预发布成功后输入 `create -h` 则可以看到命令行帮助界面了，使用 `create -v` 即可查看版本

```bash
$ create -h

Usage: index [options]

Options:
  -v, --version  output the version number
  -h, --help     display help for command
```

## 完成创建递归文件夹的功能

在 createFile.js 文件中实现递归创建文件夹的功能

```javascript
#!/usr/bin/env node

const fs = require("fs");
const absolutePath = process.cwd().replace("\\", "/");

/* 递归创建文件夹 */
async function createDir(path, deep = true) {
  let absPath = absolutePath;

  // 文件夹命名不能包含 '.'
  const pathArr = path
    .split("/")
    .filter(Boolean)
    .map((item) => item.split(".")[0]);
  if (!deep) {
    pathArr.pop();
  }

  // 递归创建文件夹
  pathArr.forEach((p) => {
    absPath += `/${p}`;
    if (fs.existsSync(absPath)) {
      return;
    }
    fs.mkdirSync(absPath);
  });
}
module.exports = { createDir };
```

在 index.js 中注册创建递归文件夹命令

```javascript
#!/usr/bin/env node

const program = require("commander");

const package = require("./package.json");
const { createDir } = require("./createFile");

program.version(package.version, "-v, --version");

// 注册递归创建文件夹命令
program
  .command("dir <directory_name>")
  .description("create a directory")
  .alias("d")
  .action(function (directory_name) {
    createDir(directory_name);
  });

program.parse(process.argv);
```

使用 `create -h` 就可以查看注册成功了

```bash
$ create -h

Usage: index [options] [command]

Options:
  -v, --version           output the version number
  -h, --help              display help for command

Commands:
  dir|d <directory_name>  create a directory
  help [command]          display help for command

```

使用 `create dir 文件夹` 命令就可以创建文件了

## 完成创建文件的功能

在 createFile.js 中实现创建文件的功能

```javascript
#!/usr/bin/env node

const fs = require("fs");
const absolutePath = process.cwd().replace("\\", "/");

/* 递归创建文件夹 */
async function createDir(path, deep = true) {
  let absPath = absolutePath;

  // 文件夹命名不能包含 '.'
  const pathArr = path
    .split("/")
    .filter(Boolean)
    .map((item) => item.split(".")[0]);
  if (!deep) {
    pathArr.pop();
  }

  // 递归创建文件夹
  pathArr.forEach((p) => {
    absPath += `/${p}`;
    if (fs.existsSync(absPath)) {
      return;
    }
    fs.mkdirSync(absPath);
  });
}

/* 写入文件 */
async function createFile(path, data = "", deep = true) {
  // 等待检查
  await createDir(path, deep);
  const pathArr = path.split("/").filter(Boolean);
  const fileName = deep ? pathArr[pathArr.length - 1] : pathArr.pop();
  // 文件夹命名不能包含 '.'
  let absPath = `${absolutePath}/${pathArr
    .map((item) => item.split(".")[0])
    .join("/")}/${fileName}`;
  fs.writeFileSync(absPath, data, "utf8");
}

module.exports = { createDir, createFile };
```

在 index.js 中注册创建文件命令

```javascript
#!/usr/bin/env node

const program = require("commander");

const package = require("./package.json");
const { createDir, createFile } = require("./createFile");

program.version(package.version, "-v, --version");

program
  .command("dir <directory_name>")
  .description("create a directory")
  .alias("d")
  .action(function (directory_name) {
    createDir(directory_name);
  });

program
  .command("file <file_name> [file_content]")
  .description("create a file")
  .alias("f")
  .option(
    "-d, --deep",
    "file will be wraped a additional directory that named file_name"
  )
  .action(function (file_name, file_content, option) {
    createFile(file_name, file_content, !!option.deep);
  });

program.parse(process.argv);
```

使用 `create -h` 查看帮助
```bash
$ create -h

Usage: index [options] [command]

Options:
  -v, --version                                output the version number
  -h, --help                                   display help for command

Commands:
  dir|d <directory_name>                       create a directory
  file|f [options] <file_name> [file_content]  create a file
  help [command]                               display help for command
```

使用 `create file test/s1.js` 命令就可以成功创建文件啦