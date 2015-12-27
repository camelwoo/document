#Service Builder 是什么

Service Builder 是一个模型驱动（model-drien）的代码生成工具，Liferay 提供给开发人员定义称为实体（entities）的对象模型（object model）。Service Builder 采用对象关系映射（ORM）技术生成服务层，将对象模型和数据库代码剥离开。这样即使没有数据库也可以为应用添加必要的业务逻辑了。Service Builder 使用一个 XML 配置文件，并生成必要的模型、持久层和服务层的代码。这些层级提供清晰的关注点分离。``（译注：意思是说每层各司其职，后会会说到。）``生成了大多都需要实现的增、删、改、查操作的代码，这样你就可以专注于高层的服务设计。下面说说使用 Service Builder 主要的好处：

- 和 Liferay 集成
- 自动生成模型、持久层和服务层代码
- 自动生成本地和远程服务
- 自动生成 Hibernate 和 Spring 配置文件
- 支持生成实体的查询器（finder，后面统一叫 finder）方法和结合权限的 finder 方法
- 内置支持实体对象的缓存
- 支持自定义 SQL 查询和动态查询
- 节省开发时间

Liferay 中所有内置数据库的持久层代码都是使用 Service Builder 生成的。实际上，Liferay 的全部服务层（包括本地和远程服务）也都是使用 Service Builder 生成的。还有 Plugin SDK 的服务层代码。Liferay Portal 和 plugin 中大量使用 Service Builder，说明它是一个健壮、可靠的工具。正如在本章中将会看到的那样，Service Builder 非常易用，并能大大节省开发时间。虽然 Service Builder 生成的文件数量多得有些吓人，但是开发人员只需要和很少的文件打交道，以此来定制一些功能或添加业务逻辑。

``注意：开发 portlet 并不是一定要用 Service Builder，完全可以使用你熟悉的框架（如 JPA 或 Hibernate）。``

Service Builder 节省开发时间的一个主要方法是完全不需要写和维护数据库访问的代码。要生成基本的服务层，只需要创建一个 _service.xml_，然后运行 Service Builder。它会生成一个新的 jar 文件。这个文件中包括模型层、持久层、服务层和相关的基本架构。清晰的层级代表着健康的关注点分离。模型层负责定义实体对象，持久层负责数据库交互，服务层负责将增删改量（CRUD）和其它操作暴露为 API。Service Builder 生成的代码是数据库无关（database-agnostic）的，就像 Liferay 自己那样。``（译注：“数据库无关”并不是说和数据库没有关系，是说不限定到特定的数据库，可以支持多种数据库。）``

Service Builder 生成的实体（entity）包含一个模型实现类、一个本地服务实现类和一个可选的远程服务实现类。可以在这些类中实现定制的功能和业务逻辑；实际上 Service Builder 生成的类中只在这些类可以进行修改来定制功能。保证所有定制的代码只能写在有限的几个类中，使得 Service Builder 项目更容易维护。本地服务实现类负责调用持久层来保存、获取数据实体。本地服务层含有业务逻辑和访问持久层。在同一个 Java 虚拟机中运行的客户端代码可以调用本地服务层。Service Builder 自动生成可以访问远程服务的代码。生成的远程服务代码包含 SOAP 工具类，可以通过 SOAP 或 JSON 访问。

Service Builder 另一个节省开发时间的方法是生成项目所需的 Hibernate 和 Spring 配置文件。利用 Spring 的依赖注入技术使服务实现类在运行时可用，并使用 AOP 技术管理数据库事务。还使用了 Hibernate 的 ORM 持久化框架。Service Builder 隐藏了这些技术复杂的细节，为开发人员提供便利。开发人员可以在项目中享用控制反转（Dependency Injection）、面向切面的编程（AOP）和 OR Mapping 技术，却不用手工配置 Spring 或 Hibernate 的环境和配置文件。

另一个使用 Service Builder 的好处是可以生成生成 finder 方法。finder 方法使用指定的参数从数据库中获取数据。只需要在 service.xml 中指定要生成哪些 finder 方法，Service Builder 就能生成代码。例如，通过生成的 finder 方法可以查询某个站点（site）或某个站点的某个用户的数据。不仅如此，还可以生成结合权限的 finder 方法，例如，如果使用 Liferay 的权限系统，Service Builder 会生成另一种不同的 finder 方法，它可以查询登录用户有权限查看的数据。

Service Builder 还提供内置的缓存支持。Liferay 在三个层级缓存对象：实体、finder 和 Hibernate。Ehcache 是 Liferay 默认的缓存系统，通过 portal 配置文件（portal.properties）可以进行调整。要想开启实体和 finder 缓存支持，只需要在 service.xml 的 <entity> 元素中设置 `cache-enabled=true` 就可以了。请参考《Using Liferay Portal 6.2》的 Liferay Clustering 章节来了解更多关于 Liferay 缓存的细节。

Service Builder 是一个灵活的工具。生动生成许多与数据库操作相关的通用代码，也不拦着你用 SQL 实现 finder。允许开发人员在 XML 文件中定义 SQL，并实现 finder 方法来执行它。这在多表关联查询时非常有用。Service Builder 还支持动态查询。

总之，鼓励开发人员在开发 portlet 和 plugin 时使用 Service Builder，因为它已经经过 Liferay plugin 和 Liferay Portal 的实践检验。它不需要开发人员手工干涉就能生成清晰的模型、持久层和服务层，本地和远程服务，Spring 和 Hibernate 配置文件，还有其它基础架构。它还允许使用基础的 SQL 查询和生成的 finder 方法，并且结合 Liferay 的权限进行数据过滤。还提供了实体和查询的缓存机制。以上每个特性都节省大量的开发时间，不仅包括初始化环境的时间还包括维护、扩展和定制项目的时间。最后，Service Builder 不是束手束脚的工具，它允许使用 SQL 查询，还支持动态查询。下面，撸胳膊挽袖子准备学习如何使用 Service Builder。