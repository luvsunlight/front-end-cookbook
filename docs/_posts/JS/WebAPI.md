> 参考[MDN WebAPI](https://developer.mozilla.org/zh-CN/docs/Web/Reference/API)

## 文档对象模型（DOM）

> DOM是一个可以访问和修改当前文档的API。通过它可以修改文档里的Node和Element

具体的API参考[这里](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model)

## 设备API

> 这些API可以让网页和应用程序调用各种硬件资源，如[环境光感应器API](https://developer.mozilla.org/zh-CN/docs/Web/API/DeviceLightEvent/Using_light_events)，[电池状态API](https://developer.mozilla.org/zh-CN/docs/Web/API/Battery_Status_API)，[地理位置API](https://developer.mozilla.org/zh-CN/docs/Web/API/Geolocation/Using_geolocation)，[指针锁定API](https://developer.mozilla.org/zh-CN/docs/API/Pointer_Lock_API)，[距离感应API](https://developer.mozilla.org/en-US/docs/Web/API/Proximity_Events)，[设备定向API](https://developer.mozilla.org/zh-CN/docs/Web/API/Detecting_device_orientation)，[屏幕定向API](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Object_Model/Managing_screen_orientation)，[震动API](https://developer.mozilla.org/zh-CN/docs/Web/API/Vibration_API)

## 通信API

> 这些API可以让网页和应用程序和其他的网页或设备进行通信，如[网络通信API](https://developer.mozilla.org/zh-CN/docs/Web/API/Network_Information_API)，[Web通信](https://developer.mozilla.org/zh-CN/docs/Web/API/notification/Using_Web_Notifications)，[简单推送API(非标准)](https://developer.mozilla.org/zh-CN/docs/Archive/Firefox_OS/API/Simple_Push_API)

## 数据管理API

> 这套API可以用来存储和管理用户的数据，如[文件处理API（非标准）](https://developer.mozilla.org/en-US/docs/Web/API/File_Handle_API)，[IndexedDB
](https://developer.mozilla.org/zh-CN/docs/Web/API/IndexedDB_API)

## 特权API

> 特权应用程序是那些由用户给予了特定权限的应用程序。包括TCPSocketAPI，联系人API，设备存储API，浏览器API，相机API 

## 已认证应用程序的私有API

> 已认证的应用程序是那些直接与操作系统（比如firefox OS）打交道，执行`核心操作`的底层应用程序。较低特权的应用程序可以通过[Web Activities](https://developer.mozilla.org/en-US/docs/Archive/B2G_OS/API/Web_Activities)调用这些底层应用程序。比如`蓝牙API`，`手机连接API`.etc

!> 注意，这些api并不是标准的w3c支持的api，只在诸如`firefox os`这样的操作系统上才可以使用