#调用本地服务

Service Builder 生成完，就可以在 portlet 的 _-Portlet_ 类中调用服务了。可以在 _EventListingPortlet_ 和 _LocationListPortlet_ 中调用静态工具类 _EventLocalServiceUtil_ 或  _LocationLocalServiceUtil_ 的任意方法了。例如，执行 CRUD 操作。为此，要为 _EventListingPortlet_ 创建以下方法（_LocationListingPortlet_ 也类似）：

- addEvent
- updateEvent
- deleteEvent

在 _docroot/WEB-INF/src/com/nosester/portlet/eventlisting_ 目录中没有 _EventListingPortlet.java_ 文件，就新建。用以下内容替换 _EventListingPortlet.java_ 文件的内容：

```
package com.nosester.portlet.eventlisting;

import java.util.Calendar;

import javax.portlet.ActionRequest;
import javax.portlet.ActionResponse;

import com.liferay.portal.kernel.exception.PortalException;
import com.liferay.portal.kernel.exception.SystemException;
import com.liferay.portal.kernel.log.Log;
import com.liferay.portal.kernel.log.LogFactoryUtil;
import com.liferay.portal.kernel.util.ParamUtil;
import com.liferay.portal.service.ServiceContext;
import com.liferay.portal.service.ServiceContextFactory;
import com.liferay.util.bridges.mvc.MVCPortlet;
import com.nosester.portlet.eventlisting.model.Event;
import com.nosester.portlet.eventlisting.service.EventLocalServiceUtil;

public class EventListingPortlet extends MVCPortlet {

    public void addEvent(ActionRequest request, ActionResponse response)
            throws Exception {

        _updateEvent(request);

        sendRedirect(request, response);
    }

    public void deleteEvent(ActionRequest request, ActionResponse response)
        throws Exception {

        long eventId = ParamUtil.getLong(request, "eventId");

        EventLocalServiceUtil.deleteEvent(eventId);

        sendRedirect(request, response);
    }

    public void updateEvent(ActionRequest request, ActionResponse response)
        throws Exception {

        _updateEvent(request);

        sendRedirect(request, response);
    }

    private Event _updateEvent(ActionRequest request)
        throws PortalException, SystemException {

        long eventId = ParamUtil.getLong(request, "eventId");
        String name = ParamUtil.getString(request, "name");
        String description = ParamUtil.getString(request, "description");
        long locationId = ParamUtil.getLong(request, "locationId");

        int year = ParamUtil.getInteger(request, "dateYear");
        int month = ParamUtil.getInteger(request, "dateMonth");
        int day = ParamUtil.getInteger(request, "dateDay");
        int hour = ParamUtil.getInteger(request, "dateHour");
        int minute = ParamUtil.getInteger(request, "dateMinute");
        int amPm = ParamUtil.getInteger(request, "dateAmPm");

        if (amPm == Calendar.PM) {
            hour += 12;
        }

        ServiceContext serviceContext = ServiceContextFactory.getInstance(
            Event.class.getName(), request);

        Event event = null;

        if (eventId <= 0) {
            event = EventLocalServiceUtil.addEvent(
                serviceContext.getUserId(), serviceContext.getScopeGroupId(),
                name, description, month, day, year, hour, minute, locationId,
                serviceContext);
        }
        else {
            event = EventLocalServiceUtil.getEvent(eventId);

            event = EventLocalServiceUtil.updateEvent(
                serviceContext.getUserId(), eventId, name, description, month,
                day, year, hour, minute, locationId, serviceContext);
        }

        return event;
    }

    private static Log _log = LogFactoryUtil.getLog(EventListingPortlet.class);
}
```
Event Listing portlet 的 _addEvent_、_updateEvent_ 和 _deleteEvent_ 方法可以调用 _EventLocalServiceUtil_ 相应的方法了。 如果从 portlet 请求中无法获取某个请求参数 Liferay 的 _ParamUtil_ 的 getter 方法（如 _getLong_ 或 _getString_）会返回 0 或 "" 这样的默认值。假如在添加一个新的 event 时，没有有效的 event ID，那么 _ParamUtil.getLong("request", "eventId")_ 会返回 0。portlet 的 _addEvent_ 方法调用 _EventLocalServiceUtil_ 的 _addEvent_ 方法，新 event 的 ID 在服务层的 _addEvent_ 方法中生成，这个方法需要在 _EventLocalServiceImpl_ 类中添加。Service Builder 生成的 _EventLocalServiceUtil_ 包含不同的 CRUD 方法：

- createEvent
- addEvent
- deleteEvent
- updateEvent
- fetchEvent
- getEvent

下图中列出的是由 Service Builder 生成的可以被 _ventListingPortlet_ 调用的全部方法：

![local-service-util-outline.png](https://www.liferay.com/c/document_library/get_file?groupId=14&uuid=d2496e10-7aa9-4e1b-ab5d-5356f6c68bba)

图 5.5：_EventListingPortlet_ 可以访问的 _EventLocalServiceUtil_ 方法，其中很多都是 CRUD 操作。

portlet 类应该只访问（或者说调用） _-LocalServiceUtil_ 类。相应地 _-LocalServiceUtil_ 类应该只调用注入的 _-LocalServiceImpl_ 类。注意，上图中的 _EventLocalServiceUtil_ 工具类有一个私有实例变量 _\_service_，在运行期间这个 _EventLocalService_ 类型的实例变量会通过依赖注入（dependency injection）赋值为 _EventLocalServiceImpl_ 的一个实例。这样，在运行期间，_EventLocalServiceUtil_ 工具类中的所有方法就能调用 _EventLocalServiceImpl_ 类相应的方法了。

像实现 _EventListingPortlet_ 类那样实现 _LocationListingPortlet_ 类。如果 _docroot/WEB-INF/src/com/nosester/portlet/eventlisting_ 目录中没有 _LocationListingPortlet.java_ 文件，就建一个新文件。打开 _LocationListingPortlet.java_ 文件，将用以下代码替换文件内容：

```
package com.nosester.portlet.eventlisting;

import javax.portlet.ActionRequest;
import javax.portlet.ActionResponse;

import com.liferay.portal.kernel.exception.PortalException;
import com.liferay.portal.kernel.exception.SystemException;
import com.liferay.portal.kernel.log.Log;
import com.liferay.portal.kernel.log.LogFactoryUtil;
import com.liferay.portal.kernel.util.ParamUtil;
import com.liferay.portal.service.ServiceContext;
import com.liferay.portal.service.ServiceContextFactory;
import com.liferay.util.bridges.mvc.MVCPortlet;
import com.nosester.portlet.eventlisting.model.Location;
import com.nosester.portlet.eventlisting.service.LocationLocalServiceUtil;

public class LocationListingPortlet extends MVCPortlet {

    public void addLocation(ActionRequest request, ActionResponse response)
            throws Exception {

        _updateLocation(request);

        sendRedirect(request, response);
    }

    public void deleteLocation(ActionRequest request, ActionResponse response)
        throws Exception {

        long locationId = ParamUtil.getLong(request, "locationId");

        LocationLocalServiceUtil.deleteLocation(locationId);

        sendRedirect(request, response);
    }

    public void updateLocation(ActionRequest request, ActionResponse response)
        throws Exception {

        _updateLocation(request);

        sendRedirect(request, response);
    }

    private Location _updateLocation(ActionRequest request)
            throws PortalException, SystemException {

        long locationId = (ParamUtil.getLong(request, "locationId"));
        String name = (ParamUtil.getString(request, "name"));
        String description = (ParamUtil.getString(request, "description"));
        String streetAddress = (ParamUtil.getString(request, "streetAddress"));
        String city = (ParamUtil.getString(request, "city"));
        String stateOrProvince = (ParamUtil.getString(request, "stateOrProvince"));
        String country = (ParamUtil.getString(request, "country"));

        ServiceContext serviceContext = ServiceContextFactory.getInstance(
                Location.class.getName(), request);

        Location location = null;

        if (locationId <= 0) {

            location = LocationLocalServiceUtil.addLocation(
                serviceContext.getUserId(), serviceContext.getScopeGroupId(), name, description,
                streetAddress, city, stateOrProvince, country, serviceContext);
        }
        else {
            location = LocationLocalServiceUtil.getLocation(locationId);

            location = LocationLocalServiceUtil.updateLocation(
                    serviceContext.getUserId(), locationId, name,
                    description, streetAddress, city, stateOrProvince, country,
                    serviceContext);
        }

        return location;
    }

    private static Log _log = LogFactoryUtil.getLog(LocationListingPortlet.class);

}
```

上面说明了如何在 portlet 类中调用 Service Builder 生成的本地服务，后面还会学习如何调用 Liferay 的本地服务。