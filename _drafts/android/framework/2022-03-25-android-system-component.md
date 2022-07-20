## ActivityManagerService

AMS 是整个四大组件最为核心的对象，所有的组件或多或少都依赖该对象的数据结构信息。

![four_component](../../images/android/android_four_component_ams.png)



## ProcessRecord

Android 系统中用于描述进程的数据结构是 ProcessRecord 对象。AMS 便是管理进程的核心模块。

![process_record](../../images/android/android_process_record.png)

| 成员变量          | 说明                                        | 对应组件        |
| :---------------- | :------------------------------------------ | :-------------- |
| activities        | 记录进程的ActivityRecord列表                | Activity        |
| services          | 记录进程的ActivityRecord列表                | Service         |
| executingServices | 记录进程的正在执行的ActivityRecord列表      | Service         |
| connections       | 记录该进程bind的ConnectionRecord集合        | Service         |
| receivers         | 动态注册的广播接收者ReceiverList集合        | Broadcast       |
| curReceiver       | 当前正在处理的一个广播BroadcastRecord       | Broadcast       |
| pubProviders      | 该进程发布的ContentProviderRecord的map表    | ContentProvider |
| conProviders      | 该进程所请求的ContentProviderConnection列表 | ContentProvider |



## ServiceRecord

ServiceRecord 位于 system_server 进程，是 AMS 管理各个 app 中 service 的基本单位。

ServiceRecord 继承与 Binder 对象，作为 Binder IPC 的 **Bn** 端，Binder 将其传递到 Service 进程的 **Bp** 端，保存在 `Service.mToken`，即 ServiceRecord 的代理对象。

![service_record](../../images/android/android_service_record.png)



## BroadcastRecord

![broadcast_record](../../images/android/android_broadcast_record.png)



## ContentProviderRecord

![content_provider_record](../../images/android/android_content_provider_record.png)



## ActivityRecord

![activity_record](../../images/android/android_activity_record.png)

ActivityRecord 没有继承与 Binder，但成员变量 `appToken` 继承于 `IApplicationToken.Stub`。