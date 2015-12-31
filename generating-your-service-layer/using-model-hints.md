#使用 Model Hints (没想到好的翻译)

已经创建了模型实体，也实现了新增、修改这些实体的业务逻辑，你或许想帮助用户校验模型实体的数据。例如，在 Event Listing 项目中，想让用户创建未来的社交活动而不是以前的。并且使用一个好用的文本编辑器来输入活动描述可能更好。不能在 portal 项目的某一个地方定制这样的功能吗？好消息！Service Builder 允许在一个名为 _portlet-model-hints.xml_ 的文件中以 _model hints_ 的方式来指定这些信息，这个文件在 _docroot/WEB-INF/src/META-INF_ 目录。Liferay 称它们为 _model hints_ 是因为它们为实体应该如何呈现给用户提供建议，也可以说明保存实体的数据库列的大小。

model hints 提供配置 AlloyUI 标签库（aui）如何哪一天模型数据。当 Liferay Portal 显示应用中的表单域时，首先检查设置的 model hints，然后根据设置信息显示表单输入域。

先看一下 Service Builder 为 Event Listing portlet 生成的 model hints。查看 _docroot/WEB-INF/src/META-INF/portlet-model-hints.xml_ 文件。如果按照前面章节的方式做，Service Builder 生成的 _portlet-model-hints.xml 有如下内容：

```
<?xml version="1.0"?>

<model-hints>
    <model name="com.nosester.portlet.eventlisting.model.Event">
        <field name="eventId" type="long" />
        <field name="companyId" type="long" />
        <field name="groupId" type="long" />
        <field name="userId" type="long" />
        <field name="createDate" type="Date" />
        <field name="modifiedDate" type="Date" />
        <field name="name" type="String" />
        <field name="description" type="String" />
        <field name="date" type="Date />
        <field name="locationId" type="long" />
    </model>
    <model name="com.nosester.portlet.eventlisting.model.Location">
        <field name="locationId" type="long" />
        <field name="companyId" type="long" />
        <field name="groupId" type="long" />
        <field name="userId" type="long" />
        <field name="createDate" type="Date" />
        <field name="modifiedDate" type="Date" />
        <field name="name" type="String" />
        <field name="description" type="String" />
        <field name="streetAddress" type="String" />
        <field name="city" type="String" />
        <field name="stateOrProvince" type="String" />
        <field name="country" type="String" />
    </model>
</model-hints>
```

根级（root-level）元素是 _model-hints_。所有模型实体都使用 _model-hints_ 的 _model_ 子元素表达。每个 _model_ 元素必须有一个 _name_ 属性，描述模型类的全限定名（就是包含包名和类名）。每个 _model_ 元素都有 _field_ 元素，用来说明模型实体的属性。最后，每个 _field_ 元素都必须有 _name_ 和 _type_ 属性。_field_ 元素的名字（name）和类型（type）对应 service.xml 中定义的实体属性的名字和类型。Service Builder 根据 service.xml 文件生成所有上述内容。

要为域（field）添加 hint，就在 _field_ 标签中添加 一个 _hint_ 标签。例如，可以添加 _display-with_ hint 来说明这个域显示的宽度是多少像素（pixel）。默认宽度是 350 像素。为了让某个输入域显示为 50 像素宽，需要嵌入一个 _name_ 为 _display-with_ 的 _hint 元素，并设置值为 50 来表示 50 像素。示例如下：

```
<field name="name" type="String">
    <hint name="display-width">50</hint>
</field>
```

为了检查一个域上的 hint 是否生效，需要再运行一次 Service Builder，并且重新部署 portlet。修改 _display-with_ 并不会影响 _name_ 域实际可以输入的字符个数；它只是控制这个域在 AlloyUI 输入表单中的显示宽度。

配置模型域的数据库字段的最大长度（例如，这个域可多可以保存的字符数），使用 _max-length_ hint。_max-legnth_ 的默认值是 75 字符。如果想让 _name_ 域保存 100 个字符，需要给域洄 _max-length_ hint：

```
<field name="name" type="String">
    <hint name="display-width">50</hint>
    <hint name="max-length">100</hint>
</field>
```

修改 portlet-model-hints.xml 文件后，记得运行 Service Builder 并重新部署 portlet。

已经提到了几个不同的 hint。下表中是可用的模型 hint：

*模型 hint 值及说明*

| 名称 | 值类型 | 说明 | 默认值 |
| --- | --- | --- | --- |
| auto-escape | boolean | 文字内容是否使用 HtmlUtil.escape 进行处理 | true |
| autoSize | boolean | 是否使用带激动条的 text area 显示 | false |
| day-nullable | boolean | 是否允许日期域的 day 为空 | false |
| default-value | String | 域的默认值 | 空字符串 |
| display-height | integer | 使用 aui 标签库时表单域的显示高度 | 15 |
| display-width | integer | 使用 aui 标签库时表单域的显示宽度 | 350 |
| editor | boolean | 是否使用编辑器（例如 FCKEditor） | false |
| max-length | integer | 设置生成 SQL 语句时字段的最大长度 | 75 |
| month-nullable | boolean | 是否允许日期域的 month 为空 | false |
| secret | boolean | 是否隐藏用户输入的内容（例如输入密码？） | false |
| show-time | boolean | 显示日期时是否显示时间 | true |
| upper-case | boolean | 是否将字符转为大写 | false |
| year-nullable | boolean | 是否允许日期域的 year 为空 | false |
| year-range-delta | integer | 使用 aui 标签库时，显示日期域允许从今天开始往后多少年 | 5 |
| year-range-future | boolean | 是否包含未来的日期 | true |
| year-range-past | boolean | 是否包含过去的日期 | true |

Liferay Portal 还有自己的模型 hint XML  配置文件，就是 Liferay 的 _portal-impl/classes/META-INF_ 目录中的 _portal-model-hints.xml_ 文件。

可以用 _default_hints_ 元素来定义一个 hint 列表，它会应用到模型的每个域。例如，在 _model_ 元素内添加以下元素会使用 _display-with_ 300 应用到每个域：

```
<default-hints>
    <hint name="display-width">300</hint>
</default-hints>
```
可以在 _nodel-hints_ 元素内定义 _hint-collection_ 元素，这样可以把一组 hint 定义在一起。hint 集合必须设置 _name_ 属性。例如，Liferay 的 _portal-hints.xml_ 中定义了以下 hint 集合：

```
<hint-collection name="CLOB">
    <hint name="max-length">2000000</hint>
</hint-collection>
<hint-collection name="URL">
    <hint name="max-length">4000</hint>
</hint-collection>
<hint-collection name="TEXTAREA">
    <hint name="display-height">105</hint>
    <hint name="display-width">500</hint>
    <hint name="max-length">4000</hint>
</hint-collection>
<hint-collection name="SEARCHABLE-DATE">
    <hint name="month-nullable">true</hint>
    <hint name="day-nullable">true</hint>
    <hint name="year-nullable">true</hint>
    <hint name="show-time">false</hint>
</hint-collection>
```

可以在模型域中通过 hint 集合的名字可以引用 hint 集合。例如，如果像上面那样在 _model-hints 元素中定义了一个 _SEARCHABLE-DATE_ 集合，在 Event 模型的 _date_ 域可以使用 _hint-collection_ 元素通过名字引用它：

```
<field name="date" type="Date">
    <hint-collection name="SEARCHABLE-DATE" />
</field>
```

同样地，修改 _portlet-model-hints.xml_ 后别忘了运行 Service Builder。

现在可以在 Event Listing Portlet 和 Location Listing Portlet 中使用很多模型 hint 了。先给用户来一个编辑器来输入 _description_ 域。因为想给 event 和 location 实体都应用这个 hint，那就先定义 hint 集合，再引用它。

在 _portlet-model-hints.xml 文件的 _model-hints_ 根元素的下面定义 hint 集合：

```
<hint-collection name="DESCRIPTION-TEXTAREA">
    <hint name="display-height">105</hint>
    <hint name="display-width">500</hint>
    <hint name="max-length">4000</hint>
</hint-collection>
```

然后像上面说明的那样用以下内容替换 event 和 location 的 _description_ 域：

```
<field name="description" type="String">
    <hint-collection name="DESCRIPTION-TEXTAREA" />
</field>
```

现在运行 Service Builder 生成服务，重新部署 portlet 项目，并且使用 portlet 添加或修改一个活动。看看 _description_ 的 textarea 输入域的大小是否和实体 hint 中定义的一样。

好的，你已经通过 Liferay 的模型 hint 学会了“说服的艺术”。现在，不仅可以影响模型的域如何显示，还可以设置它在数据库表中列的大小。还会组织 hint。直接在域上定义 hint，为模型的所有域应用默认的 hint，或者定义 hint 集合并在任意范围应用它。看样子你已经掌握了如何使用 Liferay 模型  hint 的技巧。

下面，来搞清楚远程服务的实现。