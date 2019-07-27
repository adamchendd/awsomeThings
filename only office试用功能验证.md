### 1. onlyoffice document server下载
验证环境为windows，需要在[onlyoffice官网]("https://www.onlyoffice.com/developer-edition-request.aspx?from=onlyoffice-integration.aspx&plan=5&utm_term=&utm_source=&utm_campaign=&utm_content=")下载windows安装包，windows安装包需要在官网填写申请，填写之后邮箱会收到下载链接，按照步骤安装即可，需要注意的是在安装过程中会安装postresql，mysql，redis等软件，如果机器无法联网会导致安装无法成功。另外，在安装过程中可以指定服务的启动端口，默认为80，若机器安装有nignx等占用了80端口的软件，可以设置另外的端口。

### 2. document sever功能试用
onlyoffice document sever主要提供在线编辑word，excel，ppt等office办公文件等功能，如果需要使用在线文件管理功能可以使用onlyoffice community sever或者自行开发前端文件管理功能。

onlyoffice官网提供了详尽的[api接口说明]("https://api.onlyoffice.com/editors/basic")，为了弄清楚onlyoffice document sever的工作流程，下载了官网提供的java example，官网提供的代码稍微有点问题，需要在IndexServlet类的Track方法中修改行：

    status = (int) jsonObj.get("status");
改为

    status = (Long) jsonObj.get("status");
通过tomcat启动服务，跳转页面可以上传office文件，点击edit后会跳转新页面，若做断点调试，在新页面加载时，IndexServlet的Track方法中会收到json报文，关于json报文的具体含义，可以参考api页面的Callback Handler章节，需要重点关注的是son报文中的status和key字段，
* status

该字段有如下返回值：

| 内容 | 含义 |
|---|---|
|0|找不到带有密钥标识符的文档|
|1|正在编辑文档|
|2|文档已准备好保存|
|3|发生了文档保存错误|
|4|文档编辑页面关闭，没有变化|
|6|正在编辑文档，但保存当前文档状态|
|7|强制保存文档时发生错误|

其中6和7状态为多人协同编辑并配置强制保存时才会返回。

* key

key为定义编辑的文档标识符，如果需要对文档进行多次编辑与保存，由于document sever会缓存每个key值对应的文件，每次请求的key值需要不同才能保证编辑文档的数据能完整无误的保存。

### 3. 其他说明
关于打开文件，协同编辑，保存文件等api文档已经有很清晰的描述，关于document server的安装使用，汉化等有很多大神给出了自己的解决方案，这里推荐一位牛人:https://github.com/3xxx

如有其他问题可以加本人WX讨论：adamchendd
