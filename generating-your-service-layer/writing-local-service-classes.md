#写本地服务类

服务层的核心就是 _\-LocalServiceImpl_ 类，在这个类中可以添加业务逻辑。通过本节，会完成 Nose-ster Event Listing 示例 portlet 项目的服务层。先来看看 Service Builder 刚生成的服务类。

注意，Service Builder 生成的 _EventLocalService_ 类是本地服务的接口，接口中包含 _EventLocalservcieBaseImpl_ 和 _EventLocalserviceImpl_ 中所有方法。_EventLocalServiceBaseImpl_ 中只有少量生成的通用方法。因为 _EventLocalService_ 类（译注：按照上下文来看，感觉这里应该是 _EventLocalServiceBaseImpl_ 类。）是生成的，不应该修改它。如果修改了，下次执行 Service Builder 时会被覆盖掉。所有的自定义代码都应该写在 _EventLocalServiceImpl_ 中。

打开 _docroot/WEB-INF/src/com/nosester/portlet/eventlisting/service/impl_ 目录中的 _EventLocalServiceImpl.java_ 文件。

添加以下数据库交互方法：

```
public Event addEvent(
        long userId, long groupId, String name, String description,
        int month, int day, int year, int hour, int minute, long locationId,
        ServiceContext serviceContext)
    throws PortalException, SystemException {

    User user = userPersistence.findByPrimaryKey(userId);

    Date now = new Date();

    long eventId = counterLocalService.increment(Event.class.getName());

    Event event = eventPersistence.create(eventId);

    event.setName(name);
    event.setDescription(description);

    Calendar dateCal = CalendarFactoryUtil.getCalendar(
        user.getTimeZone());
    dateCal.set(year, month, day, hour, minute);
    Date date = dateCal.getTime();
    event.setDate(date);

    event.setLocationId(locationId);

    event.setGroupId(groupId);
    event.setCompanyId(user.getCompanyId());
    event.setUserId(user.getUserId());
    event.setCreateDate(serviceContext.getCreateDate(now));
    event.setModifiedDate(serviceContext.getModifiedDate(now));

    super.addEvent(event);

    // Resources

    resourceLocalService.addResources(
        event.getCompanyId(), event.getGroupId(), event.getUserId(),
        Event.class.getName(), event.getEventId(), false,
        true, true);

    return event;
}

public Event deleteEvent(Event event) throws SystemException {

    return eventPersistence.remove(event);
}

public Event deleteEvent(long eventId)
    throws PortalException, SystemException {

    Event event = eventPersistence.findByPrimaryKey(eventId);

    return deleteEvent(event);
}

public Event getEvent(long eventId)
    throws SystemException, PortalException {

    return eventPersistence.findByPrimaryKey(eventId);
}

public List<Event> getEventsByGroupId(long groupId) throws SystemException {

    return eventPersistence.findByGroupId(groupId);
}

public List<Event> getEventsByGroupId(long groupId, int start, int end)
    throws SystemException {

    return eventPersistence.findByGroupId(groupId, start, end);
}

public int getEventsCountByGroupId(long groupId) throws SystemException {

    return eventPersistence.countByGroupId(groupId);
}

public Event updateEvent(
        long userId, long eventId, String name, String description,
        int month, int day, int year, int hour, int minute,
        long locationId, ServiceContext serviceContext)
    throws PortalException, SystemException {

    User user = userPersistence.findByPrimaryKey(userId);

    Date now = new Date();

    Event event = EventLocalServiceUtil.fetchEvent(eventId);

    event.setModifiedDate(serviceContext.getModifiedDate(now));
    event.setName(name);
    event.setDescription(description);

    Calendar dateCal = CalendarFactoryUtil.getCalendar(
        user.getTimeZone());
    dateCal.set(year, month, day, hour, minute);
    Date date = dateCal.getTime();
    event.setDate(date);

    event.setLocationId(locationId);

    super.updateEvent(event);

    return event;
}
```

记得导入（import）需要的类。为了方便，可以将以下内容复制到类中（译注：在 IDE 中使用 Ctrl-Shift-O 可能更方便）：

```
import java.util.Calendar;
import java.util.Date;
import java.util.List;

import com.liferay.portal.kernel.exception.PortalException;
import com.liferay.portal.kernel.exception.SystemException;
import com.liferay.portal.kernel.util.CalendarFactoryUtil;
import com.liferay.portal.model.User;
import com.liferay.portal.service.ServiceContext;

import com.nosester.portlet.eventlisting.model.Event;
import com.nosester.portlet.eventlisting.service.EventLocalServiceUtil;
```

为了将 Event 保存到数据库，需要给它一个 ID。Liferay 提供了一个 _counter_ 服务，可以调用它为每个新的 Event 实体设置唯一的 ID。调用 Liferay 的 _CounterLocalServiceUtil_ 的 _increment_ 方法也行，不过 Service Builder 已经通过 Spring 的依赖注入机制为生成的 _EventLocalServiceBaseImpl_ 添加了 _CounterLocalService_ 实例。因为 _EventLocalServiceImpl_ 类继承了 _EventLocalServiceBaseImpl_，所以你可以在 _EventLocalServiceImpl_ 中访问 _CounterLocalService_。通过 Spring 使 _EventLocalServiceBaseImpl_ 中可用的服务包括这些：

- eventLocalService
- eventPersistence
- locationLocalService
- locationPersistence
- counterLocalService
- resourceLocalService
- resourceService
- resourcePersistence
- userLocalService
- userService
- userPersistence

可以使用注入类的 _increment_ 方法，也可以直接调用 _CounterLocalSerice_ 的 _increment_ 方法。

```
long eventId = counterLocalService.increment(Event.class.getName());
```

将生成的 _eventId_ 赋值到新 Event 的 ID 属性：

```
Event event = eventPersistence.create(eventId);
```

_eventPersistence_ 是 Service Builder 注入到 _EventLocalServiceBaseImpl_ 中 Spring bean 中的一个。

接下来，用输入的值为 Event 的属性赋值，设置 Event 的 _name_ 和 _description_ 。然后用日期时间值作为 Event 的 _date_，再将 location 关联到 Event。

然后写入 Event 的审计信息。先设置实体的 _group_ 或 scope。本例中 group 是站点。然后设置 _company_ 和  _user_。_company_ 对应于 portal 的实例。_createDate_ 和 _modifiedDate_ 设置为当前时间。做完这些之后 ，调用生成的 _EventLocalServiceBaseImpl_ 的 _addEvent_ 方法。最后，把 Event 添加到资源（resource），这样就可以为它设置权限。在 [Asset Framework](http://www.liferay.com/documentation/liferay-portal/6.2/development/-/ai/asset-framework-liferay-portal-6-2-dev-guide-06-en) 章节有添加资源的详细说明。

因为活动需要有地点（location），那就也把 Location 的本地服务也实现了。打开 _docroot/WEB-INF/src/com/nosester/portlet/eventlisting/service/impl_ 目录中的 _LocationLocalServiceImpl.java_ 文件，并添加以下方法：

```
public Location addLocation(
        long userId, long groupId, String name, String description,
        String streetAddress, String city, String stateOrProvince,
        String country, ServiceContext serviceContext)
throws PortalException, SystemException {

    User user = userPersistence.findByPrimaryKey(userId);

    Date now = new Date();

    long locationId =
        counterLocalService.increment(Location.class.getName());

    Location location = locationPersistence.create(locationId);

    location.setName(name);
    location.setDescription(description);
    location.setStreetAddress(streetAddress);
    location.setCity(city);
    location.setStateOrProvince(stateOrProvince);
    location.setCountry(country);

    location.setGroupId(groupId);
    location.setCompanyId(user.getCompanyId());
    location.setUserId(user.getUserId());
    location.setCreateDate(serviceContext.getCreateDate(now));
    location.setModifiedDate(serviceContext.getModifiedDate(now));

    super.addLocation(location);

    return location;
}

public Location deleteLocation(Location location)
    throws SystemException {

    return locationPersistence.remove(location);
}

public Location deleteLocation(long locationId)
    throws PortalException, SystemException {

    Location location = locationPersistence.fetchByPrimaryKey(locationId);

    return deleteLocation(location);
}

public List<Location> getLocationsByGroupId(long groupId)
    throws SystemException {

    return locationPersistence.findByGroupId(groupId);
}

public List<Location> getLocationsByGroupId(
        long groupId, int start, int end)
    throws SystemException {

    return locationPersistence.findByGroupId(groupId, start, end);
}

public int getLocationsCountByGroupId(long groupId) throws SystemException {

    return locationPersistence.countByGroupId(groupId);
}

public Location updateLocation(
        long userId, long locationId, String name, String description,
        String streetAddress, String city, String stateOrProvince,
        String country, ServiceContext serviceContext)
    throws PortalException, SystemException {

    User user = userPersistence.findByPrimaryKey(userId);

    Date now = new Date();

    Location location = locationPersistence.findByPrimaryKey(locationId);

    location.setName(name);
    location.setDescription(description);
    location.setStreetAddress(streetAddress);
    location.setCity(city);
    location.setStateOrProvince(stateOrProvince);
    location.setCountry(country);
    location.setModifiedDate(serviceContext.getModifiedDate(now));

    super.updateLocation(location);

    return location;
}
```

确认添加了以下导入：

```
import java.util.Date;
import java.util.List;

import com.liferay.portal.kernel.exception.PortalException;
import com.liferay.portal.kernel.exception.SystemException;
import com.liferay.portal.model.User;
import com.liferay.portal.service.ServiceContext;

import com.nosester.portlet.eventlisting.model.Location;
import com.nosester.portlet.eventlisting.service.base.LocationLocalServiceBaseImpl;
```

event 和 location 的本地服务都准备就绪了。

在使用为 _EventLocalServiceImpl_ 和 _LocationLocalServiceImpl_ 类添加的自定义方法前，需要再运行一次 Service Builder 来将这些方法签名添加到 _EventLocalService_ 和 _LocationLocalService_ 接口中。

*使用 Liferay IDE：* 像以前一样，打开 service.xml文件，并确认处于 Overview 视图，然后选择 _Build Services_。

*使用终端：* 转到项目的根目录，并执行：

```
ant build-service
```

