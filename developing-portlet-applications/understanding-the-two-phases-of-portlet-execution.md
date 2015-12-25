#理解 Portlet 执行的两个阶段

portlet 有两个执行阶段：action 阶段和 render 阶段。多个执行阶段对于传统的 Servlet 开发人员或其它如 PHP、Python 或 Ruby 的开发人员来说会感觉迷惑。然而一旦熟悉了，你会觉得 action 阶段和 render 阶段很简单易用。我们来看一下为什么两个阶段是必须的。

portlet 不能拥有整个 HTML 页面，而是和其它 portlet 以及 portal 共享页面。portal 调用一个或多个 portlet，并且添加一些额外的 HTML 包裹上它们，这样就形成了完整的页面。当用户在一个 portlet 中执行一个动作，页面上的所有 portlet 都会重新渲染。portal 不能简单地让每个 portlet 重复最后一次操作，下面的场景会说明为什么会这样。

假设一个页面上有两个 portlet：一个导航 portlet 和一个购物 portlet。如果不是两阶段执行将会是下面这样：

1. 首先，用户找到她想要购买的东西，提交了订单并且用信用卡付了款。做完这个操作 portal 还会调用导航 portlet 的默认视图（view）。
2. 接下来，假设用户点击了导航 portlet 上的链接。这时会初始化一个 HTTP 请求/响应过程，并改变 portlet 的内容。**但是在这个请求的参数包含页面所有 portlet 的参数，当然也包含上面提到的订单支付数据。**因为 portal 必须显示购物 portlet 的内容，而它会重复最后的操作，也就是会再进行一次信用卡支付，而且会产生一条新的物流信息。[译注：这个购物 portlet 功能也是有够弱的，居然可以重复支付。]

为什么会这样？因为 portal 不知道用户在页面上添加了哪些 portlet。在写一个标准的 web 应用时，可以为执行操作和页面跳转（navigation）设计不同的 URL。用户可以在页面上添加任意可用的 portlet，portal 必须把 “action” 和简单的重绘（re-draw or re-render）区分开来。

显然，我们希望避免上面第二步中的那种情况，但是没有两阶段 portal 不知道最后一次操作是否为 _action_ 操作。这样就会一次又一次地执行最后的操作，或许直到超过了信用卡的支付限额。

幸运的是，portal 并不是这样工作的。为了避免上述情况，portlet 规范为每个 portlet 请求中定义了两个阶段，使得 portal 可以区分什么时候执行一个 action 操作（不会重复执行），什么时候显示内容（render）：

- Action 阶段： action 阶段一次只能由一个 portlet 调用。这是用户和 portlet 交互的结果。在这个阶段可以改变 portlet 的状态，例如修改偏好设置。任何像数据库插入、更新这样的只应该执行一次的操作都应该放在这个阶段中。
- Render 阶段： 在 action 阶段执行（也可能不存在 action 阶段）完之后，会执行页面上所有的 portlet 的 render 阶段。包括刚刚执行完 action 阶段的那个 portlet。需要特别注意，portlet 规范中并没有规定页面上各 portlet 渲染的顺序。Liferay 通过 liferay-portlet.xml 中的 _render-weigth_ 元素扩展了标准规范。权重高的 portlet 先渲染。

目前，例子中使用的是 _MVCPortlet_ 类。如果 portlet 只有 render 阶段，这些就够用了。要想添加在 _action_ 阶段（这样它就不会在重新显示 portlet 时重复执行）执行的自定义功能，需要做一个 _MVCPortlet_ 的子类或直接做一个 _GenericPortlet_ 的子类（如果你不想使用 Liferay 的轻量级框架）。

创建下面的类来增强现在例子的功能：

```
package com.liferay.samples;

import java.io.IOException;
import javax.portlet.ActionRequest;
import javax.portlet.ActionResponse;
import javax.portlet.PortletException;
import javax.portlet.PortletPreferences;
import com.liferay.util.bridges.mvc.MVCPortlet;

public class MyGreetingPortlet extends MVCPortlet {
    @Override
    public void processAction(
        ActionRequest actionRequest, ActionResponse actionResponse)
        throws IOException, PortletException {
        PortletPreferences prefs = actionRequest.getPreferences();
        String greeting = actionRequest.getParameter("greeting");

        if (greeting != null) {
            prefs.setValue("greeting", greeting);
            prefs.store();
        }

        super.processAction(actionRequest, actionResponse);
    }
}
```
在 src/main/java 目录中创建上述包和类。

还要修改 portlet.xml，使它使用这个新的 portlet 类 `com.liferay.samples.MyGreetingPortlet` 而不是 `com.liferay.util.bridges.mvc.MVCPortlet`：

```
<portlet>
<portlet-name>my-greeting</portlet-name>
<display-name>My Greeting</display-name>
<portlet-class>com.liferay.samples.MyGreetingPortlet</portlet-class>
<init-param>
    <name>view-template</name>
    <value>/view.jsp</value>
</init-param>
...
```
最后，微调一下 edit.jsp，修改表单的 URL，让 portal 知道是要执行一个 _action_ 阶段的请求。可以生成三种  portlet 的 URL：

- _renderURL_： 仅执行 portlet 的 _render_ 阶段。
- _actionURL_： 在渲染页面所有 portlet 之前执行这个 portlet 的 _action_ 阶段。
- _resourceURL_： 用来接收图片、XML、JSON或其它类型的资源。经常用来动态生成图片或其它媒体文件，也用来提交 AJAX 请求。最重要的是，它和其它两个阶段不同，它可以控制响应中的完整数据（不像前两个阶段只能控制一部分）。

修改 edit.jsp，使用相同的标签来生成 _actionURL_。还要删除以前保存偏好设置的代码。使用以下内容覆盖 edit.jsp：

```
<%@ taglib uri="http://java.sun.com/portlet_2_0" prefix="portlet" %>
<%@ taglib uri="http://liferay.com/tld/aui" prefix="aui" %>

<%@ page import="com.liferay.portal.kernel.util.ParamUtil" %>
<%@ page import="com.liferay.portal.kernel.util.Validator" %>
<%@ page import="javax.portlet.PortletPreferences" %>

<portlet:defineObjects />

<%
    PortletPreferences prefs = renderRequest.getPreferences();
    String greeting = (String)prefs.getValue(
        "greeting", "Hello! Welcome to our portal.");
%>

<portlet:actionURL var="editGreetingURL">
    <portlet:param name="mvcPath" value="/edit.jsp" />
</portlet:actionURL>

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
修改后重新部署 portlet，运行起来和以前几乎一样。如果不仔细观察，你可能错过一些东西：在点击保存按钮后不再显示偏好设置已保存的提示了。要想显示它，需要将信息从 _action_ 阶段传递到 _render_ 阶段，这样 JSP 才能知道偏好设置已经保存了，从来显示相应的提示信息。