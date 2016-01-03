#Portlet 本地化

如果 portlet 的受众是世界范围的，可以本地化 portlet 用户界面。要想本地化 portlet，需要为每种需要支持的语言创建语言属性文件，也叫做资源包（resource bundle）。可以手工翻译语言属性文件，或者调用 web service 翻译。plugin 项目可以方便地使用 Liferay Portal 的翻译信息。要想本地化 除了 portal 本地化信息之外的信息，必须在 plugin 项目中的一个或多个语言中创建语言 key。计划本地化 portlet 时，需要考虑以下问题。

Portal 现有的信息有你要用的吗？你的 plugin 包含多个 portlet 吗？如果是，这些 portlet 需要在控制面板（Control Panel）中使用吗？如果任何 portlet 需要在控制面板中使用，就应该为每个 portlet 创建单独的语言包。否则，portlet 应该共享语言包，这样就可以通过 Liferay IDE 和 Plugins SDK 来使用 Liferay 的语言构建（language building）工具了。下面会演示这几种情况下如何本地化 portlet。先从使用 Liferay Portal 已经本地化在其核心集中的语言码（language key）的开始。

## 使用 Liferay 的语言码

Liferay 已经在核心文件 _Language.properties_ 中定义了大量语言码。这个文件可以在 _portal-impl.jar_ 的 contnet 目录或 Liferay 源码的 _portal-impl/src/content_ 目录中找到。使用 Liferay 核心语言码可以节约时间，因为这些语言码已经翻译为多种语言。此外，你的 portlet 可以更好地和 Liferay 的用户界面融合。

在 JSP 中可以通过 _<liferay-ui:message />_ 标签来使用语言码。

```
<liferay-ui:message key="message-key" />
```

声明想要显示的信息对应 _Language.properties_ 的语言码。例如，使用用户的语言来打招呼，指定信息的语言码为 _welcome_。

```
<liferay-ui:message key="welcome" />
```

这个码映射到单词 "Welcome"，可以把它翻译为用户所在地区的语言。下面是 Liferay 的 _Language.properties_ 文件中 _welcome_ 语言码。

```
welcome=Welcome
```

在前面创建的 _my-greeting-portlet_ 的 _view.jsp_ 中的问候语前面添加 _welcome_ 语言码。把原来的问候语修改为以下内容：

```
< p><liferay-ui:message key="welcome" />! <%= greeting %></p>
```

重新访问页面来确认在原来问候语前是否有 "Welcome"（来自 _Language.properties_）。

注意，为了使用 _<liferay-ui:message />_ 标签或任意 _liferay-ui_ 标签，必须在 JSP 中包含以下内容来导入 _liferay-ui_ 标签库。

```
<%@ taglib uri="http://liferay.com/tld/ui" prefix="liferay-ui" %>
```

_<liferay-ui:message />_ 标签还支持向语言码传递字符串参数。例如，_welcome-x_ 语言码需要一个参数。以下是 _Language.properties_ 文件中的 _welcome-x_ 语言码：

```
welcome-x=Welcome{0}!
```

它引用 _{0}_，表示参数列表中的第一个参数。通过 _message_ 标签可以传递任意数量参数，但是只使用语言码需要的那些参数。例如通过 _{0}_、_{1}_ 这样的顺序来引用的参数。在“My Greeting” portlet 中把用户的昵称（screen name）作为参数传递到 _welcome-x_ 语言码。

1. 打开 _view.jsp_ 文件。
2. 在 JSP 顶部，_<portlet:defineObjects />_ 标签上面添加以下内容。第一行导入 _liferay-theme_ 标签库。第二行定义库的对象，提供访问保存用户昵称的 _user_ 对象的途径。

	```
<%@ taglib uri="http://liferay.com/tld/theme" prefix="liferay-theme"%>
<liferay-theme:defineObjects />
```
3. 将原有的 _message_ 标签和感叹号 _<liferay-ui:message key="welcome" />!_ 替换为以下内容：

	```
<liferay-ui:message key="welcome-x" arguments="<%= user.getScreenName() %>" />
```

	当刷新页面时，“My Greeting” portlet 通过昵称和你打招呼。

	![03-screen-name-greeting.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=1eae3c54-a952-49c2-a271-642b631d7aad)

	图 3.10：通过把昵称作为参数传递给 Liferay 的 _welcome-x_ 语言码，可以显示个性化的问候语。

还有 _<liferay-ui:success />_ 和 _<liferay-ui:error />_ 标签。_<liferay-ui:success />_ 显示正向的反馈信息，背景是舒适的绿色。_<liferay-ui:error />_ 用来在输入错误或异常的情况时显示获信息。错误信息使用醒目的红色背景。

如果在 _SessionMessages_ 对象中找到相应的信息就会触发 _<liferay-ui:success />_ 标签。之前创建的 _MyGreetingPortlet_ 类中，用以下方式添加了 _SessionMessages_ 对象来显示操作成功的信息 _<liferay-ui:success key="success" ... />_：

```
SessionMessages.add(actionRequest, "success");
```

类似地，_<liferay-ui:error />_ 标签是在 _SessionErrors_ 对象中有相信信息时触发的。在 _MyGreetingPortlet_ 类中添加以下代码来显示错误信息 _<liferay-ui:error key="error" ... />_：

```
SessionErrors.add(actionRequest, "error");
```

使用 Liferay 的核心本地化语言码，以上这些就够用了。如果需要添加本地化语言码，按照以下方式给用户提供定制的本地化 portlet。

##共享语言码

使用自定义的语言码和使用 Liferay 的核心语言码类似，需要在 plugin 中添加用到一个或多个资源包。如果某个 portlet 要在控制面板中使用，而且需要本地化它在控制面板中的标题和描述，这个 portlet 最好使用单独的资源包。如果 portlet 都不在控制面板中使用，它们就可以共享同一个资源包。下面先说明 portlet 之间共享资源包。

给第二章中创建的 _event-listing-portlet_ plugin 项目添加一个资源包：

1. 在 _doocroot/WEB-INF/src_ 目录中创建 _content_ 包。
2. 在 _content_ 包中添加 _Language.properties_ 文件，在文件中添加以下语言码：

	```
your-nose-knows-best=Your nose knows best
```
3. 在 _content_ 包中添加另一个语言码文件 _Language_es.properties_，在文件中添加 _your-nose-knows-best_ 码翻译为西班牙语的语言码：

	```
your-nose-knows-best=La nariz sabe mejor
```
4. 在每个 portlet 的 _view.jsp_ 中最后一个包含文件指令下面（如，在 _<%@include file="/html/init.jsp" %>_ 下面）添加下面这行内容。它为 JSP 带来翻译好的语言码：

	```
Nose-ster - <liferay-ui:message key="your-nose-knows-best" />!
```
5. 修改这些 portlet 在 _portlet.xml_ 文件中的 _<portlet>_ 节点，让它们引用同一个资源包：

	```
<portlet>
    <portlet-name>eventlisting</portlet-name>
    ...
    <resource-bundle>content.Language</resource-bundle>
    <portlet-info>...</portlet-info>
    ...
</portlet>
<portlet>
    <portlet-name>locationlisting</portlet-name>
    ...
    <resource-bundle>content.Language</resource-bundle>
    <portlet-info>...</portlet-info>
    ...
</portlet>
```
	确保每个 _resource-bundle_ 元素放在了 _portlet_ 元素中正确的位置。查看 _portlet.xml_ 的纲要（schema）http://java.sun.com/xml/ns/portlet/portlet-app_2_0.xsd 获取详细信息。
6. 重新部署 plugin，转到添加 _Event Lising_ 和 _Location Listing_ portlet 的页面，验证显示的是不是“Nose-ster - Your nose knows best!”。
7. 在 URL 地址 localhost:8080 后面添加 _/es_，重新加载页面，这样会把 portal 的地区（locale）切换为西班牙。注意两个 portlet 显示的是翻译好的语言码。

	![portlet-localization-shared-bundle-spanish.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=379bf2b5-df01-49e7-9b89-f0716e32b66d)

	图 3.11：多个 portlet 共享资源包可以使用常用的翻译文本。Liferay IDE 的语言构建工具协助你使用翻译服务。

任何 portlet 都可以使用 _Language.properties_ 文件中声明的所有语言码。

``
注意：语言包和目录最好使用 Liferay 的命名规则，这样 portlet 可以访问包，还可以使用 Liferay IDE 和 Plugins SDK 的语言构建工具。
``

在本地化控制页面中的 portlet 之前，先学习如何快捷地生成语言码文件并将码翻译为需要的语言。

##生成语言文件并自动翻译

为了让用户看到他自己语言的信息，信息值必须在一个文件名以表示用户所在地区的两个字符结尾的语言包文件中定义。例如，一个名为 _Language_es.properties_ 的资源包文件包含的 _welcome_ 信息码必须对应单词“Welcome”翻译为西班牙文字。不用担心，Plugins SDK 提供了一个翻译默认的资源包的方法。

Plugins SDK 使用 _Bing_ 翻译服务 http://www.microsofttranslator.com/ 将 _Language.properties_ 文件中的全部资源翻译成多种语言。它提供了基本的翻译作为初始内容。要使用 Bing 翻译服务完成基本翻译，需要完成以下步骤：

1. 登录 Azure 市场帐号并且注册你的应用。记下分配给你的 _client ID_ 和 _client secret_。
2. 确认在 Plugins SDK 根目录中有正确的 _build.[username].properties_ 文件。这个文件中有 Liferay 包的引用。例如，如果有一个 Liferay Tomcat 的包，文件中的引用类似以下内容：

	```
app.server.dir=[Liferay Home]/tomcat-7.0.42
auto.deploy.dir=[Liferay Home]/deploy
```
	_[Liferay Home]_ 代表你的包的根目录。
3. 编辑 Liferay 主目录中的 _portal-ext.properties_ 文件，添加下面两行，并用你自己会值替换相应的值：

	```
microsoft.translator.client.id=your-[client-id]
microsoft.translator.client.secret=your-[client-secret]
```
	Liferay 根据配置把主目录中的 _portal-ext.properties_ 文件复制到 _tomcat-[version]/webapps/ROOT/WEB-INF/classes_ 目录。因此，可以手动复制文件或者通过启动 Liferay 进行复制。
4. 编辑想要进行翻译的 plugin 的 _Language.properties_ 文件。例如，如果在 Plugins SDK 中有一个 _hello-world_ portlet，就应该编辑以下文件：

	```
[Liferay Plugins SDK]/portlets/hello-world-portlet/docroot/WEB-INF/src/content/Language.properties
```
	可以增加、删除或编辑属性，但*不会*翻译已经存在的属性。
5. 在需要翻译的 plugin 的 _plugin_ 目录中执行 _ant build-lang_。例如，是 _hello-world_ 示例 portlet，应该在 _[Liferay Plugins SDK]/portlets/hello-world-portlet_ 目录中运行 _ant build-lang_。

当构建完成时，在 _language.properties_ 文件所在目录中会看到所有翻译好的文件。

``
注意：因为翻译不会生成已存在的属性，如果需要编辑已存在的属性就需要做两步。首先从 _Language.properties_ 文件中删除对应的属性，并且运行 _ant build-lang_ 来删除所有其它资源包中对应的属性。然后重新添加属性和新值，并且再运行一次 _ant build-lang_。现在微软的翻译机就会生成新的翻译了。
`` 

``
注意：如果使用 Maven 构建项目，需要将 _content_ 目录复制到 portlet 的 _src/main/webapp/WEB-INF/classes_ 目录。
``

使用 Plugins SDK 的语言构建工具，可以同步翻译默认的 _Language.properties_ 的内容。开发过程中可以随时运行它。这样大量节省了翻译和维护的时间。但是，记住这是微软翻译机的*机器翻译*。机器翻译往往不够准确或不流畅，因此生成后应该检查这些翻译。

##本地化控制页面中的 portlet

可能已经注意到可以在控制面板中使用的 portlet 在控制面板中显示时丢失了非常棒而且必须有的标题和描述信息。要想让 portlet 在控制面板中显示良好，需要在每个 portlet 单独的 _Language.properties_ 文件中创建特定的标题和描述码。应该使用 _javax.portlet.title_ 和 _javax.portlet.description_ 语言码。

为了演示，假设有一个名为 _eventlisting_ 的 portlet 和另一个名为 _locationlisting_ 的 portlet。需要为两个 portlet 分别创建资源包来指定本地化的标题和描述值。

``
注意：如果项目中只有一个 portlet，最好直接把资源包放到 _content_ 文件夹。在 _content/Language.properties_ 是设置设置信息，可以方便地使用 Plugins SDK 的语言构建工具。在 IDE 中右键点击 _Language.properties_ 文件，选择 _Liferay → Build Languages_ 或者在终端窗口中运行 _ant build-lang_。
``

以下是本地化项目中 portlet 的标题和描述应该做的工作：

1. 如果还没有这样做，配置每个需要在控制面板中显示的 portlet。本例中，会让它们显示在 _Content_ 部分，然后设置一个 _weight_ 值来决定它显示的位置。以下是项目的 _liferay-portlet.xml 文件示例：

	```
<portlet>
    <portlet-name>eventlisting</portlet-name>
    <icon>/icon.png</icon>
    <control-panel-entry-category>site_administration.content</control-\
    panel-entry-category>
    <control-panel-entry-weight>1.5</control-panel-entry-weight>
    ....
</portlet>
<portlet>
    <portlet-name>locationlisting</portlet-name>
    <icon>/icon.png</icon>
    <control-panel-entry-category>site_administration.content</control-\
    panel-entry-category>
    <control-panel-entry-weight>1.6</control-panel-entry-weight>
    ....
</portlet>
```
2. 创建命名空间目录来放 portlet 的资源包。最佳实践是根据 portlet 的名字来命名资源包目录。
例如，可以为 _eventListing_ portlet 创建资源包目录 _content/eventlisting_，为 _locationlisting_ portlet 创建 _content/locationlisting_ 目录。
3. 在刚才创建的资源包目录中新建 _Language.properties_ 文件。在每个 _Language.properties_ 文件中添加 _javax.portlet.title_ 和 _javax.portlet.description_ 语言码/值。
_eventlisting_ 的 _content/eventlisting/Language.properties_ 文件中有以下码/值对：

	```
javax.portlet.title=Event Listing Portlet
javax.portlet.description=Lists important upcoming events.
```

	而 _locationlisting_ 的 _content/eventlisting/Language.properties_ 文件中有以下码/值对：

	```
javax.portlet.title=Location Listing Portlet
javax.portlet.description=Lists event locations.
```
4. 在项目的 _portlet.xml_ 文件中为每个 portlet 设置资源包。以下示例 _portlet.xml_ 文件代码片断展示的是 _eventlisting_ 和 _locationlisting_ 示例 portlet 的资源包配置：

	```
<portlet>
    <portlet-name>eventlisting</portlet-name>
    ...
    <resource-bundle>content.eventlisting.Language</resource-bundle>
    <portlet-info>...</portlet-info>
    ...
</portlet>
<portlet>
    <portlet-name>locationlisting</portlet-name>
    ...
    <resource-bundle>content.locationlisting.Language</resource-bundle>
    <portlet-info>...</portlet-info>
    ...
</portlet>
```
5. 重新部署 plugin 项目。
6. 转到控制面板，并选择 _Event Location_ portlet。
7. 在 URL 中添加 _en_ 来显示西班牙语的用户界面。例如，URL 应该是类似下面的 URL 开头：

	```
http://localhost:8080/es/group/control_panel/...
```

控制面板显示 portlet 本地化的标题及描述。

![localized-portlet-title-desc-in-control-panel.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=c0fa4c87-5c49-4cf7-b9bd-68ae51431810)

图 3.12：为项目中的多个 portlet 本地化标题及描述操作简单。

你已经是一位本地化专家了！

``
Tip：你知道 portlet 的标题是如何处理的吗？如果 portlet 没有定义资源包或资源包中没有 _javax.portlet.title_，portal 容器接着检查 _portlet.xml_ 文件中的 _<portlet-info>_ 节点和它的 _<portlet-title>_ 节点的内容，如果也没有，_<portlet-name>_ 节点的值就作为 portlet 的标题。
``

``
注意：使用 Struts portlet 并且在 _portlet.xml 中引用 _StrutsResource_ 资源包会导致不同的标题及描述处理策略。标题和长标题（long title）使用两个不同的码：_javax.portlet.long-title_、_javax.portlet.title_。
``

现在可以熟练地本地化 portlet 内容了，或许想学习如何使翻译机制在整个 portal 可用，或者如何重写现有的翻译。请参考第 10 章，尤其是 _Overriding a Language.properties File_ 节来获取相应的信息。章节中描述了如何用钩子（hook）来重写 Liferay 现有的翻译内容。通过这种方式，可以和其它 portlet 共享语言码，也可以重写 Liferay 原有的翻译。

下面，学习如何通过 _configuration action_ 来设置 portlet 的偏好设置。