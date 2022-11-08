Boradcast用于响应来自其他应用程序或者系统的广播消息。

<!--more-->



## 系统广播

| 系统操作                                                     | action                               |
| ------------------------------------------------------------ | ------------------------------------ |
| 监听网络变化                                                 | android.net.conn.CONNECTIVITY_CHANGE |
| 关闭或打开飞行模式                                           | Intent.ACTION_AIRPLANE_MODE_CHANGED  |
| 充电时或电量发生变化                                         | Intent.ACTION_BATTERY_CHANGED        |
| 电池电量低                                                   | Intent.ACTION_BATTERY_LOW            |
| 电池电量充足（即从电量低变化到饱满时会发出广播               | Intent.ACTION_BATTERY_OKAY           |
| 系统启动完成后(仅广播一次)                                   | Intent.ACTION_BOOT_COMPLETED         |
| 按下照相时的拍照按键(硬件按键)时                             | Intent.ACTION_CAMERA_BUTTON          |
| 屏幕锁屏                                                     | Intent.ACTION_CLOSE_SYSTEM_DIALOGS   |
| 设备当前设置被改变时(界面语言、设备方向等)                   | Intent.ACTION_CONFIGURATION_CHANGED  |
| 插入耳机时                                                   | Intent.ACTION_HEADSET_PLUG           |
| 未正确移除SD卡但已取出来时(正确移除方法:设置–SD卡和设备内存–卸载SD卡) | Intent.ACTION_MEDIA_BAD_REMOVAL      |
| 插入外部储存装置（如SD卡）                                   | Intent.ACTION_MEDIA_CHECKING         |
| 成功安装APK                                                  | Intent.ACTION_PACKAGE_ADDED          |
| 成功删除APK                                                  | Intent.ACTION_PACKAGE_REMOVED        |
| 重启设备                                                     | Intent.ACTION_REBOOT                 |
| 屏幕被关闭                                                   | Intent.ACTION_SCREEN_OFF             |
| 屏幕被打开                                                   | Intent.ACTION_SCREEN_ON              |
| 关闭系统时                                                   | Intent.ACTION_SHUTDOWN               |
| 重启设备                                                     | Intent.ACTION_REBOOT                 |
|                                                              |                                      |



参考链接
[BroadcastReceiver广播机制原理](https://skytoby.github.io/2019/BroadcastCast%E5%B9%BF%E6%92%AD%E6%9C%BA%E5%88%B6%E5%8E%9F%E7%90%86/)