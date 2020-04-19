# lerna

## 一 什么是lerna

lerna提供的能力与yarn的工作区类似。当一个项目（一个git仓库）要维护多个独立开发包时，就会需要lerna。这种开发方式被称为`monorepo`。

什么时候项目会要使用多个独立的开发包？比如有如下目录

```bash
m-proj
  package-Core
  package-Test
  package-Cli
# ...
```

一个稍微大型点的项目，可能需要有核心功能代码，单测代码。有些工具库，是以脚手架客户端形式存在的，那么就会需要一个cli/bin目录。

这种一个大型项目需要多个不同却又有依赖关系的包的时候，通常有两种项目组织方式：

1. MultiRepo：多个仓库，分别维护，联调时使用`yarn link`之类的方式连接在一起。
2. MonoRepo：单个仓库，集中维护，集中管理依赖，无需`link`。调试时就像一个项目一样。

目前许多大型项目都是选择的MonoRepo方式。因为一个项目存在多个包时，这些包的开发内容上相互独立，但是又相互依赖，如果分开多个仓库管理，那么链接，开发，调试，发布都很繁琐。monorepo的好处在于统一的工作流和依赖管理。

## 二 lerna的安装与启用

### 1. 安装和初始化

```bash
# 全局安装
npm install --global lerna

# 使用lerna初始化项目
npx lerna init
```

运行`npx lerna init`之后，会创建一个特殊的npm项目结构。首先，作为一个基本的npm项目，包括`package.json`文件。其次，作为lerna项目，它会默认创建一个lerna.json，以及一个默认的工作区包裹目录`packages`。

```bash
m-proj
  packages
  lerna.json
  package.json
```

### 2. lerna.json与package.json

lerna.json是lerna仓库的主要配置文件。一个初始的配置如下：

```json
{
  "packages": [
    "packages/*"
  ],
  "version": "0.0.0"
}
```

同时，package.json的配置如下：

```json
{
  "name": "lerna-demo",
  "private": true,
  "devDependencies": {
    "lerna": "^3.20.2"
  }
}
```

> lerna.json中，配置了项目的工作区packages目录，
> package.json里面配置了`private: true`，也与yarn工作区类似，表示私有项目，与发布到npm的版本解耦

## 三 基本使用

### 1. 创建package

```bash
lerna create @项目名/包名
```

例如当前项目名为lerna-demo，我想创建一个目录用来存放开发的脚手架工具的代码，那么就需要如下命令：

```bash
lerna create @lerna-demo/cli
```

运行上述命令，会在lerna.json定义的工作区目录下创建一个cli目录。并且提供一个cli工具需要的基础目录结构：

```json
cli
  __test__      // 测试文件目录
  lib           // cli工具主目录
  package.json
  README.md
```

再运行下面的命令，创建一个存放工具代码的目录:

```bash
lerna create @lerna-demo/utils
```

### 2. 添加依赖

#### (1) 添加全局依赖

```bash
lerna add <package>
```

例如全局安装commander:

```bash
lerna add <package>
```

#### (2) 给单个模块添加依赖

```bash
lerna add <package> --scope @<project>/<package>
```

例如，我们在utils模块下，安装一个chalk模块，这是npm上一个用来美化控制台输出的模块，使用如下命令：

```bash
lerna add chalk --scope @lerna-demo/utils
```

> 上述命令把chalk模块安装到指定的utils模块中。此时的依赖会被写入packages/utils/package.json文件，并且依赖也被安装在utils/node_modules目录下。

#### (3) 给项目模块之间添加依赖

```bash
lerna add @<project>/<source package> --scope @<project>/<target package>
```

例如，在我们的lerna-demo项目中，utils包提供工具方法，供cli包使用，那么就需要把utils作为依赖添加到cli中，使用如下命令:

```bash
lerna add @lerna-demo/utils --scope @lerna-demo/cli
```

> 运行上面的命令后，@lerna-demo/utils会被作为依赖加入到@lerna-demo/cli/node_modules目录下面

#### (4) 依赖管理

前面提到的安装全局依赖，通常来说，全局安装后，可能是装在根目录下的`node_modules`。但前面安装commander的结果是，对每一个packages目录下都装了commander，各自依赖各自的node_modules。这个时候，我们就需要把公共依赖从各自packages目录下提升至全局，并且各个package创建一个指向全局node_moduels的依赖链接。运行下面的命令: 

```bash
lerna bootstrap --hoist
```

bootstrap命令是用来整理所有包的依赖关系， `--hoist`选项用来把公共的npm包移动到根目录的node_modules，在各个package内部只存放一个结构与依赖包类似的链接，以便能够使用npm脚本。

为了防止每次运行bootstrap命令都要添加这个选项，可以在lerna.json中添加如下配置：

```json
{
  "packages": [
    "packages/*"
  ],
  "command": {
    "bootstrap": {
      "hoist": true
    }
  },
  "version": "0.0.0"
}
```

就算运行了上述命令，我们会发现模块下的node_modules还是会有残留，此时需要运行:

```bash
lerna clean
```

清理模块下的残留依赖，然后由于配置了`bootstrap`命令默认提供`--hoist`参数，可以直接运行`bootstrap`命令来重新引导依赖：

```bash
lerna bootstrap
```

### 3. 发布





