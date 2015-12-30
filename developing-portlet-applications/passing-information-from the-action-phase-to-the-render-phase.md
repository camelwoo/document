#从 Action 阶段向 Render 阶段传递信息

有两种从 Action 阶段向 Render 阶段传递信息的方法。

第一种方法是通过 render 参数。在 _processAction 方法中，可以调用 _setRenderParameter_ 方法来给 request 添加一个新参数；在 render 阶段读取这个参数。
设置参数：

```
actionResponse.setRenderParameter("parameter-name", "value");
```

在 render 阶段（例子中是 JSP）读取参数：

```
renderRequest.getParameter("parameter-name");
```

有一点需要特别注意，action URL 的参数只能在 action 阶段读取（在 _processAction_ 方法中）。要想把参数传递到 render 阶段，必须先从 _actionRequest_ 中读取要传递的参数，然后调用 _setRenderParamete_ 方法将参数设置到 request 中。

``
Tip：Liferay 提供了一个方便的扩展，通过 _MVCPortlet_ 类可以将所有的 action 参数直接作为 render 参数。在 portlet.xml 中设置 init-param 来实现：
``

```
<init-param>
    <name>copy-request-parameters</name>
    <value>true</value>
</init-param>
```

``使用 render 参数需要特别注意：一旦设置了 render 参数，在后面这个 portal 执行时都会带着这些参数，直到使用不同的参数再次调用这个 portlet。也就是说，如果一个用户点击了 portlet 中的一个链接并且代码中也设置了 render 参数，随后用户又操作页面上的其它 portlet，每次页面重新加载时 portal 都会使用前面设置的 render 参数来渲染这个 portlet。本例中，如果设置了 render 参数，操作成功的提示信息不仅会在刚保存完时显示，每次页面重新加载时都会显示这个提示信息，除非 portlet 又被不带 render 参数地调用过一次。``

第二种方法是 portlet 特有的，你可能很熟悉————使用 session。可以在 _actionRequest_ 中设置一个 session 的属性（attribute），随后在 JSP 中读取它。在本例中，JSP 显示提示信息后应该立即从 session 中移除这个属性，这样提示信息就只显示一次。Liferay 提供了 helper 类和标签库来轻松实现这些操作。在 _processAction_ 方法中，需要用到 _SessionMessages_ 类：

```
package com.liferay.samples;

import java.io.IOException;
import javax.portlet.ActionRequest;
import javax.portlet.ActionResponse;
import javax.portlet.PortletException;
import javax.portlet.PortletPreferences;
import com.liferay.portal.kernel.servlet.SessionMessages;
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

        SessionMessages.add(actionRequest, "success");
        super.processAction(actionRequest, actionResponse);
    }
}
```

然后在 view.jsp 中添加 _liferay-ui:success_ 标签及标签库的声明：

```
<%@ taglib uri="http://java.sun.com/portlet_2_0" prefix="portlet" %>
<%@ taglib uri="http://liferay.com/tld/ui" prefix="liferay-ui" %>
<%@ page import="javax.portlet.PortletPreferences" %>

<portlet:defineObjects />

<liferay-ui:success key="success" message="Greeting saved successfully!" />

<% PortletPreferences prefs = renderRequest.getPreferences(); String
greeting = (String)prefs.getValue(
    "greeting", "Hello! Welcome to our portal."); %>

< p><%= greeting %></p>

<portlet:renderURL var="editGreetingURL">
    <portlet:param name="mvcPath" value="/edit.jsp" />
</portlet:renderURL>

< p><a href="<%= editGreetingURL %>">Edit greeting</a></p>

```

修改完后，重新部署 portlet，到 edit 视图，修改问候语，然后保存。会看到像下面这样友好的提示信息：

![03-greeting-saved.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=8d305751-68da-426f-94ec-d42627c9499f)

图 3.7：My Greeting 示例 portlet 显示操作成功提示信息。

还有一个类似的工具类处错误提示信息，可以在 view.jsp 中 _liferay-ui:success_ 标签下面再添加一个 _liferay-ui:error_ 标签：

```
<liferay-ui:error key="error" message="Sorry, an error prevented saving
your greeting" />
```

这个错误提示信息工具通常在 _processAction_ 方法中捕捉到错误信息之后使用。例如：

```
try {
    prefs.setValue("greeting", greeting);
    prefs.store();
    SessionMessages.add(actionRequest, "success");
}
catch(Exception e) {
    SessionErrors.add(actionRequest, "error");
}
```

如果在处理 action 请求时出现错误，view.jsp 中会显示错误提示信息。

![portlet-invalid-data.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=56955990-e415-496e-b218-cf4759c427e6)

图 3.8：My Greeting 示例 portlet 显示错误提示信息。

第一个信息是 Liferay 自动添加的。第二个是自己添加的。这样就创建并显示了错误提示信息。

想过 Liferay Portal 在接收到不同 portlet 的多个同名参数时是如何判断哪些参数是哪个 portlet 的吗？每个 Liferay 的核心 portlet 都会给请求参数加上命名空间，因此 Liferay 可以从其它参数中分辨出它们。你也可以给自己的 portlet 添加命名空间。下面说明 portlet 的命名空间以及如何开启/关闭 portlet 的命名空间。

##使用 portlet 命名空间

命名空间保证 portlet 的名字关联到发送到服务器的请求参数元素， 这样避免了与同一页面上其它 portlet 元素的命名冲突。给 portlet 元素添加命名空间很简单。只要为 portlet 的元素使用 <portlet:namespace /> 标签产生一个唯一值就可以了。下面的示例代码中，提交时使用 <portlet:namespace /> 标签来引用 portlet 的 _fm_ 表单。

```
submitForm(document.<portlet:namespace />fm);
```

下面说明给上面 _fm_ 表单这样的元素使用命名空间的好处，假设有 A 和 B 两个 portlet，它们都有一个名为 _fm_   的表单，不使用命名空间 portal 就无法区分两个表单，也就无法判断它们应该关联到哪个 portlet。但是，使用 _<portlet:namespace />fm_ 命名 A 和 B 两个 portlet 的表单，就可以将两个表单区分为 _\_Afm_ 和 _\_Bfm_。Liferay 使用 portlet 名字来关联每个使用命名空间的元素，例如上面提到的表单。

Liferay 默认只允许使用命名空间的参数传递到 portlet，然而许多第三方的 portlet 会使用没有命名空间的参数，因此 Liferay 提供了命名空间参数的开关选项，以避免阻断第三方 portlet 的请求。要关闭某个 portlet 的过滤器，找到 liferay-portlet.xml 文件，输入以下内容：

```
<requires-namespaced-parameters>false</requires-namespaced-parameters>
```

这个选项是针对 portlet 的，因此对每一个不使用命名空间参数的 portlet 都要设置 <requires-namespaced-parameters/> 值为 false。

开发有多个 action 的 portlet 有意思吗？那你一定想看下一章节。