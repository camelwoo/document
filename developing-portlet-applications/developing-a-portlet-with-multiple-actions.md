#开发多 action 的 portlet

现在， portlet 只有两个视图（view）：默认的 _view_ 视图和 _edit_ 视图。添加更多视图也很简单，并且可以在 _renderURL_ 中使用参数 _mvcPath_ 来链接这些视图。但是现在只有一个 action，如果我们想添加其它的 action，比如给用户发送 email 该怎么做呢？

可以给 portlet 添加多个需要的 action。每个 action 都是 portlet 的一个方法，它可以接收两个参数：_ActionRequest_ 和 _ActionResponse_。可以任意命名这个方法，但是注意方法名必须和 renderURL 的 name 值相同。

重写上一章节中的例子，使用自定义方法名的 action 来设置 greeting 的内容，然后添加发送 email 的 action。

```
public class MyGreetingPortlet extends MVCPortlet {
    public void setGreeting(
            ActionRequest actionRequest, ActionResponse actionResponse)
    throws IOException, PortletException {
        PortletPreferences prefs = actionRequest.getPreferences();
        String greeting = actionRequest.getParameter("greeting");

        if (greeting != null) {
            try {
                prefs.setValue("greeting", greeting);
                prefs.store();
                SessionMessages.add(actionRequest, "success");
            }
            catch(Exception e) {
                SessionErrors.add(actionRequest, "error");
            }
        }
    }

    public void sendEmail(
            ActionRequest actionRequest, ActionResponse actionResponse)
    throws IOException, PortletException {
        // Add code here to send an email
    }
}
```

不再需要调用超类的 _processAction_ 方法了，因为不需要重写（overriding）它。

修改了名字，就还需要修改 URL，以使它的 name 匹配要调用的方法名。在 edit.jsp 中，像下面这样修改 _actionURL_：

```
<portlet:actionURL var="editGreetingURL" name="setGreeting">
    <portlet:param name="mvcPath" value="/edit.jsp" />
</portlet:actionURL>
```

现在，应该了解 portlet 开发的基本方法了，也可以用你熟悉的开发 portlet，并集成到 Liferay 中。最后来看看如何使用 Liferay 扩展的规范来为 portlet 生成更优雅的 URL。