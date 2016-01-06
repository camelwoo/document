#创建用户界面

现在为 Location Listing Portlet 创建用户界面。_location_ 实体很容易操作，因为它不依赖其它实体（不像 _evnet_ 实体，它依赖 _location_ 实体）。创建的界面允许完成以下操作：

- 列出已有地点
- 添加新的地点
- 编辑地点
- 删除地点

能看到已有的地点是好的体验，因此先创建一个地点列表的页面。下图是我们希望看到的列表的样子：

![jsp-list-locations.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=5e2c1486-3980-4fd3-aa7a-82a8152c0cd2)

图 5.9：使用 AlloyUI 和 Liferay UI 标签库，可以创建这样友好的 portlet 页面。

因为这部分是 location 实体的“视图”，用一个名为 view.jsp 的 JSP 来实现它，这个文件放在 _docroot/html/locationlisting_ 目录中。如果没有这个目录或文件就建一个。然后打开 _view.jsp_，并且用以下代码替换文件内容：

```jsp
<%@ taglib uri="http://java.sun.com/portlet_2_0" prefix="portlet" %>

<%@ taglib uri="http://liferay.com/tld/theme" prefix="liferay-theme" %>
<%@ taglib uri="http://liferay.com/tld/ui" prefix="liferay-ui" %>

<%@ page import="com.liferay.portal.util.PortalUtil" %>

<%@ page import="com.nosester.portlet.eventlisting.model.Location"%>
<%@ page import="com.nosester.portlet.eventlisting.service.LocationLocalServiceUtil"%>

<liferay-theme:defineObjects />

<portlet:defineObjects />

This is the <b>Location Listing Portlet</b> in View mode.

<liferay-ui:search-container emptyResultsMessage="There are no locations to display">
    <liferay-ui:search-container-results
        results="<%= LocationLocalServiceUtil.getLocationsByGroupId(scopeGroupId, searchContainer.getStart(), searchContainer.getEnd()) %>"
        total="<%= LocationLocalServiceUtil.getLocationsCountByGroupId(scopeGroupId) %>"
    />

    <liferay-ui:search-container-row
        className="com.nosester.portlet.eventlisting.model.Location"
        keyProperty="locationId"
        modelVar="location" escapedModel="<%= true %>"
    >
        <liferay-ui:search-container-column-text
            name="name"
            value="<%= location.getName() %>"
        />

        <liferay-ui:search-container-column-text
            name="description"
            value="<%= location.getDescription() %>"
        />

        <liferay-ui:search-container-column-text
            name="streetAddress"
            value="<%= location.getStreetAddress() %>"
        />

        <liferay-ui:search-container-column-text
            name="city"
            value="<%= location.getCity() %>"
        />

        <liferay-ui:search-container-column-text
            name="stateOrProvince"
            value="<%= location.getStateOrProvince() %>"
        />

        <liferay-ui:search-container-column-text
            name="country"
            value="<%= location.getCountry() %>"
        />
    </liferay-ui:search-container-row>

    <liferay-ui:search-iterator />
</liferay-ui:search-container>
```

仔细查看这些代码，从 JSP 指令（directive）开始。_taglib_ 指令告诉 JSP 到哪去找用到的标签，并说明使用这些标签时应该加什么命名空间前缀。例如，_<%@ taglib uri="http://liferay.com/tld/theme" prefix="liferay-theme" %>_ 指令说明 Liferay _theme_ 标签库的描述（TLD）和标签使用的前缀。JSP 中还有 Portlet 2.0 和 Liferay UI 标签库的指令。然后使用指令来导入 _PortalUtil_、_Location_ 和 _LocationLocalServiceUtil_ 类。

在导入指令下面有一些有意思的标签：_<liferay-theme:defineObjects />_ 和 _<portlet:defineObjects />_。这些标签让 JSP 可以访问请求对象和 portal 主题上下文中的很多变量。

在说明 portlet 文字之后的内容才是这个 JSP 真正的“干货”。使用 Liferay 的 <liferay-ui:search-container> 标签从数据库返回 _location_ 实体，并用表格展示实体的数据。查询容器（search container）调用 _LocationLocalServiceUtil_ 来获取 _location_。然后在表格的每行显示一个 _com.nosester.portlet.eventlisting.model.Location_ 类的实例。每个 _location_ 通过它的 _locatoinId_ 标识，而且 _location_ 的每个属性都通过 _<liferay-ui:search-container-column-text>_ 显示在单元格中。

重新部署 portlet 项目，看一下已有的地点。

![jsp-list-no-locations.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=9281b9e8-d352-43da-bf92-0cef46407656)

图 5.10：地点呢？不要着急，还没添加呢。下面会创建一个页面用来添加地点。

现在需要创建一个用来添加地点的页面，命名为 _edit\_location.jsp_，添加、修改地点都用它。在 _docroot/html/locationlisting_ 目录创建这个文件，将以下内容复制到文件中：

```
<%@ taglib uri="http://java.sun.com/portlet_2_0" prefix="portlet" %>

<%@ taglib uri="http://liferay.com/tld/aui" prefix="aui" %>
<%@ taglib uri="http://liferay.com/tld/ui" prefix="liferay-ui" %>

<%@ page import="com.liferay.portal.kernel.util.ParamUtil" %>

<%@ page import="com.nosester.portlet.eventlisting.model.Location"%>
<%@ page import="com.nosester.portlet.eventlisting.service.LocationLocalServiceUtil"%>

<%
    Location location = null;

    long locationId = ParamUtil.getLong(request, "locationId");

    if (locationId > 0) {
        location = LocationLocalServiceUtil.getLocation(locationId);
    }

    String redirect = ParamUtil.getString(request, "redirect");
%>

<aui:model-context bean="<%= location %>" model="<%= Location.class %>" />
<portlet:renderURL var="viewLocationURL" />
<portlet:actionURL name='<%= location == null ? "addLocation" : "updateLocation" %>' var="editLocationURL" windowState="normal" />

<liferay-ui:header
    backURL="<%= viewLocationURL %>"
    title='<%= (location != null) ? location.getName() : "New Location" %>'
/>

<aui:form action="<%= editLocationURL %>" method="POST" name="fm">
    <aui:fieldset>
        <aui:input name="redirect" type="hidden" value="<%= redirect %>" />

        <aui:input name="locationId" type="hidden" value='<%= location == null ? "" : location.getLocationId() %>'/>

        <aui:input name="name" />

        <aui:input name="description" />

        <aui:input name="streetAddress" />

        <aui:input name="city" />

        <aui:input name="stateOrProvince" />

        <aui:input name="country" />

    </aui:fieldset>

    <aui:button-row>
        <aui:button type="submit" />

        <aui:button onClick="<%= viewLocationURL %>"  type="cancel" />
    </aui:button-row>
</aui:form>
```

像 _view.jsp_ 中那样添加访问标签库和导入类的指令。

然后检查请求中的地点ID（location ID），如果有，就是编辑地点；否则就是新增地点。使用 _<portlet:actionURL>_ 标签来让 portlet 跳转到新增一个地点或者编辑一个现有的地点。最后，在写表单之前，显示 portlet 的头，说明是新增地点还是修改地点。

使用 AlloyUI 的 _<aui:form>_ 标签表示一个录入地点信息和提交的表单。里面用很多 _<aui:iput>_ 标签表示地点的属性。每一个的 _name_ 都是 _LocationListingPortlet_ 类中用到的。最后，添加一个用来提交表单的按钮。按钮的属性通过 _viewLocationURL_ 说明会跳转到 _view.jsp_。

现在，已经实现了 _edit_location.jsp_，必须提供一个用户能访问到它的方法。在 _view.jsp_ 中添加一个按钮，跳转到 _edit_location.jsp_：

1. 打开地点的 _view.jsp_，在 _<liferay-ui:search-container>_ 上方添加以下代码：

	```
<%
    String redirect = PortalUtil.getCurrentURL(renderRequest);
%>
<aui:button-row>
    <portlet:renderURL var="addLocationURL">
        <portlet:param name="mvcPath" value="/html/locationlisting/edit_location.jsp" />
        <portlet:param name="redirect" value="<%= redirect %>" />
    </portlet:renderURL>
    <aui:button onClick="<%= addLocationURL.toString() %>" value="add-location" />
</aui:button-row>
```
2. 在 _view.jsp_ 的顶端添加以下指令来导入 AlloyUI 的 _aui_ 标签库：

	```
<%@ taglib uri="http://liferay.com/tld/aui" prefix="aui" %>
```

这些代码添加一个按钮，并且通过 _mvcPath_ 的值 _/html/locationlisting/edit\_location.jsp._ 使用户跳转到 _edit_location.jsp_。请求中还有一个当前URL，也就是 view.jsp 的 URL，这使用户在添加完新的地点后 _edit\_location.jsp_ 可以跳转回来。

现在，已经实现了添加、删除地点的 JSP，而且提供了一个按钮让用户可以访问这个 JSP。修改完后必须重新部署 portlet 项目以使修改生效。

重新部署 portlet 项目，并添加一个新的地点。

![jsp-add-new-location.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=daf0d273-0d42-4cf7-817c-240daa9f2c12)

图 5.11：AlloyUI 的 <aui:form> 标签帮助你创建动人的表单来编辑实体信息。

添加一两个地点之后，就可以看到每个地点是如何优雅地在 _view.jsp_ 中显示。

![jsp-list-locations-with-no-action-button.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=42e83a27-238e-42bd-9dbc-3b7a9d28be7b)

图 5.12：注意页面上的 _Add Location_ 按钮。

正如前面提到的，不仅可以添加地点，还需要编辑现有的地点。另外，还要能删除地点。下面提供修改和删除单个地点的方法。页面上会显示一个下拉按钮，来让用户编辑和删除地点。将会创建一个 JSP 来处理每个 action 请求，并将请求发送到 portlet。

先来创建一个名为 _location_actions.jsp。在 _html/locationlisting_ 目录中创建 _location\_actions.jsp_ 文件，将下面代码复制到文件中：

```html
<%@ taglib uri="http://java.sun.com/portlet_2_0" prefix="portlet" %>

<%@ taglib uri="http://liferay.com/tld/ui" prefix="liferay-ui" %>

<%@ page import="com.liferay.portal.kernel.dao.search.ResultRow" %>
<%@ page import="com.liferay.portal.kernel.util.WebKeys" %>
<%@ page import="com.liferay.portal.util.PortalUtil" %>

<%@ page import="com.nosester.portlet.eventlisting.model.Location"%>

<portlet:defineObjects />

<%
    ResultRow row = (ResultRow) request
            .getAttribute(WebKeys.SEARCH_CONTAINER_RESULT_ROW);
    Location location = (Location) row.getObject();

    long groupId = location.getGroupId();
    String name = Location.class.getName();
    long locationId = location.getLocationId();

    String redirect = PortalUtil.getCurrentURL(renderRequest);
%>

<liferay-ui:icon-menu>
    <portlet:renderURL var="editURL">
        <portlet:param name="mvcPath" value="/html/locationlisting/edit_location.jsp" />
        <portlet:param name="locationId" value="<%= String.valueOf(locationId) %>" />
        <portlet:param name="redirect" value="<%= redirect %>" />
    </portlet:renderURL>

    <liferay-ui:icon image="edit" url="<%= editURL.toString() %>" />

    <portlet:actionURL name="deleteLocation" var="deleteURL">
        <portlet:param name="locationId" value="<%= String.valueOf(locationId) %>" />
        <portlet:param name="redirect" value="<%= redirect %>" />
    </portlet:actionURL>

    <liferay-ui:icon-delete url="<%= deleteURL.toString() %>" />
</liferay-ui:icon-menu>
```

上面的代码把地点信息从请求对象中取出来，然后提供了删除和编辑地点的图标。如果用户点击编辑图标就会跳转到 _edit\_location.jsp_；如果点击删除图标，请求就会提交到服务器，删除地点后会跳转到 _view.jsp_。实现了 _location\_actions.jsp_ 后，将它链接到 _view.jsp_。

在 _view.jsp_ 中关闭 _</liferay-ui:search-container-row>_ 标签之前添加一个容器来显示在 _location\_actions.jsp_ 中做好的编辑和删除图标：

```
<liferay-ui:search-container-column-jsp
    align="right"
    path="/html/locationlisting/location_actions.jsp"
/>
```

_<liferay-ui:search-container-column-jsp>_ 标签显示 _Actions_ 按钮，给用户显示 _location\_actions.jsp_ 中做好的操作图标。

做好 _Actions_ 按钮了，重新部署 portlet，刷新浏览器页面，就能看到每个地点都有一个漂亮的 _Actions_ 按钮。试试编辑、删除一个地点。

![jsp-edit-location-entry.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=907801f0-01cd-4e9b-afd3-1d5559875b1a)

图 5.13：可以创建有用的操作图标，用户点击后就可以编辑、删除实体了。

创建的 Location Listing 界面非常棒！

如果感兴趣的话，完整的 event-listing-portlet 示例项目，包括 Location Listing Portlet 和 Event Listing Portlet 的界面，可以在 [Dev Guide SDK](https://github.com/liferay/liferay-docs/tree/master/devGuide/code/devGuide-sdk) 的 _portlets/event-listing-portlet_ 目录中找到。

接下来，学习如何调用 Liferay 的核心服务。在 portlet 中调用 Liferay 的核心服务像调用 Service Builder 生成的服务一样简单。