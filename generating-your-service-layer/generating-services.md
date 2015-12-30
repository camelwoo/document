#生成服务

可以用 Liferay IDE、Liferay Developer Studio 或者命令行（终端窗口）来生成 service.xml 中定义的服务。下面，生成前面章节已经开发的 Event Listing 示例项目的服务代码。项目在 Plugins SDK 的 portlets/event-listing-portlet 目录。

``
注意：Windows 系统中，Liferay Portal 必须和 Plugins SDK 在同一个分区。比如，如果 Liferay Portal 在 C:\ 分区，要想正确运行 Service Builder，Plugins SDK 也必须在 C:\ 分区。
``

##使用 Liferay IDE 或 Liferay Developer Stuido

从 _Package Explorer_ 中打开 event-listing-portlet/docroot/WEB-INF 目录下的 service.xml 文件。这个文件默认会用 Service Builder 编辑器打开，确认显示 Overview 视图。然后点击视图右上角附件的 _Build Services_ 按钮。_Build Services_ 按钮是一个内容为 010 的文档图标。

确认点击的是 _Build Services_ 按钮而不是旁边的 _Build WSDD_ 按钮。生成 WSDD 并不会破坏什么，只不过生成的是远程服务而不是本地服务。关于 WSDD（web service deployment descriptors）的信息参考后面的 Liferay 远程服务章节。

![service-xml-overview.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=84bf3a8c-e0f3-4528-a23b-5968f76d18d5)

图 5.4：编辑器的 Overview 视图提供了可以展开的大纲，一个编辑基本 Service Builder 属性的表单，一个生成服务的按钮和生成 WSDD 的按钮。

运行完 Service Builder，Plugins SDK 会显示生成的文件列表和 _BUILD SUCCESSFUL_ 状态信息，还有生成文件的一些相关信息。（译注：原文中描述似乎有点问题。）

##使用终端（命令行）

打开一个终端窗口，转到 _portlets/event-listing-project-portlet_ 目录，并且输入以下命令：

```
ant build-service
```

生成成功后，会显示 _BUILD SUCCESSFUL_ 信息。（译注：其实出现这个信息并不能说明一定生成成功了，有时中间出现异常，也会显示这个提示信息。）在项目中还会看到生成的大量文件。别看这么多文件，其中只有三个是需要修改的。（译注：现在好像看不到太多文件了，好多不允许修改的类都打包到 jar 中了。）在 _EventLocalServiceImpl_ 中添加自定义方法后，再来看看 Service Builder 生成的文件，然后从 _EvnetPortlet_ 中调用它。

现在在 _EventLocalServiceImpl_ 中添加一些本地服务方法，然后学习如何调用它们。本章的后面还会为 _EventServiceImpl_ 添加一些远程服务方法，并学习如何调用它们。