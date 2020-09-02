# github pages 使用

#### 推荐全局安装 docsify-cli 工具，可以方便地创建及在本地预览生成的文档。

```shell
npm i docsify-cli -g
```

#### 初始化项目

如果想在项目的 ./docs 目录里写文档，直接通过 init 初始化项目。

```shell
docsify init ./docs
```

#### 开始写文档

初始化成功后，可以看到 ./docs 目录下创建的几个文件

* index.html 入口文件
* README.md 会做为主页内容渲染
* .nojekyll 用于阻止 GitHub Pages 忽略掉下划线开头的文件

直接编辑 docs/README.md 就能更新文档内容，当然也可以添加更多页面。

对应结构

```shell
.
└── docs
    ├── README.md
    ├── guide.md
    └── zh-cn
        ├── README.md
        └── guide.md

# 对应域名

        docs/README.md        => http://domain.com
        docs/guide.md         => http://domain.com/guide
        docs/zh-cn/README.md  => http://domain.com/zh-cn/
        docs/zh-cn/guide.md   => http://domain.com/zh-cn/guide
```

#### 本地预览

通过运行 docsify serve 启动一个本地服务器，可以方便地实时预览效果。默认访问地址 http://localhost:3000 。

```shell
docsify serve docs
```

#### 部署

和 GitBook 生成的文档一样，我们可以直接把文档网站部署到 GitHub Pages 或者 VPS 上。

#### GitHub Pages

GitHub Pages 支持从三个地方读取文件

* docs/ 目录

* master 分支

* gh-pages 分支

我们推荐直接将文档放在 docs/ 目录下，在设置页面开启 GitHub Pages 功能并选择 master branch /docs folder 选项。

然后点击 save

> 可以将文档放在根目录下，然后选择 master 分支 作为文档目录。你需要在部署位置下放一个 .nojekyll 文件（比如 /docs 目录或者 gh-pages 分支）
