#调用 Liferay 内置服务

每个服务都为同一 JVM 下的应用提供了本地接口。这些接口通常使用 _-ServiceUtil_ 类来调用。这些类隐藏了服务实现的复杂性。Liferay 核心服务也是用 Service Builder 生成的，可以用它们的 _-ServiceUtil_ 类来调用 Liferay 的服务。下面的 JSP 片断演示了如何获得组织中的博客作者列表。

```
<%@ page import="com.liferay.portlet.blogs.service.BlogsStatsUserLocalServiceUtil" %>
<%@ page import="com.liferay.portlet.blogs.util.comparator.StatsUserLastPostDateComparator" %>
...
<%@
List statsUsers = BlogsStatsUserLocalServiceUtil.getOrganizationStatsUsers(
    organizationId, 0, max, new StatsUserLastPostDateComparator());
%>
```

代码中调用 _-LocalServiceUtil_ 类 _BlogsStatsUserLocalServiceUtil_ 调用静态方法 _getOrganizationStatsUsers()_。

除了使用 Service Builder 生成的服务，还可以调用大量 Liferay 内置的服务。包括以下服务：

- UserService - 用于访问、添加、认证、删除和更新用户。
- OrganizationService - 用于访问、添加、删除和更新组织机构。
- GroupService - 用于访问、添加、删除和更新组。
- CompanyService - 用于访问、添加、检查和更新组织机构。
- ImageService - 用于访问图片。
- LayoutService - 用于访问、添加、删除、导出、导入和更新布局。
- PermissionService - 用于检查权限。
- UserGroupService - 用于访问、添加、检查、删除和更新用户组。
- RoleService - 用于访问、添加、取消分配（unassigning）、检查、删除和更新角色。

这些服务的更多信息可以在 [Liferay Portal CE 的 javadoc](http://docs.liferay.com/portal/6.2/javadocs/) 或 Liferay Portal EE 的 javadoc 中找到。Liferay Portal EE 的 javadoc 包含在 Liferay Portal EE 文档的 .zip 文件中，这个文件可以从 http://www.liferay.com 的 Customer Portal 处下载。

下面，学习如何让 Liferay Portal 更好地在用户界面中展现实体模型的数据。