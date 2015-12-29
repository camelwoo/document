#定义对象-关系映射（ORM）

为了演示如何使用 Service Builder，继续使用在 [Developing Apps with Liferay IDE](http://www.liferay.com/documentation/liferay-portal/6.2/development/-/ai/developing-apps-with-liferay-ide-liferay-portal-6-2-dev-guide-02-en) 中创建的 _event-listing-portlet_ 项目。这是一个示例项目，一个虚拟组织 Nose-ster 用它来安排社交活动。我们使用 _event-listing-portlet_ 项目来管理和显示活动列表。我们需要添加一些实体或模型来表示 Nose-ster 的活动和地点。会定义两个实体：活动（_Event_）实体和地点（_Location_）实体。_Event_ 实体表示一个可以添加到日历中的社交活动，_Location_ 实体表示社交活动发生的地点。因为一个活动必须有一个地点，活动实体会引用地点实体作为它的一个属性。

[service-builder-view-events.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=37ee21db-7d29-411f-be01-fe0ad48d8b54)

图 5.1：可以通过 Event Listing Portlet 添加社交活动。这个 portlet 凭借活动和地点实体以及 Service Builder 根据实体生成的服务组件。
[TODO: 以下内容以后有时间再翻译]
If you’d like to examine the finished example project, it’s a part of our Dev Guide SDK which you can browse at https://github.com/liferay/liferay-docs/tree/master/devGuide/code/devGuide-sdk. The project is in the SDK’s portlets/event-listing-portlet folder.

tip-pen-paper.png Note: If you’re looking for a fully-functional portlet application that can manage events, please use Liferay’s Calendar portlet instead. The example described in this section is only intended to demonstrate how to use Service Builder. The Calendar portlet provides many more features than the simple example application described here. For information about the Calendar portlet, please refer to the chapter on Liferay’s collaboration suite in Using Liferay Portal 6.2.

As with any portlet project, the event-listing-portlet project’s Java sources lie in its docroot/WEB-INF/src folder. Notice that Liferay IDE’s portlet wizard created the EventListingPortlet.java and LocationListingPortlet.java files in the com.nostester.portlet.eventlisting package. We’ll add some business logic to these portlet classes after using Service Builder to create a service layer for our event and location entities.

使用 Service Builder 第一步是在 `docroot/WEB-INF/service.xml` 中定义模型类和类的属性。在 Service  Builder 的术语中，数据模型类（_Event_ 和 _Location_）称为实体。我们对 _Event_ 和 _Location_ 实体的需求相当简单。它们有以下属性：

**_Event_ 实体的属性**

| 属性 | 属性类型 | 属性说明 |
| --- | :---: | --- |
| name | String | 活动的名称 |
| description | String | 活动描述 |
| date | Date | 活动发生的日期和时间 |
| locationId | long | 活动发生的地点，使用地点的ID表示地点 |

**_Location_ 实体的属性**

|属性|属性类型|属性说明|
|---|:---:|---|
|name|String|地点的名称|
|description|String|地点描述|
|streetAddress|String|地点所在的街道地址|
|city|String|地点所在的城市|
|stateOrProvince|String|地点所在的州或省|
|country|String|地点所在的国家|

Service Builder 使用名为 service.xml 的文件来定义实体。一旦建立这个文件就可以定义实体了。我们将会带你过一遍以上实体完整的生成过程，使用 Liferay IDE 可以轻易完成，只需要7步：

1. docroot/WEB-INF 目录中没有 service.xml，就创建这个文件。
2. 定义服务的全局信息。
3. 定义实体。
4. 为每个实体定义列（属性）
5. 定义实体之间的关系。
6. 定义获取数据是的默认排序。
7. 定义根据指定参数从数据库获取数据的 finder 方法。

下面开始使用 Liferay IDE 来创建服务。

##创建 service.xml 文件

要生成服务层，必须先创建 service.xml 文件。这个文件的详细定义信息及格式要求可以查看它的 [DTD(Document Type Declaration) 文件](http://www.liferay.com/dtd/liferay-service-builder_6_2_0.dtd)。可以根据 DTD 手工创建 service.sml，也可以使用 Liferay IDE 创建。Liferay IDE 帮助你一步一步创建出符合 DTD 定义的 service.xml。本例中会使用 Liferay IDE 创建 service.xml。

如果在 docroot/WEB-INF/src 目录中已经有默认的 service.xml 了，检查文件中是否有一个 name 值是 “Foo” 的 <entity /> 元素，如果有就删除 <entity name="Foo" ...> ... </entity> 的内容。项目向导生成 Foo 作为示例，不过没什么用。

如果还没有 service.xml，可以用 Liferay IDE 方便的创建它。在 Package Explorer 中右键点击项目名称，选择菜单中的 File → New → Liferay Service Builder。Liferay IDE 会在 docroot/WEB-INF/src 目录中创建 service.xml，并且使用 Overview 视图显示文件内容（译注：我操作时不是用这个模式打开的）。

Liferay IDE 还提供了 Diagram 视图和 Source 视图，让你可以从不同的角度查看 service.xml 文件。Diagram 视图在建立和呈现实体之间关系时非常有用。源码视图显示原始的 XML 格式内容。可以按照自己的喜好在这几种方式之间任意切换。因为 Overview 视图中操作很方便，我们就用这种模式创建服务。

先填写服务的全局信息。

##定义全局服务信息

服务的全局信息对所有实体有效，因此我们先指定这些信息。在 service.xml 的 Overview 方式窗口左上角的 _Service Builder_ 节点，这时右侧显示 Service Builder 表单，在这里可以输入服务的全局信息。输入的内容包括服务的包、作者和命名空间。以下是例子中用到的值：

- Package path: com.nosester.portlet.eventlisting
- Auto namespace tables: no
- Author: [your name]
- Namespace: Event

包路径指定服务以及持久化类生成到哪个包（package）中。上面指定的是生成到 docroot/WEB-INF/service 目录的 com.nosester.portlet.eventlisting 包中。服务和持久化类所在包的完整路径分别是 docroot/WEB-INF/service/com/nosester/portlet/eventlisting 和 docroot/WEB-INF/src/com/nosester/portlet/eventlisting。参考下面的“生成服务”章节，了解这些包的内容。

Service Builder 命名数据库表的时候使用了命名空间（其实就是加了前缀），本例中使用 _Event_ 作为命名空间。Service Builder 在 docroot/WEB-INF/sql 目录中生成的以下 SQL 脚本中使用命名空间：

- indexes.sql
- sequences.sql
- tables.sql

Liferay Portal 使用这些脚本创建 service.xml 中定义的全部实体。Service Builder 在表名前面添加命名空间。因为上面指定的命名空间是 _Event_ ，所以生成的实体对应的表名以 _Event\__ 作为前缀。每个 Service Builder 项目的命名空间应该是唯一的。不同的项目应该使用不同的命名空间，并且不能使用 Liferay 已经使用的命名空间（如 Users 或 Groups）。如果不清楚哪些命名空间已经被使用了，可以看看 Liferay 数据库中的表名。

全局信息的最后，在 service.xml 中输入你的名字作为服务的作者。Service Bilder 会在所有生成的类和接口中添加 _@author_ 注解和指定的作者名字。保存 service.xml。下面，添加 _Event_ 实体和 _Location_ 实体。

##定义服务实体

实体是服务的核心和灵魂。实体表达了 Java 中的模型对象与数据库中表、字段的映射关系。定义好后 Service Builder 会自动处理映射，提供工具类来处理和持久化 Java 对象。

__Event 实体的信息__

- _Name_：Event
- _Local service_：yes
- _Remote service_：yes

__Location 实体的信息__

- _Name_：Location
- _Local service_：yes
- _Remote service_：yes

要创建这些实体，要在 service.xml 编辑器的 Overview 视图左侧的 Outline 中选择 Service Builder 下面的 Entities 节点，这时窗口右侧的 Entities 表格是空的。点击表格右侧上方的 _Add Event_ 图标（绿色的加号）来添加实体。在实体名称（entity's name）中输入 _Event_，并且选中本地服务（Local Service）和远程服务（Remote Service）的选项。用同样的方法创建地点（_Location_）实体。

[service-add-entity.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=e000c195-9ff2-4055-9034-704e9df52aee)

图 5.2：使用 Liferay IDE 在 service.xml 的 Overview 视图中添加实体非常简单。

实体的名字用来命名持久化实体实例的数据库表名。实体上数据库的表名是实体名加上命名空间前缀；对于这个例子，有一个表名为 _Event\_Event_，另一个是 _Event\_Location_。

设置 _local service_ 属性为 _true_ 来告诉 Service Builder 生成服务的本地接口。_local service_ 的默认值是 _false_。不过在这个例子中要求可以调用服务，它就在 portlet 的 .war 文件中。这个 portlet 会部署到 Liferay 服务器中。因此，从 Liferay 服务器的角度来看，它个服务是“本地”的。

设置 _remote serivce_ 属性为 _true_ 来告诉 Service Builder 生成服务的远程接口。_remote service_ 的默认值是 _true_。不需要生成远程服务就可以构建功能完善的活动列表应用，因此可以对所有实体设置 _local service_ 为 _true_，_remote service_ 为 _false_。

``提示：假设已经通过像 JPA 这样的框架实现了 DAO 服务，可以设置 _local service_ 为 _false_，_remote service_ 为 _true_，这样远程服务实现类就可以调用已有的 DAO 了。这样实体可以集成 Liferay 的权限检查系统（permission-checking system），并且可以访问 Service Builder 生成的 web service API。这样做非常方便，非常强大，也是 Liferay 经常被用到的特性。``

现在，我们已经创建了 Event 和 Location 实体，下面用 _column_ 来定义它们的属性。


##定义实体列（属性）

每个实体都由它的列（也称为属性）来描述。这些属性映射的两端是表的字段和对象的属性。要添加 _Event_ 实体属性，需要在 service.xml 的 Overview 视图中 outline 窗口中展开 Entities 节点和 Event 实体的节点。然后选择 Columns 节点，这时会显示 Event 实体列的表格。

Service Builder 会生成 service.xml 中定义的所有 column 对应的数据库字段。它会将定义中 column 的 Java 数据类型映射为合适的数据库列的数据类型，并且会适配 Liferay 支持的所有数据库类型。Service Builder 运行时会自动生成 Hibernate 处理对象关系映射的配置文件；还会生成模型所有属性的访问方法。column 的 _Name_ 属性用于生成 Java 实体的 getter 和 setter 方法的名称。column 的 _Type_ 属性表示它在 Java 中的数据类型。如果 column 的 _Primary_ 属性值为 _true_，这个 column 就是实体主键的组成部分。实体的主键是实体的唯一标识。如果只有一个 column 的 _Primary_ 属性值是 _true_，那么这个 column 就是实体的主键。例子中就是这样的。也可以将实体的多个 column 设置为主键，这时这几个 column 组合到一起作为实体的联合主键。

和添加表对应的实体类似，为实体添加下面这些属性：

__Event 实体的属性__

| Name | Type | Primary |
| --- | --- | --- |
| eventId | long | yes |
| name | String | no |
| description | String | no |
| date | Date | no |

__Location 实体的属性__

| Name | Type | Primary |
| --- | --- | --- |
| locationId | long | yes |
| name | String | no |
|description | String | no |
| streetAddress | String | no |
|city | String | no|
|stateOrProvince | String | no |
| country | String | no |

点击添加图标来添加每一个属性。然后填写属性的名称、选择类型并且指定它是否是主键。当属性在 _Type_ 域时会出现选项图标。点击这个图标来选择合适的 column 类型。为 Event 实体的每个属性创建 column。给 Location 实体添加 column 的操作类似。

除了添加主键和其它属性，建议添加保存“站点ID”和“portal 实例ID”的属性。这样 portlet 就可以支持 Liferay 的多租户（multi-tenancy）特性，这样每个 portal 实例的每个站点上的每个该 portlet 的实例都可以有单独的 portlet 数据。要保存站点的 ID，需要添加一个名为 _groupId_ 且类型为 _long_ 的 column。要保存 portal 实例的 ID，需要添加一个名为 _companyId_ 并且类型为 _long_ 的 column。将这两个 column 添加到 _Event_ 和 _Location_ 实体中。

__Portal 和 站点的 column__

| Name | Type | Primary |
| --- | --- | --- |
| companyId | long | no |
| groupId | long | no |

可能还要知道谁拥有实体的实例，要想跟踪这些信息就添加一个名为 _userId_ 且类型为 _long_ 的 column。

__用户 column__

| Name | Type | Primary |
| --- | --- | --- |
| userId | long | no |

最后，为 _Event_ 和 _Location_ 实体添加用于审计的 column。添加一个名为 _createDat_ 类型为 _Date_ 的 column 来记录实例创建的日期。再添加一个名为 _modifiedDate_ 类型为 _Date_ 的 column 来跟踪实体最后一次被修改的时间。

__审计 column__

| Name | Type | Primary |
| --- | --- | --- |
| userId | long | no |
| createDate | Date | no |
| modifiedDate | Date | no |

非常棒！实体不仅包含自身的属性还支持多租户和实体审计。接下来，我们要指定 _Event_ 实体和 _Location_ 实体之间的关系 。

##定义实体之间的关系

经常需要在某个实体中引用另一个实体，也就是说，需要关联实体。下面会演示在这个活动列表项目中如何定义。

前面说到，每个活动必须有一个地点，因此每个 _Event_ 实体必须依赖于 _Location_ 实体。通过 Liferay IDE 的 Diagram 视图打开 service.xml 可以方便地设置实体之间的关系。选中视图右侧 Palette 的 Connections 下面的 Relationship 选项，点击 _Event_ 实体然后移动属性到 _Location_ 实体。Liferay IDE 会画一条从 _Event_ 实体到鼠标的虚线，点击 _Location_ 实体来完成关系的设置。Liferay IDE 将虚线转换为实线，并且在 _Location_ 实体一端添加箭头。保存 service.xml。

恭喜！你已经完成了实体的关联。在 Diagram 视图中会显示它们的关联关系，就像下面这样。

[service-builder-relate-entities.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=cebb3060-d882-4bac-a9cc-cd94a8f42518)

图 5.3：在 Liferay IDE 的 Diagram 视图中瞬间完成实体关联。

将 service.xml 的编辑器切换到 Source 视图，可以看到 Liferay IDE 在 _Event_ 实体中添加了一个 column 来保存 _Location_ 实体实例的引用：

```xml
<column name="locationId" type="long"></column>
```

现在，实体的 column 也设置好了，下面来设置从数据库中获取数据时的默认排序。

##定义排序

在获取（查询）实体的多个实例（多条数据）时往往需要按照特定的顺序进行排序。可以在 service.xml 中指定默认的排序。

假设想要按照日期正序返回 _Event_ 实体，按照 _name_ 属性的字母顺序返回 _Location_ 实体。通过 Liferay IDE 可以轻松实现。把 service.xml 的编辑器切换回 Overview 视图，然后选择视图左侧 outline 中 _Evente_ 节点下面的 Order 节点，IDE 会显示设置实体排序的表单。选中 _Specify ordering_ 复选框，就会显示设置排序的表单。点击表格右侧的添加（add）图标（绿色加号），在 column 中输入 _date_ 来作为 _Event_ 实体的排序。点击 _by_ 输入框右侧的浏览（Browse）图标，选择 _asc_ 选项。这会让 _Event_ 实体按照日期升序排序。指定 _Location_ 实体的排序类似，设置 _name_ 作为 column，_asc_ 作为 _select by_ 的值。

最后要设置从数据库获取这些实例的查询器（finder）方法。

##定义查询器（finder）方法

Finder 方法可以根据指定的参数从数据库中获取实体对象。一般每个实体至少有一个 finder 方法。Service Builder 可以根据你在实体上定义的 finder 生成多个 finder 方法。它可以根据参数生成实体的获取、查询、删除和 count 方法。

在这个例子中，我们想获取每个站点的 _Event_ 和 _Location_ 实体。通过 Liferay IDE 的 Overview 视图来设置 finder。选中左侧 outline 中 Event 实体下的 _Finders_ 节点，点击表格右侧的添加图标（绿色加号）来添加 finder，右侧会显示空的 _Finders_ 表格，_Name_ 输入 _GroupId_，_Return Type_ 输入 _Collection_。使用驼峰命名法（camel-case）来命名 finder，因为这个名字会用作查询的方法名。IDE 会在 _Finders_ 节点下新建 _GrupId_ 节点。下面我们在这个 _GroupId_ 节点下设置 finder 的 column。

在 _GrupId_ 节点下，IDE 创建了 _Finder Columns_ 节点，选中这个节点来设置 finder 的参数。点击添加图标（绿色加号）来创建新的 finder column，并且设置 _Name_ 为 _gorupId_。可以为 finder 设置多个参数（column）；这是个简单的例子。用同样的方法创建 _Location_ 实体的 finder 方法，参数也是 _groupId_。

运行 Service Builder，它就会在 \-Persistence 和 \-PersistenceImpl 类中生成 _Event_ 实体和 _Location_ 实体的 finder 方法（fetchByGroupId, findByGroupId, removeByGroupId, countByGroupId）。前面那个是接口，第二个是实现类。接口在 docroot/WEB-INF/service/com/nosester/portlet/eventlisting/service/persistence 目录，实现在 docroot/WEB-INF/src/com/nosester/portlet/eventlisting/service/persistence 目录。



好极了（老外感叹词真多）！你已经创建了 Event Listing 示例项目的服务及 _Event_ 和 _Location_ 实体。

我们已经把服务及完整的 Event Listing 示例项目的源代码放到了 Dev Guide SDK，你可以浏览 https://github.com/liferay/liferay-docs/tree/master/devGuide/code/devGuide-sdk。这个项目就在 SDK 的 portlets/event-listing-portlet 目录中。

为了方便，将 service.xml 的内容列在下面。我们还添加了一些注释。你的 service.xml 应该和下面的内容类似：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE service-builder PUBLIC "-//Liferay//DTD Service Builder 6.2.0//EN"
"http://www.liferay.com/dtd/liferay-service-builder_6_2_0.dtd">
<service-builder package-path="com.nosester.portlet.eventlisting">
    <author>Joe Bloggs</author>
    <namespace>Event</namespace>

    <entity name="Event" local-service="true" remote-service="true">

        <!-- PK fields -->

        <column name="eventId" type="long" primary="true" />

        <!-- Audit fields -->

        <column name="companyId" type="long" />
        <column name="groupId" type="long" />
        <column name="userId" type="long" />
        <column name="createDate" type="Date" />
        <column name="modifiedDate" type="Date" />

        <!-- Other fields -->

        <column name="name" type="String" />
        <column name="description" type="String" />
        <column name="date" type="Date" />
        <column name="locationId" type="long" />

        <!-- Order -->

        <order by="asc">
                <order-column name="date" />
        </order>

        <!-- Finder methods -->

        <finder name="GroupId" return-type="Collection">
                <finder-column name="groupId" />
        </finder>
    </entity>

    <entity name="Location" local-service="true" remote-service="true">

        <!-- PK fields -->

        <column name="locationId" type="long" primary="true" />

        <!-- Audit fields -->

        <column name="companyId" type="long" />
        <column name="groupId" type="long" />
        <column name="userId" type="long" />
        <column name="createDate" type="Date" />
        <column name="modifiedDate" type="Date" />

        <!-- Other fields -->

        <column name="name" type="String" />
        <column name="description" type="String" />
        <column name="streetAddress" type="String" />
        <column name="city" type="String" />
        <column name="stateOrProvince" type="String" />
        <column name="country" type="String" />

        <!-- Order -->

        <order by="asc">
                <order-column name="name" />
        </order>

        <!-- Finder methods -->

        <finder name="GroupId" return-type="Collection">
                <finder-column name="groupId" />
        </finder>
    </entity>
</service-builder>
```

现在，已经配置好了 Event Listing 示例项目的服务，运行 Service Builder 进行构建，然后就能看到生成的代码了。