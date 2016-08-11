# Node.js命令行工具开发教程
使用**Node.js**开发命令行工具是开发者应该掌握的一项技能，适当编写命令行工具以提高开发效率。


## hello world
老规矩第一个程序为`hello world`。在工程中新建**bin**目录，在该目录下创建名为**helper**的文件，具体内容如下：
 
```js
#!/usr/bin/env node

console.log('hello world');
```
 
修改**helper**文件的权限：

```sh
$ chmod 755 ./bin/helper 
```

执行**helper**文件，终端将会显示**hello world**：

```sh
$ ./bin/helper
hello world
```


## 符号链接
接下来我们创建一个符号链接，在全局的**node_modules**目录之中，生成一个符号链接，指向模块的本地目录，使我们可以直接使用`helper`命令。
在工程的**package.json**文件中添加**bin**字段：

```json
{
  "name": "helper",
  "bin": {
    "helper": "bin/helper"
  }
}
```

在当前工程目录下执行`npm link`命令，为当前模块创建一个符号链接：

```sh
$ npm link

/node_path/bin/helper -> /node_path/lib/node_modules/myModule/bin/helper
/node_path/lib/node_modules/myModule -> /Users/ipluser/myModule
```


现在我们可以直接使用`helper`命令：

```sh
$ helper
hello world
```


## commander模块
为了更高效的编写命令行工具，我们使用**TJ**大神的[commander](https://github.com/tj/commander.js)模块。

```sh
$ npm install --save commander
```

**helper**文件内容修改为：

```js
#!/usr/bin/env node

var program = require('commander');

program
  .version('1.0.0')
  .parse(process.argv);
```

执行`helper -h`和`helper -V`命令：

```sh
$ helper -h

 Usage: helper [options]

 Options:

  -h, --help     output usage information
  -V, --version  output the version number

$ helper -V
1.0.0
```

**commander**模块提供`-h, --help`和`-V, --version`两个内置命令。


### 创建命令
创建一个`helper hello <author>`的命令，当用户输入`helper hello ipluser`时，终端显示`hello ipluser`。修改**helper**文件内容：

```js
#!/usr/bin/env node

var program = require('commander');

program
  .version('1.0.0')
  .usage('<command> [options]')
  .command('hello', 'hello the author')  // 添加hello命令
  .parse(process.argv);
```

在**bin**目录下新建**helper-hello**文件：

```js
#!/usr/bin/env node

console.log('hello author');
```

执行`helper hello`命令：

```sh
$ helper hello ipluser
hello author
```

#### 解析输入信息
我们希望**author**是由用户输入的，终端应该显示为`hello ipluser`。修改**helper-hello**文件内容，解析用户输入信息：

```js
#!/usr/bin/env node

var program = require('commander');

program.parse(process.argv);

const author = program.args[0];

console.log('hello', author);
```

再执行`helper hello ipluser`命令：

```sh
$ helper hello ipluser
hello ipluser
```

哦耶，终于达到完成了，但作为程序员，这还远远不够。当用户没有输入`author`时，我们希望终端能提醒用户输入信息。

#### 提示信息
在**helper-hello**文件中添加提示信息：

```js
#!/usr/bin/env node

var program = require('commander');

program.usage('<author>');

// 用户输入`helper hello -h`或`helper hello --helper`时，显示命令使用例子
program.on('--help', function() {
  console.log('  Examples:');
  console.log('    $ helper hello ipluser');
  console.log();
});

program.parse(process.argv);
(program.args.length < 1) && program.help();  // 用户没有输入信息时，调用`help`方法显示帮助信息

const author = program.args[0];

console.log('hello', author);
``` 

执行`helper hello`或`helper hello -h`命令，终端将会显示帮助信息：

```sh
$ helper hello

 Usage: helper-hello <author>

 Options:

  -h, --help  output usage information

 Examples:
  $ helper hello ipluser

$ helper hello -h

 Usage: helper-hello <author>

 Options:

  -h, --help  output usage information

 Examples:
  $ helper hello ipluser

```

到此我们编写了一个`helper`命令行工具，并且具有`helper hello <author>`命令。
更多的使用方式可以参考[TJ - commander.js](https://github.com/tj/commander.js)文档。


## 关键知识点
> [npm link](http://javascript.ruanyifeng.com/nodejs/npm.html#toc17)
>>ruanyifeng

> [commander.js](https://github.com/tj/commander.js)
>>TJ


