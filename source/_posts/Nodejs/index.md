---
layout: page
title: NodeJs
tags: 后端
categories: nodejs
---

# [Nodejs](https://nodejs.org/zh-cn)

在任何地方运行 JavaScript

Node.js® 是一个免费、开源、跨平台的 JavaScript 运行时环境, 它让开发人员能够创建服务器 Web 应用、命令行工具和脚本。

## WebServer 
使用 Node.js 编写的 web服务器，响应返回 'Hello World'：
```js
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, hostname, () => {
  console.log(`服务器运行在 http://${hostname}:${port}/`);
});
```
终端运行：
```bash
$ node example.js
服务器运行在 http://127.0.0.1:3000/
```

## 模块api [https://www.nodeapp.cn/synopsis.html](https://www.nodeapp.cn/synopsis.html)
### crypto (加密)
crypto 模块提供了加密功能，包含对 OpenSSL 的哈希、HMAC、加密、解密、签名、以及验证功能的一整套封装。
使用 require('crypto') 来访问该模块。
```js
const crypto = require('crypto');

const secret = 'abcdefg';
const hash = crypto.createHmac('sha256', secret)
                   .update('I love cupcakes')
                   .digest('hex');
console.log(hash);
// Prints:
//   c0fa1bc00531bd78ef38c628449c5102aeabd49b5dc3a2a516ea6ea959d6658e
```
### Cipher
Cipher类的实例用于加密数据。这个类可以用在以下两种方法中的一种:

作为stream，既可读又可写，未加密数据的编写是为了在可读的方面生成加密的数据，或者
使用cipher.update()和cipher.final()方法产生加密的数据。
crypto.createCipher()或crypto.createCipheriv()方法用于创建Cipher实例。Cipher对象不能直接使用new关键字创建。

示例:使用Cipher对象作为流:
```js
const crypto = require('crypto');
const cipher = crypto.createCipher('aes192', 'a password');

let encrypted = '';
cipher.on('readable', () => {
  const data = cipher.read();
  if (data)
    encrypted += data.toString('hex');
});
cipher.on('end', () => {
  console.log(encrypted);
  // Prints: ca981be48e90867604588e75d04feabb63cc007a8f8ad89b10616ed84d815504
});

cipher.write('some clear text data');
cipher.end();
```

示例:使用Cipher和管道流:
```js
const crypto = require('crypto');
const fs = require('fs');
const cipher = crypto.createCipher('aes192', 'a password');

const input = fs.createReadStream('test.js');
const output = fs.createWriteStream('test.enc');

input.pipe(cipher).pipe(output);
```

示例:使用cipher.update()和cipher.final()方法:
```js
const crypto = require('crypto');
const cipher = crypto.createCipher('aes192', 'a password');

let encrypted = cipher.update('some clear text data', 'utf8', 'hex');
encrypted += cipher.final('hex');
console.log(encrypted);
// Prints: ca981be48e90867604588e75d04feabb63cc007a8f8ad89b10616ed84d815504
```