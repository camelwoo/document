#实现 My Greeting Portlet

`说明：可能是 Markdown 处理的问题，有些 <p> 标签会引起格式错误，代码中有一些做了处理，改为 < p>。`

我们来做一些有用的功能。首先，做两个页面：

- _view.jsp_：显示问候语，并提供一个到 edit 页面的链接。
- _edit.jsp_：显示一个表单，表单里有一个文本输入框，用来修改问候语；还有一个回到 view 页面的链接。

_MVCPortlet_ 类会控制 jsp 的渲染，在这个例子中我们不需要再写一个 Java 类了。

首先，我们不想在同一个页面上显示多个问候语，只要修改 liferay-portlet.xml 就可以了。如果已经有 instanceable 元素，把它的值修改为 _false_；如果没有就加一个。示例：

```
<portlet>
    <portlet-name>my-greeting</portlet-name>
    <icon>/icon.png</icon>
    <instanceable>false</instanceable>
    <header-portlet-css>/css/main.css</header-portlet-css>
    <footer-portlet-javascript>/js/main.js</footer-portlet-javascript>
    <css-class-wrapper>my-greeting-portlet</css-class-wrapper>
</portlet>
```
现在，我们来创建 JSP 模板。先修改 src/main/webapp 目录中的 view.jsp。用下面的内容替换原有内容：

```
<%@ taglib uri="http://java.sun.com/portlet_2_0" prefix="portlet" %>
<%@ page import="javax.portlet.PortletPreferences" %>

<portlet:defineObjects />

<%
PortletPreferences prefs = renderRequest.getPreferences();
String greeting = (String)prefs.getValue(
"greeting", "Hello! Welcome to our portal.");
%>

< p><%= greeting %></p>

<portlet:renderURL var="editGreetingURL">
    <portlet:param name="mvcPath" value="/edit.jsp" />
</portlet:renderURL>

< p><a href="<%= editGreetingURL %>">Edit greeting</a></p>
```
然后，在同一目录中创建 edit.jsp，内容如下：

```
<%@ taglib uri="http://java.sun.com/portlet_2_0" prefix="portlet" %>
<%@ taglib uri="http://liferay.com/tld/aui" prefix="aui" %>

<%@ page import="javax.portlet.PortletPreferences" %>

<portlet:defineObjects />

<%
PortletPreferences prefs = renderRequest.getPreferences();
String greeting = renderRequest.getParameter("greeting");
if (greeting != null) {
    prefs.setValue("greeting", greeting);
    prefs.store();
%>

    <p>Greeting saved successfully!</p>

<%
}
%>

<%
greeting = (String)prefs.getValue(
    "greeting", "Hello! Welcome to our portal.");
%>

<portlet:renderURL var="editGreetingURL">
    <portlet:param name="mvcPath" value="/edit.jsp" />
</portlet:renderURL>

<aui:form action="<%= editGreetingURL %>" method="post">
    <aui:input label="greeting" name="greeting" type="text" value="<%=
greeting %>" />
    <aui:button type="submit" />
</aui:form>

<portlet:renderURL var="viewGreetingURL">
    <portlet:param name="mvcPath" value="/view.jsp" />
</portlet:renderURL>

< p><a href="<%= viewGreetingURL %>">&larr; Back</a></p>
```
重新部署（redeploy）这个 portlet，在浏览器中刷新页面，现在应该能保存并显示自定义的问候语了。

![图 3.5：My Greeting portlet 的 view 页面](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=78fc11e1-833b-440e-b7bf-ae740848cc32)

图 3.5：My Greeting portlet 的 view 页面。

![图 3.6：My Greeting portlet 的 edit 页面](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=304a85a2-9f43-4946-b45f-a9faa37757ba)

图 3.6：My Greeting portlet 的 edit 页面。

``
Tip：如果 portlet 重新部署成功了，但在浏览器中刷新页面后仍然没有变化，可能是因为 Tomcat 没能正确重新构建（rebuild） JSP 页面。删除 _liferay-portal-[version]/tomcat-[tomcat-version]_ 中的 _work_ 目录，然后刷新页面，这样可以强制它 rebuild 页面。
``

这个实现的几个重要的细节需要注意。

首先，页面之间跳转的链接是用 _<portlet:renderURL>_ 标签生成的，标签由 http://java.sun.com/portlet_2_0 标签库定义。这些链接只有一个 _mvcPath_ 参数，MVCPortlet 通过它来确定每次请求时需要显示哪个 JSP 页面。建议用标签库来生成 portlet 的 URL，因为 portlet 只“拥有”页面的一块，而不是完整页面。URL必须由 portal 负责渲染，并且应用到页面上你的和其它的 portlet。portal 可以解析标签，并且生成包含足够渲染整个页面信息的 URL。

其次，注意 edit.jsp 中的表单有 _aui_ 前缀，表明它是 _AlloyUI_ 标签库的一部分。_AlloyUI_ 通过标签同时生成 label 和 field，大大简化了美观、易用的表单的制作。你也可以根据自己的喜好使用普通的 HTML 标签或其它标签库来制作表单。

另一个可能注意到的标签是 _<portlet:defineObjects/>_。portlet 规范中定义这个标签用来向 JSP 页面中添加一些内置的变量，这些变量有助于开发。内置的变量包括 _renderRequest_、_portletConfig_、_portletPreferences_等。注意，JSR-286 规范定义了 portlet 的四个生命周期方法：_processAction_、_processEvent_、_render_ 和 _serveResource_。一些由 _<portlet:dfineObjects/>_ 定义的变量只有当 JSP 包含在 portlet 生命周期相应的阶段时才会有效。这个标签引入了以下 portlet 对象：

- _RenderRequest renderRequest_： 表示渲染请求。_renderRequest_ 只在渲染请求（render request）阶段有效。
- _ResourceRequest resourceRequest_： 表示资源请求。_resourceRequest_ 只在 _resource-serving_ 阶段有效。
- _ActionRequest actionRequest_： 表示动作（action）请求。_actionRequest_ 只在 _action-processing_ 阶段有效。
- _EventRequest eventRequest_： 表示事件请求。_eventRequest_ 只在 _event-processing_ 阶段有效。
- _RenderResponse renderResponse_： 表示渲染请求的响应。[原文描述好像有点问题]
- _ResourceResponse resourceResponse_： 表示资源请求的响应。
- _ActionResponse actionResponse_： 表示动作请求的响应。
- _EventResponse eventResponse_： 表示事件响应。
- _PortletConfig portletConfig_： 表示 portlet 配置信息，包括 portlet 的名称、初始化参数、资源包、应用上下文。_portletConfig_ 在 portlet 的 jsp 中总是有效的，不管它处在哪个阶段。
- _PortletSession portletSession_： 提供了一种识别多个请求中是否为同一用户的方法，还可以暂存用户的信息。会为每个客户端创建一个 _portletSession_。不管 jsp 处在哪个阶段，它都有效。如果会话（session）不存在，它的值是 null。
- _Map<String, Object> portletSessionScope_： 相当于调用 _PortletSession.getAttributeMap()_ 获得的值，如果不存在会话属性（session attribute），它的值是空（empty）的。
- _Portletpreferences portletPreferences_： 用于访问 portlet 的偏好设置。不管 jsp 处在哪个阶段，它都有效。
- _Map<String, String[]> portletPreferencesValues_： 相当于调用  _portletPreferences.getMap()，如果 protlet 偏好设置不存在，它的值是空的。

这些因 _<portlet:defineObjects/>_ 标签而有效的变量和保存在 JSP 的 request 变量中的 portlet API 对象是一样的。阅读[Liferay Portlet 2.0 的 Javadoc](http://docs.liferay.com/portlet-api/2.0/javadocs/) 获取更多信息。

``
注意：为了让例子简单易懂，我们做了一些妥协（we cheated a little bit）。portlet 规范不允许在 JSP 中设置偏好设置，因为这样它在 render 阶段执行。这也是这样做限制的原因，下一节将做出解释。
``

``译注：上面说的"cheat a little bit" 说的是 edit.jsp 中的以下内容（代码中做了偏好设置的保存）：``

```
<%
PortletPreferences prefs = renderRequest.getPreferences();
String greeting = renderRequest.getParameter("greeting");
if (greeting != null) {
    prefs.setValue("greeting", greeting);
    prefs.store();
%>
    <p>Greeting saved successfully!</p>
<%
}
%>
```