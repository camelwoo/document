#实现 Portlet 偏好设置

portlet 偏好设置保存的是基本的 portlet 配置信息。偏好设置往往是管理员用来给某些或全部用户提供定制化视图的，有时用户也可以按照自己喜好的配置 portlet。Liferay 中的 portlet 可以在 JSP 页面进行偏好的配置。在本节中，演示如何创建默认的 JSP 配置页面，并且在页面中添加偏好配置。

这里使用[生成服务层代码](/generating-your-service-layer/README.md)章节中开发的 Location Listing portlet。给它添加配置页面，并且添加一个自定义选项，允许管理隐藏地点的地址（address）信息。通过创建配置页面并添加一个偏好设置。

首先，如果在 Location Listing portlet 的配置（configuration）页面没有设置（setup）标签页（tab），就需要按以下步骤创建它。要检查有没有，点击 portlet 右上角的选项（option）图标并且选择 _Configuration_，如果没有可以按下面演示的方式给 portlet 的配置页面添加设置标签页。

##给配置页面添加设置标签页

打开 _liferay-portlet.xml 文件，并且在 Location Listing portlet 的 <icon>...</icon> 标签下面添加 _<configuration-action-class>com.liferay.portal.kernel.portlet.DefaultConfigurationAction</configuration-action-class>_ 元素。下面是展示的是添加的这个元素在 _liferay-porltet.xml_ 中的位置：

```
....
<portlet>
    <portlet-name>locationlisting</portlet-name>
    <icon>/icon.png</icon>
    <configuration-action-class>com.liferay.portal.kernel.portlet.DefaultConfigurationAction</configuration-action-class>
    <header-portlet-css>/css/main.css</header-portlet-css>
    <footer-portlet-javascript>
       /js/main.js
    </footer-portlet-javascript>
    <css-class-wrapper>locationlisting-portlet</css-class-wrapper>
</portlet>
....
```

注意，使用的是_默认（default）_配置 action 类，后面的练习中会用自定义的配置类换掉它。重新部署 portlet 并打开 portlet 的 _Configuration_ 页面，不能看到新的 _Setup_ 标签页。标签页是空的，不过很快就会给它添加一个 portlet 偏好设置。

![portlet-default-configuration-jsp.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=358b29e8-f17a-453a-a2fa-c44a8ab9c406)

图 3.13：简单地在 portlet 的 _liferay-portlet.xml_ 文件中指定 Liferay 的默认设置 action 就可以添加用来添加设置选项的 _Setup_ 标签页。

要添加 portlet 偏好设置，必须完成以下工作：

1. 在 _portlet.xml_ 中指定设置 JSP。
2. 创建一个显示 portlet 偏好设置选项的 JSP。
3. 创建处理偏好设置信息 _Configuration Action_ 的实现类。
4. 编辑响应偏好设置的 JSP。

先来指定设置 JSP。

###第一步：在 _portlet.xml_ 中指定设置 JSP

portlet 要给用户提供一个显示设置选项的途径。Liferay 检查在 _portlet.xml_ 中是否通过 _config-template_ 设置的初始化参数来指定设置 JSP。来为 Location Listing portlet 指定一个。

打开 _portlet.xml_ 文件并 Location Listing portlet 的 <portlet-class>...</portlet-class> 标签下面插入以下信息：

```
 <init-param>
     <name>config-template</name>
     <value>/html/locationlisting/configuration.jsp</value>
 </init-param>
```

###第二步：创建一个显示 portlet 偏好设置选项的 JSP

创建一个 JSP 文件，并且添加 Javascript 以使用户可以选择偏好设置值。对于这个例子，添加一个自定义选项，让管理员可以控制显示或隐藏地点的地址信息。

_docroot/html/locationlisting_ 目录中如果没有 _configuration.jsp_ 就新建一个。

现在添加设置信息。假设 _configuration.jsp_ 没有内容，添加以下内容：

```
<%@include file="/html/init.jsp" %>

<liferay-portlet:actionURL portletConfiguration="true" var="configurationURL" />

<%
boolean showLocationAddress_cfg = GetterUtil.getBoolean(portletPreferences.getValue("showLocationAddress", StringPool.TRUE));
%>

<aui:form action="<%= configurationURL %>" method="post" name="fm">
    <aui:input name="<%= Constants.CMD %>" type="hidden" value="<%= Constants.UPDATE %>" />

    <aui:input name="preferences--showLocationAddress--" type="checkbox" value="<%= showLocationAddress_cfg %>" />

    <aui:button-row>
       <aui:button type="submit" />
    </aui:button-row>
</aui:form>
```

_showLocationAddress_cfg_ 变量保存是否显示地点的地址信息的当前值。用户可以通过复选框设置偏好设置中 _showLocationAddress_ 键（key）的值。要持久化设置数据，_input_ 表单域的 _name_ 必须是约定的 _preferences--somePreferenceKey--_ 格式。本例中，_preferences--showLocationAddress--_ 对应 portlet 偏好设置的 _showLocationAddress_。

你可能注意到 JSP 中刚才添加的代码有编译错误或警告，在设置 JSP 包含的 _init.jsp_ 中添加指令（directive）来处理它。添加的指令允许 JSP 访问用到的类和标签库。

在 _init.jsp_ 中添加以下内容：

```
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<%@ taglib uri="http://liferay.com/tld/portlet" prefix="liferay-portlet" %>
<%@ taglib uri="http://liferay.com/tld/theme" prefix="liferay-theme" %>

<%@ page import="com.liferay.portal.kernel.util.Constants" %>
<%@ page import="com.liferay.portal.kernel.util.GetterUtil" %>
<%@ page import="com.liferay.portal.kernel.util.StringPool" %>

<liferay-theme:defineObjects />
```

_taglib_ 指令访问 JSP 标准标签库（JSP Standard Tag Library，JSTL）、Liferay 的 theme 标签库和 Liferay 的 portlet 标签库；然后添加指令来导入用到的类；最后插入 _<portlet:defineObjects />_ 标签访问需要的隐式变量。隐式的变量非常有用，包括 _renderRequest_、_portletConfig_、_portletPreferences_ 等。

_configuration.jsp_ 已经可以显示 portlet 偏好设置选项了。下面实现一个处理设置信息的自定义类。

###第三步：创建处理偏好设置信息 _Configuration Action_ 的实现类

现在创建一个自定义的设置类来操作 portlet 偏好设置，这个类继承 _DefaultConfigurationAction_ 类。

在 _docroot/WEB-INF/src_ 目录创建 _com.nosester.portlet.eventlisting.action_ 包，并在包中创建 _ConfigurationActionImpl_ 类，并且指定它的父类为 [DefaultConfigurationAction](http://docs.liferay.com/portal/6.2/javadocs/com/liferay/portal/kernel/portlet/DefaultConfigurationAction.html)。

用以下代码替换 _ConfigurationActionImpl.java_ 的内容：

```
package com.nosester.portlet.eventlisting.action;

import javax.portlet.ActionRequest;
import javax.portlet.ActionResponse;
import javax.portlet.PortletConfig;
import javax.portlet.PortletPreferences;

import com.liferay.portal.kernel.portlet.DefaultConfigurationAction;

public class ConfigurationActionImpl extends DefaultConfigurationAction {

    @Override
    public void processAction(
        PortletConfig portletConfig, ActionRequest actionRequest,
        ActionResponse actionResponse) throws Exception {

        super.processAction(portletConfig, actionRequest, actionResponse);

        PortletPreferences prefs = actionRequest.getPreferences();

        String showLocationAddress = prefs.getValue(
            "showLocationAddress", "true");

        System.out.println("showLocationAddress=" + showLocationAddress +
            " in ConfigurationActionImpl.processAction().");
    }
}
```

这个类扩展了 _DefaultConfigurationAction_ 类，并添加了一个新的 _processAction()_ 方法。父类的 _processAction() 方法负责读取设置表单中的偏好设置数据，并把数据保存到数据库。通常，应该添加适当的逻辑来验证从表单接收到的数据。这个简单的示例只是为了演示从 action 请求中获取偏好设置数据。

添加的另一个常规方法是 _render()_ 方法。用户点击设置按钮时调用这个 render 方法。示例中继续使用继承的 _DefaultConfigurationAction_ 类的 render 方法。

``
注意：不应该调用 _preferences.store()_ 来保存偏好设置，因为父类 _DefaultConfigurationAction_ 会自动进行保存。
``

最后，在 _liferay-portlet.xml_ 中配置这个自定义的新配置类。将现有的 _<configuration-action-class>...</configuration-action-class>_ 替换为 _<configuration-action-class>com.nosester.portlet.eventlisting.action.ConfigurationActionImpl</configuration-action-class>_。下面的片断说明新配置在 _liferay-portlet.xml_ 中的位置：

```
....
<portlet>
    <portlet-name>locationlisting</portlet-name>
    <icon>/icon.png</icon>
    <configuration-action-class>com.nosester.portlet.eventlisting.action.ConfigurationActionImpl</configuration-action-class>
    <header-portlet-css>/css/main.css</header-portlet-css>
    <footer-portlet-javascript>
        /js/main.js
    </footer-portlet-javascript>
    <css-class-wrapper>locationlisting-portlet</css-class-wrapper>
</portlet>
....
```

配置类已经实现好了，修改视图 JSP 来响应 portlet 的偏好设置。

###第四步：编辑响应偏好设置的 JSP

在 view.jsp 中添加根据偏好设置中 _showLocationAddress_ 键的值来显示或隐藏地点的地址今年的逻辑。

下面是 _view.jsp_ 文件的内容。在代码后面，会指出处理 portlet 偏好设置的代码：

```
<%@ include file="/html/init.jsp" %>

This is the <b>Location Listing Portlet</b> in View mode.

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

<%
boolean showLocationAddress_view = GetterUtil.getBoolean(portletPreferences.getValue("showLocationAddress", StringPool.TRUE));
%>

<liferay-ui:search-container emptyResultsMessage="location-empty-results-message">
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
            property="description"
        />

        <c:choose>
            <c:when test="<%= showLocationAddress_view == true %>">
                <liferay-ui:search-container-column-text
                    name="street-address"
                    property="streetAddress"
                />

                <liferay-ui:search-container-column-text
                    name="city"
                    property="city"
                />

                <liferay-ui:search-container-column-text
                    name="state-province"
                    property="stateOrProvince"
                />

                <liferay-ui:search-container-column-text
                    name="country"
                    property="country"
                />
            </c:when>
        </c:choose>

        <liferay-ui:search-container-column-jsp
            align="right"
            path="/html/locationlisting/location_actions.jsp"
        />
    </liferay-ui:search-container-row>

    <liferay-ui:search-iterator />

</liferay-ui:search-container>
```

分解上面的代码。从获取偏好设置 _showLocationAddress_ 的值，并保存到 _showLocationAddress\_view_ 变量开始：

```
<%
boolean showLocationAddress_view = GetterUtil.getBoolean(portletPreferences.getValue("showLocationAddress", StringPool.TRUE));
%>
```

根据传到 _portletPreferences.getValue(key, default)_ 方法中 _default_ 参数的 _StringPool.TRUE_，如果没找到键 _showLocationAddress_ 默认值就是 _true_。然后在 _street address_、_city_、_state_ 和 _contry_ 文本域的周围用 _<c:choose ><c:when test="..."> ... <c:when></c:choose>_ 标签添加条件代码块。

```
<liferay-ui:search-container-row ... />
    ...
    <c:choose>
        <c:when test="<%= showLocationAddress_view == true %>">
            <liferay-ui:search-container-column-text
                name="street-address"
                property="streetAddress"
            />

            <liferay-ui:search-container-column-text
                name="city"
                property="city"
            />

            <liferay-ui:search-container-column-text
                name="state-province"
                property="stateOrProvince"
            />

            <liferay-ui:search-container-column-text
                name="country"
                property="country"
            />
        </c:when>
    </c:choose>
    ...
</liferay-ui:search-container-row>
```

如果 _showLocationAddress\_view_ 是 _true_，就显示所有地点属性；如果是 _false_，_address_ 会被忽略。

就是这样创建造福一方的设置页面和为 portlet 添加偏好设置。浏览设置页面，并且实际修改一下偏好设置。打开 Location Listing portlet 的 _Configuration_ 页面，会看到一个 _show-location-address_ 复选框。

![show-location-address-pref.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=d982f292-5bc0-4b67-bdf6-5427cd7e5067)

图 3.14：_Cofiguration_ 页面上新的 portlet 偏好设置已经可以使用了。

每次修改 _show-location-address_ 复选框并点击 _Save_ 时，_ConfigurationActionImpl_ 类的 _ConfigurationActionImpl.processAction(...)_ 方法会输出 portlet 偏好设置 _showLocationAddress_ 的值：

```
showLocationAddress=true in ConfigurationActionImpl.processAction().
```

取消选中复选框，地点的地址信息会从 Location Listing portlet 的视图中消失。

![portlet-preferences-modifying-view.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=2cca9ad5-bfe6-4a44-b68d-97c5ed83e090)

图 3.15：Liferay Portal 中可以轻松自定义 portlet 的用户界面。有权限的用户可以在 portlet 的配置页面调整偏好设置。

非常好！现在你知道如何开发 Liferay 的 portlet 偏好设置了。

接下来，使用 Plugins SDK 来创建一个扩展其它 plugin 的 plugin。