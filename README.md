# Garwer的小笔记

本项目内容为我的java笔记，用于整理自己的学习笔记和分享,发布博客

本项目基于 [Docsify](https://docsify.js.org) 使用[advanced-java](https://github.com/doocs/advanced-java)作为模板进行构建，并同步部署（这里用到 [Gitee Pages Action](https://github.com/yanglbme/gitee-pages-action) 自动部署工具，非常好用的一个开源工具，欢迎 Star 关注）

如果你同时希望在本地查看，请按照以下步骤进行操作：

1. 安装 NodeJS 环境：https://nodejs.org/zh-cn/
2. 安装 Docsify：`npm i docsify-cli -g`
3. 使用 Git 克隆项目到你的本地环境：`git clone git@github.com:garwer/garwer-java.git`
4. 进入 garwer-java 根目录：`cd garwer-java`
5. 执行命令，启动一个本地服务器：`docsify serve`
6. 浏览器访问地址：http://localhost:3000

##### 如需要自动生成pdf的功能，可以添加以下依赖后执行 docsify-pdf-converter 相关命令 
"devDependencies": {
    "docsify-pdf-converter": "2.0.7"
},