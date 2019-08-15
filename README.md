<!-- <div align="center">
    <a href="https://gitstar-ranking.com/repositories"> <img src="https://badgen.net/badge/Rank/20?icon=github&color=4ab8a1"></a>
    <a href="assets/download.md"> <img src="https://badgen.net/badge/OvO/%E7%A6%BB%E7%BA%BF%E4%B8%8B%E8%BD%BD?icon=telegram&color=4ab8a1"></a>
    <a href="https://believeszw.github.io/CS-Notes"> <img src="https://badgen.net/badge/CyC/%E5%9C%A8%E7%BA%BF%E9%98%85%E8%AF%BB?icon=sourcegraph&color=4ab8a1"></a>
    <a href="#微信公众号"> <img src="https://badgen.net/badge/%e5%85%ac%e4%bc%97%e5%8f%b7/believeszw?icon=rss&color=4ab8a1"></a>
</div>
<br> -->

| 算法 | 操作系统 | 网络 | 面向对象 | 数据库 | C++ | Java | 系统设计 |  工具 | 编码实践 | 后记 | 问题记录 |
| :---: | :----: | :---: | :----: | :----: | :----: | :----: | :----: | :----: | :----: |:----:|:----:|
| [:pencil2:](#pencil2-算法) | [:computer:](#computer-操作系统) | [:cloud:](#cloud-网络) | [:art:](#art-面向对象) | [:floppy_disk:](#floppy_disk-数据库) |[:tea:](#tea-Cpp)|[:coffee:](#coffee-java)| [:bulb:](#bulb-系统设计) |[:wrench:](#wrench-工具)| [:watermelon:](#watermelon-编码实践) |[:memo:](#memo-后记)|[:book:](#book-问题记录)||

<br>

<div align="center">
    <img src="assets/LogoMakr_SzW.png" width="200px">
</div>

<br>

## :pencil2: 算法


## :computer: 操作系统


## :cloud: 网络


## :art: 面向对象


## :floppy_disk: 数据库


## :tea: Cpp

- [Qt](https://github.com/believeszw/CS-Notes/blob/master/notes/C++/Qt)
- [C++11新特性](https://github.com/believeszw/CS-Notes/blob/master/notes/C++/C++11)
- 性能提升

## :coffee: Java


## :bulb: 系统设计


## :wrench: 工具

- [Git](https://github.com/believeszw/CS-Notes/blob/master/notes/工具/Git.md)
- [Docker](https://github.com/believeszw/CS-Notes/blob/master/notes/工具/Docker.md)
- [构建工具](https://github.com/believeszw/CS-Notes/blob/master/notes/工具/构建工具.md)
- [正则表达式](https://github.com/believeszw/CS-Notes/blob/master/notes/工具/正则表达式.md)

## :watermelon: 编码实践

- [代码可读性](https://github.com/believeszw/CS-Notes/blob/master/notes/编码实践/代码可读性.md)
- [代码风格规范](https://github.com/believeszw/CS-Notes/blob/master/notes/编码实践/代码风格规范.md)

## :memo: 后记

### 更多内容

<!-- - 内推：[Job-Recommend](https://github.com/believeszw/Job-Recommend)
- 简历模版：[Markdown-Resume](https://github.com/believeszw/Markdown-Resume)
- 面经：[2018 这一年](https://www.nowcoder.com/discuss/137593)
- 简历：https://believeszw.github.io -->
<!-- - 小专栏：[后端面试进阶指南](https://xiaozhuanlan.com/believeszw) -->
<!-- - QQ 交流群：[857210598](assets/group.png) -->

### 工具

- Github Pages：[docsify](https://docsify.js.org/#/)
- 云笔记：[为知笔记](http://www.wiz.cn/)
- 绘图：[draw.io](https://www.draw.io/)
- Logo：[logomakr](https://logomakr.com/)

## :book: 问题记录
- Ringbuffer
- Queue

<!-- ### 微信公众号

更多精彩内容将发布在微信公众号 believeszw 上，你也可以在公众号后台和我交流学习和求职相关的问题。另外，公众号提供了该项目的 PDF 等离线阅读版本，后台回复 "下载" 即可领取。公众号也提供了一份技术面试复习大纲，不仅系统整理了面试知识点，而且标注了各个知识点的重要程度，从而帮你理清多而杂的面试知识点，后台回复 "大纲" 即可领取。我基本是按照这个大纲来进行复习的，对我拿到了 BAT 头条等 Offer 起到很大的帮助。你们完全可以和我一样根据大纲上列的知识点来进行复习，就不用看很多不重要的内容，也可以知道哪些内容很重要从而多安排一些复习时间。 -->
<!--
<br>

<div align="center"><img width="300px" src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/other/公众号海报6.png"></img></div> -->

### 排版

笔记内容按照 [中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines) 进行排版，以保证内容的可读性。

不使用 `![]()` 这种方式来引用图片，而是用 `<img>` 标签。一方面是为了能够控制图片以合适的大小显示，另一方面是因为 [GFM](https://github.github.com/gfm/) 不支持 `<center> ![]() </center>` 这种方法让图片居中显示，只能使用 `<div align="center"> <img src=""/> </div>` 达到居中的效果。

在线排版工具：[Text-Typesetting](https://github.com/believeszw/Text-Typesetting)。

### 上传方案

为了方便将本地笔记内容上传到 Github 上，实现了一整套自动化上传方案，包括提取图片、Markdown 文档转换、Git 同步。进行 Markdown 文档转换是因为 Github 使用的 GFM 不支持 MathJax 公式和 TOC 标记，所以需要替换 MathJax 公式为 CodeCogs 的云服务和重新生成 TOC 目录。

GFM 转换工具：[GFM-Converter](https://github.com/believeszw/GFM-Converter)。

### License

本仓库内容是在CyC2018的[CS-Notes](https://github.com/CyC2018/CS-Notes)基础上所写。在您引用本仓库内容或者对内容进行修改演绎时，请署名并以相同方式共享，谢谢。

转载文章请在开头明显处标明该页面地址，公众号等其它转载请联系 zhengyc101@163.com。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a>

### 致谢

感谢以下人员对本仓库做出的贡献，当然不仅仅只有这些贡献者，这里就不一一列举了。如果你希望被添加到这个名单中，并且提交过 Issue 或者 PR，请与我联系。

<a href="https://github.com/linw7">
    <img src="https://avatars3.githubusercontent.com/u/21679154?s=400&v=4" width="50px">
</a>
<a href="https://github.com/g10guang">
    <img src="https://avatars1.githubusercontent.com/u/18458140?s=400&v=4" width="50px">
</a>
<a href="https://github.com/Sctwang">
    <img src="https://avatars3.githubusercontent.com/u/33345444?s=400&v=4" width="50px">
</a>
<a href="https://github.com/ResolveWang">
    <img src="https://avatars1.githubusercontent.com/u/8018776?s=400&v=4" width="50px">
</a>
<a href="https://github.com/crossoverJie">
    <img src="https://avatars1.githubusercontent.com/u/15684156?s=400&v=4" width="50px">
</a>
<a href="https://github.com/jy03078584">
    <img src="https://avatars2.githubusercontent.com/u/7719370?s=400&v=4" width="50px">
</a>
<a href="https://github.com/kwongtailau">
    <img src="https://avatars0.githubusercontent.com/u/22954582?s=400&v=4" width="50px">
</a>
<a href="https://github.com/xiangflight">
    <img src="https://avatars2.githubusercontent.com/u/10072416?s=400&v=4" width="50px">
</a>
<a href="https://github.com/mafulong">
    <img src="https://avatars1.githubusercontent.com/u/24795000?s=400&v=4" width="50px">
</a>
<a href="https://github.com/yanglbme">
    <img src="https://avatars1.githubusercontent.com/u/21008209?s=400&v=4" width="50px">
</a>
<a href="https://github.com/OOCZC">
    <img src="https://avatars1.githubusercontent.com/u/11623828?s=400&v=4" width="50px">
</a>
<a href="https://github.com/5renyuebing">
    <img src="https://avatars1.githubusercontent.com/u/32872430?s=400&v=4" width="50px">
</a>
