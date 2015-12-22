#创建 Portlet 项目

使用 Plugins SDK 创建 portlet 非常简单。在 Plugins SDK 中有一个 portlets 目录，你的项目就应该放在这里。先给你的 portlet 起一个“项目名（projtect name）”（不能有空格）和一个“显示名（display name）”（可以有空格）。

对于 greeting portlet，项目名是 _my-greeting_，portlet 标题是 _My Greeting_ 。 起好名字后就可以开始创建项目了。可以用不同的方法来创建这个项目，先使用 Liferay Developer Studio**（因使用的是 Liferay IDE，两者应该是类似的，后面都会改为 Liferay IDE）**，然后再尝试使用命令行方式。

##使用 Liferay IDE

1. 点击菜单 File -> New -> Liferay Project。
2. 在“Project Name”和“Display Name”中分别填写“my-greeting-portlet”和“My Greeting”。
3. 保持“Use default location checkbox”选中。默认的“default location”就是当前使用的 workspace。如果你想把项目保存到其它位置，取消选项并且指定存储目录。
![图3.1：使用 Liferay IDE 创建 portlet 项目非常简单。](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=3880921d-b014-4016-9ed2-dba03275e7a4)

	图3.1：使用 Liferay IDE 创建 portlet 项目非常简单。

4. 选择“Ant(liferay-plugins-sdk)”构建方式。如果你想用 Maven 构建方式，请浏览 [Developing Plugins Using Maven](https://www.liferay.com/documentation/liferay-portal/6.2/development/-/ai/developing-plugins-using-maven-liferay-portal-6-2-dev-guide-02-en) 章节。
5. 设置好的 SDK 和 Liferay 运行环境应该已经被选中了。如果你还没有设置 Plugins SDK，点击 _Configure SDKs_ 来打开 _Install Plugin SDKs_ 向导。如果要设置运行环境，需要点击 _Liferay Portal Runtime_ 下拉菜单后面的 _New Liferay Runtime_ 来打开 _New Server Runtime Environment_ 向导。
6. 选择 _Portlet_ 插件类型。
7. 点击 _Next_。
8. 在下一个客串中，选择 _Liferay MVCframework_，然后点击 _Finish_。

可以创建一个新的 plugin 项目，也可以在现有 Liferay 项目中创建一个新的 plugin。一个 Liferay 项目可以包含多个 plugin。

## 使用命令行

*实际上和使用 IDE 创建是类似的，只不过 IDE 通过图形界面收集参数，然后帮我们去执行 ant 或 maven 了。就不翻译了。*

# 部署 Portlet 项目

Liferay 提供了一种称为 _auto-deploy_ 的机制来轻松地部署 portlet 和其它类型的 plugin 项目。只要将 war 文件放到 deploy 目录中，就会自动部署到应用服务器中（portal 会做一些处理）。本文中都会使用这种方式进行部署。

**注意：** 这种 _auto-deploy_ 的机制只在像 Tomcat、Jboss 这样的应用服务器中可用。在 Weblogic、Websphere 等应用服务器中需要使用相应的工具进行部署，这种机制就不好用了。

## 在 Liferay IDE 中部署

将 portlet 项目拖到应用服务器中。部署 plugin 项目时，应用服务器会输出一些信息表示 plugin 已经读取、注册并且已经可以使用了。

```
Reading plugin package for my-greeting-portlet
Registering portlets for my-greeting-portlet
1 portlet for my-greeting-portlet is available for use
```
需要重新部署时，右键点击应用服务器下面的相应的名称，在弹出菜单上选择 _Redeploy_ 就可以了。

## 使用命令行

*不翻译了*

*后面关于 portlet 部署成功的验证也不翻译了* 