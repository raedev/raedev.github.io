# RAE 博客网站

我的个人博客网站，基于[hexo](https://hexo.io)，在第一次下载之前确保已经安装了[npm](https://www.npmjs.com/get-npm)环境。


> 安装主题

采用[Next](http://theme-next.iissnan.com/)主题，其中`_config.yml`包含了一些敏感的配置信息，需要单独配置。

`$ cd your-hexo-site`
`$ git clone https://github.com/iissnan/hexo-theme-next themes/next`


> 初始化

`$ npm install`

> 运行

`$ hexo server`

执行命令后如无意外就可以正常访问`http://localhost:4000`

> 编写

`$ hexo new [layout] <title> `

执行完命令之后会在`~\source\_posts` 生成md文件，敬请编写吧，边写边预览。 
