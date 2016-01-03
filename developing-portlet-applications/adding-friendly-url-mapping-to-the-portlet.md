#为 Portlet 设置友好的 URL

当点击 _Edit_ 问候语的链接时，新页面的链接就像下面这样：

```
http://localhost:8080/web/guest/home?p_p_id=mygreeting_WAR_mygreetingportlet
    &p_p_lifecycle=0&p_p_state=normal&p_p_mode=view&p_p_col_id=column-1
    &p_p_col_count=2&_mygreeting_WAR_mygreetingportlet_mvcPath=%2Fedit.jsp
```

从 Liferay 6 开始，可以通过内置的功能轻松地将上面的地址修改为：

```
http://localhost:8080/web/guest/home/-/my-greeting/edit
```

这个功能就是“友好的 URL 地址映射”，这个功能让无关的参数不出现在 URL 中，并且允许替换 URL 路径（path）参数，而不仅是查询字符串（query）。要添加这个功能，要先编辑 _liferay-portlet.xml_，在 _< /icon>_ 后面、_< instanceable>_ 前面添加以下内容：

```
<friendly-url-mapper-class>com.liferay.portal.kernel.portlet.DefaultFriendlyURLMapper</friendly-url-mapper-class>
<friendly-url-mapping>my-greeting</friendly-url-mapping>
<friendly-url-routes>com/liferay/samples/my-greeting-friendly-url-routes.xml</friendly-url-routes>
```

然后创建以下文件：

```
my-greeting-portlet/docroot/WEB-INF/src/com/liferay/samples/my-greeting-friendly-url-routes.xml
```

文件的内容如下：

```
<?xml version="1.0"?>
<!DOCTYPE routes PUBLIC "-//Liferay//DTD Friendly URL Routes 6.2.0//EN"
"http://www.liferay.com/dtd/liferay-friendly-url-routes_6_2_0.dtd">

<routes>
    <route>
        <pattern>/{mvcPathName}</pattern>
        <generated-parameter name="mvcPath">/{mvcPathName}.jsp</generated-parameter>
    </route>
</routes>
```

重新部署 portlet，刷新页面，看看点击 _Edit_ 后页面的 URL。URL 又短又对用户友好，还不用修改 JSP 文件。

![portlets-my-greeting-edit-friendly.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=f0c42107-db77-4121-9837-d1ce506b823a)

图 3.9：在 Liferay 中为 portlet 设置友好的 URL 很简单。看 edit.jsp 优雅的 URL。

关于友好地 URL 映射更多的信息，请参考 [Liferay in Aciton](https://www.manning.com/books/liferay-in-action) 中的详细讨论。下一步探索用户界面的国际化。