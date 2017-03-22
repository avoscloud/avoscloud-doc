{% set release = "[Github releases 页面](https://releases.leanapp.cn/#/leancloud/lean-cli/releases)" %}
{% set login = "lean login" %}

# 命令行工具 CLI 使用指南

命令行工具是用来管理和部署云引擎项目的工具。它不仅可以部署、发布和回滚云引擎代码，对同一个云引擎项目做多应用管理，还能查看云引擎日志，批量将文件上传到 LeanCloud 云端。

## 安装命令行工具

### macOS

使用 [Homebrew](http://brew.sh/) 进行安装：

```sh
brew update
brew install lean-cli
```

如果之前使用 `npm` 安装过旧版本的命令行工具，为了避免与新版本产生冲突，建议使用 `npm uninstall -g leancloud-cli` 卸载旧版本命令行工具。或者直接按照 `homebrew` 的提示，执行 `brew link --overwrite lean-cli` 覆盖掉之前的 `lean` 命令来解决。

### Windows

Windows 用户可以在 {{release}} 根据操作系统版本下载最新的 32 位 或 64 位 **msi** 安装包进行安装，安装成功之后在 Windows 命令提示符（或 PowerShell）下直接输入 `lean` 命令即可使用。

也可以选择编译好的绿色版 **exe** 文件，下载后将此文件更名为 `lean.exe`，并将其路径加入到系统 **PATH** 环境变量（[设置方法](https://www.java.com/zh_CN/download/help/path.xml)）中去。这样使用时在 Windows 命令提示符（或 PowerShell）下，在任意目录下输入 `lean` 就可以使用命令行工具了。当然也可以将此文件直接放到已经在 PATH 环境变量中声明的任意目录中去，比如 `C:\Windows\System32` 中。

### Linux

从 {{release}} 下载预编译好的二进制文件 `lean_linux_amd64`，重命名为 `lean` 并放到 已经在 PATH 环境变量中声明的任意目录中即可。

### 通过源码安装

请参考项目源码 [README](https://github.com/leancloud/lean-cli)。

## 使用

安装成功之后，直接在 terminal 终端运行 `lean -h`，输出帮助信息：

```sh
$ lean help


 _                        ______ _                 _
| |                      / _____) |               | |
| |      ____ ____ ____ | /     | | ___  _   _  _ | |
| |     / _  ) _  |  _ \| |     | |/ _ \| | | |/ || |
| |____( (/ ( ( | | | | | \_____| | |_| | |_| ( (_| |
|_______)____)_||_|_| |_|\______)_|\___/ \____|\____|
NAME:
   lean - Command line to manage and deploy LeanCloud apps

USAGE:
   lean [global options] command [command options] [arguments...]

VERSION:
   0.7.0

COMMANDS:
     login     登录 LeanCloud 账户
     info      查看当前登录用户以及应用信息
     up        本地启动云引擎应用
     init      初始化云引擎项目
     switch    切换当前项目关联的 LeanCloud 应用
     deploy    部署云引擎项目到服务器
     publish   部署当前预备环境的代码至生产环境
     upload    上传文件到当前应用 File 表
     logs      查看 LeanEngine 产生的日志
     env       输出运行当前云引擎应用所需要的环境变量
     cache     LeanCache 管理相关功能
     cql       进入 CQL 交互查询
     search    根据关键词查询开发文档
     help, h   显示全部命令或者某个子命令的帮助

GLOBAL OPTIONS:
   --version, -v  print the version
```

可以通过 `--version` 选项查看版本：


```sh
$ lean --version
lean version 0.3.0
```

下文中凡是以 `$ lean` 开头的文字即表示在终端里执行命令。

## 登录

安装完命令行工具之后，首先第一步需要登录 LeanCloud 账户。

```sh
# {% if node != 'qcloud' %}美国节点用户需要使用参数 `--region=US` 进行登录。{% else %}腾讯云 TAB 的用户需要使用参数 `--region=TAB` 进行登录。{% endif %}
$ {{ login }} {% if node == 'us' %}--region=US{% endif %}{% if node == 'qcloud' %}--region=TAB{% endif %}
```
然后按照提示输入 LeanCloud 用户名和密码完成登录。

以 GitHub、微博或 QQ 这种第三方登录方式来注册 LeanCloud 账户的用户，如果未曾设置过账户密码，需要先使用 [忘记密码](/dashboard/login.html#/forgotpass) 功能重新设置一个密码，再进行登录。

### 切换账户

要切换到另一账户，重新执行 `{{ login }}` 即可。

## 初始化项目

登录完成之后，可以使用 `init` 命令来初始化一个项目，并且关联到已有的 LeanCloud 应用上。

```sh
$ lean init
[?] 请选择应用节点
 1) 国内
 2) 美国
 3) TAB
 =>
```

选择项目节点，然后会列示出所选节点上当前用户的所有应用：

```sh
[?] 请选择 APP
 1) AwesomeApp
 2) Foobar
```

选择项目语言／框架：

```sh
[?] 请选择需要创建的应用模版：
 1) node-js-getting-started
 2) python-getting-started
 3) slim-getting-started
 4) java-war-getting-started
 5) django-getting-started
 6) static-getting-started
```

之后命令行工具会将此项目模版下载到本地，这样初始化就完成了：

```sh
 ✓ 下载模版文件 5.93 KB / 5.93 KB [=======================================] 100.00% 0s
 ✓ 正在创建项目...
```

进入以应用名命名的目录就可以看到新建立的项目。

## 关联已有项目

如果已经使用其他方法创建好了项目，可以直接在项目目录执行：

```sh
$ lean checkout
```
将已有项目关联到 LeanCloud 应用上。


## 本地运行

如果想将一份代码简单地部署到服务器而不在本地运行和调试，可以暂时跳过此章节。

进入项目目录：

```sh
$ cd AwesomeApp
```

之后需要安装此项目相关的依赖，需要根据项目语言来查看不同文档：

- [Python](leanengine_webhosting_guide-python.html#本地运行和调试)
- [Node.js](leanengine_webhosting_guide-node.html#本地运行和调试)
- [PHP](leanengine_webhosting_guide-php.html#本地运行和调试)
- [Java](leanengine_webhosting_guide-java.html#命令行工具)

启动应用：

```sh
$ lean up
```

- 在浏览器中打开 <http://localhost:3000>，进入 web 应用的首页。
- 在浏览器中打开 <http://localhost:3001>，进入云引擎云函数和 Hook 函数调试界面。

<div class="callout callout-info">
  <ul>
    <li>如果想变更启动端口号，可以使用 `lean up --port 新端口号` 命令来指定。</li>
    <li>命令行工具的所有命令都可以通过 `-h` 参数来查看详细的参数说明信息，比如 `lean up -h`。</li>
  </ul>
</div>

旧版命令行工具可以在 `$ lean up` 的过程中，监测项目文件的变更，实现自动重启开发服务进程。新版命令行工具移除了这一功能，转由项目代码本身来实现，以便更好地与项目使用的编程语言或框架集成。

- 使用旧版命令行工具创建的 Node.js 项目，请参考 [Pull Request #26](https://github.com/leancloud/node-js-getting-started/pull/26/files) 来配置。
- 使用旧版命令行工具创建的 Python 项目，请参考 [Pull Request #12](https://github.com/leancloud/python-getting-started/pull/12/files) 来配置。

除了使用命令行工具来启动项目之外，还可以**原生地**启动项目，比如直接使用 `node server.js` 或者 `python wsgi.py`。这样能够将云引擎开发流程更好地集成到开发者管用的工作流程中，也可以直接和 IDE 集成。但是直接使用命令行工具创建的云引擎项目，默认会依赖一些环境变量，因此需要提前设置好这些环境变量。

使用命令 `lean env` 可以显示出这些环境变量，手动在当前终端中设置好之后，就可以不依赖命令行工具来启动项目了。另外使用兼容 `sh` shell 的用户，还可以直接使用 `eval $(lean env)`，自动设置好所有的环境变量。

更多关于云引擎开发的内容，请参考 [云引擎服务总览](leanengine_overview.html)。

## 部署

### 从本地代码部署

当开发和本地测试云引擎项目通过后，你可以直接将本地源码推送到 LeanCloud 云引擎平台运行：

```sh
$ lean deploy
```

对于使用了<u>免费版</u>云引擎的应用，这个命令会将本地源码部署到线上的生产环境，无条件覆盖之前的代码（无论是从本地仓库部署、Git 部署还是在线定义）；而对于使用了<u>专业版</u>云引擎的应用，这个命令会先部署到**预备环境**，后续需要使用 `lean publish` 来完成向生产环境的部署。

部署过程会实时打印进度：

```sh
 ✓ 获取应用信息
 ✓ 准备部署至目标应用：AwesomeApp (xxxxxx)
 ✓ 获取应用分组信息
 ✓ 准备部署应用到生产环境: web
 ✓ 检测到 Python 运行时
 ✓ 压缩项目文件
 ✓ 上传应用文件 6.41 KB / 6.41 KB [=======================================] 100.00% 0s
 ✓ 开始构建 20161021-171836
 ✓ 正在下载应用代码 ...
 ✓ 正在解压缩应用代码 ...
 ✓ 运行环境: python (leanengine/python-base-2.7)
 ✓ 从之前的构建中恢复依赖项 ...
 ✓ 正在下载和安装依赖项 ...
 ✓ 缓存最新的依赖项 ...
 ✓ 存储镜像到仓库 ...
 ✓ 镜像构建完成：20161021-171836
 ✓ 开始部署 20161021-171836 到 web1
 ✓ 正在创建新实例 ...
 ✓ 正在启动新实例 ...
 ✓ 实例启动成功：{"version": "1.6.5", "runtime": "cpython-2.7.6"}
 ✓ 正在统一切换新旧实例 ...
 ✓ 正在更新云函数信息 ...
 ✓ 部署完成：1 个实例部署成功
 ✓ 删除临时文件
```

默认部署备注为「从命令行工具构建」，显示在 [应用控制台 > 云引擎 > 日志](/cloud.html?appid={{appid}}#/log) 中。你可以通过 `-m` 选项来自定义部署的备注信息：

```sh
$ lean deploy -m 'Be more awesome! 这是定制的部署备注'
```

部署之后可以通过 curl 命令来测试你的云引擎代码，或者访问你已设置的二级域名的测试地址 `stg-${应用的域名}.leanapp.cn`。

#### 部署时忽略部分文件

部署项目时，如果有一些临时文件或是项目源码管理软件用到的文件，不需要上传到服务器，可以将它们加入到 `.leanignore` 文件。

`.leanignore` 文件格式与 Git 使用的 `.gitignore` 格式基本相同，每行写一个忽略项，可以是文件或者文件夹。如果项目没有 `.leanignore` 文件，部署时会根据当前项目所使用的语言创建一个默认的 `.leanignore` 文件。请确认此文件中的默认配置是否与项目需求相符。

### 从 Git 仓库部署

如果代码保存在某个 Git 仓库上，例如 [Github](https://github.com)，并且在 LeanCloud 控制台已经正确设置了 git repo 地址以及 deploy key，你也可以请求 LeanCloud 平台从 Git 仓库获取源码并自动部署。这个操作可以在云引擎的部署菜单里完成，也可以在本地执行：

```sh
$ lean deploy -g
```

- `-g` 选项要求从 Git 仓库部署，Git 仓库地址必须已经在云引擎菜单中保存。
- 默认部署使用 **master** 分支的最新代码，你可以通过 `-r <revision>` 来指定部署特定的 commit 或者 branch。
- 设置 git repo 地址以及 deploy key 的方法可以参考[云引擎网站托管指南 · Git 部署](leanengine_webhosting_guide-node.html#Git_部署)。

## 发布到生产环境

以下步骤仅适用于 [专业版云引擎](leanengine_plan.html#专业版) 用户。

如果预备环境如果测试没有问题，此时需要将预备环境的云引擎代码切换到生产环境，可以在 [应用控制台 > 云引擎 > 部署](cloud.html?appid={{appid}}#/deploy) 中发布，也可以直接运行 `publish` 命令：

```sh
$ lean publish
```

这样预备环境的云引擎代码就发布到了生产环境：

```sh
 ✓ 获取应用信息
 ✓ 准备部署至目标应用：AwesomeApp (xxxxxx)
 ✓ 开始部署 20161021-173118 到 web1,web2
 ✓ 正在创建新实例 ...
 ✓ 正在创建新实例 ...
 ✓ 正在启动新实例 ...
 ✓ 实例启动成功：{"version": "1.6.5", "runtime": "cpython-3.5.1"}
 ✓ 正在启动新实例 ...
 ✓ 实例启动成功：{"version": "1.6.5", "runtime": "cpython-3.5.1"}
 ✓ 正在统一切换新旧实例 ...
 ✓ 正在更新云函数信息 ...
 ✓ 部署完成：2 个实例部署成功
```

## 查看日志

使用 `logs` 命令可以查询云引擎的最新日志：

```sh
$ lean logs
2016-05-16 16:03:53 [PROD] [INFO]
2016-05-16 16:03:53 [PROD] [INFO] > playground@1.0.0 start /home/leanengine/app
2016-05-16 16:03:53 [PROD] [INFO] > node server.js
2016-05-16 16:03:53 [PROD] [INFO]
2016-05-16 16:03:54 [PROD] [INFO] Node app is running, port: 3000
2016-05-16 16:03:54 [PROD] [INFO] connected to redis server
2016-05-16 16:03:54 [PROD] [INFO] 实例启动成功：{"runtime":"nodejs-v4.4.3","version":"0.4.0"}
2016-05-16 16:03:54 [PROD] [INFO] 正在统一切换新旧实例 ...
2016-05-16 16:03:55 [PROD] [INFO] 正在更新云函数信息 ...
2016-05-16 16:03:55 [PROD] [INFO] 部署完成：2 个实例部署成功
```

默认返回最新的 30 条，最新的在最下面。

可以通过 `-l` 选项设定返回的日志数目，例如返回最近的 100 条：

```sh
$ lean logs -l 100
```

也可以加上 `-f` 选项来自动滚动更新日志，类似 `tail -f` 命令的效果：

```sh
$ lean logs -f
```

新的云引擎日志产生后，都会被自动填充到屏幕下方。

## 多应用管理

一个项目的代码可以同时部署到多个 LeanCloud 应用上。

### 查看当前应用状态

使用 `lean info` 可以查看当前项目关联的应用：

```sh
$ lean info
当前登录用户: asaka (lan@leancloud.rocks)
当前目录关联应用：AwesomeApp (xxxxxx)
```

此时，执行 `deploy`、`publish`、`logs` 等命令都是针对当前被激活的应用。

### 切换应用

如果需要将当前项目切换到其他 LeanCloud 应用，可以通过 `checkout` 命令来添加一个应用：

```sh
$ lean checkout
```

之后运行向导会给出可供切换的应用列表。

另外还可以直接执行 `$ lean checkout 其他应用的id` 来快速切换关联应用。


## 上传文件

使用 `upload` 命令既可以上传单个文件，也可以批量上传一个目录下（包括子目录）下的所有文件到 LeanCloud 云端。

```sh
$ lean upload public/index.html
Uploads /Users/dennis/programming/avos/new_app/public/index.html successfully at: http://ac-7104en0u.qiniudn.com/f9e13e69-10a2-1742-5e5a-8e71de75b9fc.html
```

文件上传成功后会自动生成在 LeanCloud 云端的 URL，即上例中 `successfully at:` 之后的信息。

上传 images 目录下的所有文件：

```sh
$ lean upload images/
```

## CQL 交互查询

可以通过 `$ lean cql` 命令来使用 [CQL](cql_guide.html) 语言查询存储服务数据：

```
$ lean cql
CQL > select objectId, mime_type, createdAt, updatedAt from _File where mime_type != null limit 10;
objectId                   mime_type                                   createdAt                  updatedAt
5583bc44e4b0ef6154cb1b9e   application/zip, application/octet-stream   2015-06-19T06:52:52.106Z   2015-06-19T06:52:52.106Z
559a63bee4b0c4d3e72432f6   application/zip, application/octet-stream   2015-07-06T11:17:18.885Z   2015-07-06T11:17:18.885Z
55cc4d3b60b28da5fc3af7c5   image/jpeg                                  2015-08-13T07:54:35.119Z   2015-08-13T07:54:35.119Z
55cc4d7660b2d1408c770cde   image/jpeg                                  2015-08-13T07:55:34.496Z   2015-08-13T07:55:34.496Z
55cc4df460b2c0a2834d63e2   image/jpeg                                  2015-08-13T07:57:40.013Z   2015-08-13T07:57:40.013Z
55cc4eb660b2597462bc093e   image/jpeg                                  2015-08-13T08:00:54.983Z   2015-08-13T08:00:54.983Z
55cc4ece60b2597462bc0e06   image/jpeg                                  2015-08-13T08:01:18.323Z   2015-08-13T08:01:18.323Z
563b2fc360b216575c579204   application/zip, application/octet-stream   2015-11-05T10:30:27.721Z   2015-11-05T10:30:27.721Z
564ae21400b0ee7f5ca4e11a   application/zip, application/octet-stream   2015-11-17T08:15:16.951Z   2015-11-17T08:15:16.951Z
564da57360b2ed36207ad273   text/plain                                  2015-11-19T10:33:23.854Z   2015-11-19T10:33:23.854Z
```

如果需要查询的 Class 有大量 Object / Array 等嵌套的数据结构，但以上的表格形式不便于查看结果，可以尝试用 `$ lean cql --format=json` 将结果以 JSON 格式来展示：

```
$ lean cql --format=json
CQL > select objectId, mime_type from _File where mime_type != null limit 3;
[
  {
    "createdAt": "2015-06-19T06:52:52.106Z",
    "mime_type": "application/zip, application/octet-stream",
    "objectId": "5583bc44e4b0ef6154cb1b9e",
    "updatedAt": "2015-06-19T06:52:52.106Z"
  },
  {
    "createdAt": "2015-07-06T11:17:18.885Z",
    "mime_type": "application/zip, application/octet-stream",
    "objectId": "559a63bee4b0c4d3e72432f6",
    "updatedAt": "2015-07-06T11:17:18.885Z"
  },
  {
    "createdAt": "2015-08-13T07:54:35.119Z",
    "mime_type": "image/jpeg",
    "objectId": "55cc4d3b60b28da5fc3af7c5",
    "updatedAt": "2015-08-13T07:54:35.119Z"
  }
]
```

## 其他命令

使用 `search` 命令可以方便地查询文档和资料：

```sh
$ lean search AVObject
```

这将打开浏览器，显示 [搜索结果](/search.html?q=AVObject)。

也可以查询多个关键字，空格隔开即可：

```sh
$ lean search 云引擎 命令行
```

## 贡献

`lean-cli` 是开源项目，基于 [Apache](https://github.com/leancloud/lean-cli/blob/master/LICENSE.txt) 协议，源码托管在  <https://github.com/leancloud/lean-cli>，欢迎大家贡献。
