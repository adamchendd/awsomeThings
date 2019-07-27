e# start
upsource是由jetbrains公司推出的一款代码仓库浏览器和代码审查工具，方便公司内部各团队做code review，

### upsource服务端安装
1. 官网[下载压缩包](https://www.jetbrains.com/upsource/download//)并解压；
2. 进入安装包目录\bin目录下，cmd窗口执行upsource.bat start,upsource开始启动安装，安装过程中涉及到访问url和端口配置，可以参考google参考下安装过程

### upsource用户，组和角色
1. upsource免费许可包含10个用户（1个Admin账户，1个Guest账户，8个普通用户），其中的8个普通用户可由Admin账户创建，或者通过邀请注册等；
2. upsource配置的Roles中默认存在五个角色：

角色  |  含义
--|--
System Admin  |  系统管理员角色，hub中的所有可用权限
Project Admin  | 项目管理员角色，管理项目的用户使用，默认可以更新项目，创建以及更新用户，创建，更新，删除组等  
Developer  |  开发人员角色，对应项目分配的开发人员可以查看项目基本信息，更新个人信息等
Observer  |  监控人员角色，监控项目进展的用户可使用，默认权限较小
Code Viewer  | 代码审查员角色，可提查看对应项目，交代码审查

系统管理员，项目管理员或开发人员角色无法删除，但是针对这些角色的权限重新分配，用户也可以自行创建新角色并分配对应权限。

3. 管理员创建组关联到指定项目，并针对创建的组分配角色，然后对应用户添加进组。
4. 创建用户，组等具体细节详见jetbrains官网链接：https://www.jetbrains.com/help/hub/Access-Management.html，
关注其中的Users, Groups, Roles等

### upsource创建工程
设置代码库类型，Git或Subversion，填写代码库URL，访问账号密码，测试连接，不报错即表示已成功配置好，点击Create project按钮upsource会自动同步添加项目的提交记录，测试填写如下图
 ![image](https://raw.githubusercontent.com/adamchendd/markdownPics/master/create_project.jpg)

### idea配置upsource插件
