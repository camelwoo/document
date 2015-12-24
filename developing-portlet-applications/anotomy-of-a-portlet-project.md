#Portelt项目概览

```
注：因主要使用 Maven，原文中主要是 Ant 的项目结构，有些内容不会翻译。
```

portlet 项目至少由三部分构成：

1. Java 源文件
2. 配置文件
3. 客户端文件（.jsp, .css, .js, 图形文件等）

使用 Maven 构建时，项目结构如下：

- 项目名称
	- pom.xml
	- src
		- main/
			- java/
			- resources/
			- webapp/
				- css/
					- main.css
				- js/
					- main.js
				- WEB-INF/
					- liferay-display.xml
					- liferay-plugin-package.properties
					- liferay-portlet.xml
					- portlet.xml
					- web.xml
				- icon.png
				- view.jsp

创建后就是一个可用的 portlet，可以部署到 Liferay Portal 服务器中。

创建的 portlet 默认使用 MzvcPortlet 框架，这是一个轻量级的框架，它隐藏了 portlet 复杂的方面，可以轻松实现常规的功能。默认的 MVCPortlet 项目为每种 portlet 模式（mode）分别提供一个 jsp：每个注册的 portlet 模式对应一个和它名字一样的 jsp。如：edit.jsp 对应 edit 模式，help.jsp 对应 help 模式。

Java 源文件保存在 src/main/java 目录。

配置文件保存在 src/main/webapp/WEB-INF 目录。这个目录中有 JSR-286 标准的配置文件 portlet.xml，还有三个可选的 Liferay 的配置文件。Liferay 的配置文件是可选的，如果要部署到 Liferay Portal 服务器，这些文件就很重要。以下是这些文件的说明：

- _liferay-display.xml_ 描述 portlet 显示在 _Dockbar_（用户登录后，显示在页面顶端的横条） 的 _Add_ 菜单的哪个分类下。
- _liferay-plugin-package.properties_ 为热部署器（hot deployer）描述 portlet。可以配置 _Portal Access Control List(PACL)_、.jar 依赖 等。
- _liferay-portlet.xml_ 描述在部署到 Liferay Portal 服务器时比 JSR-286 增强的 Liferay 特性。例如，可以为 portlet 设置图标、激活一个定时任务 等等。这个文件的完整配置可以在 Liferay 源代码的 _definitions_ 目录中该文件的 DTD 文件中找到。 

客户端文件是用来实现用户界面的 .jsp、.css 和 .js 文件。这些文件都在 src/webapp 目录下，.jsp 文件在这个目录中，.css 和 .js 文件都在它们各自的目录中。 *注意，portlet 只能处理返回到浏览器页面的一部分。任意 HTML 代码中都不应该包含像 <html>、<head> 这样的全局标签。此外，应该为所有 CSS class 和元素的 ID 添加命名空间（namespace），以避免和其它 portlet 的内容冲突。* Liferay 提供了两个工具（标签和 API 方法）生成命名空间。阅读 _Using Portlet Namespaceing_ 章节了解更多关于命名空间的内容。

##My Greeting Portlet 详解

如果你是新手，这些内容会提高你对 portelt 配置选项的理解。

portlet 的配置内容在 src/main/webapp/WEB-INF/portlet.xml 中，如下：
![](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=13621bcc-4bd4-4c14-86f4-7701a431acc3)
图 3.3：My Greeting portlet 的 portlet.xml 文件。

这些内容的简要说明：

- _portlet-name_ portlet 的标准名称。应用（是指 portlet 项目）中的每个 portlet 名称都应该是唯一的。在 Liferay Portal 中也称为 portelt ID。
- _display-name_ portlet 的短名，是在 portal 中显示出来的名字。它不需要是唯一的（建议也不要重名，否则不好区分）。
- _portlet-class_ 处理 portlet 操作的类的全限定名（也就是包括包名和类名的名称，如 cn.shuto.portlet.example.action.MyGreetingAction）。
- _init-param_ portlet 的初始化参数，是一个键/值对。
- _expiration-cache_ 表明 portlet 的输出多少秒后就过期了。如果值是 -1 表示不会过期。
- _supports_ portlet 模式（mode）支持的 MIME 类型，"portlet 模式"的概念是由 portlet 规范定义的。模式用来区分 portlet 不同的特定视图（view）。portal 识别 portlet 模式，并提供模式之间跳转（导航）的通用方法（这句话没太理解：for example, using links in the box surrounding the potlet when it's added to a page），因此，这对大多数 portlet 的常规操作非常有用。最常用的常规用法是建立一个用户可以填写偏好设置的编辑页面。[TODO: 这部分不太理解，以后再改]
- _portlet-info_ 定义用于 portlet 的 _title-bar_ 和分类的信息。JSR-286 规范只定义了几个资源元素：title、short-title、keywords。可以直接在 _portlet-info_ 中添加资源元素，也可以把它们放到资源包（resouce bundle）中。*这里所说的“资源”应该是指语言资源吧。*

	直接在 portlet.xml 的 portlet-info 元素中填写信息非常简单，例如，可以像这样填写一个天气 portlet 的信息：

	```
<portlet>
    ...
  	<portlet-info>
       	<title>Weather Portlet</title>
       	<short-title>Weather></short-title>
       	<keywords>weather,forecast</keywords>
   	</portlet-info>
   	...
</portlet>
	```
或者，在资源包文件中添加相同的信息。例如，在 src/main/resources/content 目录中创建 Language.properties 文件，在文件中填写 portlet 的标题、短标题和关键字：

	```
# Default Resource Bundle
#
# filename: Language.properties
# Portlet Info resource bundle example
javax.portlet.title=Weather Portlet
javax.portlet.short-title=Weather
javax.portlet.keywords=weather,forecast
	```
	需要在 portlet.xml 中引用这个资源包：
	
	```
<portlet>
    ...
    <resource-bundle>content.Language</resource-bundle>
    <portlet-info>...</portlet-info>
    ...
</portlet>
	```
	如果不打算支持本地化的标题、短标题和关键字，就简单地在 portlet.xml 中定义，否则就应该写到资源包中。
	
	```
注意：不应该在 portlet.xml 和资源文件中重复定义标题、关键字等。如果这样做了，资源文件中的内容优先于 portlet.xml 中的内容。
	```
	在资源文件中指定标题、短标题、关键字非常简单。例如，如果要支持德语和英语，应该在 src/main/resources/content目录（和缺省的语言文件在相同的目录）中创建 Language_de.properties 和 Language_en.properties 文件。这两个文件的内容类似以下内容：
	
	```
# English Resource Bundle
#
# filename: Language_en.properties
# Portlet Info resource bundle example
javax.portlet.title=Weather Portlet
javax.portlet.short-title=Weather
fjavax.portlet.keywords=weather,forecast
	```
	```
# German Resource Bundle
#
# filename: Language_de.properties
# Portlet Info resource bundle example
javax.portlet.title=Wetter Portlet
javax.portlet.short-title=Wetter
javax.portlet.keywords=wetter,vorhersage
	```
	还应该在 portlet.xml 中引用缺省的和本地化的资源文件，如下：

	```
<portlet>
    ...
    <resource-bundle>content.Language</resource-bundle>
    <resource-bundle>content.Language_de</resource-bundle>
    <resource-bundle>content.Language_en</resource-bundle>
    <portlet-info>...</portlet-info>
    ...
</portlet>
	```
	浏览 [JSR-286 portlet规范](http://www.jcp.org/en/jsr/detail?id=286) 获取更多信息。
- _security-role-ref 可以访问 portlet 的角色。

_src/main/webapp/WEB-INF/liferay-portlet.xml_ portlet.xml 之外的附加参数，是 Java 标准 portlet 部署到 Liferay portal 服务器时用到的可选的 Liferay 增强配置。Liferay IDE 默认会创建并填充内容：
![](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=ddcfe4bc-479c-4c04-a231-5b9003e2ccbc)
图 3.4：My Greeting portlet 的 liferay-portlet.xml 文件。

简要说明：

- _portlet-name_ portlet 的标准名称，要和 portlet.xml 中的 portlet-name 一致。
- _icon_ portlet 图标的文件名（包括路径）。
- _instanceable_ 同一个页面中是否可以包含多个该 portlet 的实例。
- _header-portlet-css_ 包含到页面 <head> 标签中的该 portlet 的 .css 的文件名。
- _footer-portlet-javascript_ 包含到页面底部 </body> 标签前的该 portlet 的 .js 的文件名。

在更高级的开发中还会用到很多元素。这个文件的完整配置可以在 Liferay 源代码的 _definitions_ 目录中该文件的 DTD 文件中找到。 